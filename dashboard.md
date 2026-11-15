# Dashboard

The e-paper screen is BitsyCasa's face. `~/server_dash.py` renders an
800 × 480, 1-bit image with Pillow and sends it to the HAT+ through the
official `epd3in97` driver. It redraws every `REFRESH_SEC` seconds (300 by
default).

## Layout

| Region | Contents |
|---|---|
| Header bar (black) | Hostname, short date, time |
| Left panel: weather | Large current temperature (96 px), condition icon, and a 5-slot hourly forecast. Icons switch to a moon or moon-and-cloud after dark, using Open-Meteo's `is_day` flag. |
| Right panel: WORK | Bulleted notes from `~/dash_notes.txt` |
| Metric tiles | **CPU %**, **CPU temp**, **memory**, **disk**, with bars that always show a sliver of fill so low values stay visible |
| Info row | **Network** throughput, **uptime**, **Tailscale IP** (`—` if offline) |
| Footer | Last-updated time |

All text uses **DejaVu Sans Bold**. Small labels were enlarged to 18–22 px
after testing on the real panel, and lines and icon strokes were thickened.

### Notes file

```text title="~/dash_notes.txt"
Finish the e-paper layout pass
Order a spare microSD card
# lines starting with # are ignored
```

One note per line. Blank lines and `#` comments are skipped, and long lines are
truncated with an ellipsis to fit.

### Data sources

| Data | Source |
|---|---|
| Weather | `https://api.open-meteo.com/v1/forecast` (no key), cached in `/tmp/server_dash_weather.json` |
| CPU, memory, disk, network, uptime, temperature | `psutil`, with fallbacks |
| Network rate | Delta between runs, cached in `/tmp/server_dash_net.json` |
| Tailscale IP | `tailscale ip -4` (2 s timeout) |

## Refresh strategy

E-paper ghosts if you only ever use fast refreshes, so the script mixes refresh
types:

- **Cycle 0:** `init()`, then `Clear()` and a full `display()`
- **Every `FULL_REFRESH_EVERY` (6) cycles:** full `init()` + `display()`
- **Otherwise:** `init_Fast()` + `display_Fast()`, falling back to `display()`
- **After every frame:** `sleep()`, which also releases the GPIO/SPI handles

SIGINT and SIGTERM are caught, so `systemctl stop epaper-dash` puts the panel
to sleep cleanly.

## Command-line options

```bash
python3 ~/server_dash.py                       # run forever (what the service does)
python3 ~/server_dash.py --once                # push one frame and exit
python3 ~/server_dash.py --preview out.png     # render a PNG, no hardware needed
python3 ~/server_dash.py -v                    # debug logging
```

## Customising

- **Location and units:** `LAT`, `LON`, `TEMP_UNIT` at the top of the script.
- **Fonts:** DejaVu is searched in `/usr/share/fonts/truetype/dejavu` and `~/.fonts`.
  Install `fonts-dejavu-core` if text looks tiny, since that means the bitmap
  fallback is being used.
