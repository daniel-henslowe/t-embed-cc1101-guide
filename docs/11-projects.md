---
layout: default
title: "11 — Practical Projects"
nav_order: 12
---

# 11 --- Practical Projects
{: .fs-9 }

10 real-world builds from beginner to expert. Each one teaches you something fundamental about RF, wireless security, and embedded systems.
{: .fs-6 .fw-300 }

---

{: .tip }
> These projects are designed to be completed in order. Each one builds on skills from the previous projects. If you jump ahead, you may need to reference earlier sections for foundational techniques.

{: .danger }
> **Every project in this guide must be performed on devices and networks you own or have explicit written authorization to test.** Unauthorized transmission, interception, or access is a federal crime. See [Chapter 13 --- Legal Reference](13-legal-reference/) for full details.

---

## Project 1: Personal RF Environment Audit
{: .d-inline-block }

Beginner
{: .label .label-green }

### Overview

Your home is full of invisible radio signals. Weather stations, doorbells, car key fobs, baby monitors, smart home sensors, utility meters --- they are all transmitting around you constantly. This project teaches you to systematically discover, identify, and document every signal in your environment.

### What You'll Learn

- How to use the CC1101's frequency analyzer across all supported bands
- Signal identification by modulation type, frequency, and timing
- Systematic scanning methodology
- Documentation practices used in professional RF surveys

### Materials Needed

- T-Embed CC1101 Plus with antenna
- Notebook or spreadsheet for logging
- (Optional) Computer with URH or rtl_433 for cross-reference

### Step-by-Step Instructions

**Step 1 --- Prepare Your Scanning Plan**

The CC1101 covers three frequency ranges with gaps between them. You need to scan each one systematically:

| Band | Range | Common Devices |
|:-----|:------|:---------------|
| Band 1 | 300--348 MHz | Older garage doors, some car fobs, industrial sensors |
| Band 2 | 387--464 MHz | Weather stations, doorbells, tire pressure monitors, key fobs |
| Band 3 | 779--928 MHz | Newer smart home devices, utility meters, LoRa, some garage doors |

**Step 2 --- Run the Frequency Analyzer**

Open Bruce firmware and navigate to:

```
Main Menu → RF → Frequency Analyzer
```

Start with Band 2 (387--464 MHz) because it is the most active in most homes. Let the analyzer run for at least 5 minutes. Every time the display shows a signal spike, note:

- Frequency (e.g., 433.92 MHz)
- Approximate signal strength
- Whether it repeats periodically or only on events
- Time of detection

**Step 3 --- Identify Periodic Transmitters**

Many devices transmit on a schedule. Weather stations typically send data every 30--60 seconds. Smart home sensors check in every few minutes. Leave the analyzer running for 15--30 minutes on each band and watch for patterns.

**Step 4 --- Trigger Event-Based Transmitters**

Some devices only transmit when activated:

- Press your doorbell
- Open/close a window or door with sensors
- Walk past a motion sensor
- Lock/unlock your car (stand at a safe distance)
- Press buttons on wireless remotes

Note the frequency and timing for each.

**Step 5 --- Attempt Signal Capture**

For each signal you found, switch to the Sub-GHz capture mode:

```
Main Menu → RF → Sub-GHz → Read
```

Set the frequency to what you observed. Try both common modulations:

- **AM (OOK/ASK)** --- Most consumer devices use this
- **FM (FSK/GFSK)** --- Some newer or digital devices

Capture a few transmissions from each device.

**Step 6 --- Build Your Frequency Map**

Create a document with this structure:

| # | Frequency | Modulation | Device | Interval | Signal Strength | Notes |
|:--|:----------|:-----------|:-------|:---------|:----------------|:------|
| 1 | 433.92 MHz | OOK | Weather station | 60s | Strong | Temp/humidity |
| 2 | 315.00 MHz | OOK | Doorbell | Event | Medium | Fixed code |
| 3 | 868.35 MHz | FSK | Smart plug | 120s | Weak | Encrypted |

### Expected Results

A typical home will reveal 5--15 active RF devices. You will likely find:

- At least one device on 433.92 MHz (the most common ISM frequency worldwide)
- Periodic transmitters you did not know existed (neighbor's weather station, nearby utility meters)
- A mix of OOK and FSK modulation types
- Varying signal strengths based on distance and obstructions

### Extensions

- Repeat the survey at different times of day (some devices are more active at certain times)
- Survey other locations you have permission to scan (office, workshop, vehicle)
- Use a computer with URH to decode the actual data in captured signals
- Map signal strengths by moving around your home to find transmitter locations

---

## Project 2: Universal IR Remote Replacement
{: .d-inline-block }

Beginner
{: .label .label-green }

### Overview

Capture the IR codes from every remote control in your home. Build organized profiles for each device. Replace your entire collection of remotes with the T-Embed.

### What You'll Learn

- IR signal capture and storage
- Protocol identification (NEC, Sony SIRC, RC-5, RC-6, Samsung, etc.)
- Building a personal IR code library
- The T-Embed's IR hardware capabilities and limitations

### Materials Needed

- T-Embed CC1101 Plus
- All your existing IR remote controls
- Fresh batteries in each remote (weak batteries = weak IR signal)
- (Optional) Smartphone camera to verify IR LED activity

### Step-by-Step Instructions

**Step 1 --- Inventory Your Remotes**

Walk through your home and collect every IR remote. Common ones:

- TV remote(s)
- Soundbar / audio receiver remote
- Streaming device remote (Roku, Fire TV, Apple TV)
- Air conditioner / fan remote
- Set-top box / cable box remote
- Projector remote
- LED strip / smart light remote

{: .tip }
> Not sure if a remote uses IR? Point it at your smartphone's front-facing camera and press a button. If you see a purple/white flash on screen, it is IR. (Most front cameras do not have IR filters.)

**Step 2 --- Set Up IR Capture**

Navigate to:

```
Main Menu → IR → Read
```

Point the remote directly at the T-Embed's IR receiver. Hold the remote 5--15 cm away for best results.

**Step 3 --- Capture Each Button Systematically**

For each remote, capture these buttons in order:

1. **Power** (most important)
2. **Volume Up / Down**
3. **Channel Up / Down** (if applicable)
4. **Mute**
5. **Input / Source**
6. **Menu / Home**
7. **Navigation arrows + OK/Select**
8. **Number pad** (0--9)
9. **Any device-specific buttons** (Netflix, settings, etc.)

After each capture, save it with a descriptive name:

```
TV_Power
TV_VolUp
TV_VolDown
TV_Mute
TV_Input
Soundbar_Power
Soundbar_VolUp
AC_Power
AC_TempUp
AC_TempDown
AC_Mode
```

**Step 4 --- Verify Each Code**

After capturing, immediately test by replaying:

```
Main Menu → IR → Saved → [select signal] → Send
```

Point the T-Embed at the target device and send. Verify the device responds correctly. If it does not:

- Re-capture at closer range
- Ensure the remote has fresh batteries
- Try capturing multiple times (some protocols send different data on alternating presses)

**Step 5 --- Organize Into Profiles**

Group your saved codes by device. Most firmware supports folders or profile organization. Create groups like:

```
/ir_codes/
├── living_room_tv/
├── bedroom_tv/
├── soundbar/
├── air_conditioner/
└── led_strips/
```

### Expected Results

- You should successfully capture 90--95% of IR codes from modern remotes
- NEC protocol devices (most common) will capture reliably every time
- Some protocols may show as "RAW" if the firmware does not recognize the encoding --- these still replay correctly
- Air conditioner remotes are the most complex (they send full state packets, not single commands) and may require multiple captures

### Extensions

- Add codes from the built-in TV-B-Gone database for devices you encounter outside your home
- Create "scene" macros that send multiple codes in sequence (e.g., "Movie Mode" = TV On + Soundbar On + Input HDMI2 + Lights Dim)
- Share your IR code library with the community
- Experiment with raw IR capture for devices that use unusual protocols

---

## Project 3: Weather Station Decoder
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Most consumer weather stations transmit temperature, humidity, wind speed, and rain gauge data on 433 MHz using simple, unencrypted protocols. In this project, you will capture these transmissions, decode the protocol, and build a live weather display on the T-Embed's TFT screen.

### What You'll Learn

- RF protocol analysis and reverse engineering
- Binary data decoding (Manchester encoding, pulse width modulation)
- Using URH (Universal Radio Hacker) for signal analysis
- Cross-referencing with rtl_433's protocol database
- How weather sensors encode and transmit data

### Materials Needed

- T-Embed CC1101 Plus with antenna
- A wireless weather station (most transmit on 433.92 MHz)
- Computer with [URH](https://github.com/jopohl/urh) installed
- (Optional) RTL-SDR dongle for cross-verification with rtl_433

### Step-by-Step Instructions

**Step 1 --- Capture the Weather Station Signal**

Navigate to Sub-GHz capture and set the frequency to 433.92 MHz with OOK/ASK modulation. Most weather stations transmit every 30--60 seconds. Wait for a capture.

```
Main Menu → RF → Sub-GHz → Read
Frequency: 433.92 MHz
Modulation: AM (OOK)
```

Capture at least 5 separate transmissions. Save each one.

**Step 2 --- Analyze with URH**

Transfer the captured `.sub` files to your computer. Open them in URH:

1. Load the signal file
2. Switch to the **Analysis** tab
3. URH will attempt to auto-detect the modulation and bit encoding
4. Look for repeating patterns --- weather data is typically sent 2--3 times per transmission for reliability

**Step 3 --- Identify the Protocol Structure**

Most 433 MHz weather stations use a structure like this:

```
[Preamble] [Sync] [ID] [Channel] [Battery] [Temp] [Humidity] [Checksum]
  8 bits    4 bits  8b    2b        1b       12b      8b         8b
```

Compare your decoded bits against the known temperature and humidity values. If the weather station reads 72.5 degrees F and 45% humidity, look for:

- Temperature: Often transmitted as (value + offset) in tenths of a degree Celsius
- Humidity: Usually a direct percentage value
- ID: A fixed device identifier that changes when you reset the sensor

{: .tip }
> Cross-reference your findings with the [rtl_433 protocol list](https://github.com/merbanan/rtl_433). There are 200+ documented weather station protocols. Yours is probably already documented, which makes decoding much faster.

**Step 4 --- Build the Decoder Logic**

Once you understand the bit layout, write decoding logic:

```c
// Example: Decode temperature from captured bits
// Assuming bits 24-35 contain temperature as (value + 400) in tenths of °F
int raw_temp = (bits[24] << 11) | (bits[25] << 10) | ... ;
float temp_f = (raw_temp - 400) / 10.0;
float temp_c = (temp_f - 32.0) * 5.0 / 9.0;
```

**Step 5 --- Display Live Data**

If you are running custom firmware or have development access, create a TFT display layout:

```
┌──────────────────────┐
│   Weather Station    │
│                      │
│   Temp: 22.5°C       │
│   Humidity: 45%      │
│                      │
│   Last update: 30s   │
│   Signal: Strong     │
└──────────────────────┘
```

### Expected Results

- You will receive clear signal captures every 30--60 seconds
- Most consumer weather stations use well-documented protocols
- Temperature values will match your weather station's display within 0.1--0.5 degrees
- You may also pick up your neighbors' weather stations (each has a unique ID)

### Extensions

- Decode additional sensor data: wind speed, wind direction, rain gauge, UV index
- Build a logging system that records readings over time
- Compare multiple weather stations in your area
- Use the decoded protocol to build your own weather sensor transmitter (TX on ISM bands only)

---

## Project 4: Home Security RF Audit
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

This is a real penetration test applied to your own home. You will systematically test every RF-based security device for vulnerabilities, document your findings, and create a professional-grade security assessment report with remediation recommendations.

### What You'll Learn

- Penetration testing methodology applied to RF devices
- The difference between fixed codes and rolling codes (and why it matters)
- Vulnerability assessment and risk scoring
- Professional security reporting

### Materials Needed

- T-Embed CC1101 Plus with antenna
- Access to all RF security devices in your home
- Notebook or document for findings
- (Optional) Second person to operate devices while you capture

{: .danger }
> **Test only devices you own.** Capturing and replaying signals from your neighbor's garage door, gate, or alarm system is a federal crime under the CFAA and violates FCC regulations. No exceptions.

### Step-by-Step Instructions

**Step 1 --- Inventory Your RF Security Devices**

Document every RF-controlled security device in your home:

| Device | Location | Frequency | Brand/Model | Age |
|:-------|:---------|:----------|:------------|:----|
| Garage door opener | Garage | TBD | Chamberlain B2405 | 2019 |
| Gate remote | Front gate | TBD | LiftMaster | 2020 |
| Alarm system sensors | Windows/Doors | TBD | SimpliSafe | 2021 |
| Wireless doorbell | Front door | TBD | Ring (RF, not WiFi) | 2022 |
| Car key fob | Driveway | TBD | Toyota | 2023 |

**Step 2 --- Capture Signals from Each Device**

For each device, capture multiple transmissions:

1. Set the frequency analyzer to find the exact frequency
2. Switch to Sub-GHz capture mode
3. Activate the device (press the remote, trigger the sensor)
4. Capture at least 3 separate activations
5. Save each capture with a descriptive filename

**Step 3 --- Analyze for Fixed vs. Rolling Codes**

Compare your multiple captures of the same device:

**Fixed Code Indicators:**
- All captures produce identical or nearly identical signal data
- Signal structure is simple (short burst, simple encoding)
- Device is older (pre-2005 era designs)

**Rolling Code Indicators:**
- Each capture produces different data
- Signal is longer and more complex
- Device uses KeeLoq, AUT64, or similar algorithm
- Modern garage door openers (Chamberlain Security+ 2.0, etc.)

```
Fixed Code:   [A7 B3 C2 A7 B3 C2 A7 B3 C2]  ← Same every time
Rolling Code: [A7 B3 C2] [D1 E4 F5] [G8 H2 I9]  ← Different each time
```

**Step 4 --- Test Replay Vulnerability**

For devices that appear to use fixed codes:

1. Capture a signal
2. Wait 30 seconds
3. Replay the captured signal
4. Observe whether the device responds

{: .warning }
> If a device responds to a replayed signal, it is vulnerable to replay attacks. This means anyone with a $25 device could capture and replay the signal to activate that device. Document this as a **Critical** vulnerability.

**Step 5 --- Create Your Vulnerability Report**

Use this template for each device:

```
DEVICE: [Name]
LOCATION: [Where]
FREQUENCY: [MHz]
MODULATION: [OOK/FSK]
CODE TYPE: Fixed / Rolling / Encrypted
REPLAY VULNERABLE: Yes / No
RISK LEVEL: Critical / High / Medium / Low

FINDINGS:
[Description of what you discovered]

RECOMMENDATION:
[What to do about it]
```

**Risk Level Definitions:**

| Risk | Meaning | Example |
|:-----|:--------|:--------|
| **Critical** | Fixed code, replay works, secures physical access | Garage door with fixed code |
| **High** | Fixed code but limited impact | Doorbell with fixed code |
| **Medium** | Rolling code but other weaknesses exist | Older rolling code with known attacks |
| **Low** | Rolling code, properly implemented | Modern Security+ 2.0 garage opener |

**Step 6 --- Remediation Recommendations**

For each vulnerable device, provide actionable upgrades:

- **Fixed-code garage door**: Replace with Security+ 2.0 or myQ-compatible opener ($150--300)
- **Fixed-code gate remote**: Upgrade to rolling-code system or add secondary authentication
- **Unencrypted alarm sensors**: Upgrade to encrypted sensor protocol (e.g., SimpliSafe Gen 3)
- **Old car key fob**: Faraday pouch for storage, dealer reprogramming for newer protocol

### Expected Results

- Older devices (10+ years) are very likely to be fixed-code and vulnerable
- Modern garage door openers from major brands should use rolling codes
- Alarm system sensors vary widely --- some expensive systems still use fixed codes
- Car key fobs from 2015+ almost always use rolling codes
- You will likely find at least one vulnerable device

### Extensions

- Research specific attacks against rolling-code systems you find (RollJam, RollBack)
- Test at different distances to determine effective attack range
- Monitor for signal jamming (someone attempting to block your devices)
- Set up continuous monitoring to detect unauthorized RF signals near your security devices

---

## Project 5: Bluetooth Device Tracker / Counter
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Build a portable BLE scanner that counts unique Bluetooth devices, identifies device types, tracks devices appearing and disappearing over time, and displays real-time statistics on the TFT. This is useful for foot traffic analysis, crowd density estimation, and understanding the BLE device landscape.

### What You'll Learn

- BLE advertising and scanning fundamentals
- MAC address types (public, random, resolvable)
- Device identification by advertisement data
- Data collection and visualization on embedded displays

### Materials Needed

- T-Embed CC1101 Plus (battery-powered for portability)
- External battery pack (JST connector)
- (Optional) Computer for post-analysis

### Step-by-Step Instructions

**Step 1 --- Understand BLE Advertisements**

Every Bluetooth device periodically broadcasts advertisement packets. These contain:

- MAC address (may be randomized)
- Device name (sometimes)
- Service UUIDs (indicate device type)
- Manufacturer data (company identifier)
- TX power level (allows distance estimation)

{: .note }
> Modern devices (iOS 14+, Android 10+) use **MAC address randomization**, which means the MAC address changes periodically. This makes exact device counting harder but not impossible --- manufacturer data and service UUIDs remain consistent.

**Step 2 --- Run a Passive BLE Scan**

Navigate to:

```
Main Menu → BLE → Scan
```

Let the scan run for 60 seconds. The T-Embed will discover all advertising BLE devices in range (typically 10--30 meters indoors, up to 100 meters outdoors with line of sight).

**Step 3 --- Categorize Discovered Devices**

Sort discovered devices by type using their advertisement data:

| Category | Identifier | Examples |
|:---------|:-----------|:---------|
| Phones | Apple (0x004C) or Google manufacturer data | iPhones, Pixels, Samsung |
| Wearables | Heart Rate Service UUID | Fitbit, Apple Watch, Garmin |
| Audio | Audio service UUIDs | AirPods, Bluetooth speakers |
| Beacons | iBeacon or Eddystone format | Retail beacons, Tile trackers |
| Smart Home | Specific service UUIDs | Smart locks, sensors, bulbs |
| Unknown | No identifiable data | Randomized, minimal advertisements |

**Step 4 --- Build a Counting Display**

Create a TFT display layout showing real-time counts:

```
┌──────────────────────┐
│  BLE Device Counter  │
│                      │
│  Total:     47       │
│  Phones:    23       │
│  Wearables:  8       │
│  Audio:      5       │
│  Beacons:    3       │
│  Other:      8       │
│                      │
│  New/min:    4       │
│  Lost/min:   2       │
│  Scan time: 12:34    │
└──────────────────────┘
```

**Step 5 --- Track Device Persistence**

Monitor which devices stay in range vs. pass through:

- **Stationary devices**: Same MAC/manufacturer data seen consistently (your own devices, nearby smart home equipment)
- **Transient devices**: Appear and disappear within minutes (people walking by, cars passing)
- **Periodic devices**: Appear at regular intervals (devices with long advertisement intervals)

Log entries with timestamps to build a traffic pattern:

```
12:00 - 12:05: 15 unique devices (3 new, 1 lost)
12:05 - 12:10: 17 unique devices (4 new, 2 lost)
12:10 - 12:15: 14 unique devices (1 new, 4 lost)
```

### Expected Results

- In a typical home, you will find 10--30 BLE devices (many you did not know about)
- Near a street or public area, the count will be significantly higher
- Apple devices will dominate in most US locations (iPhones, AirPods, Apple Watch)
- You will discover BLE devices you forgot about (old Fitbit in a drawer, smart light bulbs)
- MAC randomization will cause some overcounting --- expect 10--20% inflation

### Extensions

- Graph device count over 24 hours to see traffic patterns
- Estimate crowd size in public spaces (with appropriate consent/signage)
- Detect specific device types (e.g., count only Apple devices using manufacturer data)
- Build an alert system that notifies you when a new persistent device appears (potential security monitoring)
- Export data to CSV for analysis in a spreadsheet or Python

---

## Project 6: Portable WiFi Security Auditor
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Build a portable WiFi security assessment workflow that scans all nearby networks, identifies security weaknesses, checks for common misconfigurations, monitors for rogue access points, and generates a clean report. Run this in your own home or office to understand your wireless security posture.

### What You'll Learn

- WiFi security fundamentals (WEP, WPA, WPA2, WPA3, OWE)
- Network reconnaissance techniques
- Common WiFi misconfigurations
- Rogue AP detection methodology
- Security assessment reporting

### Materials Needed

- T-Embed CC1101 Plus (WiFi uses the ESP32-S3 radio, not the CC1101)
- Charged battery for portable scanning
- Computer for report generation
- (Optional) Known-good network list for rogue AP comparison

{: .danger }
> **Scan only networks you own or have written authorization to audit.** Connecting to, deauthenticating, or interfering with networks you do not own is illegal under the CFAA and potentially under wiretapping laws.

### Step-by-Step Instructions

**Step 1 --- Perform a Full Network Scan**

Navigate to:

```
Main Menu → WiFi → Scan
```

Let the scan complete. It will discover all 2.4 GHz networks in range. For each network, note:

- SSID (network name)
- BSSID (access point MAC address)
- Channel
- Signal strength (RSSI)
- Encryption type (Open, WEP, WPA, WPA2, WPA3)
- Hidden networks (blank SSID)

**Step 2 --- Assess Each Network's Security**

Score each of your networks against this checklist:

| Check | Secure | Vulnerable |
|:------|:-------|:-----------|
| **Encryption** | WPA3 or WPA2 | WEP or Open |
| **SSID** | Custom name | Default (e.g., "NETGEAR-5G") |
| **Hidden** | Optional | Not a security measure by itself |
| **WPS** | Disabled | Enabled (PIN brute-force risk) |
| **Channel** | Non-overlapping (1, 6, 11) | Overlapping with neighbors |
| **Signal** | Appropriate for coverage area | Overpowered (leaks far outside) |

{: .warning }
> **WEP is completely broken.** If any of your networks still use WEP, upgrade immediately. WEP can be cracked in minutes with readily available tools.

**Step 3 --- Check for Default SSIDs**

Default SSIDs indicate the router was never properly configured:

- `NETGEAR-xxxx`, `Linksys-xxxx`, `TP-LINK_xxxx` --- likely still using default admin password
- `xfinitywifi`, `ATT-xxxx` --- ISP defaults, may have known vulnerabilities
- `DIRECT-xxxx` --- WiFi Direct, often has weak/no authentication

If any of your networks have default SSIDs, change them and update the admin password.

**Step 4 --- Monitor for Rogue Access Points**

A rogue AP is an unauthorized access point on your network or an evil twin mimicking your network. To detect them:

1. Create a baseline list of your known access points (SSID + BSSID + Channel)
2. Scan periodically and compare
3. Flag any new access points that match your SSID but have a different BSSID
4. Flag any unknown access points on your network's channels

```
BASELINE:
HomeNetwork    AA:BB:CC:DD:EE:FF    Channel 6
HomeNetwork_5G AA:BB:CC:DD:EE:00    Channel 36

ALERT - NEW AP DETECTED:
HomeNetwork    11:22:33:44:55:66    Channel 6    ← ROGUE AP!
```

**Step 5 --- Test Deauthentication Resilience (Your Own Network)**

Modern WPA3 networks with Protected Management Frames (PMF/802.11w) are immune to deauth attacks. Test your own network:

1. Connect a test device to your WiFi
2. Send a deauth frame targeting your own test device
3. If the device disconnects, your network is vulnerable to deauth
4. If the device stays connected, PMF is working

{: .tip }
> To enable PMF, update your router firmware and enable WPA3 or WPA2/WPA3 transition mode. Most routers manufactured after 2020 support this.

**Step 6 --- Generate Your Security Report**

```
═══════════════════════════════════════
       WiFi Security Audit Report
       [Your Name] — [Date]
       Location: [Your Home/Office]
═══════════════════════════════════════

NETWORK: HomeNetwork
  Encryption:  WPA2-PSK     [PASS - but recommend WPA3]
  SSID:        Custom       [PASS]
  WPS:         Disabled     [PASS]
  PMF:         Not enabled  [FAIL - vulnerable to deauth]
  Channel:     6            [OK]
  Admin login: Changed      [PASS]

OVERALL RISK: MEDIUM
RECOMMENDATIONS:
  1. Enable WPA3 transition mode
  2. Enable Protected Management Frames
  3. Update router firmware to latest version
```

### Expected Results

- Most home networks will show WPA2 (acceptable but improvable)
- You will likely find neighboring networks with poor security (do not test them)
- Default SSIDs are extremely common and indicate poor security hygiene
- PMF is rarely enabled on home networks, making them vulnerable to deauth
- The exercise will reveal security improvements you can make immediately

### Extensions

- Schedule weekly automated scans and compare against your baseline
- Test all IoT devices for WiFi security (smart bulbs, cameras, etc. often use weak encryption)
- Build a persistent rogue AP detector that alerts you in real time
- Audit guest network isolation (can guest devices reach your main network?)

---

## Project 7: Custom Sub-GHz Remote Control System
{: .d-inline-block }

Advanced
{: .label .label-red }

### Overview

Design and implement a custom RF communication protocol from scratch. Build a transmitter on the T-Embed and a receiver on a second ESP32. Create a multi-button remote with proper encoding, error detection, and basic encryption. This project teaches you how RF protocols actually work at the lowest level.

### What You'll Learn

- RF protocol design principles
- Manchester encoding and decoding
- Packet structure design (preamble, sync word, payload, checksum)
- Basic encryption for RF communications
- Two-device communication systems
- Error detection with CRC

### Materials Needed

- T-Embed CC1101 Plus (transmitter)
- Second ESP32 board with CC1101 module or 433 MHz receiver module (receiver)
- Breadboard and jumper wires
- LEDs or relay module for receiver output
- Arduino IDE or PlatformIO

{: .note }
> You can also use a second T-Embed CC1101 Plus as the receiver if you have one. The code is the same --- just different firmware roles.

### Step-by-Step Instructions

**Step 1 --- Design Your Protocol**

Before writing any code, design the packet structure on paper:

```
┌──────────┬──────────┬────────┬─────────┬─────────┬──────────┐
│ Preamble │ Sync Word│ Length │ Address │ Payload │ CRC-16   │
│ 4 bytes  │ 2 bytes  │ 1 byte │ 1 byte  │ N bytes │ 2 bytes  │
└──────────┴──────────┴────────┴─────────┴─────────┴──────────┘

Preamble:  0xAA 0xAA 0xAA 0xAA  (alternating bits for clock sync)
Sync Word: 0x2D 0xD4             (unique pattern to mark packet start)
Length:    Total payload length
Address:   Device address (0x01-0xFE, 0xFF = broadcast)
Payload:   Your data (button press, command, etc.)
CRC-16:    Error detection checksum
```

**Step 2 --- Define Your Command Set**

```c
// Command definitions
#define CMD_BUTTON_1    0x01  // Button 1 pressed
#define CMD_BUTTON_2    0x02  // Button 2 pressed
#define CMD_BUTTON_3    0x03  // Button 3 pressed
#define CMD_BUTTON_4    0x04  // Button 4 pressed
#define CMD_ALL_OFF     0x10  // All outputs off
#define CMD_ALL_ON      0x11  // All outputs on
#define CMD_TOGGLE      0x20  // Toggle specific output
#define CMD_STATUS_REQ  0x30  // Request status from receiver
#define CMD_PING        0xFF  // Keepalive ping
```

**Step 3 --- Implement Manchester Encoding**

Manchester encoding ensures clock synchronization by guaranteeing a transition in every bit period:

```c
// Manchester encode: each data bit becomes two bits
// Logic 0 → 01 (low-to-high transition)
// Logic 1 → 10 (high-to-low transition)

uint16_t manchester_encode(uint8_t byte) {
    uint16_t encoded = 0;
    for (int i = 7; i >= 0; i--) {
        encoded <<= 2;
        if (byte & (1 << i)) {
            encoded |= 0b10;  // 1 → high-low
        } else {
            encoded |= 0b01;  // 0 → low-high
        }
    }
    return encoded;
}
```

**Step 4 --- Add Basic Encryption**

Add a simple XOR cipher with a rolling counter to prevent replay attacks:

```c
// Simple encryption: XOR with key + rolling counter
uint8_t encrypt_byte(uint8_t data, uint8_t key, uint8_t counter) {
    return data ^ key ^ counter;
}

// Packet includes a counter field that increments with each transmission
// Receiver rejects packets with counter <= last received counter
```

{: .warning }
> XOR encryption is not cryptographically secure. For a real security application, use AES-128 (supported by the ESP32-S3 hardware). This project uses XOR for simplicity and learning --- never rely on it for actual security.

**Step 5 --- Write the Transmitter Firmware**

```c
#include <SPI.h>

// CC1101 configuration for custom protocol
void setupCC1101() {
    // Frequency: 433.92 MHz (ISM band)
    // Modulation: 2-FSK
    // Data rate: 4.8 kbps (reliable for beginners)
    // TX power: +10 dBm
    // Packet format: Variable length with CRC
}

void sendCommand(uint8_t address, uint8_t command) {
    uint8_t packet[8];
    packet[0] = address;
    packet[1] = command;
    packet[2] = counter++;
    packet[3] = encrypt_byte(command, SECRET_KEY, counter);

    // Calculate CRC-16
    uint16_t crc = crc16(packet, 4);
    packet[4] = crc >> 8;
    packet[5] = crc & 0xFF;

    // Transmit via CC1101
    cc1101_sendPacket(packet, 6);
}
```

**Step 6 --- Write the Receiver Firmware**

```c
void loop() {
    if (cc1101_packetAvailable()) {
        uint8_t packet[8];
        uint8_t len = cc1101_readPacket(packet, sizeof(packet));

        // Verify CRC
        if (!verifyCRC(packet, len)) {
            Serial.println("CRC error - packet dropped");
            return;
        }

        // Verify counter (anti-replay)
        if (packet[2] <= lastCounter) {
            Serial.println("Replay detected - packet dropped");
            return;
        }
        lastCounter = packet[2];

        // Decrypt and execute command
        uint8_t command = decrypt_byte(packet[3], SECRET_KEY, packet[2]);
        executeCommand(command);
    }
}

void executeCommand(uint8_t cmd) {
    switch (cmd) {
        case CMD_BUTTON_1: digitalWrite(LED_1, !digitalRead(LED_1)); break;
        case CMD_BUTTON_2: digitalWrite(LED_2, !digitalRead(LED_2)); break;
        case CMD_ALL_OFF:  allOutputsOff(); break;
        case CMD_ALL_ON:   allOutputsOn(); break;
    }
}
```

**Step 7 --- Test and Debug**

1. Flash the transmitter firmware to the T-Embed
2. Flash the receiver firmware to the second ESP32
3. Open Serial Monitor on the receiver
4. Press buttons on the T-Embed and verify the receiver responds
5. Test at increasing distances to find your range limit
6. Verify replay protection by capturing and replaying a signal --- the receiver should reject it

### Expected Results

- Reliable communication up to 50--100 meters in open air at +10 dBm
- 10--30 meter range indoors through walls
- Replay protection will successfully reject captured-and-replayed signals
- CRC will catch corrupted packets (test by partially obstructing the antenna)
- You will gain a deep understanding of how every RF remote control protocol works

### Extensions

- Add bidirectional communication (receiver sends acknowledgments)
- Implement AES-128 encryption using ESP32-S3 hardware acceleration
- Add frequency hopping for interference resistance
- Build a protocol analyzer that displays your custom packets on the TFT
- Increase the data rate for faster response time

---

## Project 8: Signal Replay Defense Tester
{: .d-inline-block }

Advanced
{: .label .label-red }

### Overview

Build a comprehensive tool that systematically tests your fixed-code devices against replay attacks, documents every vulnerability, and generates a remediation report with specific product upgrade recommendations. This is a formalized, repeatable security testing methodology.

### What You'll Learn

- Formalized vulnerability testing methodology
- Signal capture, storage, and replay techniques
- Vulnerability documentation and reporting
- Risk assessment and remediation planning
- The real-world security impact of fixed-code systems

### Materials Needed

- T-Embed CC1101 Plus with antenna
- All your RF-controlled devices (garage doors, gates, doorbells, etc.)
- Computer for report generation
- SD card for signal storage (if supported by firmware)

{: .danger }
> This project is exclusively for testing **your own devices**. Testing devices belonging to others --- neighbors, businesses, public infrastructure --- is a federal crime. Always have documentation proving ownership before testing.

### Step-by-Step Instructions

**Step 1 --- Create a Test Plan**

Before touching the T-Embed, document your test plan:

```
═══════════════════════════════════════
    RF Replay Defense Test Plan
    Tester: [Your Name]
    Date: [Date]
    Location: [Your Home Address]
    Authorization: Self (homeowner)
═══════════════════════════════════════

DEVICES UNDER TEST:
1. Garage door opener — [Brand/Model]
2. Wireless doorbell — [Brand/Model]
3. Fan remote — [Brand/Model]
4. LED strip remote — [Brand/Model]

TEST METHODOLOGY:
- Capture 3 signals per device
- Compare signals for fixed vs. rolling code
- Attempt replay for fixed-code devices
- Document all results
- Generate remediation report
```

**Step 2 --- Systematic Signal Capture**

For each device, capture three separate activations:

```
Capture 1: Press button → Save as [device]_capture_1
Wait 10 seconds
Capture 2: Press button → Save as [device]_capture_2
Wait 10 seconds
Capture 3: Press button → Save as [device]_capture_3
```

**Step 3 --- Compare Captures**

Analyze the three captures for each device:

| Result | Meaning | Vulnerability |
|:-------|:--------|:-------------|
| All 3 identical | Fixed code | **Critical** --- trivially replayable |
| All 3 different, structured | Rolling code | Low --- resistant to simple replay |
| All 3 different, random-looking | Encrypted rolling code | Very Low --- properly secured |
| 2 identical, 1 different | Hybrid or alternating code | **High** --- partially replayable |

**Step 4 --- Replay Test (Fixed-Code Devices Only)**

For devices with identical captures:

1. Move to normal operating distance from the device
2. Load the saved capture
3. Transmit the replay
4. Document whether the device activates

Record the results:

```
DEVICE: Ceiling fan remote
REPLAY RESULT: SUCCESS — Fan turned on
EFFECTIVE RANGE: 8 meters (same room)
RELIABILITY: 3/3 attempts successful
```

**Step 5 --- Generate Remediation Report**

For each vulnerable device, research and recommend specific replacements:

```
═══════════════════════════════════════
    VULNERABILITY: Garage Door Opener
    Risk Level: CRITICAL

    CURRENT STATE:
    - Fixed code on 315 MHz
    - 100% replay success rate
    - Any $25 device can clone access

    IMPACT:
    - Unauthorized physical access to garage
    - Access to home interior (if garage connects)
    - Vehicle theft if garage stores vehicles

    REMEDIATION OPTIONS:
    1. Replace opener: Chamberlain B6753T
       - Security+ 2.0 rolling code
       - myQ smart home integration
       - Cost: ~$280
       - Difficulty: DIY installation

    2. Add secondary security:
       - Deadbolt on garage-to-home door
       - Motion-activated camera in garage
       - Cost: ~$100-150
       - Difficulty: Easy

    PRIORITY: Immediate
═══════════════════════════════════════
```

### Expected Results

- Consumer devices like fan remotes, LED strip controllers, and wireless outlets almost always use fixed codes and will be replayable
- Garage door openers manufactured after 2005 usually use rolling codes (but verify --- some budget brands still use fixed codes)
- Car key fobs from the last 10 years should use rolling codes
- You will likely find 2--5 vulnerable devices in a typical home
- Non-security devices (fan remotes, LED strips) are low risk despite being vulnerable

### Extensions

- Test effective replay range for each device (how far away can you successfully replay?)
- Time-based analysis (do captured codes expire? Some systems invalidate old codes after a window)
- Build an automated scanner that tests for fixed codes across common frequencies
- Create a "defense grade" scoring system for your home's overall RF security

---

## Project 9: Multi-Band Spectrum Monitor
{: .d-inline-block }

Advanced
{: .label .label-red }

### Overview

Create a real-time spectrum display on the T-Embed's TFT that shows signal activity across the full CC1101 frequency range. Build a color-coded waterfall display with peak hold, frequency logging, and battery-powered portable operation. You are essentially building a pocket spectrum analyzer.

### What You'll Learn

- Spectrum analysis principles and techniques
- RSSI (Received Signal Strength Indicator) measurement with the CC1101
- Real-time data visualization on embedded displays
- Sweep scanning across frequency ranges
- Waterfall display rendering
- Power management for portable operation

### Materials Needed

- T-Embed CC1101 Plus with antenna
- External battery (JST connector, 3.7V LiPo recommended)
- Arduino IDE or PlatformIO
- (Optional) RTL-SDR for comparison and verification

### Step-by-Step Instructions

**Step 1 --- Understand CC1101 RSSI**

The CC1101 provides an RSSI (Received Signal Strength Indicator) register that reports the power level of any signal at the currently tuned frequency. By rapidly sweeping across frequencies and reading RSSI at each step, you can build a spectrum display.

Key specifications:
- RSSI range: approximately -110 dBm to 0 dBm
- RSSI resolution: 0.5 dBm per step
- Frequency resolution: depends on sweep step size
- Sweep speed: limited by CC1101 settling time (~800 microseconds per frequency change)

**Step 2 --- Implement Frequency Sweep**

```c
#define FREQ_START  430000000  // 430 MHz
#define FREQ_END    440000000  // 440 MHz
#define FREQ_STEP     50000    // 50 kHz steps
#define NUM_STEPS   ((FREQ_END - FREQ_START) / FREQ_STEP)  // 200 steps

int8_t rssi_values[NUM_STEPS];

void sweepBand() {
    for (int i = 0; i < NUM_STEPS; i++) {
        uint32_t freq = FREQ_START + (i * FREQ_STEP);
        cc1101_setFrequency(freq);
        delayMicroseconds(800);  // Wait for PLL to settle
        rssi_values[i] = cc1101_readRSSI();
    }
}
```

**Step 3 --- Build the Spectrum Display**

Map RSSI values to colors and pixel heights on the 170x320 TFT:

```c
// Color mapping for signal strength
// -110 dBm (noise floor) → dark blue
// -90 dBm (weak signal) → green
// -70 dBm (medium signal) → yellow
// -50 dBm (strong signal) → red

uint16_t rssiToColor(int8_t rssi) {
    if (rssi < -90) return TFT_NAVY;
    if (rssi < -80) return TFT_BLUE;
    if (rssi < -70) return TFT_GREEN;
    if (rssi < -60) return TFT_YELLOW;
    if (rssi < -50) return TFT_ORANGE;
    return TFT_RED;
}

void drawSpectrum() {
    int bar_width = 320 / NUM_STEPS;  // Width of each frequency bin
    for (int i = 0; i < NUM_STEPS; i++) {
        int height = map(rssi_values[i], -110, 0, 0, 150);
        int x = i * bar_width;
        uint16_t color = rssiToColor(rssi_values[i]);

        tft.fillRect(x, 170 - height, bar_width, height, color);
        tft.fillRect(x, 0, bar_width, 170 - height, TFT_BLACK);
    }
}
```

**Step 4 --- Add Waterfall Display**

A waterfall display scrolls historical spectrum data downward, showing signal activity over time:

```c
// Waterfall buffer: each row is one sweep
#define WATERFALL_HEIGHT 80
uint16_t waterfall[WATERFALL_HEIGHT][NUM_STEPS];

void updateWaterfall() {
    // Shift all rows down by one
    memmove(&waterfall[1], &waterfall[0],
            sizeof(uint16_t) * NUM_STEPS * (WATERFALL_HEIGHT - 1));

    // New sweep data becomes top row
    for (int i = 0; i < NUM_STEPS; i++) {
        waterfall[0][i] = rssiToColor(rssi_values[i]);
    }

    // Draw waterfall section below spectrum
    for (int y = 0; y < WATERFALL_HEIGHT; y++) {
        for (int x = 0; x < NUM_STEPS; x++) {
            tft.drawPixel(x * bar_width, 170 + y, waterfall[y][x]);
        }
    }
}
```

**Step 5 --- Add Peak Hold**

Peak hold shows the maximum signal level seen at each frequency:

```c
int8_t peak_rssi[NUM_STEPS];
uint32_t peak_time[NUM_STEPS];

void updatePeakHold() {
    uint32_t now = millis();
    for (int i = 0; i < NUM_STEPS; i++) {
        if (rssi_values[i] > peak_rssi[i]) {
            peak_rssi[i] = rssi_values[i];
            peak_time[i] = now;
        }
        // Decay peak after 3 seconds
        if (now - peak_time[i] > 3000) {
            peak_rssi[i] = rssi_values[i];
        }
    }
}

// Draw peak markers as white dots above the spectrum bars
void drawPeakMarkers() {
    for (int i = 0; i < NUM_STEPS; i++) {
        int peak_y = map(peak_rssi[i], -110, 0, 170, 20);
        tft.drawPixel(i * bar_width + bar_width/2, peak_y, TFT_WHITE);
    }
}
```

**Step 6 --- Add Navigation Controls**

Use the rotary encoder to:

- **Rotate**: Scroll center frequency left/right
- **Press**: Cycle between band presets (300--348, 387--464, 779--928 MHz)
- **Long press**: Toggle between spectrum view and waterfall view

Add an on-screen display showing:

```
┌──────────────────────────────────────────┐
│  430.0 MHz ────────────── 440.0 MHz      │
│  ▁▂▃▅▇█▇▅▃▂▁▁▁▂▅▇▅▃▁▁▁▁▁▂▃▂▁▁▁▁▁▁▁▁   │ ← Spectrum
│  · · · · ·   · · ·                 ·     │ ← Peak hold
│──────────────────────────────────────────│
│ ░░▓▓░░░░░▒▒▓▓▒▒░░░░░░░░░░░░░░░░░░░░░░  │
│ ░░░▒▒░░░░░▒▒▓▓▒▒░░░░░░░░░░░░░░░░░░░░░  │ ← Waterfall
│ ░░░░░░░░░░░▒▒▒▒▒░░░░░░░░░░░░░░░░░░░░░  │
│──────────────────────────────────────────│
│ Center: 435.0 MHz  Span: 10 MHz  -72dBm │
└──────────────────────────────────────────┘
```

### Expected Results

- The spectrum display will update at approximately 2--5 sweeps per second (depending on step count and settling time)
- Strong nearby signals (weather stations, doorbells) will appear as distinct peaks rising above the noise floor
- The noise floor will typically sit around -100 to -95 dBm
- The waterfall display will show intermittent transmissions as horizontal streaks
- Battery life with continuous scanning will be approximately 2--4 hours with a 1000 mAh LiPo

### Extensions

- Add dBm scale markers and frequency labels to the display
- Implement zoom (narrow the frequency span for finer resolution)
- Add signal triggering (capture when a signal exceeds a threshold)
- Log frequency/time/RSSI data to a file for post-analysis
- Build a "signal hunter" mode that auto-tunes to the strongest detected signal
- Compare your results with an RTL-SDR + SDR# waterfall for verification

---

## Project 10: Integrated Pentest Multi-Tool
{: .d-inline-block }

Expert
{: .label .label-purple }

### Overview

Combine every capability of the T-Embed --- Sub-GHz, WiFi, BLE, IR, and BadUSB --- into a cohesive, menu-driven penetration testing toolkit with a polished interface, organized logging, and professional output. You are building your own Flipper Zero from scratch, customized exactly to your needs.

### What You'll Learn

- System architecture for multi-function embedded tools
- Menu system design and user interface development on embedded displays
- Integrating multiple radio systems into one workflow
- Data logging and report generation on embedded systems
- Professional penetration testing workflow
- Full-stack embedded development

### Materials Needed

- T-Embed CC1101 Plus with antenna
- External battery (1000+ mAh LiPo recommended)
- MicroSD card module (connected via SPI/GPIO) for logging
- Arduino IDE or PlatformIO with all required libraries
- Computer for firmware development

{: .note }
> This is a significant firmware development project. Expect 40--80 hours of development time to build a polished, fully-featured tool. You can start with a subset of features and add modules incrementally.

### Step-by-Step Instructions

**Step 1 --- Architecture Design**

Design a modular architecture where each capability is an independent module:

```
┌──────────────────────────────────────────┐
│                 Main Menu                 │
├──────────┬──────────┬──────────┬─────────┤
│ RF Recon │ WiFi     │ BLE      │ IR      │
│ Module   │ Module   │ Module   │ Module  │
├──────────┼──────────┼──────────┼─────────┤
│ BadUSB   │ Spectrum │ Settings │ Logs    │
│ Module   │ Analyzer │          │         │
└──────────┴──────────┴──────────┴─────────┘
        │           │           │
   ┌────▼───────────▼───────────▼────┐
   │        Shared Services           │
   │  • Display Manager               │
   │  • Input Handler (encoder)        │
   │  • File System (SD logging)       │
   │  • Battery Monitor                │
   │  • Configuration Store            │
   └──────────────────────────────────┘
```

**Step 2 --- Build the Menu System**

Create a reusable menu engine driven by the rotary encoder:

```c
typedef struct {
    const char* label;
    void (*action)(void);
    const uint16_t* icon;  // 16x16 bitmap icon
} MenuItem;

typedef struct {
    const char* title;
    MenuItem* items;
    uint8_t item_count;
    uint8_t selected;
    uint8_t scroll_offset;
} Menu;

// Main menu definition
MenuItem main_items[] = {
    {"RF Recon",      rfReconMode,     icon_rf},
    {"WiFi Audit",    wifiAuditMode,   icon_wifi},
    {"BLE Scanner",   bleScanMode,     icon_ble},
    {"IR Remote",     irRemoteMode,    icon_ir},
    {"BadUSB",        badusbMode,      icon_usb},
    {"Spectrum",      spectrumMode,    icon_spectrum},
    {"Settings",      settingsMode,    icon_settings},
    {"View Logs",     viewLogsMode,    icon_logs},
};
```

**Step 3 --- Implement the RF Recon Module**

Combine all Sub-GHz capabilities:

```
RF Recon Menu:
├── Frequency Analyzer
│   └── Sweep all bands, show active frequencies
├── Signal Capture
│   ├── Auto-detect frequency and modulation
│   ├── Manual frequency entry
│   └── Save to SD card
├── Signal Replay
│   ├── Load from SD card
│   ├── Recent captures
│   └── Replay with options (repeat count, delay)
├── Protocol Decoder
│   ├── Known protocols (weather, remotes, sensors)
│   └── Raw bit display
└── Continuous Monitor
    └── Log all activity on a frequency
```

**Step 4 --- Implement the WiFi Audit Module**

```
WiFi Audit Menu:
├── Network Scanner
│   ├── Full scan (all channels)
│   ├── Targeted scan (specific SSID)
│   └── Export results
├── Deauth Tester
│   ├── Target selection
│   ├── Single deauth
│   └── PMF detection
├── Beacon Generator
│   ├── Custom SSID list
│   ├── Rickroll / funny SSIDs
│   └── Clone existing networks
├── Evil Portal
│   ├── Captive portal templates
│   ├── Credential capture
│   └── Custom HTML upload
└── Packet Monitor
    └── Channel activity display
```

**Step 5 --- Implement the BLE Module**

```
BLE Menu:
├── Device Scanner
│   ├── Passive scan
│   ├── Active scan (more data, less stealthy)
│   └── Device count mode
├── GATT Explorer
│   ├── Connect to device
│   ├── Enumerate services
│   └── Read characteristics
├── Advertisement Spam
│   ├── Apple notification spam
│   ├── Samsung/Android spam
│   ├── Custom advertisements
│   └── Sour Apple (crash notifications)
└── BLE Tracker
    └── Monitor specific device RSSI
```

**Step 6 --- Implement the IR Module**

```
IR Menu:
├── Capture
│   ├── Auto-decode (NEC, Sony, RC5, RC6, Samsung)
│   ├── Raw capture
│   └── Save to library
├── Replay
│   ├── From library
│   ├── Recent captures
│   └── Repeat mode
├── TV-B-Gone
│   ├── All codes
│   └── Region selection
└── Custom Remote
    ├── Load device profile
    └── Button layout editor
```

**Step 7 --- Implement the BadUSB Module**

```
BadUSB Menu:
├── Payload Browser
│   ├── SD card payloads
│   ├── Built-in payloads
│   └── Recent payloads
├── Payload Editor
│   ├── On-device text editor
│   └── Template selection
├── Execute
│   ├── Run selected payload
│   ├── Delayed execution
│   └── OS auto-detect
└── Keyboard Tester
    └── Type test strings
```

**Step 8 --- Build the Logging System**

All modules log to a structured format on the SD card:

```
/logs/
├── 2026-02-28/
│   ├── rf_scan_14-30-00.json
│   ├── wifi_audit_14-45-00.json
│   ├── ble_scan_15-00-00.json
│   └── session_summary.json
```

Log format:

```json
{
    "timestamp": "2026-02-28T14:30:00",
    "module": "rf_recon",
    "action": "signal_capture",
    "frequency": 433920000,
    "modulation": "OOK",
    "rssi": -65,
    "data": "AA BB CC DD EE FF",
    "notes": "Doorbell signal - fixed code"
}
```

**Step 9 --- Add the Settings Module**

```
Settings Menu:
├── Display
│   ├── Brightness
│   ├── Screen timeout
│   └── Color theme
├── Radio
│   ├── Default frequency
│   ├── TX power level
│   └── Region (affects legal frequencies)
├── Storage
│   ├── SD card info
│   ├── Clear logs
│   └── Export all data
├── Battery
│   ├── Voltage display
│   ├── Low battery threshold
│   └── Power saving mode
└── About
    ├── Firmware version
    ├── Hardware info
    └── Legal notice
```

**Step 10 --- Polish the User Interface**

Design a professional status bar and consistent visual language:

```
┌──────────────────────────────────────────┐
│ 🔋 87%  📡 433.9MHz   📶 WiFi   14:30  │ ← Status bar
│──────────────────────────────────────────│
│                                          │
│    ┌────────────────────────────────┐    │
│    │  ► RF Recon                    │    │
│    │    WiFi Audit                  │    │
│    │    BLE Scanner                 │    │
│    │    IR Remote                   │    │ ← Menu items
│    │    BadUSB                      │    │
│    │    Spectrum                    │    │
│    └────────────────────────────────┘    │
│                                          │
│──────────────────────────────────────────│
│ [ROTATE] Navigate    [PRESS] Select     │ ← Control hints
└──────────────────────────────────────────┘
```

### Expected Results

- A fully functional multi-tool that boots to your custom menu system
- Smooth navigation with the rotary encoder (no lag, no missed inputs)
- Each module works independently without interfering with others
- All actions are logged to the SD card automatically
- Battery life of 3--6 hours depending on active radio usage
- A tool that is genuinely useful for authorized penetration testing

{: .tip }
> Start with the menu system and one module (RF Recon is recommended). Get that working perfectly before adding more modules. A well-built single module is more useful than five half-finished ones.

### Extensions

- Add OTA firmware update capability so you can update without USB
- Build a companion web app that parses and visualizes log data
- Add a "quick actions" mode triggered by specific encoder gestures (double-click, long press patterns)
- Implement a scripting engine for automated test sequences
- Add WiFi-based remote control from a phone/laptop
- Share your firmware on GitHub for the community to use and improve
- Create 3D-printable cases with labeled controls and antenna routing

---

## Project Difficulty Summary

| # | Project | Difficulty | Time Estimate | Key Skill |
|:--|:--------|:-----------|:-------------|:----------|
| 1 | RF Environment Audit | Beginner | 2--3 hours | RF scanning |
| 2 | Universal IR Remote | Beginner | 1--2 hours | IR capture/replay |
| 3 | Weather Station Decoder | Intermediate | 4--8 hours | Protocol analysis |
| 4 | Home Security RF Audit | Intermediate | 3--5 hours | Pentest methodology |
| 5 | BLE Device Tracker | Intermediate | 4--6 hours | BLE fundamentals |
| 6 | WiFi Security Auditor | Intermediate | 3--5 hours | WiFi security |
| 7 | Custom RF Protocol | Advanced | 10--20 hours | Protocol design |
| 8 | Replay Defense Tester | Advanced | 5--10 hours | Vulnerability testing |
| 9 | Spectrum Monitor | Advanced | 15--25 hours | DSP and visualization |
| 10 | Integrated Multi-Tool | Expert | 40--80 hours | Full-stack embedded |

{: .tip }
> Do not skip the beginner projects even if you are experienced. They build foundational familiarity with the T-Embed's interface and capabilities that every advanced project depends on.

---

{: .warning }
> **Reminder**: Every project in this chapter must be performed only on devices and networks you own or have explicit written permission to test. Unauthorized access, interception, or interference is a crime. When in doubt, do not transmit. See [Chapter 13 --- Legal Reference](13-legal-reference/) for comprehensive legal guidance.
