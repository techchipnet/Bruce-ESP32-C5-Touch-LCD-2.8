<div align="center">


# Bruce for Waveshare ESP32-C5-Touch-LCD-2.8
<img width="400" height="400" alt="bruce-c5 logo" src="https://github.com/user-attachments/assets/d783cb62-5f91-448b-a41c-fe661c59c53f" />
</div>
Community port of [Bruce Firmware](https://github.com/BruceDevices/firmware) for the Waveshare **ESP32-C5-Touch-LCD-2.8** board, prepared for the TechChip community.

> This is an unofficial board port. It is not an official Bruce Devices or Waveshare release.
All primary firmware functionality and the Bruce user interface come from the excellent **Bruce Firmware** project:

## What this port adds

- ESP32-C5 board target for the Waveshare ESP32-C5-Touch-LCD-2.8
- ST7789 240 x 320 display initialization
- Touch support
- TF/microSD support on the board's shared SPI bus
- Onboard SHTC3 temperature and humidity screen under `Others > Environment`
- Optional external nRF24L01(+) support using `CE=GPIO2`, `CSN=GPIO3` and shared SPI pins `GPIO6/7/8`

## Release file

Flash this merged image at address `0x0`:

[Releases](https://github.com/techchipnet/Bruce-ESP32-C5-Touch-LCD-2.8/releases) 

SHA-256:

```text
8ece27c994e9d7ae9d569c1d42a4c7c9cfdf8761846a40acc8a39fe0bc53a136
```

The merged image contains the bootloader, partition table and application. Do not flash it at `0x10000`.

## Requirements

- Waveshare ESP32-C5-Touch-LCD-2.8
- Data-capable USB-C cable
- Python 3
- Espressif `esptool`

Install esptool:

```bash
python -m pip install --upgrade esptool
```

After installation, close and reopen PowerShell so the updated PATH is loaded.
On Linux/macOS, use `python3` if `python` is not recognized. The Windows
`py` launcher is optional and is not installed on every system.

## Flashing on Windows

1. Connect the board by USB-C.
2. Open PowerShell in the folder containing the BIN file.
3. Find the COM port in Device Manager, for example `COM5`.
4. Run:

```powershell
python -m esptool --chip esp32c5 --port COM5 --baud 115200 write-flash 0x0 Bruce-waveshare-c5-touch-28.bin
```

If `python` is not recognized but the standalone esptool command is available,
use:

```powershell
esptool --chip esp32c5 --port COM5 --baud 115200 write-flash 0x0 Bruce-waveshare-c5-touch-28.bin
```

If your Windows installation includes the optional Python Launcher, `py -m
esptool` is also valid. Replace `COM5` with the actual port. If normal
connection fails, hold **BOOT**, press and release **RESET**, release **BOOT**,
then run the command again.

## Flashing on Linux

Find the serial port:

```bash
ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null
```

Flash, replacing `/dev/ttyACM0` when needed:

```bash
python3 -m esptool --chip esp32c5 --port /dev/ttyACM0 --baud 115200 write-flash 0x0 Bruce-waveshare-c5-touch-28.bin
```

If permission is denied, add your account to the serial-port group and log in again:

```bash
sudo usermod -aG dialout "$USER"
```

## Flashing on macOS

Find the port:

```bash
ls /dev/cu.usbmodem* /dev/cu.usbserial* 2>/dev/null
```

Flash, replacing the port when needed:

```bash
python3 -m esptool --chip esp32c5 --port /dev/cu.usbmodem101 --baud 115200 write-flash 0x0 Bruce-waveshare-c5-touch-28.bin
```

For reliability this release uses 115200 baud. A higher baud rate may work, but is more sensitive to the USB cable and hub.

## nRF24L01(+) connection

### Recommended connection used by this firmware

| nRF24L01 pin | ESP32-C5 signal | Physical connection |
|---|---:|---|
| `VCC` | `3V3` | SH1.0 12-pin connector |
| `GND` | `GND` | SH1.0 12-pin connector |
| `CE` | `GPIO2` | SH1.0 12-pin connector |
| `CSN` / `CS` | `GPIO3` | SH1.0 12-pin connector |
| `SCK` | `GPIO6` | Shared LCD/TF SPI pad or trace |
| `MOSI` | `GPIO7` | Shared LCD/TF SPI pad or trace |
| `MISO` | `GPIO8` | TF-card SPI pad or trace |
| `IRQ` | Not used | Leave disconnected |

The 12-pin connector does **not** expose GPIO6, GPIO7 and GPIO8. Therefore the recommended wiring is only partly made through the SH1.0 cable; the three SPI wires must connect to the corresponding board pads/traces. LCD, TF card and nRF24 share the SPI clock/data lines, while each device has a separate chip-select signal.

### SH1.0 12-pin board labels

With the board oriented exactly like the Waveshare interface illustration, the connector labels are:

| Position | Label |
|---:|---|
| 1 | `RST` |
| 2 | `BOOT / GPIO28` |
| 3 | `EXIO7` |
| 4 | `EXIO6` |
| 5 | `GPIO3` |
| 6 | `GPIO2` |
| 7 | `3V3` |
| 8 | `GND` |
| 9 | `D+ / GPIO14` |
| 10 | `D- / GPIO13` |
| 11 | `VBUS / 5V` |
| 12 | `GND` |

Cable wire order can appear mirrored at the loose end. Verify the board silkscreen or check continuity before applying power.

### nRF24 power precautions

- Power the radio from **3.3V only**. Never connect its VCC pin to VBUS/5V.
- Place a `10 uF` to `47 uF` capacitor between VCC and GND close to the radio module. A `100 nF` ceramic capacitor in parallel is also useful.
- High-power PA/LNA nRF24 modules may require a separate clean 3.3V regulator; do not assume the connector can supply their peak current reliably.
- Keep SPI wires short, especially SCK.
- Disconnect power before changing wiring.

Using GPIO13/GPIO14 as an all-SH1.0 software SPI alternative is not enabled in this release because those pins are the board's USB D-/D+ connection and can interfere with flashing and USB serial.

## Temperature and humidity

Open:

```text
Others > Environment
```

The screen reads the onboard Sensirion SHTC3 over the shared I2C bus and displays:

- Temperature in degrees Celsius
- Relative humidity in percent
- `Refresh` to request a new sample
- `Back` to return to the Others menu

The touch controller and SHTC3 share GPIO0/GPIO1 I2C. This port serializes their bus transactions so touch input does not corrupt sensor reads.

The sensor is mounted inside the device enclosure and can be warmed by the ESP32, display backlight, charging circuit and the user's hand. Treat it as an onboard/environment indication rather than a calibrated ambient-weather instrument. For a more representative reading, allow the board to stabilize and avoid holding or charging it during measurement.


## Restoring or troubleshooting

- Keep a backup of the original factory flash before experimenting.
- If the board is not detected, try another data-capable USB cable and connect directly without a hub.
- For download mode: hold **BOOT**, tap **RESET**, then release **BOOT**.
- If flashing disconnects at a high baud rate, retry at `115200`.
- If nRF24 is not detected, first confirm 3.3V, common ground, CE/CSN assignment and SPI continuity; then add the local decoupling capacitor.

## Responsible use

Bruce includes wireless and security-testing capabilities. Use them only on systems, networks and devices you own or have explicit authorization to test. You are responsible for complying with local laws and radio regulations.

## Useful links

- Bruce Firmware: <https://github.com/BruceDevices/firmware>
- Bruce documentation: <https://bruce.computer>
- Waveshare board documentation: <https://docs.waveshare.com/ESP32-C5-Touch-LCD-2.8>
- Waveshare product page: <https://www.waveshare.com/esp32-c5-touch-lcd-2.8.htm>
