# smart plug ZLD-44EU(FR)-W-YT

ESPHome firmware for the ZLD-44EU(FR)-W-YT power strip: 4 sockets and a USB port, controlled from
Home Assistant. Config: [`smart-plug-zld-44eufr-w-yt.yaml`](smart-plug-zld-44eufr-w-yt.yaml).

## Device

- PCB marking: `ZLD-44EU(FR)-W-YT (V:2.0)`
- Chip: BK7231N, Tuya QFN32 module (`generic-bk7231n-qfn32-tuya`)
- 4 relay-switched sockets + switchable USB power
- The stock Tuya firmware could not be exploited by tuya-cloudcutter or tuya-convert, so the first
  flash has to go over serial (see [First flash](#first-flash)).

## Features

Exposed to Home Assistant:

- **Socket 1–4**, **USB**: switches. They come back on after a power loss and after waking from
  the power button (`RESTORE_DEFAULT_ON`); the power button cuts the relays without saving "off".
- **Turn off**: button, same as clicking the power button.
- **Uptime**, **Internal Temperature**: sensors.

## Button & LED

| Action | Result |
|---|---|
| Click (50 ms – 1 s) | All sockets and USB off, LED off, deep sleep |
| Long press while asleep | Wake up (short clicks don't wake it) |
| Hold | LED starts strobing after 5 s |
| Hold ≥ 6 s, then release | Factory reset |

The LED is on while the plug runs normally and blinks while it isn't connected or reports an error.

## Setup

No Wi-Fi credentials or API key are compiled in.

1. After a fresh flash or a factory reset the plug starts an open hotspot, `smart-plug-44`.
2. Join it from a phone; the captive portal opens. Pick your Wi-Fi network.
3. Add the discovered device in Home Assistant. HA generates the API encryption key and pushes it to
   the plug, which keeps it in flash.

A factory reset wipes both the Wi-Fi credentials and the API key; remove and re-add the device in
Home Assistant afterwards.

> While the hotspot is up, its captive portal accepts firmware uploads without authentication.
> It only runs while the plug is unconfigured, so set it up right after flashing or resetting.

## Pinout

| Pin | Function | Notes |
|---|---|---|
| P10 | Power button | Active-low, internal pull-up; also the deep-sleep wake pin |
| P11 | Status LED | Inverted |
| P6 | Socket 1 relay | |
| P7 | Socket 2 relay | |
| P24 | Socket 3 relay | |
| P8 | Socket 4 relay | |
| P26 | USB power | Inverted |

## First flash

Follow the LibreTiny guide: <https://docs.libretiny.eu/docs/platform/beken-72xx/#flashing_1>.
In short:

1. Solder wires to RX1, TX1, 3.3 V and GND on the module.
2. Connect a 3.3 V USB-UART adapter (TX ↔ RX). **NOTE**: better to use an exernal power supply (with it's GND connected to the adapter GND) than powering the plug from the adapter.
3. Put the chip into download mode by resetting it (CEN to GND briefly).
4. Flash the ESPHome build with `ltchiptool`.

After that, all updates go over OTA.

## Build & OTA

- ESPHome ≥ 2026.7.0 (built with ESPHome Device Builder).
- One secret, `smart_plug_zld_44eufr_w_yt__encryption_key`: the OTA encryption key. The API key comes
  from Home Assistant, not from secrets.
- Logger level is `INFO`. For debugging, raise it to `DEBUG` and reflash.

## Known limitations

- Waking from deep sleep needs a long press on the power button; short clicks are ignored by the
  BK7231N.
- Power monitoring (BL0937) doesn't work yet: its pins aren't mapped and the `hlw8012` block is
  commented out in the YAML.
