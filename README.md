# Bruce firmware — CYD ESP32 2.4" (resistive touch)

Working build of the [Bruce](https://github.com/pr3y/Bruce) firmware (tag `1.16.1`) for the
**ESP32 "Cheap Yellow Display" 2.4" with resistive touch (S024R)**.

The file `Bruce-CYD-2432W328R-or-S024R.bin` is a **merged image**: bootloader + partition table +
firmware in one file. Flash it at offset `0x0`.

## Hardware tested

| Item | Value |
|---|---|
| Board | CYD ESP32-2432S024R (2.4" TFT, resistive touch) |
| Chip | ESP32-D0WD-V3, revision v3.1 |
| Features | Wi-Fi, Bluetooth, dual core, 240 MHz |
| Crystal | 40 MHz |
| PSRAM | none |
| Touch controller | XPT2046 (SPI, resistive) |
| USB-serial port (Linux) | `/dev/ttyUSB0` |
| SD card | not required (Bruce boots without one) |

## Build environment

PlatformIO env: **`CYD-2432W328R-or-S024R`** (defined in `boards/CYD-2432S028/CYD-2432S028.ini`).

Key flags:

```ini
-DTFT_INVERSION_ON
-DTFT_BL=27
-DTOUCH_OFFSET_ROTATION=1
-DSPI_FREQUENCY=55000000
-DSPI_READ_FREQUENCY=20000000
-DSPI_TOUCH_FREQUENCY=2500000
-DTOUCH_CS=33
-DUSE_TFT_eSPI_TOUCH
-DTOUCH_CONFIG_INT_GPIO_NUM=36
```

## Flash (Linux / macOS)

Install esptool:

```bash
pip install esptool
```

Find the port:

```bash
ls /dev/ttyUSB* /dev/ttyACM*
```

Erase, then flash the merged image:

```bash
esptool --chip esp32 --port /dev/ttyUSB0 erase-flash
esptool --chip esp32 --port /dev/ttyUSB0 --baud 460800 write-flash 0x0 Bruce-CYD-2432W328R-or-S024R.bin
```

If you get a permission error: `sudo usermod -aG dialout $USER`, then log out and back in.

If the board does not enter flash mode: hold **BOOT**, press **RESET**, release **BOOT**, run the command again.

### Web flasher (no command line)

Open a Web Serial flasher in Chrome or Edge (for example https://web.esphome.io), select the `.bin`
file and flash it at `0x0`.

## Build from source

```bash
git clone https://github.com/pr3y/Bruce.git
cd Bruce
git checkout 1.16.1
pio run -e CYD-2432W328R-or-S024R
```

The build produces `.pio/build/CYD-2432W328R-or-S024R/{bootloader,partitions,firmware}.bin`
and, via the merge script, `Bruce-CYD-2432W328R-or-S024R.bin`.

To merge manually:

```bash
esptool --chip esp32 merge-bin --output Bruce-CYD-2432W328R-or-S024R.bin \
  0x1000  .pio/build/CYD-2432W328R-or-S024R/bootloader.bin \
  0x8000  .pio/build/CYD-2432W328R-or-S024R/partitions.bin \
  0x10000 .pio/build/CYD-2432W328R-or-S024R/firmware.bin
```

Flash the three parts separately instead:

```bash
esptool --chip esp32 --port /dev/ttyUSB0 --baud 460800 write-flash \
  0x1000  .pio/build/CYD-2432W328R-or-S024R/bootloader.bin \
  0x8000  .pio/build/CYD-2432W328R-or-S024R/partitions.bin \
  0x10000 .pio/build/CYD-2432W328R-or-S024R/firmware.bin
```

## Serial logs

```bash
pio device monitor -p /dev/ttyUSB0 -b 115200
```

A healthy boot ends with `stage=first-mainMenu`.

## Troubleshooting

### Screen shows the Wi-Fi logo and nothing responds
The firmware is running; the touch is not being read. This happened with a custom env
(`CYD-S024R-Fix`) that was missing three flags:

- `-DUSE_TFT_eSPI_TOUCH` (enables the XPT2046 touch driver)
- `-DTOUCH_OFFSET_ROTATION=1`
- `-DTOUCH_CONFIG_INT_GPIO_NUM=36`

Using the official `CYD-2432W328R-or-S024R` env fixes it.

### Touch works but in the wrong place
Change `-DTOUCH_OFFSET_ROTATION` to `0`, `2` or `3`, rebuild and reflash.

### Touch never responds
Your board may be the capacitive version (S024C). Use the `CYD-2432W328C_2` env instead.

### Resistive screen tips
Press firmly with a fingernail or stylus; a light touch is not enough.

### Wrong serial port or no port
- Try another USB cable (many are charge-only).
- Check `dmesg | tail` for `ttyUSB0` / `ttyACM0`.
- Remove `brltty` if it grabs CH340 chips.

## Notes

- esptool prints a deprecation warning for `esptool.py`; use `esptool` instead.
- Bruce is licensed under AGPL-3.0; keep the license and credit the original project
  when redistributing binaries.
- Use at your own risk, and only on devices and networks you own or have permission to test.

## Credits

[Bruce](https://github.com/pr3y/Bruce) by pr3y and contributors.
