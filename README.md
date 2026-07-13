<div align="center">

# ❄️ TCL Air Conditioner · ESPHome Controller

**Native ESPHome (ESP-IDF) firmware for TCL / Electriq / Daichi split ACs**
Turn a dumb IR/UART air conditioner into a fully controllable **Home Assistant** climate entity — over UART, no IR blaster required.

`UART 9600 8E1` · `ESP32-C3 / C6` · `esp-idf` · `WiFi or Thread` · `MIT`

</div>

---

## 📑 Table of Contents

- [✨ Features](#-features)
- [🧰 What You Need](#-what-you-need)
- [🔌 Wiring](#-wiring)
- [🚀 Quick Start](#-quick-start)
- [🔐 Secrets (`secrets.yaml`)](#-secrets-secretsyaml)
- [💾 Flashing](#-flashing)
- [🏠 Home Assistant Integration](#-home-assistant-integration)
- [🎛️ Entities & Controls](#-entities--controls)
- [🌡️ Supported Air Conditioners](#-supported-air-conditioners)
- [🧭 Architecture](#-architecture)
- [📦 Repository Structure](#-repository-structure)
- [🛠️ Troubleshooting](#-troubleshooting)
- [🙋 Credits](#-credits)
- [📄 License](#-license)

---

## ✨ Features

| | Feature |
|---|---|
| 🌡️ | **Full climate control** — power, mode, target temperature, fan speed, swing |
| 🏠 | **Native Home Assistant API** — no MQTT broker, no cloud, no IR blaster |
| 🔌 | **Direct UART** — talks to the AC's service port at 9600 baud, 8E1 |
| 🧱 | **ESP-IDF framework** — required for Thread / 802.15.4 support |
| 📡 | **WiFi *or* Thread** — pick one per device; Thread needs an ESP32-C6/H2 |
| 📦 | **Git-pulled packages** — your device YAML stays slim; all logic lives in the repo |
| 🖥️ | **Optional OLED** + status LEDs for TX/RX activity |

---

## 🧰 What You Need

### Recommended boards

| Board | Radio | ~Price | Notes |
|-------|-------|--------|-------|
| **ESP32-C3-DevKitM-1** | WiFi | ~8 € | Cheapest path (WiFi only) |
| **ESP32-C6-DevKitC-1** | WiFi + 802.15.4 | ~10 € | Best for Thread |
| **Seeed XIAO ESP32-C6** | WiFi + 802.15.4 | ~10 € | Tiny, breadboard-friendly |

### Other parts

- **USB-A plug / cable** to reach the AC's service port — e.g. [AliExpress](https://www.aliexpress.com/item/1005005776162012.html)
- **SSD1306 OLED 128×32** (optional, I²C)
- **2× 5 mm LEDs** + resistors (optional, TX/RX activity)

> ⚠️ Not every AC model exposes a usable UART service port. Check your unit before buying hardware — see [Supported Air Conditioners](#-supported-air-conditioners).

---

## 🔌 Wiring

### USB-A service plug → ESP32

> ⚠️ The AC's "USB-A" service port is **not real USB** — it is repurposed as a UART
> port with a **non-standard pinout**. The mechanical USB-A pins do *not* carry
> their normal signals, so do **not** treat this as a USB connection. Wire it
> exactly as below (the authoritative mapping from the
> [`sorz2122/tclac`](https://github.com/sorz2122/tclac) project):

| USB-A pin | Wire color | ESP32 pin |
|-----------|-----------|-----------|
| GND  | Black | **5V / VIN** |
| D+   | Green | **GND** |
| D−   | Gray / White | **RX** |
| VBUS | Red | **TX** |

> 🔄 **Cross TX↔RX:** the AC's **D−** goes to the ESP's **RX**, and the AC's
> **VBUS** goes to the ESP's **TX**. The GND and D+ pins are *swapped* relative
> to a normal USB cable (mechanical GND carries 5V, mechanical D+ is ground),
> so double-check with a multimeter before powering up.

### Default GPIO assignment (ESP32-C3 sample)

| Function | GPIO | Package |
|----------|------|---------|
| UART TX → AC | `GPIO1` | core.yaml (`uart_tx`) |
| UART RX ← AC | `GPIO3` | core.yaml (`uart_rx`) |
| I²C SDA (OLED) | `GPIO0` | screen.yaml |
| I²C SCL (OLED) | `GPIO2` | screen.yaml |
| RX LED | `GPIO6` | leds.yaml (`receive_led`) |
| TX LED | `GPIO4` | leds.yaml (`transmit_led`) |

All UART pins are configurable via the `uart_rx` / `uart_tx` substitutions — change them to match your board.
For example the Seeed XIAO ESP32-C6 uses **GPIO16 (RX) / GPIO17 (TX)**.

---

## 🚀 Quick Start

You have **two ready-made sample configs** — pick the one matching your connectivity.
Both pull everything else (UART, climate, switches, selects, OTA, API) from this repo via `packages:`.

| Config file | Connectivity | Board |
|-------------|-------------|-------|
| `TCL-Conditioner.yaml` | 📶 **WiFi** | ESP32-C3 (or any ESP32) |
| `TCL-Conditioner-thread.yaml` | 🧵 **Thread** | ESP32-C6 / H2 (needs 802.15.4) |

### Step 1 — Clone

```bash
git clone https://github.com/steuerlexi/tclac-esphome.git
cd tclac-esphome
```

### Step 2 — Create your `secrets.yaml`

ESPHome reads `secrets.yaml` from the **same folder as your config**. Copy the template and fill in your values:

```bash
cp secrets.yaml.example secrets.yaml   # then edit secrets.yaml
```

See [🔐 Secrets](#-secrets-secretsyaml) below for the required keys.

### Step 3 — Edit your config

Open `TCL-Conditioner.yaml` (WiFi) or `TCL-Conditioner-thread.yaml` (Thread) and set the substitutions:

```yaml
substitutions:
  device_name: tclac          # unique, lowercase, no spaces
  humanly_name: "TCL AC"      # shown in Home Assistant
  uart_rx: GPIO3              # adjust to your board
  uart_tx: GPIO1
```

Pick the packages you want (Thread config shown):

```yaml
packages:
  remote_package:
    url: https://github.com/steuerlexi/tclac-esphome.git
    ref: master
    files:
      - packages/core.yaml          # mandatory — UART + climate + API + OTA
      - packages/openthread.yaml    # Thread  (WiFi config: use packages/wifi.yaml)
      # - packages/leds.yaml        # optional TX/RX LEDs
      # - packages/screen.yaml      # optional OLED
    refresh: 30s
```

### Step 4 — Flash

```bash
pip install esphome                 # or use the ESPHome Dashboard
esphome run TCL-Conditioner.yaml    # or TCL-Conditioner-thread.yaml
```

---

## 🔐 Secrets (`secrets.yaml`)

`secrets.yaml` is **git-ignored** — it never leaves your machine.
The committed configs contain only `!secret` *key* references, so they're safe to share.

### WiFi build (`TCL-Conditioner.yaml`)

```yaml
# secrets.yaml
wifi_ssid: "YourWiFi"
wifi_password: "your-wifi-password"
api_key: "BASE64_ESPHOME_API_KEY=="     # generate one: https://esphome.io/components/api.html
ota_pass: "your-ota-password"
recovery_pass: "your-fallback-password"
```

### Thread build (`TCL-Conditioner-thread.yaml`)

```yaml
# secrets.yaml
thread_network_name: "TCL-Thread"
thread_channel: "15"
thread_pan_id: "0x1234"
thread_ext_pan_id: "dead00beef00cafe"
thread_network_key: "00112233445566778899aabbccddeeff"
thread_pskc: "00112233445566778899aabbccddeeff"
thread_mesh_local_prefix: "fdde:ad00:beef:0::/64"
api_key: "BASE64_ESPHOME_API_KEY=="
ota_pass: "your-ota-password"
recovery_pass: "your-fallback-password"
```

> 💡 Get your Thread credentials from your Thread Commissioner / Border Router (e.g. Apple HomePod, Home Assistant Connect ZBT-1). Generate an ESPHome API key at <https://esphome.io/components/api.html>.

---

## 💾 Flashing

### First flash (USB)

```bash
esphome run TCL-Conditioner.yaml        # pick the right port when prompted
```

### OTA updates (after first flash)

Once the device is on your network (WiFi or Thread), updates go over the air:

```bash
esphome run TCL-Conditioner.yaml --device tclac.local
# Thread devices: use the device's Thread/mDNS address
```

### ESPHome Dashboard

Prefer a GUI? Use the [ESPHome Dashboard](https://esphome.io/guides/getting-started.html) — just drop the config folder in and click **Install**.

---

## 🏠 Home Assistant Integration

After flashing, the device shows up automatically:

1. **Settings → Devices & Services** in Home Assistant → the `tclac` device is discovered via the native API.
2. Click **Configure** and enter your `api_key` (it must match `secrets.yaml`).
3. A new **Climate** entity appears — add it to a dashboard and you're done. 🎉

> Thread devices need a **Thread Border Router** (Home Assistant Connect ZBT-1, Apple HomePod, etc.) to be reachable from HA.

---

## 🎛️ Entities & Controls

### Climate entity (`climate.tclac`)

| Control | Options |
|---------|---------|
| ⏻ Power | On / Off |
| 🌡️ Target temp | 16 – 31 °C (1° step) |
| 🌀 Mode | OFF · AUTO · COOL · HEAT · DRY · FAN_ONLY |
| 💨 Fan speed | AUTO · QUIET · LOW · MIDDLE · MEDIUM · HIGH · FOCUS · DIFFUSE |
| ↔️ Swing | OFF · VERTICAL · HORIZONTAL · BOTH |
| 🌙 Preset | NONE · ECO · SLEEP · COMFORT |

### Config switches & selects

| Entity | Type | What it does |
|--------|------|--------------|
| 🔔 **Beeper** | switch | Beep on command confirmation |
| 🖥️ **Display** | switch | Show setpoint on the indoor unit |
| 💡 **Display on module** | switch | LED indication for AC data exchange |
| ⚡ **Force config** | switch | Apply Beeper/Display immediately (sorz2122 behavior gates these on Force) |
| ↕️ **Vertical swing** | select | Top↔Bottom · Upper half · Lower half |
| ↔️ **Horizontal swing** | select | Left↔Right · Left · Center · Right |
| 📌 **Vertical fixing** | select | Last · Max up · Upper · **Center** · Lower · Max down |
| 📌 **Horizontal fixing** | select | Last · Max left · Left · **Center** · Right · Max right |

> 📌 To **park the vertical flap straight ahead** when swing is off: set **Vertical fixing → In the center**.

---

## 🌡️ Supported Air Conditioners

| Manufacturer | Models (confirmed) |
|--------------|--------------------|
| **TCL** | TAC-07CHSA, TAC-09CHSA, TAC-12CHSA, TAC-12CHDA, TAC-12CHFA |
| **Daichi** | AIR20AVQ1, AIR25AVQS1R-1, DA35EVQ1-1 |
| **Axioma** | ASX09H1, ASB09H1 |
| **Dantex** | RK-12SATI, RK-12SATIE |
| **Electriq** | (various split units sharing the TCL protocol) |

> Not all units expose the UART service port — verify before buying hardware.

---

## 🧭 Architecture

```
  ┌──────────────────────────────────────┐
  │           Home Assistant             │
  │      (native ESPHome API, TLS)       │
  └─────────────────┬────────────────────┘
                    │  WiFi  or  Thread / 802.15.4
                    │
  ┌─────────────────▼────────────────────┐
  │          ESPHome device              │
  │   esp-idf  ·  ESP32-C3 / C6          │
  │  ┌──────────────────────────────┐    │
  │  │   tclac climate component    │    │
  │  │  (UART protocol · 9600 8E1)  │    │
  │  └──────────────────────────────┘    │
  └─────────────────┬────────────────────┘
                    │  UART 9600 baud, 8 data, Even parity, 1 stop
                    │
  ┌─────────────────▼────────────────────┐
  │      TCL / Electriq / Daichi AC      │
  │       (service / USB-A port)         │
  └──────────────────────────────────────┘
```

---

## 📦 Repository Structure

```
tclac-esphome/
├── components/tclac/          # 🔧 The ESPHome climate component (C++ + Python)
│   ├── tclac.cpp / tclac.h
│   ├── climate.py
│   └── __init__.py
├── packages/                  # 📦 Shared YAML, pulled by device configs
│   ├── core.yaml              #    mandatory: UART, climate, API, OTA, switches, selects
│   ├── wifi.yaml              #    optional: WiFi connectivity
│   ├── openthread.yaml        #    optional: Thread / 802.15.4 (esp-idf only)
│   ├── leds.yaml              #    optional: TX/RX status LEDs
│   └── screen.yaml            #    optional: SSD1306 OLED
├── devices/                   # 🖥️ Real-world device configs (slim, git-pulled)
│   └── tclac-klima.yaml       #    example: XIAO ESP32-C6, Thread, GPIO16/17
├── TCL-Conditioner.yaml       # 📶 Sample config — WiFi
├── TCL-Conditioner-thread.yaml# 🧵 Sample config — Thread
└── secrets.yaml.example       # 🔑 Template for your local secrets.yaml (git-ignored)
```

The **package-based approach** keeps your device file tiny — it holds only identity,
hardware pins, connectivity, and secret references. Everything else is downloaded from
this repo over git, so updates to the component or packages reach your device on the next
compile without you editing your config.

---

## 🛠️ Troubleshooting

### ❌ No connection to the AC

```
[TCL] Wrong byte
[TCL] Invalid checksum
```

- Check the **non-standard USB-A → ESP32 wiring** (AC **D− → ESP RX**, AC **VBUS → ESP TX**, AC **GND → 5V/VIN**, AC **D+ → GND**) — see [Wiring](#-wiring). This is *not* a normal USB pinout.
- Verify UART settings: **9600 baud, 8 data bits, Even parity, 1 stop bit**.
- Confirm your `uart_rx` / `uart_tx` pins match the board you actually wired.
- Check cable continuity on the USB-A service plug.

### 📶 WiFi won't connect

- Verify `wifi_ssid` / `wifi_password` in `secrets.yaml`.
- Move the device closer to the router; check `signal_strength` sensor.
- The fallback hotspot (`{device_name} Fallback Hotspot`) appears if WiFi fails — connect with `recovery_pass` to reconfigure.

### 🧵 Thread device not reachable

- You need a **Thread Border Router** on the same network as Home Assistant.
- Confirm the Thread network credentials in `secrets.yaml` match your commissioned network.
- ESP-IDF framework + an 802.15.4-capable board (C6/H2) are **required** for Thread.

### 🔄 "My fix didn't take effect after a reflash"

ESPHome caches git-pulled packages and components under `.esphome/` and does not always
re-fetch on `refresh:`. After pulling an update, force a clean fetch:

```bash
rm -rf .esphome/packages .esphome/external_components
esphome compile your-config.yaml
```

### 📋 Enable debug logging

```yaml
logger:
  level: DEBUG
  # baud_rate: 0   # keep 0 — logging shares the UART with the AC
```

---

## 🙋 Credits

This fork builds on the work of several people in the ESPHome / TCL-AC community:

- **Miguel Angel Lopez** — original ESPHome component
- **xaxexa** — protocol modifications
- **Nightingale with a soldering iron** (*Соловей с паяльником*) — refactoring & component
- **sorz2122** — ESPHome ESP-IDF edition & the authoritative flap-encoding implementation

ESPHome ESP-IDF + package-architecture edition maintained in this repo.

☕ If this project is useful to you, consider supporting the original author:
<https://buymeacoffee.com/sorz2122>

---

## 📄 License

Licensed under the **MIT License** — see the component headers for details.
Feel free to fork, adapt, and use in your own smart home.

---

<div align="center">

**Made with ❄️ for the Smart Home community**

Contributions welcome — fork, branch, PR. See [Repository Structure](#-repository-structure) to get started.

</div>