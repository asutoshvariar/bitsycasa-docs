# Sources and placeholders

This file tracks where the wiki's facts came from. It's not part of the
published site.

## Taken from the build sessions and project files

- Pi 4B + Waveshare 3.97" e-Paper HAT+ (SKU 31063, 800×480, SPI); SPI enabled with `raspi-config nonint do_spi 0`; user in `gpio`/`spi` groups.
- `~/server_dash.py` run from `~/.venv`; driver `epd3in97`; driver path `~/3in97_e-Paper/RaspberryPi_JetsonNano/python/lib` (also `~/e-Paper/E-paper_Separate_Program/...`); folder name `RaspberryPi_JetsonNano`, not `RaspberryPi`.
- Dashboard config: `REFRESH_SEC=300`, `FULL_REFRESH_EVERY=6`, `TEMP_UNIT="F"`, notes at `~/dash_notes.txt`, Open-Meteo weather, psutil metrics, `tailscale ip -4`, `--once` / `--preview` / `-v`; refresh API `init`/`init_Fast`/`display`/`display_Fast`/`Clear`/`sleep`.
- Hidden modes: `/tmp/doom` and `/tmp/odyssey`; DOOM wins; DOOM HUD mapping (AMMO/HEALTH/ARMOR); HAL 9000 screen text and telemetry strip.
- Fixes: night icons (`is_day`), bold/larger fonts for readability.
- `epaper-dash.service` and `copyparty.service` unit files, verbatim.
- copyparty (`copyparty/min`) on `127.0.0.1:3923`; Caddy `caddy:2-alpine` with `network_mode: host`, bound to `TS_IP`; `auto_https disable_redirects`, `admin off`, `default_sni`; `local` log driver 5m×2; `no-thumb`.
- Domain `bitsy.casa`, `files.bitsy.casa` A record → Tailscale IP; DNS provider **Porkbun**; DNS-01; prebuilt Caddy from caddyserver.com download API with `github.com/caddy-dns/porkbun`; `resolvers 1.1.1.1`.
- Raw-IP site with `tls internal` + `default_sni` (fix for IP access).
- CORS fix: `xff-src: lan`, `xff-hdr`, `xf-proto`, `xf-host`; `allow-csrf` last resort.
- ~1.8 GB free storage; xcaddy build failed; cleanup commands.
- University-wide tailnet; MagicDNS names unusable; university DNS may block answers pointing to private `100.x` addresses (fixes: Tailscale DNS settings, hosts file).
- Case: `epaper_pi_case.py`/`.stl`, two-part snap-fit, 4 clips + lip + pry notch, ~221×69 mm plate, ~18 mm base, holes Ø2.75 for M2.5 at the official 58×49 mm pattern, 11.5 mm HAT stack height → 16–20 mm stacking header, "geoff" in Reenie Beanie 0.6 mm deep, HAT position estimated from photos, ~30–35 g PLA, no supports.

## Generic (standard, correct practice, not specific to this build)

- OS flashing, Tailscale and Docker install one-liners, venv package list.
- Waveshare default pin map (marked TODO: confirm).
- gpiozero troubleshooting details (marked TODO: confirm).
- "RPZ" mechanism explanation and the DoH, `nslookup`, and `tailscale set --accept-dns` steps (marked TODO: confirm).
- GitHub Actions / Read the Docs config; Expanding ideas.

## Placeholders to fill in

- `100.x.y.z` (Tailscale IP), `<you@example.com>`, `<pk1_...>` / `<sk1_...>` Porkbun keys
- `<username>` / `<strong-password>` for copyparty. The real account isn't published on purpose.
- `LAT`/`LON` shown as `<your-latitude>`/`<your-longitude>`
- `site_url` / `repo_url` in `mkdocs.yml`
- TODO admonitions: pin map, driver download source, exact venv packages, the driver-model mistake, the gpiozero error, the UMass RPZ behaviour and chosen fix, whether Tailscale DNS works on the university tailnet
