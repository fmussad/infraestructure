# infrastructure — PROJECT_CONTEXT

## What this repository does

Hosts **deployment automation** and the **Traefik reverse-proxy bundle** used by the FELIXMUSSA workloads: Ansible playbooks, inventory templates Jinja for payroll env rendering, Docker network bootstrap, and `traefik/` static + compose files shipped to the VPS.

## Purpose

- Provision **Docker networks** expected by Felix services.
- Deploy **Traefik** (Docker provider on **`felixmussa_net`**, HTTP entrypoint `:80`).
- **Git pull** landing, inventory, payroll-api, payroll-frontend on the VPS and run **`docker compose up`** with consistent env handling (templates for payroll stacks).
- Provide **verification** snippets (`VERIFY.md`) and **Makefile** shortcuts wrapping `ansible-playbook`.

## Stack

| Component | Detail |
| --------- | ------ |
| Automation | Ansible 2.14+ assumed (`README.md`) |
| Edge proxy | Traefik **v3.6**, config file **`traefik/traefik.yml`** + **`traefik/docker-compose.yml`** |
| Target OS | VPS with Docker Engine + Compose plugin (**`setup-server.yml`** verifies only; does not install Docker) |

## Fit in FELIXMUSSA

Each app repo (`landing`, `inventory`, `payroll-*`) declares **Traefik labels** on its containers; Traefik listens on **`felixmussa_net`**. Ansible copies/syncs infra and pulls app repos under **`felixmussa_root`** (default **`/opt/felixmussa`** in **`hosts.ini.example`**).
