# BitsyCasa ("Geoff")

A tiny, private home server built on a **Raspberry Pi 4 Model B** with a
**Waveshare 3.97" e-Paper HAT+** (SKU 31063, 800 × 480, SPI) showing a live
dashboard. Files are served by **copyparty** behind **Caddy** in Docker
Compose, reachable only over **Tailscale** at `files.bitsy.casa`, with real
Let's Encrypt certificates obtained by DNS-01. The case is 3D-printed and has
"geoff" engraved on the bezel, after the little electric car from Top Gear.

This repository holds the project wiki as an [MkDocs](https://www.mkdocs.org/)
site using the [Material](https://squidfunk.github.io/mkdocs-material/) theme.

## Build locally

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r docs/requirements.txt
mkdocs serve          # http://127.0.0.1:8000
mkdocs build --strict # writes ./site
```

## Publish

- **Read the Docs:** import the repo at <https://readthedocs.org/>. The
  `.readthedocs.yaml` file in the repo root is picked up automatically.
- **GitHub Pages:** in *Settings → Pages*, set *Source* to **GitHub Actions**.
  `.github/workflows/docs.yml` builds and deploys on every push to `main`.

## Layout

```
mkdocs.yml               site config and navigation
.readthedocs.yaml        Read the Docs build config
.github/workflows/docs.yml  GitHub Pages deploy
docs/requirements.txt    pinned doc dependencies
docs/*.md                the wiki pages
```

**Before publishing**, search the docs for `TODO` and `<...>` placeholders and
fill them in. Never commit real passwords, API keys, or your tailnet IP.
