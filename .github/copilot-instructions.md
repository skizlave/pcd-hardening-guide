# Copilot Instructions for pcd-hardening

## Scope and safety
- This repository hardens hypervisor data-plane hosts only.
- Do not propose or add changes for SaaS-hosted management/control-plane services.
- Keep outbound manager connectivity policy restricted to TCP 443.

## Platform assumptions
- Target OS is Ubuntu 24.04.
- Primary playbook: playbooks/cis-hardening.yml.
- Single source of project variables: vars/cis_hardening.yml.
- Image library port is 9494 and runs on the hypervisor hosts
- Console port is 6080 and runs on the hypervisor hosts.

## Editing guidelines
- Prefer minimal, surgical changes.
- Preserve idempotency and existing tags.
- Avoid introducing control-plane/OpenStack-specific logic.
- Keep comments concise and operationally useful.

## Validation commands
Run these from repository root after changes:

```bash
ansible-playbook playbooks/cis-hardening.yml --syntax-check -i inventory/prod/hosts.yml
ansible-playbook playbooks/cis-hardening.yml --check -i inventory/prod/hosts.yml
```

## Inventory and auth expectations
- Inventory group: hypervisors.
- SSH access is key-based and uses root by default.
