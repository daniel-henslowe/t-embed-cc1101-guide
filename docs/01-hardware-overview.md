---
layout: default
title: "01 — Hardware Deep Dive"
nav_order: 2
---

# 01 — Hardware Deep Dive
{: .fs-9 }

Every chip, every pin, every component. The most complete hardware reference for the LILYGO T-Embed CC1101 Plus available anywhere.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Board Overview

### Physical Specifications

| Attribute | Value |
|:----------|:------|
| **Dimensions (PCB)** | ~103 × 37 × 13 mm (L × W × H) |
| **Dimensions (with antenna)** | ~103 × 37 × 13 mm + antenna length |
| **Weight** | ~30 g (board only, without antenna or battery) |
| **PCB Layers** | 4-layer FR4 |
| **Form Factor** | USB stick / compact wand |
| **Main Color** | Black PCB, black shell (some revisions: white silkscreen) |
| **Connectors** | USB-C, JST PH 2.0 (battery), SMA female (antenna) |
| **Operating Temperature** | -10 °C to 60 °C (recommended) |
| **Storage Temperature** | -40 °C to 85 °C |

### What's in the Box

Depending on your vendor and variant, the typical package includes:

- 1x T-Embed CC1101 Plus main board
- 1x 433 MHz antenna (SMA male, whip style) — some bundles include 315/868/915 MHz antennas
- 1x USB-C cable (charging and data)
- Pin header (if expansion header is unpopulated)
- Instruction card / QR code to GitHub repository

{: .note }
> Third-party sellers on AliExpress may include different antenna bundles. Always verify the antenna frequency matches your target band. An antenna tuned for 433 MHz will perform poorly at 915 MHz and vice versa.

### Available Variants and Revisions

| Variant | Key Differences |
|:--------|:---------------|
| **T-Embed CC1101 (original)** | ESP32-S3, CC1101, 1.9" TFT, rotary encoder, no IR, no speaker on early revisions |
| **T-Embed CC1101 Plus** | Added IR TX/RX, speaker/buzzer, improved antenna connector, updated GPIO layout |
| **T-Embed CC1101 Plus (V1.1+)** | Minor PCB routing improvements, better power regulation, silkscreen updates |

{: .warning }
> Pin assignments can vary between board revisions. Always verify your specific board revision against the LILYGO GitHub repository before wiring external hardware. The pin definitions in the official `pin_config.h` file are the canonical source.

---

## ESP32-S3 Microcontroller

The heart of the T-Embed CC1101 Plus is the **Espressif ESP32-S3-WROOM-1-N16R8** module — a highly integrated SoC with WiFi, Bluetooth, and powerful processing capabilities.

### Core Specifications

| Parameter | Value |
|:----------|:------|
| **CPU** | Dual-core Xtensa LX7 |
| **Clock Speed** | Up to 240 MHz |
| **SRAM** | 512 KB |
| **ROM** | 384 KB |
| **PSRAM** | 8 MB OPI (Octal SPI, high bandwidth) |
| **Flash** | 16 MB QIO (Quad SPI) |
| **WiFi** | 802.11 b/g/n, 2.4 GHz, HT20/HT40, up to 150 Mbps |
| **Bluetooth** | Bluetooth 5.0, BLE, Long Range (Coded PHY), 2 Mbps PHY |
| **USB** | USB OTG 1.1 Full Speed (12 Mbps), native — no FTDI/CH340 |
| **GPIO** | 45 programmable GPIO pins (many used by onboard peripherals) |
| **ADC** | 2x SAR ADC, 12-bit, up to 20 channels |
| **DAC** | None (removed from S3 compared to ESP32) |
| **SPI** | 4x SPI controllers (SPI0/1 for flash/PSRAM, SPI2/SPI3 general purpose) |
| **I2C** | 2x I2C controllers |
| **I2S** | 2x I2S controllers |
| **UART** | 3x UART controllers |
| **PWM** | LED PWM controller: 8 channels, Motor PWM: 2 modules |
| **Touch** | 14 capacitive touch GPIOs |
| **RTC GPIO** | 22 RTC-capable GPIOs (for deep sleep wakeup) |
| **Temperature Sensor** | Built-in, -20 °C to 110 °C range |
| **Security** | AES-128/256, SHA, RSA-3072, ECC (ECDSA), HMAC, Digital Signature |
| **RNG** | True hardware random number generator |
| **JTAG** | Built-in USB-JTAG for debugging |

### Power Modes

| Mode | Description | Typical Current | Wake Sources |
|:-----|:------------|:---------------|:-------------|
| **Active** | Both cores running, WiFi/BT active | 80-240 mA | N/A (running) |
| **Modem Sleep** | CPU active, WiFi/BT radios powered down | 20-30 mA | Automatic on WiFi DTIM |
| **Light Sleep** | CPU paused, RAM retained, peripherals idle | 0.3-2 mA | Timer, GPIO, touch, UART, WiFi |
| **Deep Sleep** | RTC core only, ULP coprocessor available | 7-10 μA | Timer, RTC GPIO, touch, ULP |
| **Hibernation** | RTC timer only, no RAM retention | ~5 μA | RTC timer only |

{: .tip }
> In Deep Sleep, the ESP32-S3 draws as little as **7 μA**. For battery-powered portable use, implement deep sleep between user interactions. The rotary encoder push button can be connected to an RTC GPIO for wake-on-press functionality.

### Clock Sources

| Source | Frequency | Use |
|:-------|:----------|:----|
| **Main Crystal (XTAL)** | 40 MHz | CPU clock, PLL source |
| **RTC Crystal** | 32.768 kHz | RTC timer (deep sleep wakeup) |
| **Internal RC (8 MHz)** | ~8 MHz | Fallback, calibration |
| **Internal RC (150 kHz)** | ~150 kHz | RTC backup clock |
| **PLL** | Configurable | Derived from 40 MHz XTAL |

### Memory Map Overview

```
┌─────────────────────────────────┐  0x4200_0000
│     External Flash (16 MB)      │  Instruction bus (cached, read-only)
│     Firmware, SPIFFS/LittleFS   │
├─────────────────────────────────┤  0x3C00_0000
│     External PSRAM (8 MB OPI)   │  Data bus (cached, read/write)
│     Large buffers, heap         │
├─────────────────────────────────┤  0x3FC8_8000
│     Internal SRAM (512 KB)      │  Instruction + Data bus
│     Stack, heap, .bss, .data    │
├─────────────────────────────────┤  0x4000_0000
│     Internal ROM (384 KB)       │  Bootloader, crypto, API tables
├─────────────────────────────────┤  0x5000_0000
│     RTC FAST Memory (8 KB)      │  ULP coprocessor, deep sleep stub
├─────────────────────────────────┤  0x5000_2000
│     RTC SLOW Memory (8 KB)      │  ULP program storage
└─────────────────────────────────┘
```

{: .note }
> The 8 MB OPI PSRAM is critical for RF operations. Captured signal buffers, WiFi packet dumps, and display framebuffers can easily consume megabytes. The OPI (Octal SPI) interface provides significantly higher bandwidth than QPI PSRAM variants used on cheaper ESP32-S3 boards.

---

## CC1101 Sub-GHz Transceiver

The **Texas Instruments CC1101** is a fully integrated sub-1 GHz ISM-band transceiver designed for very low power wireless applications. It is the primary RF component of the T-Embed CC1101 Plus and the chip that gives this board its name.

### Architecture

```
┌───────────────────────────────────────────────────────────┐
│                     CC1101 Block Diagram                   │
│                                                           │
│  ┌─────────┐   ┌──────────┐   ┌─────────────────────┐    │
│  │   LNA   │──▶│  Mixer   │──▶│  IF Amp + Filter    │    │
│  │         │   │(Low-IF)  │   │  (IF ≈ 152 kHz)     │    │
│  └────┬────┘   └──────────┘   └──────────┬──────────┘    │
│       │                                   │               │
│  ┌────┴────┐                    ┌─────────▼──────────┐    │
│  │ Antenna │                    │ ADC + Demodulator   │    │
│  │  Match  │                    │ (2-FSK/GFSK/       │    │
│  │ Network │                    │  4-FSK/MSK/OOK/ASK)│    │
│  └────┬────┘                    └─────────┬──────────┘    │
│       │                                   │               │
│  ┌────┴────┐                    ┌─────────▼──────────┐    │
│  │   PA    │◀──────────────────│   Packet Handler    │    │
│  │(+12 dBm)│                    │ Preamble|Sync|CRC  │    │
│  └─────────┘                    │ Address|Length      │    │
│                                 └─────────┬──────────┘    │
│  ┌─────────────────────────┐              │               │
│  │  Frequency Synthesizer  │    ┌─────────▼──────────┐    │
│  │  Σ-Δ Fractional-N PLL  │    │ 64B TX / 64B RX    │    │
│  │  26 MHz Crystal Ref     │    │      FIFOs         │    │
│  └─────────────────────────┘    └─────────┬──────────┘    │
│                                           │               │
│                                 ┌─────────▼──────────┐    │
│                                 │    SPI Interface    │    │
│                                 │  (to ESP32-S3)      │    │
│                                 └────────────────────┘    │
└───────────────────────────────────────────────────────────┘
```

- **Receiver Type**: Low-IF (Intermediate Frequency ~152 kHz)
- **Synthesizer**: Fully integrated Sigma-Delta fractional-N PLL
- **Reference Crystal**: 26 MHz
- **Digital Interface**: 4-wire SPI + 2 configurable GDO pins

### Frequency Bands

The CC1101 supports three frequency bands, each covering key ISM (Industrial, Scientific, Medical) allocations worldwide:

| Band | Frequency Range | Common ISM Frequencies | Regions |
|:-----|:---------------|:----------------------|:--------|
| **Band 1** | 300 — 348 MHz | 315 MHz | Americas, Asia |
| **Band 2** | 387 — 464 MHz | 433.92 MHz | Europe, Asia, Africa, Americas |
| **Band 3** | 779 — 928 MHz | 868 MHz, 915 MHz | Europe (868), Americas (915), Japan (920) |

{: .warning }
> The CC1101 has a **gap** between 348 MHz and 387 MHz, and another between 464 MHz and 779 MHz. These frequencies are physically impossible for the CC1101 to access. Claims of "300-928 MHz continuous" coverage are incorrect. The three bands listed above are the actual ranges.

### Modulation Support

| Modulation | Description | Typical Use Cases |
|:-----------|:------------|:-----------------|
| **2-FSK** | Binary Frequency Shift Keying | General-purpose data links |
| **4-FSK** | 4-level FSK (2 bits per symbol) | Higher throughput links |
| **GFSK** | Gaussian FSK (filtered, reduced bandwidth) | Bluetooth-adjacent, low interference |
| **MSK** | Minimum Shift Keying (constant envelope) | Spectral efficiency |
| **OOK** | On-Off Keying | Garage doors, weather stations, legacy remotes |
| **ASK** | Amplitude Shift Keying | Simple remotes, sensors, key fobs |

{: .tip }
> **OOK/ASK** is by far the most common modulation for consumer sub-GHz devices (garage doors, car key fobs, weather stations, doorbells, simple remotes). Master OOK first — it covers the majority of real-world targets.

### RF Performance

| Parameter | Value |
|:----------|:------|
| **Maximum TX Power** | +12 dBm (approximately 16 mW) |
| **Programmable TX Power** | -30 dBm to +12 dBm (stepped) |
| **RX Sensitivity** | -116 dBm @ 0.6 kbps, 2-FSK |
| | -104 dBm @ 38.4 kbps, 2-FSK |
| | -95 dBm @ 250 kbps, 2-FSK |
| | -104 dBm @ 1.2 kbps, OOK |
| **Data Rate** | 1.2 kbps — 500 kbps (FSK/GFSK/MSK) |
| | 1.2 kbps — 600 kbps (OOK/ASK, asynchronous serial) |
| **Channel Spacing** | Programmable (25 kHz, 50 kHz, 100 kHz, 200 kHz typical) |
| **Channel Filter BW** | 58 kHz — 812 kHz (programmable, 16 steps) |
| **Frequency Resolution** | ~397 Hz (26 MHz / 2^16) |
| **Settling Time** | 88.4 μs from IDLE to TX/RX |
| **RSSI Range** | -138 dBm to approximately 0 dBm |
| **RSSI Resolution** | 0.5 dB |

### Packet Engine

The CC1101 has a sophisticated hardware packet handler that offloads packet assembly and parsing from the ESP32-S3:

```
┌──────────┬──────────┬─────────┬──────────┬─────────┬──────┐
│ Preamble │   Sync   │ Length  │ Address  │ Payload │ CRC  │
│ (config) │  Word    │ (opt.)  │  (opt.)  │  (data) │(opt.)│
│ 1-255 B  │ 2 or 4 B │  1 B   │   1 B    │ 1-255 B │ 2 B  │
└──────────┴──────────┴─────────┴──────────┴─────────┴──────┘
```

| Feature | Details |
|:--------|:--------|
| **Preamble** | Configurable: 2, 3, 4, 6, 8, 12, 16, or 24 bytes of alternating 0/1 |
| **Sync Word** | 2-byte or 4-byte, configurable, carrier sense qualifier |
| **Length Field** | Optional 1-byte length for variable-length packets |
| **Address Filtering** | 1-byte, hardware-filtered, 0x00 and 0xFF broadcast |
| **CRC** | CRC-16 (CCITT), hardware computed, auto-check on RX |
| **Max Packet Length** | 255 bytes (fixed or variable length modes) |
| **TX FIFO** | 64 bytes |
| **RX FIFO** | 64 bytes |
| **FIFO Threshold** | Programmable interrupt threshold for streaming |

{: .note }
> For raw signal capture (recording OOK signals from unknown remotes), bypass the packet engine entirely by using **asynchronous serial mode**. Configure GDO0 to output raw demodulated data and sample it with the ESP32-S3 GPIO at high speed.

### Wake-on-Radio (WOR)

The CC1101 supports low-power listening with its Wake-on-Radio feature:

| Parameter | Value |
|:----------|:------|
| **WOR Sleep Current** | ~900 nA (Event 0) |
| **WOR RX Sniff Current** | Averaged ~15 μA @ 1-second interval |
| **WOR Timeout** | Programmable (Event 0 period: 15.6 ms to 1.95 s) |
| **RSSI-Based Wake** | Carrier Sense threshold configurable |

### Key Registers

The CC1101 has 47 configuration registers, 12 status registers, and an 8-byte PA (Power Amplifier) table. The most critical registers for practical use:

| Register | Address | Function |
|:---------|:--------|:---------|
| `IOCFG2` | 0x00 | GDO2 pin configuration |
| `IOCFG0` | 0x02 | GDO0 pin configuration |
| `FIFOTHR` | 0x03 | FIFO thresholds |
| `SYNC1/SYNC0` | 0x04-0x05 | Sync word |
| `PKTLEN` | 0x06 | Packet length |
| `PKTCTRL1/0` | 0x07-0x08 | Packet automation control |
| `ADDR` | 0x09 | Device address |
| `CHANNR` | 0x0A | Channel number |
| `FSCTRL1/0` | 0x0B-0x0C | Frequency synthesizer control |
| `FREQ2/1/0` | 0x0D-0x0F | **Frequency control word** (sets carrier frequency) |
| `MDMCFG4/3/2/1/0` | 0x10-0x14 | **Modem configuration** (data rate, modulation, channel spacing) |
| `DEVIATN` | 0x15 | Frequency deviation |
| `MCSM2/1/0` | 0x18-0x1A | Main Radio Control State Machine |
| `FOCCFG` | 0x19 | Frequency offset compensation |
| `AGCCTRL2/1/0` | 0x1B-0x1D | AGC control |
| `FREND1/0` | 0x21-0x22 | Front-end TX/RX configuration |
| `FSCAL3/2/1/0` | 0x23-0x26 | Frequency synthesizer calibration |

**Frequency Calculation Formula:**

```
f_carrier = (f_XOSC / 2^16) × FREQ[23:0]

Where:
  f_XOSC = 26,000,000 Hz (26 MHz crystal)
  FREQ[23:0] = (FREQ2 << 16) | (FREQ1 << 8) | FREQ0

Examples:
  433.92 MHz → FREQ = 0x10B071  (FREQ2=0x10, FREQ1=0xB0, FREQ0=0x71)
  315.00 MHz → FREQ = 0x0C1D89
  868.00 MHz → FREQ = 0x216276
  915.00 MHz → FREQ = 0x234000
```

### CC1101 Command Strobes

| Strobe | Address | Function |
|:-------|:--------|:---------|
| `SRES` | 0x30 | Reset |
| `SFSTXON` | 0x31 | Enable frequency synthesizer, stay in TX |
| `SXOFF` | 0x32 | Turn off crystal oscillator |
| `SCAL` | 0x33 | Calibrate frequency synthesizer |
| `SRX` | 0x34 | **Enter RX mode** |
| `STX` | 0x35 | **Enter TX mode** |
| `SIDLE` | 0x36 | Exit RX/TX, enter IDLE |
| `SWOR` | 0x38 | Start Wake-on-Radio |
| `SPWD` | 0x39 | Power down |
| `SFRX` | 0x3A | Flush RX FIFO |
| `SFTX` | 0x3B | Flush TX FIFO |
| `SNOP` | 0x3D | No operation (read status byte) |

### CC1101 vs Related Chips

| Feature | CC1101 | CC1100 | CC1101Q1 | CC1200 |
|:--------|:-------|:-------|:---------|:-------|
| **Frequency** | 300-928 MHz | 300-928 MHz | 300-928 MHz (automotive) | 136-960 MHz |
| **Max Data Rate** | 500 kbps | 500 kbps | 500 kbps | 1.2 Mbps |
| **Sensitivity** | -116 dBm | -112 dBm | -116 dBm | -122 dBm |
| **TX Power** | +12 dBm | +10 dBm | +12 dBm | +16 dBm |
| **FIFO** | 64 + 64 B | 64 + 64 B | 64 + 64 B | 128 + 128 B |
| **Package** | QFN-20 | QFN-20 | QFN-20 (AEC-Q100) | QFN-32 |
| **Status** | Active | Not Recommended | Active | Active |

{: .tip }
> The CC1101 on the T-Embed is a genuine Texas Instruments part, not a clone. Some ultra-cheap CC1101 modules from China use counterfeit chips with degraded sensitivity and unstable frequency synthesis. The LILYGO board uses an authentic CC1101, which is why its RF performance is notably better than bargain-bin modules.

---

## Display — 1.9" ST7789 TFT

### Specifications

| Parameter | Value |
|:----------|:------|
| **Panel Size** | 1.9 inches diagonal |
| **Resolution** | 170 × 320 pixels |
| **Orientation** | Portrait native (rotatable in software) |
| **Driver IC** | ST7789V2 |
| **Interface** | SPI (4-wire) |
| **Color Depth** | 262K colors (18-bit RGB666) |
| **Typical Drive Mode** | 16-bit RGB565 (65K colors) for performance |
| **Pixel Format** | RGB565 (16-bit), RGB666 (18-bit), RGB888 (24-bit) |
| **Backlight** | White LED, PWM-controllable brightness |
| **Viewing Angle** | IPS-class, ~170° horizontal, ~170° vertical |
| **SPI Clock** | Up to 80 MHz (typically driven at 40-60 MHz for stability) |
| **Refresh Rate** | Up to 60 fps at 40 MHz SPI (depends on partial/full update) |

### SPI Timing

Full-screen refresh at 16-bit color:

```
Pixels:     170 × 320 = 54,400 pixels
Data:       54,400 × 2 bytes = 108,800 bytes
At 40 MHz:  108,800 × 8 bits / 40,000,000 = ~21.8 ms per frame
Max FPS:    ~45 fps (theoretical at 40 MHz SPI)
```

{: .tip }
> Use **partial screen updates** whenever possible. Updating only a 170 × 40 pixel status bar instead of the full screen reduces transfer time by 87.5% and dramatically improves UI responsiveness.

### Display Pin Connections

| Function | ESP32-S3 GPIO | Notes |
|:---------|:-------------|:------|
| **MOSI (SDA)** | GPIO 11 | Shared SPI bus data out |
| **SCK (SCL)** | GPIO 12 | Shared SPI bus clock |
| **CS** | GPIO 10 | Display chip select (active low) |
| **DC (A0)** | GPIO 13 | Data/Command select |
| **RST** | GPIO 9 | Hardware reset (active low) |
| **Backlight** | GPIO 15 | PWM brightness control (LEDC channel) |

{: .pinout }
> The display and CC1101 share the SPI bus (MOSI, SCK) but use separate CS lines. Never assert both CS lines simultaneously. Most SPI libraries handle this automatically, but custom bare-metal code must manage CS arbitration manually.

---

## Rotary Encoder

### Specifications

| Parameter | Value |
|:----------|:------|
| **Type** | Mechanical incremental rotary encoder with integrated push switch |
| **Detents** | 20 per full revolution (typical) |
| **Output** | Quadrature (A/B channels, 90° phase offset) |
| **Push Switch** | Momentary, normally open |
| **Debounce** | Required (mechanical contacts, ~5ms bounce typical) |
| **Rotation** | Continuous (no mechanical stop) |

### Pin Connections

| Function | ESP32-S3 GPIO | Notes |
|:---------|:-------------|:------|
| **Channel A (CLK)** | GPIO 2 | Quadrature output A |
| **Channel B (DT)** | GPIO 1 | Quadrature output B |
| **Push Button (SW)** | GPIO 0 | Active low (has internal pull-up) |

### Quadrature Decoding

```
Clockwise Rotation:
  CLK (A): ─┐ ┌─┐ ┌─┐ ┌─
             └─┘ └─┘ └─┘
  DT  (B): ──┐ ┌─┐ ┌─┐ ┌
              └─┘ └─┘ └─┘
              ▲ B lags A by 90°

Counter-Clockwise Rotation:
  CLK (A): ──┐ ┌─┐ ┌─┐ ┌
              └─┘ └─┘ └─┘
  DT  (B): ─┐ ┌─┐ ┌─┐ ┌─
             └─┘ └─┘ └─┘
              ▲ A lags B by 90°
```

{: .tip }
> GPIO 0 serves double duty as the encoder push button and the ESP32-S3 **BOOT/STRAPPING pin**. Holding it low during reset enters download mode. This is by design — it means pressing the encoder button while plugging in USB puts the board into firmware flashing mode. During normal operation, GPIO 0 works as a standard input with internal pull-up.

### Debouncing Recommendations

| Method | Approach | Latency |
|:-------|:---------|:--------|
| **Software Timer** | Ignore state changes within 5 ms of last change | ~5 ms |
| **Interrupt + Filter** | Use hardware timer ISR to sample at fixed rate | ~2 ms |
| **ESP-IDF PCNT** | Pulse Counter peripheral with hardware glitch filter | ~0 ms (hardware) |

{: .note }
> The ESP32-S3's built-in **Pulse Counter (PCNT)** peripheral is ideal for rotary encoder decoding. It performs quadrature decoding entirely in hardware with configurable glitch filtering, consuming zero CPU cycles. The Arduino `ESP32Encoder` library uses PCNT automatically.

---

## IR Transceiver

The T-Embed CC1101 Plus includes both an IR transmitter and receiver for infrared remote control operations.

### IR Transmitter

| Parameter | Value |
|:----------|:------|
| **Component** | IR LED (940 nm wavelength) |
| **Viewing Angle** | ~20-30° half-angle |
| **Carrier Frequency** | Software-generated, typically 38 kHz |
| **Effective Range** | ~5-10 meters (depends on target sensitivity, ambient IR) |
| **GPIO Pin** | GPIO 44 |
| **Drive Method** | PWM carrier modulated by RMT peripheral |

### IR Receiver

| Parameter | Value |
|:----------|:------|
| **Component** | Integrated IR receiver module (TSOP-style, 38 kHz tuned) |
| **Carrier Frequency** | 38 kHz center (accepts 36-40 kHz range) |
| **Effective Range** | ~10-15 meters |
| **Output** | Demodulated digital signal (active low) |
| **GPIO Pin** | GPIO 3 |

### Supported IR Protocols

The ESP32-S3's **RMT (Remote Control Transceiver)** peripheral provides hardware-level IR signal timing. Common protocols supported by firmware:

| Protocol | Carrier | Encoding | Bits | Common Devices |
|:---------|:--------|:---------|:-----|:---------------|
| **NEC** | 38 kHz | Pulse distance | 32 | TVs, set-top boxes, most consumer electronics |
| **NEC Extended** | 38 kHz | Pulse distance | 32 | Extended address space NEC |
| **RC5** | 36 kHz | Manchester / bi-phase | 14 | Philips, older European |
| **RC6** | 36 kHz | Manchester / bi-phase | 16+ | Microsoft MCE remotes |
| **Sony SIRC** | 40 kHz | Pulse width | 12/15/20 | Sony TVs, PlayStation |
| **Samsung** | 38 kHz | Pulse distance | 32 | Samsung TVs, appliances |
| **Panasonic** | 36.7 kHz | Pulse distance | 48 | Panasonic / Kaseikyo |
| **LG** | 38 kHz | Pulse distance | 28 | LG electronics |
| **Sharp** | 38 kHz | Pulse distance | 15 | Sharp TVs |
| **Dish** | 57.6 kHz | Pulse distance | 16 | Dish Network |
| **JVC** | 38 kHz | Pulse distance | 16 | JVC audio/video |
| **Whynter** | 38 kHz | Pulse distance | 32 | AC units |
| **Raw** | Any | Raw timing capture | Any | Unknown / proprietary |

{: .tip }
> The **raw capture mode** is the most versatile. It records exact pulse timing without protocol decoding, letting you capture and replay virtually any IR signal — even proprietary protocols from obscure devices. The ESP32-S3 RMT peripheral can capture timing with microsecond precision.

---

## Speaker / Buzzer

### Specifications

| Parameter | Value |
|:----------|:------|
| **Type** | Electromagnetic buzzer / micro speaker |
| **Resonant Frequency** | ~2.7 kHz (peak loudness) |
| **Usable Frequency Range** | ~200 Hz - 8 kHz |
| **Drive Method** | PWM via ESP32-S3 LEDC or DAC emulation |
| **GPIO Pin** | GPIO 46 |
| **Volume** | ~75-80 dB at 10 cm (at resonant frequency) |

### Use Cases

| Application | Implementation |
|:------------|:--------------|
| **RTTTL Ringtones** | Parse Ring Tone Text Transfer Language strings, play melodies |
| **UI Feedback** | Click sounds for encoder rotation, confirmation beeps |
| **Morse Code** | Encode text to audible Morse, adjustable WPM |
| **Signal Audio** | Audible representation of RF signal strength (Geiger-counter style) |
| **Alerts** | Proximity warnings, scan-complete notifications |
| **Frequency Counter** | Output captured RF frequency as audible tone |

{: .note }
> The speaker is driven via PWM, so it produces square waves (fundamental + odd harmonics). For cleaner audio, use the ESP32-S3's I2S peripheral with a PDM output, though the small buzzer element won't reproduce high-fidelity audio regardless of the drive signal.

---

## USB-C Connector

### Specifications

| Parameter | Value |
|:----------|:------|
| **Connector** | USB Type-C receptacle |
| **USB Standard** | USB 2.0 Full Speed (12 Mbps) |
| **Controller** | ESP32-S3 native USB OTG (no external UART bridge chip) |
| **Power Input** | 5V via USB-C (VBUS) |
| **Data Pins** | D+/D- directly connected to ESP32-S3 GPIO 19 (D-) and GPIO 20 (D+) |

### USB Functions

| Function | Class | Description |
|:---------|:------|:------------|
| **CDC-ACM** | Communications Device Class | Virtual serial port for debug console, AT commands |
| **HID** | Human Interface Device | Keyboard/mouse emulation (BadUSB payloads) |
| **MSC** | Mass Storage Class | Expose flash filesystem as USB drive |
| **USB-JTAG** | Debug | Built-in JTAG debugging via USB (no external probe needed) |
| **DFU** | Device Firmware Update | Firmware flashing without external tools |

{: .warning }
> The ESP32-S3 uses **native USB** — there is no CH340, CP2102, or FTDI chip. This means:
> - **No extra drivers needed** on modern operating systems
> - The USB serial port **disappears during reset** (brief disconnection is normal during firmware flash)
> - You must hold **GPIO 0 (encoder button) during power-on** to force download mode if firmware is bricked
> - The COM port name changes between CDC mode (firmware running) and download mode (bootloader)

---

## Antenna System

### Built-in vs External Antenna

The T-Embed CC1101 Plus features an **SMA female connector** for attaching external antennas. The CC1101 is connected to this SMA connector through a matching network on the PCB.

{: .warning }
> **Never transmit without an antenna connected.** Operating the CC1101 without a proper antenna (or with a severely mismatched antenna) can reflect power back into the PA stage and damage the CC1101 output transistors. Always connect an appropriate antenna before enabling TX mode.

### Antenna Length Recommendations

For optimal performance, use a **quarter-wave monopole** antenna matched to your target frequency:

| Frequency | Quarter-Wave Length | Common Use |
|:----------|:-------------------|:-----------|
| **315 MHz** | 23.8 cm (9.37") | Garage doors, car remotes (Americas/Asia) |
| **433.92 MHz** | 17.3 cm (6.81") | ISM Europe/Asia, weather stations, remotes |
| **868 MHz** | 8.6 cm (3.39") | ISM Europe, LoRa, smart home |
| **915 MHz** | 8.2 cm (3.23") | ISM Americas, LoRa, Z-Wave |

**Formula:**

```
λ (wavelength) = c / f
Quarter-wave = λ / 4 = (299,792,458 m/s) / (f in Hz) / 4

Example for 433.92 MHz:
  λ = 299,792,458 / 433,920,000 = 0.6909 m
  λ/4 = 0.1727 m = 17.27 cm
```

### Antenna Considerations

| Factor | Guidance |
|:-------|:---------|
| **SWR** | Target SWR < 2:1 for good performance; SWR > 3:1 causes significant power loss |
| **Gain** | Quarter-wave monopole: ~2.15 dBi; Telescopic whip: variable |
| **Ground Plane** | Quarter-wave monopole needs a ground plane; the PCB provides a minimal one |
| **Multi-band** | Telescopic antennas adjustable to different lengths cover multiple bands |
| **Coaxial** | Use 50Ω coaxial cable (RG-174, RG-58) for SMA extensions |
| **Polarization** | Most sub-GHz devices use vertical polarization; orient antenna vertically |

{: .tip }
> For maximum flexibility, get a **telescopic whip antenna** with SMA connector. Extend it to the appropriate quarter-wave length for your target frequency. This single antenna covers all bands with reasonable performance, unlike fixed-length antennas optimized for one frequency.

---

## Power System

### Power Input

| Source | Voltage | Connector | Notes |
|:-------|:--------|:----------|:------|
| **USB-C** | 5V | USB Type-C | Primary power, also provides data |
| **LiPo Battery** | 3.7V nominal (4.2V full, 3.0V cutoff) | JST PH 2.0 (2-pin) | Check polarity before connecting |

{: .warning }
> **Verify battery polarity before connecting.** The JST PH 2.0 connector has no polarity protection on some board revisions. Connecting a battery with reversed polarity will likely destroy the charging IC and potentially the ESP32-S3. The positive (+) pin is typically closest to the board edge — but **always verify with a multimeter** on your specific board.

### Charging Circuit

| Parameter | Value |
|:----------|:------|
| **Charging IC** | TP4054 or equivalent single-cell Li-Ion charger |
| **Charge Current** | ~500 mA (set by programming resistor) |
| **Charge Voltage** | 4.2V (±1%) |
| **Charging Indicator** | LED on PCB (red = charging, green/off = complete) |
| **Charge Time** | ~2 hours for a typical 1000 mAh cell |

### Voltage Regulation

| Rail | Voltage | Regulator | Supplies |
|:-----|:--------|:----------|:---------|
| **Main 3.3V** | 3.3V | LDO (ME6211 or equivalent) | ESP32-S3, CC1101, display logic |
| **USB VBUS** | 5V | Direct from USB | Charging IC input |
| **Battery** | 3.0-4.2V | Direct from LiPo | LDO input when USB disconnected |

### Power Consumption Measurements

Typical current draw from the 3.3V rail (approximate — varies by firmware, activity, and measurement conditions):

| State | Current Draw | Notes |
|:------|:------------|:------|
| **Idle (screen on, backlight 50%)** | ~80 mA | ESP32-S3 active, no radio |
| **Idle (screen on, backlight 100%)** | ~110 mA | Backlight dominates |
| **Idle (screen off)** | ~40 mA | ESP32-S3 active, display off |
| **WiFi Scanning** | ~160-200 mA | Peaks during active scan |
| **WiFi Connected (idle)** | ~100 mA | DTIM-based power save |
| **BLE Scanning** | ~100-130 mA | Active BLE scan |
| **CC1101 RX (listening)** | ~55 mA + ESP32 | CC1101 RX current ~15 mA |
| **CC1101 TX (+12 dBm)** | ~75 mA + ESP32 | CC1101 TX current ~30 mA at max power |
| **CC1101 TX (0 dBm)** | ~55 mA + ESP32 | Reduced PA current |
| **IR Transmitting** | ~100-150 mA | LED current during pulses |
| **Full Load** | ~250-350 mA | WiFi + CC1101 TX + display + speaker |
| **Deep Sleep** | ~10-50 μA | ESP32-S3 deep sleep, all peripherals off |
| **Deep Sleep (ideal)** | ~7 μA | ESP32-S3 only, minimal leakage |

### Battery Life Estimates

For a **1000 mAh** LiPo battery (common JST PH size):

| Use Case | Estimated Runtime |
|:---------|:-----------------|
| Continuous WiFi scanning (screen on) | ~5-6 hours |
| Continuous CC1101 RX monitoring (screen on) | ~8-10 hours |
| Intermittent use (10 min on / 50 min sleep per hour) | ~24-48 hours |
| Deep sleep standby | ~4,000+ hours (~6 months) |
| Mixed active use (casual hacking session) | ~4-6 hours |

{: .tip }
> For extended field use, carry a small USB power bank. A 5000 mAh power bank will run the T-Embed CC1101 Plus for roughly 20+ hours of active use. This is more practical than hunting for larger LiPo cells that fit the compact form factor.

---

## Complete Pin Mapping

{: .pinout }
> This is the canonical GPIO pin mapping for the T-Embed CC1101 Plus. Pin assignments may vary between board revisions. Always cross-reference with the `pin_config.h` file from the official LILYGO GitHub repository for your specific board revision.

### Full GPIO Assignment Table

| GPIO | Function | Peripheral | Direction | Notes |
|:-----|:---------|:-----------|:----------|:------|
| **GPIO 0** | Encoder Push Button (SW) | Rotary Encoder | Input (pull-up) | BOOT/STRAPPING pin — hold low during reset for download mode |
| **GPIO 1** | Encoder Channel B (DT) | Rotary Encoder | Input | Quadrature output B |
| **GPIO 2** | Encoder Channel A (CLK) | Rotary Encoder | Input | Quadrature output A |
| **GPIO 3** | IR Receiver Data | IR Receiver | Input | Demodulated IR input (active low) |
| **GPIO 4** | CC1101 CS | CC1101 SPI | Output | Chip select (active low) |
| **GPIO 5** | CC1101 GDO0 | CC1101 | Input | General Digital Output 0 — configurable (IRQ, RX data, sync) |
| **GPIO 6** | CC1101 GDO2 | CC1101 | Input | General Digital Output 2 — configurable (status, RSSI, CCA) |
| **GPIO 7** | CC1101 MOSI | CC1101 SPI | Output | SPI data to CC1101 |
| **GPIO 8** | CC1101 SCK | CC1101 SPI | Output | SPI clock |
| **GPIO 9** | Display RST | Display | Output | Hardware reset (active low) |
| **GPIO 10** | Display CS | Display SPI | Output | Chip select (active low) |
| **GPIO 11** | Display MOSI (SDA) | Display SPI | Output | SPI data to display |
| **GPIO 12** | Display SCK (SCL) | Display SPI | Output | SPI clock |
| **GPIO 13** | Display DC (A0) | Display | Output | Data/Command select |
| **GPIO 14** | CC1101 MISO | CC1101 SPI | Input | SPI data from CC1101 |
| **GPIO 15** | Display Backlight | Display | Output (PWM) | Backlight brightness (LEDC) |
| **GPIO 16** | Free / Expansion | User | I/O | Available for user projects |
| **GPIO 17** | Free / Expansion | User | I/O | Available for user projects |
| **GPIO 18** | Free / Expansion | User | I/O | Available for user projects |
| **GPIO 19** | USB D- | USB | Bidirectional | Native USB data minus |
| **GPIO 20** | USB D+ | USB | Bidirectional | Native USB data plus |
| **GPIO 21** | Free / Expansion | User | I/O | Available for user projects |
| **GPIO 33** | Free / Expansion | User | I/O | Available (PSRAM may use some high GPIOs) |
| **GPIO 34** | Free / Expansion | User | I/O | Available |
| **GPIO 35** | Free / Expansion | User | I/O | Available |
| **GPIO 36** | Free / Expansion | User | I/O | Available |
| **GPIO 37** | Free / Expansion | User | I/O | Available |
| **GPIO 38** | Free / Expansion | User | I/O | Available |
| **GPIO 39** | Free / Expansion | User | I/O | Available |
| **GPIO 40** | Free / Expansion | User | I/O | Available |
| **GPIO 41** | Free / Expansion | User | I/O | Available |
| **GPIO 42** | Free / Expansion | User | I/O | Available |
| **GPIO 43** | UART TX (default) | Debug UART | Output | Default serial output (if not using USB CDC) |
| **GPIO 44** | IR Transmitter | IR LED | Output (PWM) | Modulated IR output via RMT peripheral |
| **GPIO 45** | Strapping | Boot Config | Input | BOOT strapping pin — do not use unless you know what you're doing |
| **GPIO 46** | Speaker / Buzzer | Audio | Output (PWM) | PWM-driven via LEDC |
| **GPIO 47** | Power Control / LED | Status LED | Output | Board-specific (may be power enable or LED) |
| **GPIO 48** | Power Control / LED | Status LED | Output | Board-specific (may be WS2812 data or power enable) |

{: .note }
> GPIOs 26-32 are used internally by the ESP32-S3 for the OPI PSRAM interface and are **not available** for user GPIO on N16R8 modules. GPIOs 33-37 availability depends on whether the PSRAM uses Octal SPI mode — consult your specific module's datasheet. The above table reflects typical availability.

### SPI Bus Architecture

The T-Embed CC1101 Plus uses two separate SPI buses to avoid contention:

```
                    ESP32-S3
                 ┌──────────────┐
                 │              │
  CC1101 SPI     │   SPI2 (FSPI)│         Display SPI
  ═══════════    │              │         ═══════════
                 │              │
  MOSI ◄─── GPIO 7 ──────┐     │    GPIO 11 ───► MOSI
  MISO ───► GPIO 14 ─────┤     │    (no MISO — write-only)
  SCK  ◄─── GPIO 8  ─────┤     │    GPIO 12 ───► SCK
  CS   ◄─── GPIO 4  ─────┘     │    GPIO 10 ───► CS
                 │              │    GPIO 13 ───► DC
  GDO0 ───► GPIO 5             │    GPIO 9  ───► RST
  GDO2 ───► GPIO 6             │    GPIO 15 ───► BL (PWM)
                 │              │
                 └──────────────┘
```

{: .pinout }
> The CC1101 and display use **separate SPI peripherals** (SPI2 and SPI3) within the ESP32-S3. This is a deliberate design choice — it allows simultaneous CC1101 radio operations and display updates without bus contention. Some cheaper boards share a single SPI bus, causing RF capture dropouts during screen refreshes. The T-Embed avoids this by dedicating a full SPI peripheral to each device.

---

## Complete Pinout Diagram

```
                    LILYGO T-Embed CC1101 Plus
                    ═══════════════════════════

    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │   ┌──────────────────┐          ┌──────────────────┐     │
    │   │   1.9" ST7789V2  │          │                  │     │
    │   │   TFT Display    │          │   CC1101 Module   │     │
    │   │   170 × 320 px   │          │   Sub-GHz Radio   │     │
    │   │                  │          │                  │     │
    │   │   MOSI → GPIO 11 │          │   MOSI ← GPIO 7  │     │
    │   │   SCK  → GPIO 12 │          │   MISO → GPIO 14 │     │
    │   │   CS   → GPIO 10 │          │   SCK  ← GPIO 8  │     │
    │   │   DC   → GPIO 13 │          │   CS   ← GPIO 4  │     │
    │   │   RST  → GPIO 9  │          │   GDO0 → GPIO 5  │     │
    │   │   BL   → GPIO 15 │          │   GDO2 → GPIO 6  │     │
    │   └──────────────────┘          └────────┬─────────┘     │
    │                                          │               │
    │                                     ┌────┴────┐          │
    │                                     │   SMA   │          │
    │                                     │ Antenna │          │
    │                                     │  Port   │          │
    │                                     └─────────┘          │
    │                                                          │
    │   ┌──────────────────────────────────────────────────┐   │
    │   │                  ESP32-S3-WROOM-1                 │   │
    │   │                  N16R8 Module                     │   │
    │   │                                                  │   │
    │   │   CPU:   2× Xtensa LX7 @ 240 MHz                │   │
    │   │   RAM:   512 KB SRAM + 8 MB OPI PSRAM            │   │
    │   │   Flash: 16 MB QIO                               │   │
    │   │   WiFi:  802.11 b/g/n 2.4 GHz                   │   │
    │   │   BLE:   Bluetooth 5.0 LE                        │   │
    │   │   USB:   Native OTG (GPIO 19/20)                 │   │
    │   │                                                  │   │
    │   └──────────────────────────────────────────────────┘   │
    │                                                          │
    │   ┌────────────┐  ┌────────┐  ┌────────┐  ┌──────────┐  │
    │   │  Rotary     │  │   IR   │  │ Speaker│  │  USB-C   │  │
    │   │  Encoder    │  │        │  │        │  │          │  │
    │   │             │  │TX→ G44 │  │  GPIO  │  │ D- → G19 │  │
    │   │ CLK → GPIO 2│  │RX→ G3  │  │   46   │  │ D+ → G20 │  │
    │   │ DT  → GPIO 1│  └────────┘  └────────┘  │ VBUS: 5V │  │
    │   │ SW  → GPIO 0│                           └──────────┘  │
    │   └────────────┘                                         │
    │                                                          │
    │   ┌──────────────────────────────────────────────────┐   │
    │   │              Expansion / Free GPIOs               │   │
    │   │                                                  │   │
    │   │   GPIO 16, 17, 18, 21, 33-42                     │   │
    │   │   All 3.3V logic, max 40 mA per pin              │   │
    │   │   Supports: SPI, I2C, UART, PWM, ADC, Touch      │   │
    │   └──────────────────────────────────────────────────┘   │
    │                                                          │
    │   ┌───────────────────┐     ┌──────────────────────┐     │
    │   │   JST PH 2.0      │     │   Power System       │     │
    │   │   Battery Input    │     │                      │     │
    │   │   3.7V LiPo        │     │   LDO → 3.3V rail    │     │
    │   │   + charge circuit │     │   TP4054 charger     │     │
    │   └───────────────────┘     └──────────────────────┘     │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### Signal Flow Diagram

```
                          RF Signals (300-928 MHz)
                                  │
                                  ▼
                          ┌───────────────┐
                          │  SMA Antenna   │
                          │   Connector    │
                          └───────┬───────┘
                                  │
                          ┌───────▼───────┐
                          │  Matching     │
                          │  Network      │
                          └───────┬───────┘
                                  │
                    ┌─────────────▼──────────────┐
                    │        CC1101              │
                    │   Sub-GHz Transceiver      │
                    │                            │
                    │  300-348 / 387-464 /        │
                    │  779-928 MHz               │
                    │  OOK/ASK/FSK/GFSK/MSK     │
                    │  TX: +12 dBm              │
                    │  RX: -116 dBm             │
                    └─────────┬──────────────────┘
                              │ SPI (GPIO 4,7,8,14)
                              │ IRQ (GPIO 5,6)
                              ▼
    ┌────────────────────────────────────────────────────────┐
    │                      ESP32-S3                          │
    │               Dual-Core LX7 @ 240 MHz                 │
    │                                                        │
    │   ┌──────────┐ ┌──────────┐ ┌───────┐ ┌────────────┐  │
    │   │  WiFi    │ │   BLE    │ │  USB  │ │   Crypto   │  │
    │   │ 2.4 GHz  │ │  5.0 LE  │ │  OTG  │ │ AES/SHA/   │  │
    │   │ b/g/n    │ │ LR/2M   │ │  HID  │ │ RSA/ECC    │  │
    │   └──────────┘ └──────────┘ └───┬───┘ └────────────┘  │
    │                                 │                      │
    └───┬──────────┬──────────┬───────┼──────────┬───────────┘
        │          │          │       │          │
        ▼          ▼          ▼       ▼          ▼
   ┌────────┐ ┌────────┐ ┌──────┐ ┌──────┐ ┌────────┐
   │Display │ │Encoder │ │  IR  │ │USB-C │ │Speaker │
   │ST7789  │ │+ Button│ │TX/RX │ │      │ │Buzzer  │
   │170×320 │ │        │ │940nm │ │Power │ │        │
   │SPI     │ │Quad+SW │ │38kHz │ │Data  │ │PWM     │
   └────────┘ └────────┘ └──────┘ └──────┘ └────────┘
```

---

## GPIO Electrical Characteristics

| Parameter | Value |
|:----------|:------|
| **Logic Level** | 3.3V CMOS |
| **Output High (VOH)** | ~3.1V (at 20 mA) |
| **Output Low (VOL)** | ~0.2V (at 20 mA) |
| **Input High Threshold (VIH)** | ~2.0V |
| **Input Low Threshold (VIL)** | ~0.8V |
| **Maximum Current per GPIO** | 40 mA (absolute max) |
| **Recommended Current per GPIO** | 20 mA |
| **Total GPIO Current (all pins)** | 1200 mA absolute max |
| **Internal Pull-up Resistance** | ~45 kΩ |
| **Internal Pull-down Resistance** | ~45 kΩ |
| **Input Capacitance** | ~2 pF |

{: .warning }
> The ESP32-S3 GPIOs are **3.3V only**. They are **NOT 5V tolerant**. Connecting a 5V signal directly to any GPIO will damage the ESP32-S3 permanently. Use a level shifter (TXS0108E, BSS138-based, or resistor divider) when interfacing with 5V devices like some UART modules, Arduino Uno, or 5V sensors.

---

## Schematic Design Notes

### Decoupling and Power Filtering

- **CC1101**: Multiple 100 nF (0402) ceramic decoupling capacitors on AVDD, DVDD, and DCOUPL pins. A 100 pF cap on RF input for DC blocking.
- **ESP32-S3 module**: 10 μF bulk capacitor + 100 nF ceramic on 3.3V rail near module pins.
- **Display**: 100 nF ceramic on VCC, separate 10 μF for backlight LED driver.
- **Voltage Regulator**: 10 μF input, 22 μF output ceramic capacitors (X5R/X7R).

### Pull-Up / Pull-Down Resistors

| Pin | Resistor | Value | Purpose |
|:----|:---------|:------|:--------|
| GPIO 0 (Encoder SW) | Pull-up | 10 kΩ | BOOT strapping + encoder button |
| GPIO 45 | Pull-down | 10 kΩ (internal) | VDD_SPI voltage select strapping |
| GPIO 46 | Pull-down | 10 kΩ (internal) | ROM log output strapping |
| CC1101 CS (GPIO 4) | Pull-up | 10 kΩ | Keep CS high (deselected) during boot |
| Display CS (GPIO 10) | Pull-up | 10 kΩ | Keep CS high (deselected) during boot |
| I2C SDA/SCL (if used) | Pull-up | 4.7 kΩ | Required for I2C bus |

### RF Matching Network

The CC1101 antenna connection uses a standard TI reference matching network:

```
CC1101 RF Pin                              SMA Connector
     │                                          │
     ├──┤ L1 (balun inductor) ├──┬──┤ C1 ├──────┤
     │                          │               │
     └──┤ L2 (balun inductor) ├──┘               │
                                                 │
     Note: Component values depend on           GND
     target frequency band. TI provides
     reference designs for 315/433/868/915 MHz.
```

{: .note }
> The matching network is optimized for a specific frequency band at the factory. While the CC1101 can tune across its full range in software, the RF frontend hardware provides optimal impedance matching (50Ω to antenna) at the designed center frequency. Performance will degrade somewhat at frequencies far from the design center, though it remains usable across all three CC1101 bands.

### Level Shifting

The T-Embed CC1101 Plus does **not** include level shifters because all onboard peripherals operate at 3.3V:

- CC1101: 3.3V supply, 3.3V logic
- ST7789V2: 3.3V I/O capable (accepts 3.3V SPI directly)
- Encoder: Passive component, operates at GPIO voltage
- IR LED: Driven through current-limiting resistor from GPIO
- Speaker: Driven through transistor or direct from GPIO (low power)

For external 5V peripherals, add a level shifter on the expansion GPIOs.

---

## Physical Dimensions for Enclosure Design

### PCB Measurements

```
    ┌─────────────────────────────────────────┐
    │               103 mm                     │
    │◄─────────────────────────────────────────►│
    │                                          │ ▲
    │   ┌──────────────────────────────────┐   │ │
    │   │                                  │   │ │
    │   │        Component Area            │   │ 37 mm
    │   │                                  │   │ │
    │   └──────────────────────────────────┘   │ │
    │                                          │ ▼
    └─────┬──────────────────────────────┬─────┘
          │          USB-C              │
          │     (centered, bottom)       │
          └──────────────────────────────┘
```

| Dimension | Value |
|:----------|:------|
| **PCB Length** | ~103 mm |
| **PCB Width** | ~37 mm |
| **PCB Thickness** | ~1.6 mm (standard 4-layer) |
| **Total Height (with components)** | ~13 mm (including tallest component) |
| **Display Cutout** | ~25 × 50 mm (visible area for 1.9" diagonal) |
| **SMA Connector Protrusion** | ~10 mm from board edge |
| **USB-C Protrusion** | ~2 mm from board edge |
| **JST Battery Connector** | ~5 mm protrusion from board edge |
| **Encoder Shaft Height** | ~12 mm above PCB surface |
| **Encoder Shaft Diameter** | ~6 mm (for knob fitting) |
| **Mounting Holes** | Check your specific revision (some have 2x M2 mounting holes) |

### Enclosure Design Tips

{: .tip }
> When designing a 3D-printed enclosure:
> - Add **0.3 mm tolerance** on all sides for FDM printing
> - Include a cutout for the **SMA connector** (typically 6.35 mm hex, 9.5 mm thread diameter)
> - Allow ventilation — the ESP32-S3 can heat up during sustained WiFi TX
> - Consider a **lanyard hole** for field use
> - The encoder shaft needs a **clear path** for rotation and press — don't enclose it too tightly
> - Include a slot for the **JST battery cable** if using a LiPo
> - The IR LED and receiver need a **clear window** (IR-transparent plastic or an opening)

---

## Hardware Revision Identification

To identify your board revision:

1. **Check the silkscreen** on the back of the PCB for version markings (V1.0, V1.1, etc.)
2. **Check the LILYGO GitHub** repository's `pin_config.h` — multiple revisions may have different pin definitions
3. **Count the components** — earlier revisions may lack the IR transceiver or speaker
4. **Check the antenna connector** — some early boards used U.FL/IPEX instead of SMA

{: .note }
> When in doubt, the **software pin definitions** in the official LILYGO repository are the authoritative source. Hardware silk screen markings may not always be updated between minor revisions. Flash the official LILYGO test firmware to verify all peripherals are detected and working on your specific board before developing custom firmware.

---

## Summary — Quick Reference Card

```
╔══════════════════════════════════════════════════════════════╗
║           T-EMBED CC1101 PLUS — QUICK REFERENCE             ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  MCU:     ESP32-S3 Dual LX7 @ 240 MHz                       ║
║  Memory:  512KB SRAM + 8MB PSRAM + 16MB Flash                ║
║  Radio:   CC1101 300-348/387-464/779-928 MHz                 ║
║  WiFi:    802.11 b/g/n 2.4 GHz                               ║
║  BLE:     Bluetooth 5.0 LE                                   ║
║  Display: 1.9" ST7789 170×320 TFT (SPI)                     ║
║  Input:   Rotary Encoder + Push Button                       ║
║  IR:      TX (940nm LED) + RX (38kHz receiver)               ║
║  Audio:   PWM Buzzer/Speaker                                 ║
║  USB:     Type-C, Native OTG (CDC/HID/MSC)                   ║
║  Power:   USB-C 5V or 3.7V LiPo (JST PH 2.0)               ║
║  Antenna: SMA female connector                               ║
║                                                              ║
║  KEY PINS:                                                   ║
║  CC1101:  MOSI=7 MISO=14 SCK=8 CS=4 GDO0=5 GDO2=6          ║
║  Display: MOSI=11 SCK=12 CS=10 DC=13 RST=9 BL=15            ║
║  Encoder: CLK=2 DT=1 SW=0                                   ║
║  IR:      TX=44 RX=3                                         ║
║  Speaker: 46                                                 ║
║  USB:     D-=19 D+=20                                        ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

{: .warning }
> **Disclaimer**: GPIO assignments in this document are based on commonly reported pin configurations for the T-Embed CC1101 Plus. LILYGO occasionally updates pin assignments between board revisions without changing the product name. **Always verify against the `pin_config.h` file in the official LILYGO GitHub repository for your specific hardware revision** before wiring external components or writing firmware that depends on specific pin assignments.

---

[Next: 02 — Getting Started](/t-embed-cc1101-guide/docs/02-getting-started/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Back to Home](/t-embed-cc1101-guide/){: .btn .fs-5 .mb-4 .mb-md-0 }
