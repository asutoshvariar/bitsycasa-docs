# BitsyCasa

**A pocket-sized, private home server with a paper-like face.**

BitsyCasa is a Raspberry Pi 4 Model B wearing a Waveshare 3.97" e-Paper HAT+.
It shares files with your devices, shows its own health on an always-on
800 × 480 e-paper screen, and stays invisible to the public internet. Your
devices reach it over Tailscale at `bitsy.casa`, and it still gets
real HTTPS certificates.

## Elevator pitch

> A Raspberry Pi, a glare-free e-paper screen, and a 3D-printed case
> make a home server you can actually *see*. BitsyCasa serves your files
> privately over Tailscale with a real domain and real HTTPS, opens no ports to
> the internet, and shows weather, to-dos, and server vitals at a glance,
> plus a couple of hidden easter-egg screens. It's small, quiet, and frugal
> with power and storage, and it's simple enough for a hobbyist to rebuild in
> an afternoon.

## Why "Geoff"?

Internally the project is called **Geoff**, after *Geoff*, the tiny
homemade electric car from *Top Gear*, known for its hand-painted lettering.
BitsyCasa has the same spirit: small, a bit scrappy, and charming. The
3D-printed case has **"geoff"** engraved on the bezel in *Reenie Beanie*, a
brush-script font chosen to look like the car's paint job. See
[Hardware → Case](hardware.md#3d-printed-case).

## What's in the box

| Piece | What it does |
|---|---|
| Raspberry Pi 4 Model B | The server |
| Waveshare 3.97" e-Paper HAT+ (SKU 31063) | 800 × 480 monochrome dashboard over SPI |
| `server_dash.py` + `epaper-dash.service` | Draws the dashboard on boot ([Dashboard](dashboard.md)) |
| Tailscale | Private network; the only way in |
| Docker Compose: copyparty + Caddy | File server plus HTTPS reverse proxy ([File server](fileserver.md)) |
| `copyparty.service` | Starts the Compose stack once Docker and Tailscale are up |
| `bitsy.casa` (Porkbun DNS) | A record points to the Pi's Tailscale IP |
| 3D-printed snap-fit case | Holds Pi + HAT, with "geoff" on the bezel |

## Architecture at a glance

```text
 your laptop / phone (on the tailnet)
            │  https://bitsy.casa  (or https://100.x.y.z)
            ▼
   ┌─────────────── Raspberry Pi 4B ────────────────┐
   │ tailscale0  100.x.y.z                          │
   │   └─ Caddy :443 (bound to Tailscale IP only)   │
   │        └─ reverse_proxy → 127.0.0.1:3923       │
   │              └─ copyparty (Docker, localhost)  │
   │                                                │
   │ epaper-dash.service → server_dash.py → SPI ──► e-paper HAT+
   └────────────────────────────────────────────────┘
   Certificates: Let's Encrypt via DNS-01 (Porkbun API), so no inbound ports.
```

!!! note "Conventions in this wiki"
    Values in angle brackets, like `<your-tailnet-ip>` are placeholders you need to fill in for your own build.
    `100.x.y.z` always means *your Pi's Tailscale IPv4 address*
    (`tailscale ip -4`).
