# fony-espuino

Modified version of [ESPuino](https://github.com/biologist79/ESPuino) (RFID-controlled ESP32 music player) for the Fony player project.
Upstream documentation (mostly German): https://forum.espuino.de

## Hardware

- Board: **ESPuino Complete** (ESP32-WROVER, 16 MB flash, PSRAM, SD in SD_MMC mode, MAX98357A amp, PCA9555 port expander)
- PlatformIO environment: `complete` (HAL=6, config in `src/settings.h` + `src/settings-complete.h`)
- RFID reader: **PN5180** (autodetected at boot)
- Rotary encoder wired to the board
- Cards used: Mifare and **NTAG424 DNA**. The NTAG424 cards carry an NDEF URL (playfony.com links).

## Goals / planned modifications

- Read the NDEF URL from NTAG424 cards (playfony.com links) and use it to decide which album to play, instead of relying on the UID-derived card number. RFID handling is in `src/RfidPn5180.cpp`.
- More to come; keep changes on feature branches (see Git workflow).

## Development environment (Linux)

- VS Code + PlatformIO IDE extension. `pio` = `~/.platformio/penv/bin/pio` (on PATH).
- **PlatformIO Core is pinned to 6.1.19. Do NOT run `pio upgrade` or `pio system prune`**, and ignore the "new version 6.2.0 available" notice.
  Core 6.2.0 breaks this build at link time with `ModuleNotFoundError: No module named 'SCons.Tool.FortranCommon'`
  (pioarduino/platform-espressif32 issue #529). If that error appears, check `pio --version`, and if needed:
  `~/.platformio/penv/bin/python -m pip install "platformio==6.1.19" && rm -rf ~/.platformio/packages/tool-scons*`
- A full build from scratch takes ~4–5 minutes; incremental builds are much faster.

## Commands

```bash
pio run -e complete                              # build
pio run -e complete -t upload                    # build + flash
pio device list                                  # find serial port (board: /dev/ttyUSB0 — verify)
timeout 30 pio device monitor -e complete > serial.log 2>&1   # capture serial output (never run monitor without a timeout)
```

- Serial: 115200 baud, `esp32_exception_decoder` filter enabled.
- If upload fails with "Failed to connect", the board is probably in deep sleep: ask the user to press the encoder button, then retry.
- Machine-specific ports go in `platformio-override.ini` (not committed).

## Safety rules

- **Never erase flash** (no `-t erase`, no `esptool.py erase_flash`). NVS holds WiFi config and RFID card mappings.
- Don't change partition tables (`custom_16mb_ota.csv`) without asking; it can wipe NVS/data.
- A full flash backup of the original board lives outside the repo at `~/Documents/fony/espuino-original-backup.bin`. Never commit `.bin` backups, logs with credentials, or `platformio-override.ini`.
- Ask before flashing if the user may not be at the board, and always confirm a flash by checking serial output afterwards.

## Git workflow

- `origin` = https://github.com/nopararas/fony_espuino (our repo), `upstream` = https://github.com/biologist79/ESPuino (original).
- Base branch: `master`. Do each modification on its own branch, e.g. `fony/ndef-album-lookup`.
- Pull upstream updates with `git fetch upstream && git merge upstream/master`.
- Before committing: make sure `pio run -e complete` builds. Format C/C++ with the repo's `.clang-format`.
- Commit with clear messages; push to `origin`. Never force-push `master`.

## License

ESPuino is GPL-3.0; derived firmware must stay GPL-3.0 and source must be available if firmware is distributed.
