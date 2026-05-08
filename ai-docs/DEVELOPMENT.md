# infrastructure — DEVELOPMENT

## Prerequisites (control machine)

- **Ansible 2.14+**
- SSH access to the VPS (key or agent)
- Valid **`ansible/inventory/hosts.ini`** (copy from `hosts.ini.example`)

## Quick iteration

Run from **`infrastructure/ansible/`** (paths match `README.md`):

```bash
cd ansible
ansible-playbook -i inventory/hosts.ini playbooks/deploy-networks.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-traefik.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-landing.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-inventory.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-payroll.yml
```

Alternatively from **`infrastructure/`** root:

```bash
make deploy-networks
make deploy-traefik
make deploy-landing
make deploy-inventory
make deploy-payroll
make deploy-all
```

Override inventory relative to **`ansible/`** after `cd`:

```bash
make deploy-payroll INVENTORY=inventory/staging.ini
```

## Private Git repos

Inventory uses **`git@github.com:...`** URLs; VPS needs a **deploy key** registered per repo (read-only acceptable). Smoke test documented in `README.md` (`GIT_TERMINAL_PROMPT=0 git ls-remote ...`).

## Traefik bundle edits

Modify files under **`infrastructure/traefik/`** locally; **`deploy-traefik.yml`** **copies** them to **`{{ felixmussa_root }}/traefik`** on the next run (`ansible.builtin.copy`).

## Payroll templates

Editing **`ansible/templates/payroll-*.env.j2`** affects only the **next** `deploy-payroll` / `deploy-all` run (templates overwrite remote `.env`).

## Debugging

| Symptom | Check |
| ------- | ----- |
| Playbook SSH fails | Inventory `ansible_host`, firewall, agent |
| Git clone failures on VPS | deploy key rights, SSH known_hosts (`accept_hostkey` on Ansible git tasks) |
| Traefik port 80 collision | README pre-step removing old **`felixmussa-traefik`** / **`felixmussa-noop`** |

Do **not** commit real secrets—keep them in **`hosts.ini`** locally.
