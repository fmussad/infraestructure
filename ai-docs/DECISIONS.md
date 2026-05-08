# infrastructure — DECISIONS

Cross-cutting decisions evidenced by tracked files:

1. **Separate `docker-compose.yml` per Felix service repo** instead of one mega-compose — independent lifecycle and versioning per app (`felixmussa-project/README.md` pattern).
2. **Traefik v3.x** Docker provider limited to **`felixmussa_net`** (`traefik.yml` **`network:`** stanza).
3. **Path-based routing initially** (`PathPrefix`, StripPrefix middlewares declared on workloads). Root `/` redirects to **`/landing`** via Traefik labels bundled with infra compose.
4. **Ansible clones flat trees** (`payroll-api`, `payroll-frontend`) under **`felixmussa_root`** for operational simplicity vs nested monorepo paths on developer laptops.
5. **Extra networks **`inventory_net`**, **`payroll_net`** provisioned preemptively (`deploy-networks.yml`) yet **remaining unused by current Felix compose manifests** (`README.md` explicit note)—reserve for backend isolation iterations.
6. **Payroll split**: distinct Ansible git URLs (`payroll_api_git_url`, `payroll_frontend_git_url`) aligning with repo separation strategy.
7. **Force git reset on payroll clones** (**`force: true`**) prioritizes VPS matching remote HEAD over leftover local tweaks under deploy dirs.
8. **Template-driven payroll env** renders secrets from Ansible inventory into remote `.env`—operator must safeguard **`hosts.ini`** out of Git.
9. **English as working language for automation/config comments** aligns with organisational backend guidance layered in payroll-api docs.
10. **TLS not implemented in-repo**—only HTTP `entryPoints.web` `:80`; HTTPS left as documented future uplift (see Felix parent README narratives).
