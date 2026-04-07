# Proxy

Caddy reverse proxy that provides TLS termination and split-view routing for the attack demonstration. Runs with `network_mode: host` so it can distinguish local vs external traffic by source IP.

## Split-view routing

The key mechanism for Scenario 3 (split-world attack detection):

| Domain | Local traffic (monitor) | External traffic (victim) |
|--------|------------------------|--------------------------|
| `loga.jvgc-a.com` | Log A **public** instance | Log A **attack** instance |
| `logb.jvgc-a.com` | Log B **public** instance | Log B **attack** instance |
| `api.jvgc-a.com` | Backend API | Backend API |
| `logs.jvgc-a.com` | log-list.json | log-list.json |

The monitor runs with `network_mode: host` and `extra_hosts` resolving log domains to `127.0.0.1`, so its requests arrive from localhost. Caddy's `remote_ip` matcher routes these to the honest public log instances. The victim's browser connects from an external IP and gets routed to the attack instances.

## Prerequisites

- Docker
- DNS records for `*.jvgc-a.com` pointing to this server
- CT logs and backend already running on localhost ports
- `log-list.json` at the path specified by `LOG_LIST_PATH`

## Setup

```bash
cp .env.example .env
# Edit .env if ports differ from defaults
docker compose up -d
```

## Configuration

Environment variables (`.env`):

| Variable | Default | Description |
|----------|---------|-------------|
| `API_PORT` | 3000 | Backend API port |
| `LOG_A_ATTACK_PORT` | 8081 | Log A attack instance |
| `LOG_A_PUBLIC_PORT` | 8082 | Log A public instance |
| `LOG_B_ATTACK_PORT` | 8083 | Log B attack instance |
| `LOG_B_PUBLIC_PORT` | 8084 | Log B public instance |
| `LOG_LIST_PATH` | /root/ct-logs/log-list.json | Path to log-list.json on host |

## Project structure

```
Caddyfile             # Routing rules with split-view logic
docker-compose.yml    # Caddy container with host networking
.env.example          # Default port configuration
```
