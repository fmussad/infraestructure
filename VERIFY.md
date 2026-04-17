# Post-deploy verification

Replace `VPS_IP` with your IP or hostname (e.g. `2.24.204.53`). Use **HTTP** unless you have added TLS.

## 1. Containers

```bash
ssh root@VPS_IP 'docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep -E "felixmussa|NAMES"'
```

You should see at least: `felixmussa-traefik`, `felixmussa-noop`, `felixmussa-landing`, `felixmussa-inventory`, `felixmussa-payroll-frontend`, `felixmussa-payroll-api`.

## 2. Networks

```bash
ssh root@VPS_IP 'docker network ls | grep -E "felixmussa_net|inventory_net|payroll_net"'
```

## 3. HTTP routes

```bash
curl -sI "http://VPS_IP/" | head -n 5
curl -sI "http://VPS_IP/landing" | head -n 5
curl -s  "http://VPS_IP/landing" | head -c 200
curl -sI "http://VPS_IP/services/app/inventory" | head -n 5
curl -sI "http://VPS_IP/services/app/payroll" | head -n 5
curl -s  "http://VPS_IP/services/app/payroll" | grep -o 'Hello: [^<]*' || true
curl -s  "http://VPS_IP:9080/test/hello"
```

- `/` → redirect to `/landing`
- `/landing` → **landing** repo HTML (`main`)
- `/services/app/inventory` → inventory
- `/services/app/payroll` → payroll-frontend (should show **Hello: Felix Mussa** when the frontend points at the API; on the VPS the Ansible template sets `VITE_API_BASE_URL=http://VPS_IP:9080`)

## 4. Quick logs

```bash
ssh root@VPS_IP 'docker logs felixmussa-traefik 2>&1 | tail -20'
ssh root@VPS_IP 'docker logs felixmussa-payroll-api 2>&1 | tail -10'
```
