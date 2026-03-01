# LILYGO T-Embed CC1101 Plus — The Ultimate Guide

> Every feature. Every use case. Every hack. The definitive reference for the most versatile sub-$30 hacking tool on the market.

---

## What is this?

A comprehensive, interactive guide covering **every capability** of the [LILYGO T-Embed CC1101 Plus](https://www.lilygo.cc/) — a compact, open-source multi-tool built around the ESP32-S3 and Texas Instruments CC1101 sub-GHz transceiver.

This guide is designed for **ethical hackers, security researchers, RF enthusiasts, and makers** who want to understand and fully utilize every feature of this remarkable device.

---

## Hardware at a Glance

| Component | Specification |
|:----------|:-------------|
| **MCU** | ESP32-S3 — Dual-core Xtensa LX7 @ 240 MHz, 8MB PSRAM, 16MB Flash |
| **Sub-GHz** | CC1101 — 300-348 / 387-464 / 779-928 MHz, TX+RX, OOK/ASK/FSK/GFSK/MSK |
| **WiFi** | 802.11 b/g/n 2.4 GHz, STA + AP mode, promiscuous mode |
| **Bluetooth** | BLE 5.0, scanning, GATT, advertisements |
| **IR** | TX (940nm LED) + RX (38kHz receiver) |
| **Display** | 1.9" ST7789 TFT, 170x320, 65K colors |
| **Input** | Rotary encoder with push button |
| **Audio** | Built-in speaker/buzzer |
| **USB** | USB-C, native USB OTG (HID/CDC/MSC) |
| **Power** | USB-C 5V + JST battery connector (3.7V LiPo) |
| **Price** | ~$25 USD |

---

## Guide Contents

### Fundamentals
| Chapter | Description |
|:--------|:-----------|
| [01 — Hardware Deep Dive](docs/01-hardware-overview.md) | Every chip, pin, and component on the board |
| [02 — Getting Started](docs/02-getting-started.md) | Unboxing to first hack in 15 minutes |
| [03 — Firmware Guide](docs/03-firmware.md) | Bruce, ESP-Flipper, custom firmware, flashing |

### Capabilities
| Chapter | Description |
|:--------|:-----------|
| [04 — Sub-GHz RF Operations](docs/04-sub-ghz.md) | Capture, replay, analyze, transmit, decode protocols |
| [05 — WiFi Operations](docs/05-wifi.md) | Scanning, monitoring, deauth, evil portal, recon |
| [06 — Bluetooth / BLE](docs/06-bluetooth.md) | Scanning, GATT, advertisements, tracker detection |
| [07 — Infrared](docs/07-ir.md) | Capture, replay, protocols, universal remote, TV-B-Gone |
| [08 — BadUSB / HID](docs/08-badusb.md) | Keyboard emulation, Ducky Script, payloads |

### Advanced
| Chapter | Description |
|:--------|:-----------|
| [09 — GPIO & Hardware Hacking](docs/09-gpio-hardware.md) | Pinout, SPI, I2C, UART, expansion modules |
| [10 — Custom Firmware Development](docs/10-custom-development.md) | Arduino, PlatformIO, ESP-IDF, full code examples |
| [11 — Practical Projects](docs/11-projects.md) | 25 real-world builds from beginner to expert |

### Reference
| Chapter | Description |
|:--------|:-----------|
| [12 — Troubleshooting & FAQ](docs/12-troubleshooting.md) | Every common problem and its fix, 30+ FAQ |
| [13 — Legal Reference](docs/13-legal-reference.md) | Frequency regulations, CFAA, authorized testing |

---

## Who Is This For?

- **Ethical Hacking Students** — Preparing for CEH, OSCP, or CyberOps certifications
- **Penetration Testers** — Adding RF and physical assessment capabilities
- **RF Enthusiasts** — Learning sub-GHz protocols, signal analysis, and SDR
- **Makers & Tinkerers** — Building custom IoT, remote controls, and automation
- **Security Researchers** — Analyzing wireless protocols and device vulnerabilities
- **Home Lab Builders** — Testing and auditing your own devices and networks

---

## Recommended Firmware

**[Bruce Firmware](https://github.com/pr3y/Bruce)** — Open-source multi-tool firmware with Sub-GHz, WiFi, BLE, IR, BadUSB, and NRF24 capabilities. Install via [web flasher](https://bruce.computer/flasher) in under 2 minutes.

---

## How to Use This Guide

1. **Start with Chapter 02** — Get your device set up and running
2. **Follow the chapters in order** or jump to the capability you're most interested in
3. **Complete the practical projects** — Hands-on experience is essential
4. **Build custom firmware** (Chapter 10) once you're comfortable with the basics
5. **Always check Chapter 13** before testing — know your legal boundaries

---

## Legal Disclaimer

All techniques in this guide are for **authorized security testing and educational purposes only**.

- Only test devices and networks you **own** or have **written permission** to test
- Transmitting on radio frequencies is regulated — comply with your country's regulations
- Unauthorized access to computer systems and networks is **illegal**
- The author is not responsible for misuse of information in this guide

See [Chapter 13 — Legal Reference](docs/13-legal-reference.md) for comprehensive legal guidance.

---

## Contributing

Found an error or want to add something? Contributions are welcome:

1. Fork this repository
2. Create a feature branch (`git checkout -b improve/chapter-name`)
3. Commit your changes (`git commit -m 'Add missing protocol to Sub-GHz chapter'`)
4. Push to the branch (`git push origin improve/chapter-name`)
5. Open a Pull Request

---

## Resources

- [LILYGO T-Embed CC1101 Product Page](https://www.lilygo.cc/)
- [Bruce Firmware GitHub](https://github.com/pr3y/Bruce)
- [CC1101 Datasheet (TI)](https://www.ti.com/product/CC1101)
- [ESP32-S3 Datasheet (Espressif)](https://www.espressif.com/en/products/socs/esp32-s3)
- [ELECHOUSE CC1101 Library](https://github.com/LSatan/SmartRC-CC1101-Driver-Lib)
- [Universal Radio Hacker (URH)](https://github.com/jopohl/urh)
- [FCC Part 15 Rules](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-15)

---

## License

This guide is provided for educational purposes. Content is original and free to use for personal learning. Attribution appreciated.

---

*Built with care for the security research community.*
