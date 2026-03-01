---
layout: default
title: "06 — Bluetooth / BLE"
nav_order: 7
---

# 06 — Bluetooth / BLE Operations
{: .no_toc }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## BLE 5.0 on the ESP32-S3

The LILYGO T-Embed CC1101 is built around the **ESP32-S3** microcontroller, which includes a fully integrated Bluetooth Low Energy 5.0 radio. This is a separate radio from the CC1101 sub-GHz transceiver and operates in the **2.4 GHz ISM band** (2400--2483.5 MHz).

### Key Specifications

| Parameter | Value |
|:---|:---|
| Bluetooth Version | BLE 5.0 |
| Frequency Band | 2.4 GHz ISM |
| Channels | 40 (3 advertising, 37 data) |
| Data Rates | 1 Mbps (LE 1M), 2 Mbps (LE 2M) |
| Range (typical) | 10--30 m indoors, up to 100 m outdoors (line of sight) |
| TX Power | Up to +20 dBm |
| Advertising Extensions | Yes (BLE 5.0 extended advertising) |
| Coded PHY (Long Range) | Supported (125 kbps / 500 kbps) |
| Antenna | On-board PCB trace antenna |

> **Important: BLE only, not Classic Bluetooth.**
> The ESP32-S3 supports **only** Bluetooth Low Energy. It does **not** support Bluetooth Classic (BR/EDR). This means you cannot pair with Classic Bluetooth audio devices, file-transfer profiles (OBEX), or serial port profiles (SPP) using the T-Embed's built-in radio.
{: .warning }

### BLE 5.0 Feature Highlights

- **2 Mbps PHY**: Double the data rate of BLE 4.x, reducing time on air and power consumption.
- **Coded PHY (Long Range)**: Trades data rate for range — up to 4x the distance of standard BLE by using Forward Error Correction (FEC). Rates of 500 kbps (S=2) or 125 kbps (S=8).
- **Extended Advertising**: Advertising payloads up to 255 bytes (vs. 31 bytes in BLE 4.x), advertising on data channels, chained advertising PDUs.
- **Channel Selection Algorithm #2**: Improved frequency hopping for better coexistence with Wi-Fi and other 2.4 GHz traffic.

---

## BLE vs. Classic Bluetooth

Understanding the distinction is critical because the T-Embed can only interact with BLE devices.

| Feature | Bluetooth Classic (BR/EDR) | Bluetooth Low Energy (BLE) |
|:---|:---|:---|
| Power Consumption | Higher (continuous connection) | Ultra-low (event-driven) |
| Data Rate | Up to 3 Mbps (EDR) | 1--2 Mbps |
| Range | ~10--100 m | ~10--100 m (up to 400 m Coded PHY) |
| Latency | ~100 ms | ~6 ms (connection interval minimum) |
| Connection Topology | Point-to-point | Point-to-point, broadcast, mesh |
| Audio Streaming | A2DP, HFP | LE Audio (LC3 codec, BLE 5.2+) |
| File Transfer | OBEX / FTP | Not natively (custom GATT) |
| Use Cases | Audio, file transfer, serial | IoT sensors, fitness, beacons, smart home |
| ESP32-S3 Support | **No** | **Yes** |

> Devices like wireless headphones, Bluetooth speakers, and car audio systems typically use Classic Bluetooth for audio streaming. The T-Embed cannot directly interact with these. However, many modern devices support **both** Classic and BLE simultaneously (dual-mode), and the T-Embed can interact with their BLE advertisement and GATT services.
{: .note }

---

## BLE Architecture Deep Dive

BLE is organized into two major frameworks: **GAP** (for discovery and connections) and **GATT** (for data exchange after connection).

### GAP — Generic Access Profile

GAP controls **advertising**, **scanning**, and **connection establishment**. It defines the roles a device can play:

```
┌─────────────────────────────────────────────────────────┐
│                     GAP ROLES                           │
├──────────────┬──────────────┬──────────────┬────────────┤
│  Broadcaster │   Observer   │  Peripheral  │  Central   │
│  (TX only)   │  (RX only)   │ (advertise + │ (scan +    │
│              │              │  connect)    │  connect)  │
│  e.g. beacon │ e.g. scanner │ e.g. fitness │ e.g. phone │
│              │              │    tracker   │            │
└──────────────┴──────────────┴──────────────┴────────────┘
```

**The T-Embed with Bruce firmware acts primarily as a Central / Observer** — it scans for and connects to other devices.

#### Advertising Process

BLE devices announce their presence by broadcasting **advertising packets** on three dedicated channels (37, 38, 39). These channels are spaced across the 2.4 GHz band to minimize interference:

```
2.4 GHz Band Layout:

Ch 37          Ch 38                    Ch 39
(2402 MHz)     (2426 MHz)               (2480 MHz)
  │              │                         │
  ▼              ▼                         ▼
  ┃  ┃┃┃┃┃┃┃┃┃  ┃  ┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃  ┃
  ADV  DATA CH   ADV      DATA CHANNELS    ADV
       0-10            11-36

  ←──────────── 80 MHz total ─────────────→
```

**Advertising packet structure:**

```
┌──────────┬──────────┬──────────────────────┐
│ Preamble │ Access   │ PDU                  │
│ (1 byte) │ Address  │ (2-257 bytes)        │
│          │ (4 bytes)│                      │
└──────────┴──────────┼──────────────────────┤
                      │ Header │ Payload     │
                      │(2 bytes)│(0-255 bytes)│
                      └────────┴─────────────┘
```

**Advertising types:**

| Type | Name | Description |
|:---|:---|:---|
| `ADV_IND` | Connectable Undirected | General advertising, accepts connections from any Central |
| `ADV_DIRECT_IND` | Connectable Directed | Targeted at a specific Central device |
| `ADV_SCAN_IND` | Scannable Undirected | Allows scan requests for additional data, but not connections |
| `ADV_NONCONN_IND` | Non-connectable Undirected | Broadcast only (beacons) |
| `SCAN_RSP` | Scan Response | Additional data sent in response to a scan request |

#### Advertising Data (AD) Structures

Each advertising packet contains one or more AD structures:

```
┌──────────────────────────────────────────┐
│           Advertising Payload            │
├────────┬────────┬────────┬───────────────┤
│ Length │  Type  │  Data  │  ... more AD  │
│(1 byte)│(1 byte)│(N bytes)│  structures  │
└────────┴────────┴────────┴───────────────┘
```

**Common AD types:**

| Type Code | Name | What It Contains |
|:---|:---|:---|
| `0x01` | Flags | Discoverability mode, BR/EDR support |
| `0x02`/`0x03` | 16-bit Service UUIDs | Incomplete / Complete list |
| `0x06`/`0x07` | 128-bit Service UUIDs | Custom service identifiers |
| `0x08`/`0x09` | Shortened/Complete Local Name | Human-readable device name |
| `0x0A` | TX Power Level | Transmit power in dBm |
| `0xFF` | Manufacturer Specific Data | First 2 bytes = Company ID, rest is proprietary |

### GATT — Generic Attribute Profile

Once a connection is established, GATT defines how data is organized and exchanged. GATT uses a hierarchical structure:

```
┌─────────────────────────────────────────────┐
│                GATT Server                  │
│  (the peripheral / device being queried)    │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─ Service (UUID: 0x180D — Heart Rate) ─┐  │
│  │                                        │  │
│  │  ┌─ Characteristic ──────────────────┐ │  │
│  │  │ UUID: 0x2A37 — Heart Rate Meas.   │ │  │
│  │  │ Properties: Notify               │ │  │
│  │  │ Value: [0x06, 0x48]              │ │  │
│  │  │                                   │ │  │
│  │  │  ┌─ Descriptor ────────────────┐  │ │  │
│  │  │  │ CCCD (0x2902) — Enable      │  │ │  │
│  │  │  │ Notifications               │  │ │  │
│  │  │  └─────────────────────────────┘  │ │  │
│  │  └───────────────────────────────────┘ │  │
│  │                                        │  │
│  │  ┌─ Characteristic ──────────────────┐ │  │
│  │  │ UUID: 0x2A38 — Body Sensor Loc.   │ │  │
│  │  │ Properties: Read                 │ │  │
│  │  │ Value: [0x01] (Chest)            │ │  │
│  │  └───────────────────────────────────┘ │  │
│  └────────────────────────────────────────┘  │
│                                             │
│  ┌─ Service (UUID: 0x180F — Battery) ────┐  │
│  │  ┌─ Characteristic ──────────────────┐ │  │
│  │  │ UUID: 0x2A19 — Battery Level      │ │  │
│  │  │ Properties: Read, Notify         │ │  │
│  │  │ Value: [0x5F] (95%)              │ │  │
│  │  └───────────────────────────────────┘ │  │
│  └────────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

**Key GATT concepts:**

- **Service**: A collection of related data and behaviors. Identified by UUID (16-bit standard or 128-bit custom).
- **Characteristic**: A single data point within a service. Has properties (Read, Write, Notify, Indicate) and a value.
- **Descriptor**: Metadata about a characteristic (e.g., CCCD for enabling notifications, user description, format).

**Characteristic properties:**

| Property | Description |
|:---|:---|
| Read | Client can read the value |
| Write | Client can write a value (with acknowledgment) |
| Write Without Response | Client can write without acknowledgment |
| Notify | Server can push updates (no acknowledgment) |
| Indicate | Server can push updates (with acknowledgment) |

---

## BLE Scanning with Bruce Firmware

Bruce firmware provides comprehensive BLE scanning capabilities on the T-Embed CC1101.

### Starting a BLE Scan

1. From the main menu, navigate to **Bluetooth** (or **BLE**).
2. Select **BLE Scan** (or **Scan BLE Devices**).
3. The T-Embed will begin scanning all three advertising channels.
4. Discovered devices appear on screen with:
   - Device name (if advertised)
   - MAC address (may be random)
   - RSSI (signal strength in dBm)
   - Device type indicators

### Understanding Scan Results

```
┌──────────────────────────────────────┐
│ BLE Scan Results         Found: 12  │
├──────────────────────────────────────┤
│ ▸ Mi Band 7           -45 dBm  ████ │
│   Fitbit Charge 5     -62 dBm  ██▒  │
│   [Unknown]           -71 dBm  ██   │
│   AirPods Pro         -55 dBm  ███  │
│   Samsung TV           -78 dBm  █▒  │
│   Tile Mate            -66 dBm  ██▒ │
│   [Random: 4A:3B:...]  -83 dBm  █  │
│   Living Room Light    -48 dBm  ███ │
│                                     │
│ [A] Sort  [B] Filter  [OK] Details  │
└──────────────────────────────────────┘
```

**RSSI interpretation:**

| RSSI (dBm) | Signal Quality | Approximate Distance |
|:---|:---|:---|
| -30 to -50 | Excellent | < 1 m |
| -50 to -60 | Good | 1--3 m |
| -60 to -70 | Fair | 3--10 m |
| -70 to -80 | Weak | 10--20 m |
| -80 to -90 | Very Weak | > 20 m |
| < -90 | Marginal | Near maximum range |

> RSSI values are affected by obstacles, multipath reflections, antenna orientation, and interference. Use RSSI as a rough proximity estimate, never as a precise distance measurement.
{: .note }

### MAC Address Types

BLE devices use different MAC address strategies:

| Type | Format | Description |
|:---|:---|:---|
| Public | `XX:XX:XX:XX:XX:XX` | Globally unique, registered IEEE OUI. Permanent. |
| Random Static | `XX:XX:XX:XX:XX:XX` | Generated at boot, stays constant until power cycle. Two MSBs = `11`. |
| Random Private Resolvable | `XX:XX:XX:XX:XX:XX` | Rotates every ~15 minutes. Can be resolved by bonded devices using IRK. |
| Random Private Non-Resolvable | `XX:XX:XX:XX:XX:XX` | Rotates frequently. Cannot be resolved. Used for beacons. |

> Most modern smartphones and wearables use **Random Private Resolvable** addresses that rotate every 10--15 minutes. This makes tracking specific devices by MAC address alone unreliable. Apple, Google, and Samsung all implement address rotation as a privacy measure.
{: .warning }

---

## Understanding BLE Advertisements

### Decoding Advertisement Data

When you capture a BLE advertisement, the raw data reveals important information about the device. Here is how a typical advertising packet breaks down:

```
Raw Advertising Data (hex):
02 01 06 11 07 FB 34 9B 5F 80 00 00 80 00 10 00 00 00 00 00 00 09 09 4D 69 42 61 6E 64 20 37

Decoded:
┌─────────────────────────────────────────────────────────┐
│ AD Structure 1:                                         │
│   Length: 0x02 (2 bytes of data follow)                 │
│   Type:   0x01 (Flags)                                  │
│   Data:   0x06 = 0b00000110                             │
│           Bit 1: LE General Discoverable Mode           │
│           Bit 2: BR/EDR Not Supported                   │
├─────────────────────────────────────────────────────────┤
│ AD Structure 2:                                         │
│   Length: 0x11 (17 bytes of data follow)                │
│   Type:   0x07 (Complete 128-bit Service UUID)          │
│   Data:   FB349B5F-8000-0080-0010-000000000000          │
│           (Custom service UUID — proprietary protocol)  │
├─────────────────────────────────────────────────────────┤
│ AD Structure 3:                                         │
│   Length: 0x09 (9 bytes of data follow)                 │
│   Type:   0x09 (Complete Local Name)                    │
│   Data:   "MiBand 7" (ASCII decoded)                    │
└─────────────────────────────────────────────────────────┘
```

### Manufacturer-Specific Data (Type 0xFF)

This is the richest source of information for device fingerprinting. The first two bytes are the **Bluetooth SIG Company Identifier** (little-endian):

| Company ID (hex) | Company | Commonly Seen In |
|:---|:---|:---|
| `0x004C` | Apple | iPhones, AirPods, AirTags, MacBooks, Apple Watch |
| `0x0075` | Samsung | Galaxy phones, Galaxy Buds, SmartThings |
| `0x0006` | Microsoft | Windows devices, Xbox controllers, Surface |
| `0x00E0` | Google | Pixel, Nest, Chromecast, Fast Pair |
| `0x0059` | Nordic Semi | nRF-based devices (Tile, many IoT) |
| `0x0157` | Anhui Huami (Zepp) | Xiaomi Mi Band, Amazfit watches |
| `0x038F` | Xiaomi | Xiaomi IoT ecosystem |

**Example — Apple device advertisement:**

```
Manufacturer Specific Data: 4C 00 12 02 00 01
                            ─────  ── ──
                            Apple  │  │
                                   │  └─ Nearby Info data
                                   └─ Type 0x12 = Nearby Action
                                      (AirPods pairing popup)
```

---

## Common BLE Device Types

When you scan your environment, you will encounter many different BLE device categories:

### Wearables and Fitness

| Device | Service UUIDs | Characteristics |
|:---|:---|:---|
| Heart Rate Monitor | `0x180D` | Heart rate measurement, body sensor location |
| Fitness Tracker | `0x1816` (Cycling Speed), custom | Steps, calories, sleep data |
| Smart Watch | Various custom | Notifications, health data |
| Blood Pressure | `0x1810` | Systolic, diastolic, pulse |
| Glucose Monitor | `0x1808` | Glucose measurement, context |
| Pulse Oximeter | `0x1822` | SpO2, pulse rate |

### Smart Home

| Device | Advertising Pattern | Notes |
|:---|:---|:---|
| Smart Bulbs (Philips Hue) | Zigbee bridge has BLE commissioning | May expose light control GATT |
| Smart Locks (August, Yale) | Custom service UUIDs | Encrypted communications |
| Smart Plugs | Custom manufacturer data | Often use vendor-specific protocols |
| Thermostats | `0x181A` (Environmental Sensing) | Temperature, humidity |
| Smart Blinds | Custom | Motor control characteristics |

### Trackers and Beacons

| Device | Identifier | Behavior |
|:---|:---|:---|
| Apple AirTag | Company ID `0x004C`, Type `0x12` | Rotates address every 15 min, plays sound after separation |
| Samsung SmartTag | Company ID `0x0075` | SmartThings Find network |
| Tile | Company ID `0x0059` (Nordic) | Community find network |
| iBeacon | Company ID `0x004C`, Type `0x02` | UUID + Major + Minor + TX Power |
| Eddystone | Service UUID `0xFEAA` | URL, UID, TLM, or EID frame |

### Consumer Electronics

| Device | Pattern |
|:---|:---|
| Wireless Earbuds | Fast Pair (Google) or Nearby (Apple) advertisements |
| Game Controllers | HID over GATT (`0x1812`) |
| Smart TVs | Device Information Service (`0x180A`), custom control |
| Bluetooth Speakers | Audio service, battery service |
| E-Readers | Custom sync services |

---

## BLE Device Fingerprinting

Even when devices use random MAC addresses, you can often identify the device type from its advertisement data.

### Fingerprinting Techniques

**1. Service UUID analysis:**
Standard 16-bit UUIDs immediately identify device capabilities:

```
0x180D → Heart Rate Service     → Fitness device
0x180F → Battery Service        → Almost anything (very common)
0x1812 → HID over GATT          → Keyboard, mouse, gamepad
0x180A → Device Information     → Check manufacturer string
0x1800 → Generic Access         → Contains device name
0x1801 → Generic Attribute      → GATT service changed
0xFE9F → Google Fast Pair       → Android accessory
0xFD6F → Apple Exposure Notif.  → COVID contact tracing (legacy)
```

**2. Manufacturer data decoding:**

```python
# Pseudo-code for identifying Apple device types from advertisement
apple_types = {
    0x01: "iBeacon (legacy)",
    0x02: "iBeacon",
    0x05: "AirDrop",
    0x07: "AirPods",
    0x09: "AirPlay Target",
    0x0A: "AirPlay Source",
    0x0C: "Handoff",
    0x0D: "Wi-Fi Settings",
    0x0E: "Instant Hotspot",
    0x0F: "Wi-Fi Password Sharing",
    0x10: "Nearby Action",
    0x12: "Nearby Action (new)",
    0x19: "AirTag / Find My"
}
```

**3. Advertisement interval:**

| Interval | Typical Device |
|:---|:---|
| 20--100 ms | Device seeking fast connection (pairing mode) |
| 100--500 ms | Active wearable, interactive device |
| 1--2 s | Beacon, tracker in normal mode |
| 2--10 s | Low-power sensor, deep-sleep tracker |

**4. TX Power + RSSI correlation:**
If the advertisement includes TX Power Level (type `0x0A`), you can estimate path loss and thus approximate distance:

```
Estimated Distance = 10 ^ ((TX_Power - RSSI) / (10 * n))

Where n = path loss exponent:
  n = 2.0  (free space)
  n = 2.5  (indoor, light obstructions)
  n = 3.0  (indoor, walls)
  n = 3.5  (indoor, heavy obstructions)
```

---

## Apple BLE Spam

> This section is for educational and authorized security research only. Performing BLE spam attacks against non-consenting individuals may violate laws including the Computer Fraud and Abuse Act, UK Computer Misuse Act, and EU cybercrime directives. Only use on your own devices or in controlled lab environments.
{: .danger }

### What It Does

Apple BLE spam exploits Apple's **Nearby Action** protocol to generate fake pairing popups on iOS devices. When an iPhone or iPad detects a BLE advertisement with Apple's Company ID (`0x004C`) and specific type bytes, it displays a popup notification — even without any user interaction.

**Affected popups include:**
- AirPods pairing animation
- AirPods Pro pairing animation
- AirPods Max pairing animation
- Beats headphones pairing
- Apple TV setup
- HomePod setup
- AirTag detected nearby
- New Apple Pencil
- Transfer phone number
- Setup new iPhone

### How It Works

The attack crafts BLE advertising packets that mimic Apple's Nearby Action protocol:

```
Advertising Data Structure:
┌──────────────────────────────────────────┐
│ Flags: 0x06 (LE General, No BR/EDR)     │
├──────────────────────────────────────────┤
│ Manufacturer Specific Data:              │
│   Company ID: 0x004C (Apple)             │
│   Type: 0x0F (Nearby Action)             │
│   Action Flags: variable                 │
│   Action Type: determines which popup    │
│   Authentication Tag: randomized         │
│   Payload: device-specific data          │
└──────────────────────────────────────────┘
```

**Action types determine the popup:**

| Action Type (hex) | Popup Displayed |
|:---|:---|
| `0x01` | AirPods |
| `0x02` | AirPods Pro |
| `0x03` | AirPods Max |
| `0x04` | AirPods Pro 2 |
| `0x05` | Beats Fit Pro |
| `0x06` | Beats Studio Buds |
| `0x07` | Beats Flex |
| `0x09` | Apple TV Setup |
| `0x0A` | Setup New iPhone |
| `0x0E` | Transfer Phone Number |
| `0x14` | HomePod |

### Step-by-Step on T-Embed (Bruce Firmware)

1. Navigate to **Bluetooth** > **Apple BLE Spam** (or **BLE Spam** > **Apple**).
2. Select the **spam type**:
   - **Single**: Sends one specific device type popup.
   - **Random / Flood**: Rapidly cycles through all device types.
3. Choose the specific device popup to spoof (if single mode).
4. Press **OK** / confirm to begin broadcasting.
5. The T-Embed rapidly transmits crafted Nearby Action advertisements.
6. Any nearby iPhones / iPads within BLE range (~10--30 m) will display popup notifications.
7. Press **Back** to stop.

### Mitigations (for iPhone users)

- **Disable Bluetooth** (from Settings, not Control Center — Control Center only disconnects, does not fully disable the radio).
- **iOS 17.2+**: Apple added filtering for unexpected Nearby Action frames. Update to the latest iOS version.
- **Lockdown Mode**: Blocks many BLE-based attacks including spam popups.

---

## Samsung BLE Spam

### What It Does

Similar to Apple BLE spam, this targets **Samsung Galaxy devices** by spoofing Samsung's BLE advertising protocol. Samsung devices respond to specific manufacturer data patterns to display pairing popups for Galaxy Buds and other Samsung accessories.

### How It Works

The attack broadcasts BLE advertisements with Samsung's Company ID (`0x0075`) and specific payload structures:

```
Advertising Data Structure:
┌──────────────────────────────────────────┐
│ Flags: 0x06                              │
├──────────────────────────────────────────┤
│ Manufacturer Specific Data:              │
│   Company ID: 0x0075 (Samsung)           │
│   Protocol Type: Galaxy accessory        │
│   Device Model: e.g., Galaxy Buds2 Pro   │
│   State: Pairing mode                    │
│   Color/Variant data                     │
└──────────────────────────────────────────┘
```

**Spoofable devices include:**
- Galaxy Buds, Buds+, Buds Live, Buds Pro, Buds2, Buds2 Pro, Buds FE
- Galaxy Watch series
- Galaxy Ring
- Galaxy SmartTag

### Step-by-Step

1. Navigate to **Bluetooth** > **Samsung BLE Spam** (or **BLE Spam** > **Samsung**).
2. Select single device type or random/flood mode.
3. Press **OK** to begin broadcasting.
4. Nearby Samsung Galaxy phones display accessory pairing popups.
5. Press **Back** to stop.

### Mitigations (for Samsung users)

- Disable **Nearby Device Scanning** in Samsung settings.
- Disable Bluetooth when not in use.
- Install latest Samsung security patches.

---

## Windows Swift Pair Spam

### What It Does

Windows 10 and 11 feature **Swift Pair**, a protocol that displays toast notifications when a new Bluetooth device is detected nearby in pairing mode. By spoofing Swift Pair advertisements, the T-Embed can flood Windows machines with fake device pairing notifications.

### How It Works

Swift Pair uses Microsoft's Company ID (`0x0006`) with a specific payload format:

```
Advertising Data Structure:
┌──────────────────────────────────────────┐
│ Flags: 0x06                              │
├──────────────────────────────────────────┤
│ Manufacturer Specific Data:              │
│   Company ID: 0x0006 (Microsoft)         │
│   Beacon Type: 0x03 (Swift Pair)         │
│   RSSI Threshold: calibration value      │
│   Payload: device class, display info    │
├──────────────────────────────────────────┤
│ Complete Local Name: "Device Name"       │
│   (displayed in the Swift Pair popup)    │
└──────────────────────────────────────────┘
```

### Step-by-Step

1. Navigate to **Bluetooth** > **Windows Swift Pair Spam** (or **BLE Spam** > **Windows**).
2. Select single or flood mode.
3. Press **OK** to begin.
4. Nearby Windows PCs display "New Bluetooth device found" toast notifications.
5. Press **Back** to stop.

### Mitigations (for Windows users)

- **Settings** > **Bluetooth & devices** > **Show notifications to connect using Swift Pair** > **Off**.
- Group Policy: `Computer Configuration > Administrative Templates > Bluetooth > Allow Swift Pair notifications > Disabled`.

---

## BLE Tracker Detection

> This is a **defensive** feature. Use it to detect whether Bluetooth trackers (AirTags, SmartTags, Tile) are covertly following you.
{: .tip }

### Why This Matters

Bluetooth trackers have been misused for **stalking**. An AirTag or SmartTag placed in someone's bag, car, or belongings can track their location over time using the manufacturer's crowd-sourced network. While Apple and Samsung have built detection features into their respective phones, these only work if you own the right phone. The T-Embed provides **platform-independent tracker detection**.

### How Tracker Detection Works

```
Detection Algorithm:
┌─────────────────────────────────────────┐
│ 1. Continuous BLE scan                  │
│ 2. Record all detected devices + RSSI   │
│ 3. Track which devices persist over     │
│    time (>10 minutes, moving with you)  │
│ 4. Flag devices matching tracker        │
│    signatures:                          │
│    - Apple Find My (0x004C, type 0x12)  │
│    - Samsung SmartTag (0x0075)          │
│    - Tile (Nordic Semi, specific UUID)  │
│    - Chipolo (specific adv. pattern)    │
│ 5. Alert user of suspected trackers     │
└─────────────────────────────────────────┘
```

### Using Tracker Detection on Bruce

1. Navigate to **Bluetooth** > **Detect Trackers** (or **BLE** > **Tracker Scan**).
2. The T-Embed enters continuous scanning mode.
3. Walk around normally while the device monitors.
4. Detected trackers are displayed with:
   - Tracker type (AirTag, SmartTag, Tile, Unknown)
   - RSSI (signal strength — lower absolute value means closer)
   - First seen / duration detected
   - MAC address (may rotate for AirTags)
5. If a tracker is identified, note its approximate location based on RSSI changes as you move.

### Identifying Tracker Types

| Tracker | BLE Signature | Rotation Period | Sound Capability |
|:---|:---|:---|:---|
| Apple AirTag | Company `0x004C`, Find My payload | ~15 min address rotation | Yes (after 8--24 hours separated from owner) |
| Samsung SmartTag | Company `0x0075`, SmartThings payload | Variable | Yes |
| Samsung SmartTag 2 | Company `0x0075`, UWB capable | Variable | Yes |
| Tile Mate/Pro | Nordic Semi, custom service UUID | Static address common | Yes (community find) |
| Chipolo ONE | Custom advertisement pattern | Variable | Yes |

> Apple AirTags rotate their BLE address every ~15 minutes using a cryptographic key derived from the owner's Apple ID. This makes tracking a single AirTag by MAC address challenging. However, the advertising payload structure remains identifiable as a Find My device even across address rotations.
{: .note }

### What to Do If You Find a Tracker

1. **Do not confront anyone** — ensure your personal safety first.
2. **Document the detection** — screenshot or note the details.
3. **Try to locate it** — use RSSI readings. Move around; stronger signal = closer to tracker.
4. **If it is an AirTag**: Hold your iPhone (or NFC-enabled Android) near it to get the serial number and the last four digits of the owner's phone number.
5. **Disable it**: Remove the battery (twist the back cover counterclockwise for AirTag).
6. **Contact law enforcement** if you believe you are being stalked.

---

## BLE GATT Enumeration

GATT enumeration is the process of connecting to a BLE device and discovering all its services, characteristics, and descriptors. This is fundamental to understanding what a device exposes over BLE.

### Performing GATT Enumeration

1. Perform a BLE scan and identify your target device.
2. Select the device and choose **Connect** or **GATT Enum** (depending on Bruce firmware version).
3. The T-Embed initiates a BLE connection and performs service discovery.
4. Results display the full GATT hierarchy:

```
Device: Smart Lock XYZ
MAC: A4:C1:38:XX:XX:XX
Connected: Yes

Services Found: 4
─────────────────────────────────────────
Service: Generic Access (0x1800)
├── Device Name (0x2A00) [Read]
│   Value: "Smart Lock XYZ"
├── Appearance (0x2A01) [Read]
│   Value: 0x0000 (Unknown)
└── Preferred Conn. Params (0x2A04) [Read]

Service: Generic Attribute (0x1801)
└── Service Changed (0x2A05) [Indicate]

Service: Device Information (0x180A)
├── Manufacturer Name (0x2A29) [Read]
│   Value: "LockCo Inc."
├── Model Number (0x2A24) [Read]
│   Value: "SL-3000"
├── Firmware Revision (0x2A26) [Read]
│   Value: "v2.1.4"
└── Hardware Revision (0x2A27) [Read]
    Value: "Rev B"

Service: Custom (a1b2c3d4-e5f6-7890-abcd-ef1234567890)
├── Characteristic 1 (...) [Write, Notify]
│   └── CCCD (0x2902)
├── Characteristic 2 (...) [Read, Write]
└── Characteristic 3 (...) [Read]
    Value: 0x00 (Lock Status: Locked)
─────────────────────────────────────────
```

### Standard GATT Services Reference

| UUID | Service Name | Contains |
|:---|:---|:---|
| `0x1800` | Generic Access | Device name, appearance, connection parameters |
| `0x1801` | Generic Attribute | Service changed indication |
| `0x180A` | Device Information | Manufacturer, model, serial, firmware version |
| `0x180D` | Heart Rate | Heart rate measurement, body sensor location |
| `0x180F` | Battery | Battery level percentage |
| `0x1810` | Blood Pressure | Blood pressure measurement |
| `0x1812` | HID (Human Interface Device) | Keyboard/mouse reports |
| `0x1816` | Cycling Speed and Cadence | Wheel/crank revolutions |
| `0x181A` | Environmental Sensing | Temperature, humidity, pressure |
| `0x181C` | User Data | Name, age, weight, height |

> When enumerating GATT services on devices you do not own, you may be violating computer access laws. Only perform GATT enumeration on your own devices or devices you have explicit written authorization to test.
{: .warning }

### What GATT Enumeration Reveals

- **Device identity**: Manufacturer, model, firmware version — useful for vulnerability research.
- **Exposed data**: What information the device makes available over BLE.
- **Write capabilities**: Which characteristics accept writes — potential attack surface.
- **Security posture**: Whether sensitive characteristics are protected by encryption or authentication.
- **Custom services**: Proprietary protocols that may contain vulnerabilities.

---

## BLE Security Modes

BLE defines security at the link layer. Understanding these modes is essential for assessing the security posture of BLE devices.

### Pairing Methods

| Method | User Interaction | MITM Protection | Typical Use |
|:---|:---|:---|:---|
| **Just Works** | None | **No** | Headphones, simple IoT sensors |
| **Passkey Entry** | Enter 6-digit PIN | Yes | Keyboards, payment terminals |
| **Numeric Comparison** | Confirm matching 6-digit number on both devices | Yes | Phone-to-phone, watches |
| **Out of Band (OOB)** | NFC tap, QR code scan | Yes (if OOB channel is secure) | NFC pairing, enterprise |

### Security Levels

| Level | Name | Requirements |
|:---|:---|:---|
| 1 | No Security | No encryption, no authentication |
| 2 | Unauthenticated Encryption | Encrypted link, but Just Works pairing (MITM possible) |
| 3 | Authenticated Encryption | Encrypted link with MITM-protected pairing (Passkey/Numeric/OOB) |
| 4 | Authenticated LE Secure Connections | BLE 4.2+ Secure Connections with ECDH key exchange |

### Security Vulnerabilities in BLE

```
Attack Surface Overview:

                    ┌────────────┐
   Eavesdropping ──→│            │←── Replay attacks
                    │  BLE Link  │
  MITM (Just Works)→│            │←── GATT manipulation
                    │            │
  Passive sniffing ─→│            │←── Fuzzing
                    └────────────┘
         ▲                              ▲
         │                              │
   Address tracking              DoS (connection
   (static MACs)                  flooding)
```

**Common vulnerabilities:**

| Vulnerability | Description | Affected Devices |
|:---|:---|:---|
| Just Works MITM | No authentication allows man-in-the-middle during pairing | Cheap IoT, headphones |
| Static MAC tracking | Devices with fixed public addresses can be tracked | Older devices, cheap trackers |
| Unencrypted GATT | Sensitive data transmitted without encryption | Many consumer devices |
| Hardcoded PINs | Default PINs like 000000 or 123456 | Smart locks, medical devices |
| Replay attacks | Captured write commands replayed to control device | Devices without nonce/sequence |
| Downgrade attacks | Force legacy pairing to bypass Secure Connections | Dual-mode devices |

> Many consumer BLE devices use **Just Works** pairing with **no encryption** on GATT characteristics. This means anyone within BLE range can read and potentially write data. Always assume BLE communications are insecure unless proven otherwise.
{: .warning }

---

## BLE Sniffing Limitations

The ESP32-S3 in the T-Embed has significant limitations as a BLE sniffer compared to dedicated hardware.

### What the T-Embed Can Do

- **Scan advertisements**: See all devices broadcasting within range.
- **Read advertisement data**: Decode manufacturer data, service UUIDs, device names.
- **Connect and enumerate GATT**: Discover services and characteristics on connectable devices.
- **Measure RSSI**: Approximate distance/proximity.
- **Transmit crafted advertisements**: BLE spam, beacon spoofing.

### What the T-Embed Cannot Do

| Capability | T-Embed (ESP32-S3) | Ubertooth One | nRF52840 Dongle |
|:---|:---|:---|:---|
| Passive advertisement capture | Yes | Yes | Yes |
| Passive connection sniffing | **No** | Yes | Yes |
| Follow frequency hopping | **No** | Yes | Yes |
| Raw link-layer capture | **No** | Yes | Yes |
| Promiscuous mode | **No** | Yes | Limited |
| Wireshark integration | **No** | Yes (btbb plugin) | Yes (nRF Sniffer) |
| Capture encrypted traffic | **No** | Yes (raw) | Yes (raw) |
| Injection into connections | **No** | Yes | Limited |
| Cost | ~$20 (integrated) | ~$120 | ~$10 |

> For serious BLE security research requiring passive sniffing of connections between two devices, you need a dedicated sniffer like the **Ubertooth One** or **nRF52840 Dongle with nRF Sniffer firmware**. The T-Embed is excellent for scanning, enumeration, and active testing, but it cannot passively capture traffic between two other devices.
{: .note }

### Recommended Complementary Tools

| Tool | Purpose | Price |
|:---|:---|:---|
| nRF52840 Dongle | BLE sniffing with nRF Sniffer for Bluetooth LE | ~$10 |
| Ubertooth One | Full BLE + Classic Bluetooth sniffing and injection | ~$120 |
| nRF Connect (mobile app) | GATT exploration, advertisement logging | Free |
| Wireshark + BLE plugins | Packet analysis and protocol decoding | Free |
| BTLE (btlejack) | BLE connection hijacking research tool | Free (software) |

---

## BLE Security Research Use Cases

The T-Embed with Bruce firmware is a valuable tool for legitimate BLE security research:

### 1. IoT Device Assessment

Assess the BLE security posture of IoT devices:
- Scan for the device and examine its advertisement data.
- Enumerate GATT services and characteristics.
- Identify sensitive data exposed without encryption.
- Test for default/hardcoded credentials.
- Attempt unauthorized writes to control characteristics.

### 2. Smart Lock Testing

> Only test locks you own or have explicit written authorization to test.
{: .danger }

- Enumerate the lock's GATT services.
- Identify the lock/unlock characteristic.
- Analyze the authentication mechanism.
- Test for replay attacks (capture and replay unlock command).
- Check if the lock uses challenge-response or simple static commands.

### 3. Proximity-Based Access Control

- Assess BLE-based proximity access systems.
- Test relay attack scenarios (extend BLE range with repeaters).
- Evaluate RSSI-based proximity thresholds.

### 4. Medical Device Security

- Identify BLE-enabled medical devices in the environment.
- Assess data exposure (patient data over unencrypted BLE).
- Test for command injection on writable characteristics.

> Medical device testing requires extreme caution. Unauthorized interference with medical devices can endanger lives and violates numerous laws. Only perform such testing in controlled lab environments with decommissioned devices.
{: .danger }

### 5. Corporate BLE Exposure Audit

- Map all BLE devices in an office environment.
- Identify devices leaking sensitive information.
- Detect unauthorized BLE devices on the network.
- Assess employee wearable data exposure.

---

## Practical Exercises

### Exercise 1: Environment Survey

**Objective**: Identify and categorize all BLE devices in your immediate environment.

**Steps:**
1. Power on the T-Embed and navigate to **Bluetooth** > **BLE Scan**.
2. Let the scan run for at least 60 seconds to capture slow advertisers.
3. For each discovered device, record:
   - Device name (or "Unknown")
   - MAC address and type (public vs. random)
   - RSSI value
   - Any identified service UUIDs
4. Categorize each device: wearable, smart home, tracker, phone, computer, unknown.
5. Estimate the distance to each device based on RSSI.
6. Identify which devices use random address rotation (note MAC changes across multiple scans).

**Expected findings in a typical home:**
- 5--15 BLE devices (phones, tablets, smart home, wearables)
- Mix of public and random addresses
- Several "Unknown" devices with no name in advertisement

### Exercise 2: Tracker Sweep

**Objective**: Verify no unauthorized trackers are present in your belongings or vehicle.

**Steps:**
1. Place the T-Embed in your bag or car.
2. Start **Tracker Detection** mode.
3. Travel your normal route for 15--30 minutes.
4. Check for any persistent BLE devices that follow you.
5. Investigate any flagged devices.
6. If an unknown tracker is found, use RSSI triangulation to locate it physically.

### Exercise 3: Your Own Device Audit

**Objective**: Understand what your own BLE devices expose.

**Steps:**
1. Choose a BLE device you own (fitness tracker, smart watch, smart bulb).
2. Scan for it and note its advertisement data.
3. Connect and perform full GATT enumeration.
4. Document every service, characteristic, and their properties.
5. Identify any data exposed without authentication.
6. Read all readable characteristics and interpret the values.
7. Assess: could an attacker extract useful information from this device?

> Only enumerate devices you own. Connecting to other people's BLE devices without permission may violate computer access laws.
{: .warning }

---

## Troubleshooting BLE Operations

### Scan finds no devices

| Possible Cause | Solution |
|:---|:---|
| BLE radio not initialized | Restart the T-Embed or re-enter BLE menu |
| Interference from Wi-Fi | Disable Wi-Fi on the T-Embed before scanning (2.4 GHz coexistence) |
| No BLE devices nearby | Move to an area with known BLE devices (near your phone, smart watch) |
| Firmware bug | Update to latest Bruce firmware version |

### Cannot connect to a device

| Possible Cause | Solution |
|:---|:---|
| Device already connected to another Central | Disconnect from the other device first (e.g., unpair from phone) |
| Device not in connectable mode | Some devices only advertise, they do not accept connections from unknown devices |
| Connection timeout | Move closer to the device (within 2 m), retry |
| Pairing required | The device requires bonding which the T-Embed may not support for that device |
| Address rotation | The device's address changed between scan and connect attempt — rescan and retry immediately |

### BLE spam not working

| Possible Cause | Solution |
|:---|:---|
| Target has Bluetooth disabled | Must be enabled in Settings (not just Control Center on iOS) |
| Target OS updated | Apple patched many spam vectors in iOS 17.2+, Samsung in recent updates |
| Out of range | Move within 5--10 m of target |
| Wrong spam type selected | Ensure you selected the correct target platform (Apple/Samsung/Windows) |

### GATT enumeration incomplete

| Possible Cause | Solution |
|:---|:---|
| Services require encryption | Some services are hidden until pairing/bonding occurs |
| Connection dropped | The device may disconnect after timeout — reconnect and retry quickly |
| MTU too small | Some firmware versions negotiate a small MTU, limiting data transfer |
| Device rate limiting | Some devices limit GATT discovery requests — wait and retry |

### General BLE tips

> **Wi-Fi and BLE coexistence**: The ESP32-S3 shares the 2.4 GHz radio between Wi-Fi and BLE. Running both simultaneously can degrade performance. For best BLE results, disable Wi-Fi on the T-Embed before conducting BLE operations.
{: .tip }

> **Antenna orientation**: The T-Embed's PCB trace antenna is directional. Rotating the device can significantly affect BLE range and signal quality. If signal is weak, try different orientations.
{: .tip }

> **BLE range outdoors vs. indoors**: Expect 50--100 m range outdoors with line of sight, but only 10--20 m indoors through walls. Metal and water (including human bodies) significantly attenuate 2.4 GHz signals.
{: .note }

---

## Summary

| Capability | T-Embed Support | Notes |
|:---|:---|:---|
| BLE Scanning | Full | Discover all advertising devices |
| Advertisement Decoding | Full | Parse manufacturer data, service UUIDs, names |
| GATT Enumeration | Full | Connect, discover services, read characteristics |
| Apple BLE Spam | Full | Nearby Action popup spoofing |
| Samsung BLE Spam | Full | Galaxy accessory popup spoofing |
| Windows Swift Pair Spam | Full | Toast notification flooding |
| Tracker Detection | Full | Detect AirTags, SmartTags, Tile |
| Passive Connection Sniffing | **Not Supported** | Requires Ubertooth or nRF52840 |
| Classic Bluetooth | **Not Supported** | ESP32-S3 is BLE only |
| BLE Mesh | Limited | Depends on firmware implementation |

The T-Embed's BLE capabilities make it an excellent tool for device discovery, security assessment, and tracker detection. For passive sniffing of existing connections, complement it with dedicated BLE sniffing hardware.
