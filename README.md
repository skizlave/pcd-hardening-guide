# pcd-hardening

Ansible Playbook to implement CIS Benchmark Level 1 best practices and hardening for Platform9 hypervisor data-plane hosts on Ubuntu 24.04.

## Scope

This project hardens only hypervisors in the `hypervisors` inventory group.

- In scope: customer-managed hypervisors (data plane)
- Out of scope: SaaS-hosted management/control services

## Network policy in this repo

- Outbound HTTP/HTTPS (`80/tcp`, `443/tcp`) is allowed to any destination
- Outbound DNS/NTP and management/networking ports are explicitly allowlisted
- Management/networking port rules are scoped to `cis_management_subnets` (or fallback `pf9_management_subnet`)

## What the playbook does

| Section | Controls applied |
|---|---|
| CIS 1 | Filesystem restrictions, security updates, AppArmor, warning banners |
| CIS 2 | Remove inetd/xinetd, enable timesyncd, disable unused services, restrict MTA |
| CIS 3 | Disable unused protocols, apply CIS sysctl parameters, configure UFW |
| CIS 4 | Install/configure auditd rules, rsyslog hardening, logrotate |
| CIS 5 | SSH daemon hardening, PAM password quality, sudo logging, password aging |
| CIS 6 | Critical file permissions, world-writable file reporting, legacy account checks |
| Hypervisor | Preserve KVM/libvirt services, overlay networking rules, hypervisor routing sysctls |

## Prerequisites

- Ubuntu 24.04 LTS target host(s)
- SSH key-based root access from the Ansible control node
- Python 3 on target hosts
- Ansible 2.12+ and `community.general`

Install required collection:

```bash
ansible-galaxy collection install community.general
```

## Variable model

All project variables are in a single file:

- `vars/cis_hardening.yml`

The former `inventory/prod/group_vars/all.yml` has been merged into this file.

Important variables:

| Variable | Purpose |
|---|---|
| `pcd_manager_endpoint` | Hostname checked in verification on `443/tcp` |
| `cis_hypervisor_outbound_global_ports` | Outbound infra ports (DNS/NTP/HTTP/HTTPS) |
| `cis_hypervisor_outbound_management_ports` | Outbound management/networking ports scoped to management subnets |
| `cis_management_subnets` | Optional list of management CIDRs for scoped rules |
| `cis_ssh_allowed_sources` | Optional SSH source CIDR restrictions |
| `cis_hypervisor_ingress_whitelist` | Optional external allowlist for noVNC/image catalog |
| `pf9_management_subnet` | Source CIDR for migration-related inbound rules |
| `cis_hypervisor_console_ingress_whitelist` | Jump-host allowlist (`/32`) for direct VNC (`5900:5999`) |
| `cis_hypervisor_allow_direct_vnc` | Enable/disable direct VNC rules (recommended: `false`) |

## Setup your deployment

This repo is published with templates to avoid sharing credentials and sensitive IPs. Follow these steps to configure for your environment:

### 1. Create your inventory

Copy the template and add your hypervisor hostnames and management IP addresses:

```bash
cp inventory/prod/hosts.yml.example inventory/prod/hosts.yml
# Edit inventory/prod/hosts.yml with your hostnames and IPs
```

### 2. Create your variables

Copy the template and update all placeholders with your specific values:

```bash
cp vars/cis_hardening.yml.example vars/cis_hardening.yml
# Edit vars/cis_hardening.yml:
# - pcd_manager_endpoint (your manager URL)
# - pf9_management_subnet (your management CIDR)
# - cis_hypervisor_outbound_global_ports (global egress like DNS/NTP/HTTP/HTTPS)
```

### 3. Verify syntax

```bash
ansible-playbook playbooks/cis-hardening.yml --syntax-check -i inventory/prod/hosts.yml
```

## Quick start

```bash
cd /Users/jeffscott-pf9/ansible/pcd-hardening

# dry run
ansible-playbook playbooks/cis-hardening.yml --check -i inventory/prod/hosts.yml

# apply
ansible-playbook playbooks/cis-hardening.yml -i inventory/prod/hosts.yml
```

## Tags

```bash
ansible-playbook playbooks/cis-hardening.yml --tags ssh
ansible-playbook playbooks/cis-hardening.yml --tags firewall
ansible-playbook playbooks/cis-hardening.yml --tags audit
ansible-playbook playbooks/cis-hardening.yml --tags hypervisor
ansible-playbook playbooks/cis-hardening.yml --tags verify
```

## Firewall behavior

UFW is temporarily disabled during package installation phases and finalized at the verification phase to avoid package-install deadlocks.

Optional early enable:

```yaml
cis_enable_ufw_early: true
```

## SSH safety

Default is `PermitRootLogin prohibit-password` to preserve key-based automation access while blocking root password authentication.

## Directory layout

```text
pcd-hardening/
├── ansible.cfg
├── README.md
├── playbooks/
│   └── cis-hardening.yml
├── vars/
│   └── cis_hardening.yml
└── inventory/
    └── prod/
        └── hosts.yml
```
