# BLE EnvMon / BLE Proxy

[![ESPHome](https://img.shields.io/badge/ESPHome-2025.8.0+-blue)](https://esphome.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-ESP32--C6-orange)](https://www.espressif.com/en/products/socs/esp32-c6)

🇩🇪 [Deutsch](#deutsch) | 🇬🇧 [English](#english)

---

<a name="deutsch"></a>
# 🇩🇪 Deutsch

ESPHome-basiertes Firmware-Projekt für einen **Bluetooth Proxy** auf Basis des ESP32-C6 – wahlweise mit oder ohne **BME688 Umweltsensor**. Beide Varianten teilen eine gemeinsame Paketbasis und werden über separate Gerätedateien konfiguriert.

## Übersicht

| Variante | Datei | Funktion |
|---|---|---|
| BLE Proxy | `ble-proxy.yaml` | Bluetooth Proxy für Home Assistant |
| BLE EnvMon | `ble-envmon.yaml` | Bluetooth Proxy + Umweltsensor (BME688) |

## Features

### Beide Varianten
- **Bluetooth Proxy** (aktiv) für Home Assistant BLE-Integration
- **2× NeoPixel WS2812** Statusanzeige
  - LED 1: WiFi-Status (amber = getrennt / grün = verbunden)
- **Web UI** (ESPHome Web Server v3, Port 80)
- **Captive Portal** für einfache WLAN-Ersteinrichtung
- OTA-Update, Safe Mode Boot, Factory Reset per Knopfdruck in der UI

### BLE EnvMon (zusätzlich)
- **BME688** über BSEC2-Bibliothek (I²C)
  - Temperatur, Luftdruck, relative Feuchte
  - IAQ (Indoor Air Quality, 0–500)
  - CO₂-Äquivalent, Atemluft-VOC-Äquivalent
  - IAQ Accuracy Status (0–3, mit Klartextbeschreibung)
- **LED 0**: IAQ-Farbindikator (grün → gelb → orange → rot → lila)
- **Temperaturoffset** über Web UI / Home Assistant einstellbar (mit Flash-Persistenz)

## IAQ Farbskala

| IAQ | Farbe | Bewertung |
|---|---|---|
| 0–50 | 🟢 Grün | Excellent |
| 51–100 | 🟡 Gelbgrün | Good |
| 101–150 | 🟡 Gelb | Lightly polluted |
| 151–200 | 🟠 Orange | Moderately polluted |
| 201–250 | 🔴 Rotorange | Heavily polluted |
| 251–350 | 🔴 Rot | Severely polluted |
| 351–500 | 🟣 Lila | Extremely polluted |

## Hardware

| Komponente | Beschreibung |
|---|---|
| MCU | ESP32-C6 |
| Sensor | Bosch BME688 (nur EnvMon-Variante) |
| LED | 2× WS2812 NeoPixel (GPIO1) |
| I²C | SDA: GPIO20, SCL: GPIO19 |
| Sensor-Adresse | 0x77 |

## Dateistruktur

```
├── ble-proxy.yaml          # Gerätedatei: BLE Proxy (ohne Sensor)
├── ble-envmon.yaml         # Gerätedatei: BLE Proxy + BME688
└── packages/
    ├── base.yaml           # Gemeinsame Basis (ESP32, BLE, LEDs, Buttons)
    ├── wifi.yaml           # WiFi + NeoPixel-Statusanzeige
    └── bme688.yaml         # BME688 Umweltsensor komplett
```

## Installation

### Voraussetzungen
- [ESPHome](https://esphome.io/guides/getting_started_command_line.html) ≥ 2025.8.0
- Home Assistant mit ESPHome Add-on (empfohlen) oder ESPHome CLI

### Ersteinrichtung

1. Repository klonen:
   ```bash
   git clone https://github.com/<dein-username>/esphome-BLE_proxy_envmon.git
   cd ble-envmon
   ```

2. Firmware flashen (USB):
   ```bash
   # BLE Proxy (ohne Sensor)
   esphome run ble-proxy.yaml

   # BLE EnvMon (mit BME688)
   esphome run ble-envmon.yaml
   ```

3. Nach dem ersten Boot erscheint ein WLAN-Hotspot `ESPHome-BLEProxy` – darüber die WLAN-Zugangsdaten eingeben (Captive Portal).

## Kalibrierung (BME688)

### Temperaturoffset

Der BME688 sitzt auf einer Platine mit eigener Eigenerwärmung. Der Temperaturoffset korrigiert diesen Fehler und verbessert gleichzeitig die Genauigkeit der relativen Feuchte, da diese temperaturabhängig berechnet wird.

Einstellbar im Web UI unter **Controls → BME68x Temperature Offset** oder direkt in Home Assistant. Der Wert wird im Flash gespeichert und überlebt Reboots.

### Luftfeuchte

Eine direkte Kalibrierung per BSEC2-Parameter existiert nicht. Empfohlene Vorgehensweise:

- **Referenzsensor** (z.B. SHT30/SHT40): Beide Sensoren eine Stunde lang nebeneinander betreiben, Abweichung ermitteln.
- **Kochsalz-Methode**: Gesättigte NaCl-Lösung erzeugt bei ~25 °C definiert 75 % rH. Sensor im geschlossenen Behälter kalibrieren.

### BSEC2 Accuracy

Der BSEC2-Algorithmus kalibriert sich selbst über Zeit:

| Stufe | Bedeutung |
|---|---|
| 0 | Stabilisierung / Run-in läuft |
| 1 | Niedrig – Sensor einmal frischer und verbrauchter Luft aussetzen |
| 2 | Mittel – Auto-Trimming läuft |
| 3 | Hoch |

Stufe 3 wird nach ca. 24–48 h Betrieb mit ausreichend Luftvarianz erreicht. Der Kalibrierzustand wird von BSEC2 intern im Flash persistiert (`operating_age: 28d`).

## Home Assistant Integration

Nach dem ersten Start erscheint das Gerät automatisch unter **Einstellungen → Integrationen → ESPHome**.

Für den BLE Proxy: In Home Assistant unter **Einstellungen → Integrationen → Bluetooth** das Gerät als Bluetooth-Proxy aktivieren.

## Lizenz

MIT License – siehe [LICENSE](LICENSE)

---

<a name="english"></a>
# 🇬🇧 English

ESPHome-based firmware project for a **Bluetooth Proxy** built around the ESP32-C6 – optionally with or without a **BME688 environmental sensor**. Both variants share a common package base and are configured via separate device files.

## Overview

| Variant | File | Function |
|---|---|---|
| BLE Proxy | `ble-proxy.yaml` | Bluetooth Proxy for Home Assistant |
| BLE EnvMon | `ble-envmon.yaml` | Bluetooth Proxy + Environmental Sensor (BME688) |

## Features

### Both variants
- **Bluetooth Proxy** (active) for Home Assistant BLE integration
- **2× NeoPixel WS2812** status indicator
  - LED 1: WiFi status (amber = disconnected / green = connected)
- **Web UI** (ESPHome Web Server v3, port 80)
- **Captive Portal** for easy first-time WiFi setup
- OTA update, Safe Mode boot, Factory Reset via UI buttons

### BLE EnvMon (additional)
- **BME688** via BSEC2 library (I²C)
  - Temperature, air pressure, relative humidity
  - IAQ (Indoor Air Quality, 0–500)
  - CO₂ equivalent, breath VOC equivalent
  - IAQ Accuracy Status (0–3, with human-readable description)
- **LED 0**: IAQ color indicator (green → yellow → orange → red → purple)
- **Temperature offset** adjustable via Web UI / Home Assistant (flash-persistent)

## IAQ Color Scale

| IAQ | Color | Rating |
|---|---|---|
| 0–50 | 🟢 Green | Excellent |
| 51–100 | 🟡 Yellow-green | Good |
| 101–150 | 🟡 Yellow | Lightly polluted |
| 151–200 | 🟠 Orange | Moderately polluted |
| 201–250 | 🔴 Red-orange | Heavily polluted |
| 251–350 | 🔴 Red | Severely polluted |
| 351–500 | 🟣 Purple | Extremely polluted |

## Hardware

| Component | Description |
|---|---|
| MCU | ESP32-C6 |
| Sensor | Bosch BME688 (EnvMon variant only) |
| LED | 2× WS2812 NeoPixel (GPIO1) |
| I²C | SDA: GPIO20, SCL: GPIO19 |
| Sensor address | 0x77 |

## File Structure

```
├── ble-proxy.yaml          # Device file: BLE Proxy (no sensor)
├── ble-envmon.yaml         # Device file: BLE Proxy + BME688
└── packages/
    ├── base.yaml           # Shared base (ESP32, BLE, LEDs, buttons)
    ├── wifi.yaml           # WiFi + NeoPixel status feedback
    └── bme688.yaml         # BME688 environmental sensor (complete)
```

## Installation

### Requirements
- [ESPHome](https://esphome.io/guides/getting_started_command_line.html) ≥ 2025.8.0
- Home Assistant with ESPHome add-on (recommended) or ESPHome CLI

### First-time setup

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/esphome-BLE_proxy_envmon.git
   cd ble-envmon
   ```

2. Flash the firmware (USB):
   ```bash
   # BLE Proxy (no sensor)
   esphome run ble-proxy.yaml

   # BLE EnvMon (with BME688)
   esphome run ble-envmon.yaml
   ```

3. After the first boot a WiFi hotspot `ESPHome-BLEProxy` appears – enter your WiFi credentials via the Captive Portal.

## Calibration (BME688)

### Temperature offset

The BME688 sits on a PCB that generates some self-heating. The temperature offset corrects this error and simultaneously improves the accuracy of the relative humidity reading, since humidity is calculated as a function of temperature.

Adjustable in the Web UI under **Controls → BME68x Temperature Offset** or directly in Home Assistant. The value is stored in flash and survives reboots.

### Humidity

There is no direct BSEC2 calibration parameter for humidity. Recommended approach:

- **Reference sensor** (e.g. SHT30/SHT40): Run both sensors side by side for one hour, determine the offset.
- **Salt solution method**: A saturated NaCl solution produces a defined 75 % RH at ~25 °C. Place the sensor in a sealed container for calibration.

### BSEC2 Accuracy

The BSEC2 algorithm self-calibrates over time:

| Level | Meaning |
|---|---|
| 0 | Stabilizing / run-in in progress |
| 1 | Low – expose sensor to fresh air and polluted air once for auto-trimming |
| 2 | Medium – auto-trimming in progress |
| 3 | High accuracy |

Level 3 is typically reached after 24–48 h of operation with sufficient air variation. The calibration state is persisted internally by BSEC2 in flash (`operating_age: 28d`).

## Home Assistant Integration

After the first boot the device will appear automatically under **Settings → Integrations → ESPHome**.

For the BLE Proxy: enable it as a Bluetooth proxy in Home Assistant under **Settings → Integrations → Bluetooth**.

## License

MIT License – see [LICENSE](LICENSE)
