# infrastructure — ARCHITECTURE

## Directory layout

| Path | Content |
| ---- | ------- |
| `ansible/inventory/` | `hosts.ini` (local, not tracked as example shows), `hosts.ini.example` vars + Git SSH URLs |
| `ansible/playbooks/` | Modular deploy playbooks + `deploy-all.yml` aggregator |
| `ansible/templates/` | Jinja templates (e.g. `payroll-api.env.j2`, `payroll-frontend.env.j2`) |
| `traefik/` | **`docker-compose.yml`** (Traefik + noop whoami helper), **`traefik.yml`** static (`entryPoints.web`, Docker provider **`network: felixmussa_net`**, dashboard off) |
| `Makefile` | Convenience targets (`make deploy-payroll`, etc.) cd into **`ansible/`** |
| `README.md`, `VERIFY.md` | Human operator docs |

## Playbook graph

`deploy-all.yml` imports in order:

1. `setup-server.yml` — Docker + compose presence checks  
2. `deploy-networks.yml` — ensures **`felixmussa_net`**, **`inventory_net`**, **`payroll_net`** exist  
3. `deploy-traefik.yml` — copies `traefik/*` → **`{{ felixmussa_root }}/traefik`**, **`docker compose up`**  
4. `deploy-landing.yml` — shallow git clone **`landing/`**, compose up  
5. `deploy-inventory.yml` — same for **`inventory/`**  
6. `deploy-payroll.yml` — full clone **`payroll-api`**, guarded **`payroll-frontend`** clone, template `.env` files, API stack up, **`payroll_dev`** ensure, **`migrate`** prod + dev DB, frontend compose up  

## Traefik topology

- Binds Docker socket read-only on the VPS.
- **`exposedByDefault: false`** — only labelled services register routes.
- **Root router** (`Path('/')`) attaches redirect middleware **→ `/landing`** (implemented as labels on the Traefik container in `traefik/docker-compose.yml`).

Application-specific routing (**StripPrefix**, path rules) remains in **each app's** `docker-compose.yml`; not centrally defined here beyond static provider/network.

## Networks (declared vs used)

Ansible **`deploy-networks.yml`** ensures three networks:

- **`felixmussa_net`** — used by Traefik and all Felix service compose files inspected in-repo  
- **`inventory_net`**, **`payroll_net`** — created but **not attached** by current landing/inventory/payroll compose files (`README.md` states this explicitly)
