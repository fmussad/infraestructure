# infrastructure — TODO

## Gaps / unverified assumptions

| Item | Detail |
| ---- | ------ |
| OS baseline | **`setup-server.yml`** does not enforce Ubuntu **24.04 LTS** or Hostinger image specifics—purely verifies Docker toolchain exists |
| **`felixmussa_root` divergence** | Some Makefiles in Felix repos mention **`DEPLOY_PATH=/opt/felixmussa-project`** while inventory example chooses **`/opt/felixmussa`** — align per environment consciously |
| **TLS readiness** | No cert resolver / `websecure` entrypoint in tracked `traefik.yml` |
| **`inventory_net` / `payroll_net` usefulness** | Created but unattached currently—evaluate attach strategy if internal-only DB traffic needed |

## Risks

- Ansible **`force: true`** on payroll resets server working trees—local hotfixes directly on VPS clones will be wiped next deploy if not merged upstream first.
- Shallow clones on landing/inventory cannot represent every edge Git failure mode (already noted for payroll opting into full clones).

## Suggested backlog

1. Add optional playbook variable for **staging vs prod** inventories without duplicating playbooks entirely.
2. Consider **Molecule** smoke tests stubbing docker connection (heavyweight—nice-to-have).
3. Document rollback strategy (pinned image tags vs rebuild-only model today).

## Manual maintenance cues

Whenever Traefik/router labels drift in Felix repos, synchronize **`VERIFY.md`** curl snippets and priorities.
