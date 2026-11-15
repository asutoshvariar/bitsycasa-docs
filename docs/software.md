# Software stack

BitsyCasa is deliberately small. Every piece was chosen to run on a Pi 4 with
very little free storage and to keep the server off the public internet.

| Layer | Component | Where it lives |
|---|---|---|
| OS | Raspberry Pi OS (64-bit recommended) | microSD |
| Network | Tailscale (`tailscaled.service`) | system package |
| Display | `server_dash.py` + Waveshare `epd3in97` driver | `~/server_dash.py`, `~/3in97_e-Paper/...` |
| Display service | `epaper-dash.service` | `/etc/systemd/system/` |
| Containers | Docker Engine + Compose plugin | system package |
| File server | copyparty (`copyparty/min` image) | `~/copyparty/` |
| Reverse proxy / TLS | Caddy (`caddy:2-alpine` + prebuilt binary with the Porkbun DNS module) | `~/copyparty/` |
| Stack service | `copyparty.service` | `/etc/systemd/system/` |
| DNS | `bitsy.casa` → Tailscale IP | Porkbun dashboard |

## Dashboard script

`~/server_dash.py` is a single Python 3 file that uses Pillow to draw an
800 × 480 1-bit image and pushes it to the panel with the official `epd3in97`
driver. It runs from a virtualenv at `~/.venv`. Details are on the
[Dashboard](dashboard.md) page.

- Weather comes from [Open-Meteo](https://open-meteo.com/), which needs no API key.
- Metrics come from `psutil`, with fallbacks, plus `tailscale ip -4`.
- Notes are read from `~/dash_notes.txt`.
- Hidden modes are turned on with flag files `/tmp/doom` and `/tmp/odyssey`.

## Tailscale

Tailscale is the only way into BitsyCasa. Caddy binds **only** to the Pi's
Tailscale IPv4 address, and copyparty binds only to `127.0.0.1`.

## Docker Compose: copyparty + Caddy

Everything for the file server lives in `~/copyparty/`:

```text
~/copyparty/
├── docker-compose.yml
├── .env                 # TS_IP, DOMAIN, ACME_EMAIL, Porkbun keys (chmod 600)
├── Caddyfile
├── caddy-bin            # prebuilt Caddy binary that includes the Porkbun DNS module
├── caddy-data/          # certificates, Caddy local CA
├── caddy-config/
├── cfg/copyparty.conf
└── data/                # your files (mounted at /w in the copyparty container)
```

Full file contents are on the [File server](fileserver.md) page.

## systemd services

| Unit | Type | Purpose |
|---|---|---|
| `epaper-dash.service` | `simple` | Runs `~/.venv/bin/python ~/server_dash.py` as your user, with `gpio` and `spi` groups. Waits for `/dev/spidev0.0`. Restarts on failure. |
| `copyparty.service` | `oneshot` + `RemainAfterExit` | Waits for Docker and a Tailscale IP, then runs `docker compose up -d`; `stop` runs `docker compose down`. |
| `docker.service` | system | Must be enabled (`sudo systemctl enable docker`). |
| `tailscaled.service` | system | Installed by Tailscale. |

## The `bitsy.casa` domain

- Registrar and DNS: **Porkbun**, with API access turned on for the domain.
- Record: `A  bitsy.casa → 100.x.y.z` (the Pi's Tailscale IP).
- Certificates: Let's Encrypt through the **ACME DNS-01** challenge. Caddy
  uses the Porkbun API to create the `_acme-challenge` TXT record, so Let's
  Encrypt never has to connect to the Pi.

!!! note
    The A record is public, but a `100.64.0.0/10` address isn't routable on
    the internet. Anyone off your tailnet who looks up the name gets an
    address they can't reach.
