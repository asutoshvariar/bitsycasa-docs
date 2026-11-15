# File server

BitsyCasa serves files with [copyparty](https://github.com/9001/copyparty),
running in Docker behind [Caddy](https://caddyserver.com/). It's reachable
only over Tailscale:

- `https://bitsy.casa`: real Let's Encrypt certificate (DNS-01 via Porkbun)
- `https://100.x.y.z`: Caddy's internal CA, so expect a browser warning

## Security model

```text
Internet ──✗── (no inbound ports, no port forwarding, no Funnel)

Tailnet device ──WireGuard──► tailscale0 100.x.y.z:443 ──► Caddy
                                                          │
                                       127.0.0.1:3923 ◄───┘
                                         copyparty
```

| Layer | Control |
|---|---|
| Network | Caddy is bound to the **Tailscale IP only** (`bind {$TS_IP}`). Nothing listens on `0.0.0.0`, `[::]`, or the LAN IP. |
| App | copyparty publishes only on `127.0.0.1:3923`. |
| No port 80 | `auto_https disable_redirects` stops Caddy from opening `:80`. |
| Certificates | **DNS-01**, so Let's Encrypt never connects to the Pi. HTTP-01 and TLS-ALPN-01 would need public ports. |
| Auth | A single copyparty account with `rwmd` (read/write/move/delete) on `/`. Everyone else sees a login page. |

!!! warning "Never run `tailscale funnel`"
    Funnel publishes a service to the public internet, which is exactly what
    this design avoids.

### Why no open ports?

DNS-01 is the key idea. To prove you own `bitsy.casa`, Caddy uses the
Porkbun API to write a TXT record at `_acme-challenge.bitsy.casa`, and
Let's Encrypt checks it through public DNS. The Pi only makes **outbound**
HTTPS calls, so your router, the campus firewall, and the public internet
never need to reach it.

## Files

All files live in `~/copyparty/`.

### `.env`

```bash
TS_IP=100.x.y.z                 # tailscale ip -4
DOMAIN=bitsy.casa
ACME_EMAIL=<you@example.com>
PORKBUN_API_KEY=<pk1_...>
PORKBUN_API_SECRET_KEY=<sk1_...>
```

### Prebuilt Caddy binary (Porkbun DNS module)

The stock `caddy:2-alpine` image has no DNS provider modules. Instead of
compiling one with `xcaddy`, which takes up valuable space, download a binary
that caddyserver.com builds for you:

```bash
cd ~/copyparty
curl -fL -o caddy-bin "https://caddyserver.com/api/download?os=linux&arch=arm64&p=github.com/caddy-dns/porkbun"
chmod +x caddy-bin
./caddy-bin list-modules | grep porkbun    # → dns.providers.porkbun
```

!!! warning "For 32-bit devices:"
	On 32-bit Raspberry Pi OS, use `arch=arm&arm=7` instead (`uname -m` shows `aarch64` on 64-bit). The binary is about 40 MB.

### `docker-compose.yml`

```yaml
services:
  copyparty:
    image: copyparty/min:latest
    container_name: copyparty
    restart: unless-stopped
    user: "1000:1000"            # match `id -u`:`id -g`
    ports:
      - "127.0.0.1:3923:3923"    # localhost only
    volumes:
      - ./cfg:/cfg:z
      - ./data:/w:z
    logging:
      driver: local
      options: { max-size: "5m", max-file: "2" }

  caddy:
    image: caddy:2-alpine
    container_name: caddy
    restart: unless-stopped
    network_mode: host           # bind the Tailscale IP, reach 127.0.0.1:3923
    env_file: .env
    volumes:
      - ./caddy-bin:/usr/bin/caddy:ro     # prebuilt binary with Porkbun DNS
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-data:/data
      - ./caddy-config:/config
    depends_on: [copyparty]
    logging:
      driver: local
      options: { max-size: "5m", max-file: "2" }
```

The `local` log driver with 5 MB × 2 files keeps logs from growing on the
small SD card.

### `Caddyfile`

```caddy
{
	email {$ACME_EMAIL}
	auto_https disable_redirects
	admin off
	default_sni {$TS_IP}
}

# Real certificate, DNS-01 through Porkbun
{$DOMAIN} {
	bind {$TS_IP}
	tls {
		dns porkbun {
			api_key {$PORKBUN_API_KEY}
			api_secret_key {$PORKBUN_API_SECRET_KEY}
		}
		resolvers 1.1.1.1
	}
	encode zstd gzip
	reverse_proxy 127.0.0.1:3923
}

# Raw Tailscale IP, Caddy internal CA (browser warning)
https://{$TS_IP} {
	bind {$TS_IP}
	tls internal
	encode zstd gzip
	reverse_proxy 127.0.0.1:3923
}
```

- `resolvers 1.1.1.1` makes Caddy check the challenge TXT record through a
  public resolver instead of the campus DNS. See [UMass DNS](troubleshooting.md#campus-dns-umass-rpz-and-filesbitsycasa).
- `default_sni` is needed because browsers send no SNI when you connect to an IP.
- To remove the IP warning, trust
  `~/copyparty/caddy-data/caddy/pki/authorities/local/root.crt` on your devices.

### `cfg/copyparty.conf`

```ini
[global]
  # trust reverse-proxy headers from Caddy (arrives via the docker bridge, 172.16.0.0/12)
  rproxy: 1
  xff-src: lan
  xff-hdr: x-forwarded-for
  xf-proto: x-forwarded-proto
  xf-host: x-forwarded-host
  no-robots
  no-thumb          # no thumbnail cache, saves SD-card space

[accounts]
  <username>: <strong-password>

[/]
  /w
  accs:
    rwmd: <username>
```

!!! note "Why `xff-src: lan`?"
    Caddy reaches copyparty through Docker's port mapping, so its requests
    arrive from the Docker bridge gateway (`172.x.x.x`), not `127.0.0.1`.
    With `xff-src: 127.0.0.1`, copyparty ignored the forwarded headers, thought
    the request was plain HTTP, and failed its CORS check. See
    [Troubleshooting](troubleshooting.md#copyparty-rejected-by-cors-check).

## Start, stop, update

```bash
cd ~/copyparty
mkdir -p caddy-data caddy-config
docker compose up -d
docker compose logs -f caddy           # "certificate obtained successfully"

sudo systemctl restart copyparty       # down + up via the service
sudo systemctl reload copyparty        # apply compose/Caddyfile changes
docker compose pull && docker compose up -d     # update copyparty image
```

To update Caddy, download `caddy-bin` again and run `docker compose restart caddy`.

## Verify nothing is public

```bash
sudo ss -tlnp | grep -E ':(80|443|3923)\b'
#   100.x.y.z:443     caddy
#   127.0.0.1:3923    docker-proxy (copyparty)

curl -kI https://100.x.y.z            # from a tailnet device: copyparty answers
curl -I  https://bitsy.casa     # valid cert, no -k needed
curl -m5 https://bitsy.casa     # from OFF the tailnet: must time out
```
