# Hardware

## Bill of materials

| Part | Notes |
|---|---|
| Raspberry Pi 4 Model B | Any RAM size. Board is 85 × 56 mm. |
| Waveshare 3.97inch e-Paper HAT+ (SKU 31063) | 800 × 480, black/white, SPI, 40-pin HAT+ form factor. Driver board 99.5 × 60 mm. |
| 2×20 **stacking** header, 16–20 mm | Needed with the printed case; see [the header note](#stacking-header). |
| microSD card | The build was done on a card with only **~1.8 GB free**, so plan for that or buy bigger. See [storage limits](troubleshooting.md#storage-limits-xcaddy-build-fills-the-disk). |
| USB-C power supply | Official Pi 4 PSU (5 V / 3 A) recommended. |
| 4 × M2.5 screws | Fasten the Pi to the case standoffs. |
| 3D-printed case | ~30–35 g PLA; see [3D-printed case](#3d-printed-case). |

## The e-paper HAT+

| Property | Value |
|---|---|
| Resolution | 800 × 480 px, 1-bit (black/white) |
| Interface | SPI (4-wire) via the 40-pin header |
| Raw panel | 96.62 × 56.24 × 0.92 mm |
| Active area | 86.40 × 51.84 mm |
| Driver module | `epd3in97` from [Waveshare's site](https://files.waveshare.com/wiki/3.97inch_e-Paper_HAT%2B/3in97_e-Paper.zip) |
| Refresh | Full refresh (`init()` + `display()`) and faster refresh (`init_Fast()`) |

### SPI and GPIO

The HAT talks over **SPI0** with chip-select CE0, which appears as
`/dev/spidev0.0` once SPI is enabled. It also uses a few GPIO lines for reset,
data/command, busy, and panel power.

| Signal | BCM GPIO (Waveshare default) |
|---|---|
| MOSI / SCLK | GPIO 10 / GPIO 11 (SPI0) |
| CS | GPIO 8 (CE0) |
| DC | GPIO 25 |
| RST | GPIO 17 |
| BUSY | GPIO 24 |
| PWR | GPIO 18 |

Enable SPI with:

```bash
sudo raspi-config nonint do_spi 0
ls /dev/spidev0.*        # should list /dev/spidev0.0 (reboot once if not)
```

## 3D-printed case

| Feature | Detail |
|---|---|
| Plate footprint | ~221 × 69 mm (base tray + bezel side by side) |
| Base tray | ~18 mm tall; standoffs for the Pi; cutouts for USB-C, 2 × micro-HDMI, audio, USB-A ×4, Ethernet, microSD; floor vents |
| Pi mounting holes | Official 58 × 49 mm pattern, 3.5 mm from the SD-card and USB-C edges, opened to Ø2.75 mm for M2.5 screws |
| Bezel | 2.8 mm thick, window sized to the official 86.4 × 51.84 mm active area (+0.6 mm margin), pocket for the 0.92 mm panel |
| Closure | Locating lip plus **4 flexible clips** that catch in slots in the long walls; pry notch at the SD-card end |
| Engraving | **"geoff"** in *Reenie Beanie* (SIL OFL), 0.6 mm deep, ~4 mm tall, on the bezel strip at the USB end |
| Printing | No supports; ~30–35 g PLA |

### Stacking header

!!! warning "The HAT sits over the USB and Ethernet ports"
    The case puts the HAT PCB **11.5 mm above the Pi PCB**. That clears the tall USB-A and Ethernet housings.
    A standard short header will likely make the HAT hit those ports, so use
    a **16–20 mm 2×20 stacking/extension header**.

!!! warning "HAT position is estimated"
    Waveshare doesn't publish a dimensioned drawing of where the HAT sits
    relative to the 40-pin header. The case derives it from official product
    photos: the HAT overhangs ~13.2 mm past the SD-card end and ~1.3 mm past
    the USB end. **Measure your board before printing the bezel**.
