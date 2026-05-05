# HAProxy — Docker Compose Load Balancer

## Directory layout

```
haproxy/
├── docker-compose.yml
└── haproxy/
    ├── haproxy.cfg          ← single-file config (start here)
    ├── backends/
    │   └── app1.cfg         ← example split-file backend (for scaling)
    └── errors/
        └── 503.http         ← custom error page
```

---

## Quick start

```bash
# 1. Validate config (runs haproxy -c)
docker compose run --rm haproxy

# 2. Start everything
docker compose up -d

# 3. Open the stats dashboard
open http://localhost:8404/stats
```

---

## Adding a new service

1. Add the container to `docker-compose.yml` on the `proxy` network.
2. Add an ACL + `use_backend` rule in the `http_in` frontend inside `haproxy.cfg`.
3. Add a `backend be_<service>` block (or a new file in `backends/`).
4. Reload HAProxy without dropping connections:

```bash
docker kill -s HUP haproxy
```

---

## Scaling to multiple config files

Switch the `command` in `docker-compose.yml` to load directories:

```yaml
command: >
  haproxy
    -f /usr/local/etc/haproxy/haproxy.cfg
    -f /usr/local/etc/haproxy/frontends/
    -f /usr/local/etc/haproxy/backends/
```

Then keep one `*.cfg` file per service in `frontends/` and `backends/`.
HAProxy merges them at startup — order within a directory is alphabetical.

---

## Useful commands

| Task | Command |
|------|---------|
| Validate config | `docker compose run --rm haproxy` |
| Reload (zero-downtime) | `docker kill -s HUP haproxy` |
| Tail logs | `docker compose logs -f haproxy` |
| Stats dashboard | http://localhost:8404/stats |
| Runtime API (if socket enabled) | `echo "show info" \| socat stdio /tmp/haproxy.sock` |

---

## Load balancing algorithms

| Algorithm | `balance` value | Best for |
|-----------|----------------|----------|
| Round robin | `roundrobin` | Stateless, equal-weight apps |
| Least connections | `leastconn` | Long-lived / async workloads |
| Source IP hash | `source` | Sticky sessions without cookies |
| Random | `random` | Simple horizontal scale-out |
| URI hash | `uri` | Cache-friendly routing |
| First Available | `first` | Active-passive or single-server proxy setups |