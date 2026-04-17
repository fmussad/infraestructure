# felixmussa infrastructure (Ansible + Traefik)

Remote deploy to `/opt/felixmussa` on the VPS: Docker networks, Traefik (port **80**), and GitHub clones (`main`).

## Requirements on your machine

- Ansible 2.14+ (`ansible-playbook`)
- SSH access to the VPS (key or agent)
- On the **VPS**: Docker Engine + **docker compose** plugin already installed

## Configuration

1. Copy `ansible/inventory/hosts.ini.example` → `hosts.ini` and set `ansible_host`, `ansible_user`, `felixmussa_public_ip`, and repo URLs if needed.

2. From the `ansible/` directory:

```bash
cd ansible
ansible-playbook -i inventory/hosts.ini playbooks/deploy-networks.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-traefik.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-landing.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-inventory.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-payroll.yml
```

Or run everything:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/deploy-all.yml
```

## Before the first Traefik deploy from here

If an old Traefik is still on the VPS (e.g. removed `felixmussa-project/proxy` path or another compose using the same container name), free port **80** and names `felixmussa-traefik` / `felixmussa-noop`:

```bash
ssh root@YOUR_VPS 'docker rm -f felixmussa-traefik felixmussa-noop 2>/dev/null || true'
```

## Networks

| Network          | Purpose |
|------------------|---------|
| `felixmussa_net` | Traefik + services exposed through the proxy |
| `inventory_net`  | Created by Ansible; attach in inventory compose when you isolate a backend |
| `payroll_net`    | Created by Ansible; attach in payroll when you want private FE/API traffic |

Current **landing / inventory / payroll** compose files only use `felixmussa_net`; they were not changed for the extra networks.

## Verification

See `VERIFY.md`.

## GitHub repository

Suggested remote name: **`infraestructure`**. Local folder name: `infrastructure/`.
