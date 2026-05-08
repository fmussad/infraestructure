# infrastructure — DEPLOYMENT

## Target layout on the VPS

Playbooks use Ansible variable **`felixmussa_root`** (example default **`/opt/felixmussa`** in `hosts.ini.example`), producing sibling directories typically:

- `{{ felixmussa_root }}/traefik`
- `{{ felixmussa_root }}/landing`
- `{{ felixmussa_root }}/inventory`
- `{{ felixmussa_root }}/payroll-api`
- `{{ felixmussa_root }}/payroll-frontend`

**Note:** The monorepo checkout on a dev machine nests payroll under **`felixmussa-project/payroll/`** — Ansible deliberately uses **flat** paths on the server.

## Full stack order

**`deploy-all.yml`** orchestrates prerequisite checks → networks → Traefik → landing → inventory → payroll (API + frontend + migrations).

## Individual playbooks

| Playbook | Effect |
| -------- | ------ |
| `setup-server.yml` | `docker version`, `docker compose version` checks only |
| `deploy-networks.yml` | Ensures **`felixmussa_net`**, **`inventory_net`**, **`payroll_net`** |
| `deploy-traefik.yml` | Copy `traefik/` tree, compose up |
| `deploy-landing.yml` | Git shallow clone (**depth 1**), optional `.env` bootstrap, compose up |
| `deploy-inventory.yml` | Same pattern as landing |
| `deploy-payroll.yml` | Deep clone (**force**) payroll repos, render Jinja `.env`, API compose up (**no `-v`**), ensure **`payroll_dev`**, run **`migrate`** twice (prod DB + **`payroll_dev`**), frontend compose up |

## Docker / Compose assumptions

- **`COMPOSE_MENU=false`** injected on compose invocations used in payroll playbooks.
- VPS must have Compose v2 plugin (validated by prerequisite playbook).

## Public HTTP surface

Traefik listens on **`${TRAEFIK_HTTP_PORT:-80}:80`** (`traefik/docker-compose.yml`). Path routes are defined by labels on workloads (landing `/landing`, inventory `/services/app/inventory`, payroll UI/API paths documented in **`VERIFY.md`**).

## Payroll-specific deploy notes

- MySQL Docker volume **`payroll_mysql_data`** persists; playbook **does not** run `docker compose down -v` for payroll-api.
- Migrations CLI: **`docker compose exec -T payroll-api /app/payroll-api migrate`**

## Operational verification

Use **`VERIFY.md`** (container list expectation, curls, logs tail commands).
