# Troubleshooting

These are the problems that came up while building BitsyCasa, plus a few common
neighbours.

## Quick diagnostics

```bash
systemctl status epaper-dash copyparty
journalctl -u epaper-dash -b --no-pager | tail -50
journalctl -u copyparty -b --no-pager
docker compose -f ~/copyparty/docker-compose.yml logs --tail 50
ls -l /dev/spidev* /dev/gpiochip*
tailscale status && tailscale ip -4
df -h /
sudo ss -tlnp | grep -E ':(80|443|3923)\b'
```

---

## E-paper

### Driver model and path: `could not import epd3in97`

**Symptom:** the service log shows `could not import epd3in97`, or the panel
stays blank or garbled.

**Cause:** the script can't find the driver for **this** panel, or it's using
a driver for a different Waveshare model. The 3.97" HAT+ needs the
`epd3in97` module. Drivers for other sizes use different controllers and
resolutions and won't work.

**Fix:**

- Make sure you're using the right [repo](https://files.waveshare.com/wiki/3.97inch_e-Paper_HAT%2B/3in97_e-Paper.zip)
- Make sure this file exists:
  `~/3in97_e-Paper/RaspberryPi_JetsonNano/python/lib/waveshare_epd/epd3in97.py`
  (or the same path under `~/e-Paper/E-paper_Separate_Program/`).
- The official folder is **`RaspberryPi_JetsonNano`**, not `RaspberryPi`. The
  script checks both.
- Or point `PYTHONPATH` at `.../3in97_e-Paper/RaspberryPi_JetsonNano/python/lib`.

### `Permission denied` on `/dev/spidev0.0` or `/dev/gpiochip*`

**Fix:** enable SPI with `sudo raspi-config nonint do_spi 0` and join the groups
with `sudo usermod -aG gpio,spi "$USER"`, then log in again or reboot. The
service sets `SupplementaryGroups=gpio spi`. On newer images the device group
can differ, so check `ls -l /dev/gpiochip* /dev/spidev*` and adjust.

### Ghosting

That's normal for e-paper. The script does a full refresh every
`FULL_REFRESH_EVERY` (6) cycles. Lower that number if ghosting bothers you.

---

## Network and DNS


### Campus DNS (UMass RPZ) and `bitsy.casa`

**Symptom:** on the campus network, `bitsy.casa` doesn't resolve, or
resolves to the wrong address, even though the Porkbun record is correct.

**Cause:** the university's resolvers apply a DNS **Response Policy Zone
(RPZ)**. Like many campus and enterprise resolvers, they block or rewrite
answers, including public names that point at private or CGNAT addresses
such as `100.x.y.z` (DNS-rebinding protection).

**Fixes, easiest first:**

1. On the client, turn on **Use Tailscale DNS settings**
   (`tailscale set --accept-dns=true`), so lookups don't go to the campus
   resolver. However, this may not always work.
2. Add a hosts-file entry: `100.x.y.z  bitsy.casa`
   (`/etc/hosts`, or `C:\Windows\System32\drivers\etc\hosts`).
3. Fall back to `https://100.x.y.z`.

### copyparty: "rejected by cors-check"

**Symptom:** copyparty shows `rejected by cors-check (see fileserver log)`.

**Cause:** requests from Caddy arrive via the Docker bridge (`172.x.x.x`), but
`xff-src: 127.0.0.1` told copyparty to trust only localhost. Copyparty
ignored `X-Forwarded-Proto` and `X-Forwarded-Host`, assumed plain `http`, and saw
an `Origin` of `https://bitsy.casa` that didn't match.

**Fix:** in `cfg/copyparty.conf`, under `[global]`:

```ini
  rproxy: 1
  xff-src: lan
  xff-hdr: x-forwarded-for
  xf-proto: x-forwarded-proto
  xf-host: x-forwarded-host
```

Then run `docker compose restart copyparty`. If it still fails, the copyparty
log line shows the expected and received origins. As a last resort, add
`allow-csrf` under `[global]`. That's lower risk on a tailnet, but not
preferred.

### Certificate not issued

- Check that API Access is **on** for `bitsy.casa` in Porkbun, and that the keys
  in `.env` are correct.
- Run `docker compose logs caddy | grep -i -E 'acme|challenge|error'`.
- Let's Encrypt rate-limits repeated failures, so fix the problem before retrying.

---

## 3D-printed case

| Problem | Fix |
|---|---|
| HAT hits the USB and Ethernet ports | Use a 16–20 mm stacking header (the case puts the HAT 11.5 mm above the Pi). |
| Screws too tight | Holes are Ø2.75 mm for M2.5 (`STANDOFF_HOLE`). Raise it slightly for your printer. |
