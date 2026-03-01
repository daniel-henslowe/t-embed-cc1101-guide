---
layout: default
title: Home
nav_order: 1
description: "The ultimate guide to the LILYGO T-Embed CC1101 Plus — every feature, every use case, every hack."
permalink: /
---

# LILYGO T-Embed CC1101 Plus — The Ultimate Guide
{: .fs-9 }

Every feature. Every use case. Every hack. The definitive reference for the most versatile sub-$30 hacking tool on the market.
{: .fs-6 .fw-300 }

[Get Started](/t-embed-cc1101-guide/docs/02-getting-started/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/daniel-henslowe/t-embed-cc1101-guide){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is the T-Embed CC1101 Plus?

The LILYGO T-Embed CC1101 Plus is a compact, open-source multi-tool built around the **ESP32-S3** microcontroller and the **TI CC1101** sub-GHz radio transceiver. For under $30, you get capabilities that rival devices costing 5-10x more.

```
┌─────────────────────────────────────────────────────┐
│              LILYGO T-Embed CC1101 Plus             │
│                                                     │
│  ┌─────────────┐  ┌──────────────────────────────┐  │
│  │  ESP32-S3   │  │  CC1101 Sub-GHz Transceiver  │  │
│  │             │  │                              │  │
│  │ • WiFi 2.4G │  │ • 300-348 MHz   TX/RX        │  │
│  │ • BLE 5.0   │  │ • 387-464 MHz   TX/RX        │  │
│  │ • USB OTG   │  │ • 779-928 MHz   TX/RX        │  │
│  │ • 240 MHz   │  │ • OOK/ASK/FSK/GFSK/MSK      │  │
│  │ • 8MB PSRAM │  │ • +12 dBm output             │  │
│  └─────────────┘  └──────────────────────────────┘  │
│                                                     │
│  ┌──────────┐  ┌──────┐  ┌────┐  ┌──────┐  ┌────┐  │
│  │ 1.9" TFT │  │ IR   │  │ 🔊 │  │ GPIO │  │USB-C│ │
│  │ 170×320  │  │TX/RX │  │    │  │      │  │    │  │
│  └──────────┘  └──────┘  └────┘  └──────┘  └────┘  │
│       ┌──────────────┐                              │
│       │ Rotary Encoder│                             │
│       │  + Push Button │                            │
│       └──────────────┘                              │
└─────────────────────────────────────────────────────┘
```

---

## What Can It Do?

| Capability | Radio/Interface | Use Cases |
|:-----------|:---------------|:----------|
| **Sub-GHz RF** | CC1101 (300-928 MHz) | Signal capture, replay, analysis, custom protocols, spectrum scanning |
| **WiFi** | ESP32-S3 (2.4 GHz) | Network scanning, packet monitoring, beacon generation, evil portal |
| **Bluetooth** | ESP32-S3 (BLE 5.0) | Device scanning, BLE enumeration, advertisement spam |
| **Infrared** | IR LED + Receiver | Remote capture/replay, universal remote, TV-B-Gone |
| **BadUSB** | USB-C OTG | HID keyboard emulation, automated payloads |
| **GPIO** | Expansion header | Hardware hacking, SPI/I2C/UART, external sensors |
| **Display** | 1.9" ST7789 TFT | Real-time signal visualization, menus, status |
| **Audio** | Built-in speaker | RTTTL tones, alerts, frequency audio output |

---

## Guide Contents

| Chapter | Topic | What You'll Learn |
|:--------|:------|:------------------|
| 01 | [Hardware Deep Dive](docs/01-hardware-overview/) | Every chip, pin, and component on the board |
| 02 | [Getting Started](docs/02-getting-started/) | Unboxing to first hack in 15 minutes |
| 03 | [Firmware Guide](docs/03-firmware/) | Bruce, ESP-Flipper, custom firmware, flashing |
| 04 | [Sub-GHz RF Operations](docs/04-sub-ghz/) | Capture, replay, analyze, transmit, protocols |
| 05 | [WiFi Operations](docs/05-wifi/) | Scanning, monitoring, deauth, evil portal |
| 06 | [Bluetooth / BLE](docs/06-bluetooth/) | Scanning, GATT, advertisements, tracking |
| 07 | [Infrared](docs/07-ir/) | Capture, replay, protocols, universal remote |
| 08 | [BadUSB / HID](docs/08-badusb/) | Keyboard emulation, payloads, Ducky Script |
| 09 | [GPIO & Hardware Hacking](docs/09-gpio-hardware/) | Pinout, SPI, I2C, UART, external modules |
| 10 | [Custom Firmware Development](docs/10-custom-development/) | Arduino, PlatformIO, libraries, full examples |
| 11 | [Practical Projects](docs/11-projects/) | 25 real-world builds from beginner to expert |
| 12 | [Troubleshooting & FAQ](docs/12-troubleshooting/) | Every common problem and its fix |
| 13 | [Legal Reference](docs/13-legal-reference/) | Frequency regulations, authorized use, compliance |

---

## Quick Comparison

| Feature | T-Embed CC1101 | Flipper Zero | HackRF One | RTL-SDR v4 |
|:--------|:--------------|:-------------|:-----------|:-----------|
| **Price** | ~$25 | ~$170 | ~$350 | ~$30 |
| **Sub-GHz TX** | 300-928 MHz | 300-928 MHz | 1 MHz-6 GHz | No TX |
| **Sub-GHz RX** | 300-928 MHz | 300-928 MHz | 1 MHz-6 GHz | 24-1766 MHz |
| **WiFi** | 802.11 b/g/n | No | No | No |
| **Bluetooth** | BLE 5.0 | BLE | No | No |
| **IR** | TX + RX | TX + RX | No | No |
| **BadUSB** | Yes (USB OTG) | Yes | No | No |
| **Display** | 1.9" Color TFT | 1.4" Mono LCD | None | None |
| **GPIO** | Yes (ESP32-S3) | Yes (limited) | No | No |
| **Open Source** | Fully | Partially | Fully | Fully |
| **Custom Code** | Arduino/ESP-IDF | Limited | GNU Radio | GNU Radio |
| **Battery** | External (JST) | Built-in | N/A | N/A |
| **Form Factor** | Compact stick | Tamagotchi | Board + case | Dongle |

{: .tip }
> The T-Embed CC1101 Plus gives you **90% of Flipper Zero's capabilities** at **15% of the price**, with the added bonus of WiFi, a color display, and full Arduino/ESP-IDF programmability. It's the best value in portable hacking hardware.

---

{: .warning }
> **Legal Disclaimer**: All techniques in this guide are for **authorized security testing and educational purposes only**. Transmitting on radio frequencies you are not authorized to use is illegal. Always comply with your country's radio regulations (FCC Part 15 in the US, ETSI EN 300 220 in the EU). Only test devices and networks you own or have written permission to test.
