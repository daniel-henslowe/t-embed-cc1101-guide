---
layout: default
title: "05 — WiFi Operations"
nav_order: 6
---

# 05 — WiFi Operations
{: .fs-9 }

Turn the T-Embed CC1101 Plus into a pocket-sized WiFi reconnaissance tool. Scan networks, monitor packets, detect rogue access points, and test your own network's resilience — all from a device that fits in your palm.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## ESP32-S3 WiFi Capabilities

The T-Embed CC1101 Plus is built around the **ESP32-S3**, which includes a full WiFi transceiver alongside the CC1101 sub-GHz radio. Understanding its capabilities — and its limitations — is essential before diving into WiFi operations.

### Hardware Specifications

| Parameter | Value |
|:----------|:------|
| **Standard** | IEEE 802.11 b/g/n |
| **Band** | 2.4 GHz only (2.400–2.4835 GHz) |
| **Channels** | 1–14 (channel 14 restricted to Japan, 802.11b only) |
| **Maximum TX Power** | +20 dBm (100 mW) |
| **Antenna** | Onboard PCB trace antenna |
| **Modes** | Station (STA), Access Point (AP), STA+AP simultaneous |
| **Special Modes** | Promiscuous mode (raw 802.11 frame capture) |
| **Security** | WEP, WPA, WPA2 (Personal/Enterprise), WPA3 (limited) |
| **Max Connections (AP)** | 10 simultaneous stations |
| **Frequency Accuracy** | +/- 40 ppm |

### Operating Modes

The ESP32-S3 supports three primary WiFi operating modes, and Bruce firmware leverages all of them:

**Station Mode (STA)**
The T-Embed connects to an existing WiFi network as a client. Used for internet access, OTA firmware updates, NTP time sync, and downloading payloads.

**Access Point Mode (AP)**
The T-Embed creates its own WiFi network that other devices can connect to. Used for Evil Portal attacks, web-based configuration interfaces, and data exfiltration.

**STA+AP Simultaneous Mode**
The T-Embed can be connected to a WiFi network *and* host its own access point at the same time. This enables advanced scenarios like connected Evil Portals that route captured credentials to an external server while serving the phishing page locally.

**Promiscuous Mode**
The ESP32-S3 can be placed into promiscuous mode where it receives *all* 802.11 frames on a given channel, not just those addressed to it. This is the foundation for packet monitoring, probe request capture, and deauthentication attacks.

{: .note }
> Promiscuous mode on the ESP32-S3 is **not** the same as full monitor mode on a dedicated WiFi adapter. The ESP32 receives management and some data frames but cannot decode encrypted payloads, and packet injection capabilities are limited compared to cards using the Atheros or Ralink chipsets.

### What the ESP32-S3 CAN Do

- Scan for all visible 2.4 GHz access points (active and passive)
- Capture 802.11 management frames (beacons, probes, deauths, associations)
- Inject certain management frames (deauth, beacon, probe response)
- Operate as a rogue access point with captive portal
- Monitor packet counts and channel activity
- Capture probe requests to reveal device behavior
- Connect to networks and perform basic network operations
- Send and receive raw 802.11 frames in promiscuous mode

### What the ESP32-S3 CANNOT Do

- Operate on 5 GHz or 6 GHz bands (2.4 GHz only)
- Perform full WPA/WPA2 handshake capture reliably (limited buffer and processing)
- Crack WiFi passwords (no GPU, insufficient CPU for brute force)
- Run tools like `aircrack-ng`, `hashcat`, or `john` natively
- Achieve the same packet injection reliability as Atheros AR9271 or Ralink RT3070 chipsets
- Decode WPA2/WPA3 encrypted data frames without the key
- Perform advanced 802.11ax (WiFi 6) frame analysis
- Match the range and sensitivity of external high-gain WiFi adapters

{: .tip }
> Think of the T-Embed as your **pocket recon tool** — it excels at scanning, enumeration, and basic attacks. For heavy-duty WiFi penetration testing, pair it with a dedicated adapter like the Alfa AWUS036AXML running on Kali Linux. The T-Embed handles field reconnaissance, and Kali handles the exploitation.

---

## WiFi Scanning

WiFi scanning is the most fundamental WiFi operation and your starting point for any wireless assessment. The T-Embed can enumerate every 2.4 GHz network within range, revealing critical information about each one.

### Active vs Passive Scanning

**Active Scanning**
The T-Embed sends probe request frames on each channel, asking access points to identify themselves. This is faster (APs respond immediately) but is detectable — a WIDS (Wireless Intrusion Detection System) will see your probe requests and log your device's MAC address.

**Passive Scanning**
The T-Embed listens silently on each channel, waiting for beacon frames that access points broadcast approximately every 102.4 ms. This is slower (you need to dwell on each channel long enough to catch beacons) but is completely undetectable since no frames are transmitted.

{: .note }
> Bruce firmware uses **active scanning** by default for speed. For stealth assessments of your own network, consider that your device's MAC address will appear in AP association logs. Some Bruce builds support MAC randomization — check your firmware version.

### Understanding Scan Results

Each discovered network provides a wealth of information:

| Field | What It Tells You | Security Relevance |
|:------|:-------------------|:-------------------|
| **SSID** | Network name (human-readable) | Social engineering target, naming convention analysis |
| **BSSID** | MAC address of the access point (e.g., `AA:BB:CC:DD:EE:FF`) | Manufacturer identification (first 3 octets = OUI), rogue AP detection |
| **Channel** | Radio channel (1-14) the AP operates on | Channel overlap analysis, targeted monitoring |
| **RSSI** | Signal strength in dBm (e.g., -45 dBm = strong, -80 dBm = weak) | Physical proximity estimation, range mapping |
| **Encryption** | Security protocol: Open / WEP / WPA / WPA2 / WPA3 | Vulnerability assessment — Open and WEP are critical findings |

### Reading RSSI Values

| RSSI (dBm) | Signal Quality | Approximate Range |
|:------------|:---------------|:------------------|
| -30 to -50 | Excellent | Very close (same room) |
| -50 to -60 | Good | Nearby (adjacent room) |
| -60 to -70 | Fair | Moderate distance |
| -70 to -80 | Weak | Far away or obstructed |
| -80 to -90 | Very Weak | Edge of range |
| Below -90 | Unusable | May not even appear in scan |

### Hidden Network Detection

When an access point has SSID broadcast disabled, it still sends beacon frames — but with an empty (zero-length) SSID field. The T-Embed will display these as blank or `[Hidden]` entries in scan results.

Hidden networks are **not secure** — their SSID is revealed in:
- Probe responses (when a client connects)
- Association request frames
- Probe requests from devices that have previously connected

{: .warning }
> "Hidden" SSIDs are a cosmetic measure, not a security control. If you find hidden networks during a scan of your own infrastructure, do not rely on SSID hiding as a security mechanism. Use proper encryption (WPA3) and network segmentation instead.

### Channel Utilization Analysis

By scanning all channels, you can build a picture of 2.4 GHz spectrum utilization:

- **Channels 1, 6, and 11** are the only non-overlapping channels in the US/EU. Well-designed networks use only these three.
- Heavy concentration on one channel indicates potential congestion and interference.
- Unusual channel usage (e.g., channels 2-5, 7-10, 12-13) may indicate misconfigured or foreign devices.
- Channel overlap causes co-channel interference — relevant for your own network optimization.

### Identifying Rogue Access Points

A rogue AP is an unauthorized access point connected to your network. During authorized assessments, compare scan results against your known AP inventory:

1. **BSSID mismatch** — Unknown MAC addresses on your network's SSID
2. **Duplicate SSIDs** — Same network name on a BSSID you don't recognize (evil twin)
3. **Unexpected vendors** — Use the OUI (first 3 octets of BSSID) to identify the manufacturer. If your organization uses Cisco APs and you see a TP-Link BSSID broadcasting your SSID, that is a rogue.
4. **Unusual encryption** — Your network uses WPA2-Enterprise but a matching SSID shows WPA2-Personal. This is a classic evil twin indicator.
5. **Signal anomalies** — A familiar SSID suddenly appearing with much stronger signal than usual.

{: .tip }
> Create a baseline inventory of all authorized APs (BSSID, SSID, channel, encryption) on your network. Save this as a reference. On subsequent scans, any new entry is either a new legitimate AP or a rogue that needs investigation.

### Step-by-Step: WiFi Scanning with Bruce

1. Power on the T-Embed and navigate to the main menu using the rotary encoder
2. Select **WiFi** from the main menu
3. Select **Scan Networks** (or **WiFi Scan** depending on Bruce version)
4. The T-Embed will cycle through all 2.4 GHz channels — this takes 5-15 seconds
5. Results appear on the 1.9" display, sorted by signal strength (strongest first)
6. Scroll through results using the rotary encoder
7. Press the encoder button to view details of a selected network (BSSID, channel, encryption, exact RSSI)
8. To rescan, back out and select Scan again

### Exporting Scan Results

Depending on your Bruce firmware version, scan results can be saved to the onboard storage (SD card or SPIFFS):

1. After completing a scan, look for a **Save** or **Export** option in the menu
2. Results are typically saved as a `.txt` or `.csv` file
3. Connect the T-Embed via USB and access the filesystem to retrieve the file
4. Alternatively, if connected to WiFi, some Bruce builds offer a web interface for file download

{: .note }
> If your Bruce build does not support scan export natively, you can take photos of the display or note the results manually. For production security assessments, consider using the T-Embed for initial discovery and then performing detailed scanning with `airodump-ng` on Kali for comprehensive logging.

---

## Packet Monitor / Sniffer

The packet monitor puts the ESP32-S3 into promiscuous mode, allowing it to capture raw 802.11 frames on a selected channel. This provides real-time visibility into wireless activity.

### What Promiscuous Mode Captures

When the ESP32-S3 enters promiscuous mode, it receives **all** 802.11 frames on the selected channel, regardless of whether they are addressed to the device. The frames are passed to a callback function before being discarded by the hardware, allowing Bruce to inspect and display them in real-time.

### 802.11 Frame Types

All WiFi communication uses three categories of frames:

**Management Frames**
Control network membership and are always sent **unencrypted** (in WPA2 — WPA3 with PMF encrypts some of these):

| Frame Type | Purpose | What It Reveals |
|:-----------|:--------|:----------------|
| **Beacon** | AP announces its presence (~10/sec per AP) | SSID, BSSID, capabilities, supported rates |
| **Probe Request** | Client asks "Is [SSID] here?" | Client MAC, SSIDs the client has connected to previously |
| **Probe Response** | AP responds to probe request | AP details, available to specific client |
| **Authentication** | Client authenticates with AP | Authentication type, sequence |
| **Association Request** | Client requests to join network | Client capabilities, requested SSID |
| **Association Response** | AP accepts/denies client | Success/failure, assigned AID |
| **Deauthentication** | Disconnect notification | Reason code, can be spoofed |
| **Disassociation** | Disconnect from network | Reason code |

**Control Frames**
Manage access to the wireless medium:

| Frame Type | Purpose |
|:-----------|:--------|
| **RTS** (Request to Send) | Requests permission to transmit |
| **CTS** (Clear to Send) | Grants permission to transmit |
| **ACK** (Acknowledgment) | Confirms frame receipt |
| **Block ACK** | Acknowledges multiple frames |

**Data Frames**
Carry the actual payload (web traffic, file transfers, etc.). In WPA2/WPA3 networks, the payload is encrypted — the ESP32-S3 can see the frame headers (source/destination MAC, frame type) but **cannot** decrypt the payload without the network key.

{: .note }
> On an open (unencrypted) network, data frame payloads are visible in cleartext. This is one of many reasons why open WiFi is dangerous and why you should always use a VPN on public WiFi. On your own test network, you can observe this directly.

### Beacon Frame Analysis

Beacons are the most common frames you will see — every AP broadcasts approximately 10 per second. Analyzing beacons reveals:

- **SSID** — The network name (or empty if hidden)
- **BSSID** — The AP's MAC address
- **Timestamp** — AP uptime since last reboot
- **Beacon Interval** — Typically 102.4 ms (100 TU)
- **Capability Information** — ESS/IBSS, privacy (encryption), short preamble
- **Supported Rates** — 802.11b/g/n data rates the AP supports
- **Channel** — Current operating channel
- **RSN Information Element** — WPA2/WPA3 cipher suites, AKM (authentication) type
- **Vendor-Specific IEs** — Manufacturer features (WMM, WPS, etc.)

### Probe Request Monitoring

Probe requests are particularly valuable for security assessment. When devices have WiFi enabled, they periodically broadcast the names of networks they have previously connected to — essentially shouting "Is *MyHomeWiFi* here? Is *CoffeeShopFreeWiFi* here? Is *CorpNet-Internal* here?"

This reveals:
- **Device presence** — Which devices are nearby (by MAC address)
- **Network history** — SSIDs the device has connected to before
- **Movement patterns** — If you see the same MAC over time, the device/owner is recurring
- **Corporate targets** — Devices probing for enterprise SSIDs reveal employee presence

### Understanding the Packet Count Display

The Bruce packet monitor typically shows:

```
┌────────────────────┐
│ Packet Monitor     │
│ Channel: 6         │
│ ─────────────────  │
│ Mgmt:    1,247     │
│ Ctrl:      892     │
│ Data:    3,481     │
│ Total:   5,620     │
│ ─────────────────  │
│ Deauth:     12     │
│ Beacon:    834     │
│ Probe:     401     │
│                    │
│ [■■■■■■░░░░] 56%   │
└────────────────────┘
```

- **High beacon count** — Many APs on this channel (congested)
- **High probe count** — Many client devices searching for networks
- **Deauth frames present** — Someone may be running a deauth attack (or legitimate disconnections)
- **High data frame ratio** — Active data transfer on this channel

{: .warning }
> If you observe deauthentication frames you did **not** send during monitoring of your own network, this may indicate an active attack against your network. Investigate immediately — check for rogue APs, evil twins, or unauthorized devices.

### Limitations of ESP32-S3 Promiscuous Mode

- **Single channel** — The ESP32-S3 can only monitor one channel at a time. To cover all channels, you must hop manually or use channel-hopping firmware features.
- **2.4 GHz only** — No visibility into 5 GHz or 6 GHz traffic. Modern networks increasingly use 5 GHz, so you may miss significant activity.
- **No full packet capture** — The ESP32 has limited RAM. It can inspect frames in real-time but cannot save full PCAP files like `tcpdump` or `Wireshark` on a laptop.
- **Frame filtering** — The hardware may drop some frames under high traffic conditions due to buffer overflows.
- **No encrypted payload decoding** — Even with the network key, the ESP32 lacks the processing power to decrypt WPA2 traffic in real-time.

{: .tip }
> For comprehensive packet capture, use the T-Embed to identify the most interesting channel, then switch to a laptop running Kali Linux with `airodump-ng` or Wireshark for full PCAP capture and analysis.

---

## Probe Request Capture

Probe request capture is one of the most powerful passive reconnaissance techniques available on the T-Embed. It requires no interaction with any network and is completely silent.

### What Are Probe Requests?

When a WiFi-enabled device has its radio active, it periodically sends **probe request frames** on every channel. These frames contain:

- The device's MAC address (or randomized MAC on modern devices)
- A list of SSIDs the device has previously connected to (the Preferred Network List, or PNL)
- Supported data rates and capabilities

The purpose is efficiency — instead of waiting for beacon frames, the device proactively asks whether any known networks are nearby so it can reconnect faster.

### Privacy Implications

Probe requests are a significant privacy leak:

1. **Network history exposure** — A device probing for `HomeNet-5G`, `Hilton_WiFi`, `CorpNet-Secure`, and `Starbucks-Guest` reveals where the owner lives, travels, works, and hangs out.
2. **Device tracking** — Consistent MAC addresses allow tracking a device (and its owner) across locations and time. Modern iOS and Android use MAC randomization, but many IoT devices, laptops, and older phones still broadcast their real MAC.
3. **Evil twin enablement** — An attacker who sees a device probing for `FreeAirportWiFi` can create a fake AP with that exact name. The device will automatically connect, routing all traffic through the attacker.
4. **Organizational intelligence** — Devices probing for corporate SSIDs reveal which organizations have employees in the area.

### Capturing Probes with Bruce

1. Navigate to **WiFi** > **Probe Monitor** (or **Probe Request Capture**)
2. The T-Embed enters promiscuous mode and begins listening on the current channel
3. Detected probe requests appear on the display showing:
   - Source MAC address
   - Probed SSID
   - Signal strength (RSSI)
   - Timestamp
4. Scroll through captured probes with the rotary encoder
5. Some Bruce builds allow saving captured probes to storage

### Using Probe Capture for Security Awareness

On your own network, probe capture demonstrates:

- **Why WiFi should be turned off** when not in use — devices constantly broadcast their network history
- **Why MAC randomization matters** — compare probes from an iPhone (randomized) vs an old Android or IoT device (real MAC)
- **Why Preferred Network Lists should be pruned** — remove old/public network SSIDs from your devices
- **Why open networks are dangerous** — any probed open network name can be spoofed with an evil twin

{: .tip }
> Run a probe capture session in a meeting room at your organization (with authorization). Show colleagues how many SSIDs their devices are broadcasting. This is one of the most effective security awareness demonstrations you can perform.

---

## Beacon Flood (Educational — YOUR NETWORK ONLY)

Beacon flooding creates dozens or hundreds of fake access point advertisements, filling nearby devices' WiFi network lists with bogus entries.

### What Are Beacon Frames?

Beacon frames are management frames broadcast by access points approximately 10 times per second. They announce the AP's existence and contain:

- SSID (network name)
- BSSID (AP MAC address)
- Supported data rates
- Channel information
- Encryption type
- Timestamp

Any device can transmit beacon frames — the 802.11 standard does not authenticate them. This is the fundamental weakness that beacon flooding exploits.

### How Beacon Flooding Works

The T-Embed generates beacon frames with randomized BSSIDs and custom or random SSIDs, transmitting them rapidly. Nearby devices that scan for networks will see dozens of fake APs appear in their WiFi network list.

### Legitimate Use Cases

- **WIDS/WIPS testing** — Verify that your Wireless Intrusion Detection/Prevention System correctly identifies and alerts on beacon floods
- **Security awareness demonstrations** — Show stakeholders how easy it is to pollute the wireless environment
- **Incident response training** — Practice identifying and locating the source of a beacon flood
- **RF interference analysis** — Understand how beacon floods affect legitimate AP discovery

### Running Beacon Flood on Bruce

1. Navigate to **WiFi** > **Beacon Flood** (or **Beacon Spam**)
2. Select a mode:
   - **Random SSIDs** — Generates gibberish network names
   - **Custom List** — Uses a predefined list of SSIDs (often humorous or themed)
   - **Rickroll** — Generates SSIDs that spell out lyrics (a classic)
3. Select the channel to broadcast on (or leave on default)
4. Press the encoder to start — fake APs will begin appearing on nearby devices
5. Press again to stop

{: .note }
> The beacon flood will show up on any device within range that scans for WiFi networks. The fake networks cannot be connected to — they exist only as beacon advertisements with no backing infrastructure.

### Limitations and Detection

- **Range** — The PCB antenna limits effective range to approximately 10-30 meters indoors
- **No connectivity** — Devices that attempt to connect to fake APs will fail immediately
- **Easy to detect** — All fake beacons originate from the same physical location. A WIDS system or anyone with a directional antenna can locate the source.
- **Channel specific** — Beacons are broadcast on a single channel. APs on other channels are unaffected.
- **Modern device filtering** — Some modern devices filter out beacons that appear and disappear rapidly

{: .danger }
> Beacon flooding on networks you don't own is illegal and disruptive. It can interfere with legitimate network operations, cause denial of service, and violate FCC Part 15 regulations (intentional interference). This feature exists for security research on **YOUR OWN isolated network** only. Performing this in a public space or against someone else's network can result in criminal charges.

---

## Deauthentication (Educational — YOUR NETWORK ONLY)

Deauthentication is one of the most well-known WiFi attacks. It forces devices to disconnect from their access point by exploiting a fundamental design flaw in the 802.11 standard.

### How 802.11 Deauth Frames Work

In WPA2 (and earlier), **management frames are not encrypted or authenticated**. This means any device can forge a deauthentication frame that appears to come from a legitimate access point or client.

The attack flow:

```
1. Attacker observes AP (BSSID) and connected client (STA MAC)
2. Attacker forges deauth frame:
   - Source: AP's BSSID (spoofed)
   - Destination: Client's MAC (or broadcast FF:FF:FF:FF:FF:FF)
   - Reason code: 7 (Class 3 frame from nonassociated STA)
3. Client receives deauth and disconnects
4. Client attempts to reconnect (4-way handshake)
5. Attacker can repeat continuously to maintain denial of service
```

### Why This Works

The vulnerability is architectural — the 802.11 standard (prior to 802.11w) designed management frames to be unauthenticated. The reasoning was that deauth/disassociation frames need to work even when no encryption keys are established. Unfortunately, this makes them trivially spoofable.

### WPA3 and 802.11w (PMF) Protection

**802.11w (Protected Management Frames / PMF)** addresses this vulnerability by encrypting and authenticating management frames:

| Standard | Deauth Protection | Status |
|:---------|:-------------------|:-------|
| WPA2 (no PMF) | None — fully vulnerable | Still common |
| WPA2 (optional PMF) | Protected if both sides support it | Transitional |
| WPA2 (required PMF) | Deauth frames must be authenticated | More secure |
| WPA3 | PMF mandatory — deauth attacks blocked | Most secure |

{: .tip }
> Testing your own network against deauth attacks is one of the best arguments for upgrading to WPA3 or at minimum enabling PMF (802.11w) on your WPA2 network. If your APs and clients support it, enable PMF today.

### Running a Deauth Test on Your Own Network

{: .danger }
> Deauthentication attacks on networks you don't own violate the **Computer Fraud and Abuse Act (CFAA)**, **FCC regulations** (intentional interference, 47 U.S.C. Section 333), and equivalent laws in virtually every jurisdiction worldwide. This is a **federal crime** in the United States. You may **only** test networks you own or have explicit written authorization to test.

**Prerequisites:**
- You own the target network or have written authorization
- The test environment is isolated (no production devices will be affected)
- You have documented the test scope and approval

**Steps in Bruce:**

1. Navigate to **WiFi** > **Deauth** (or **Deauth Attack**)
2. First, run a WiFi scan to identify your target network
3. Select your target AP by BSSID
4. Bruce will display connected clients (discovered via data frame analysis)
5. Choose to deauth:
   - **Single client** — Target a specific device by MAC address
   - **All clients** — Broadcast deauth (destination `FF:FF:FF:FF:FF:FF`)
6. Set the number of deauth frames to send (or continuous mode)
7. Press the encoder to begin
8. Observe the target device(s) disconnect from the network
9. Press the encoder again to stop

### Understanding the Results

After running a deauth test on your own network, evaluate:

- **Did the device disconnect?** If yes, your network is vulnerable to deauth (no PMF)
- **How quickly did it reconnect?** Fast reconnection limits the attack's impact
- **Did your WIDS/WIPS detect it?** Check your wireless management console for deauth flood alerts
- **Are IoT devices affected?** Smart locks, cameras, thermostats — if they disconnect, your physical security may be compromised during an attack
- **What happens to VoIP calls?** WiFi-dependent phone calls drop during deauth attacks

### Implications for IoT Device Availability

Deauthentication attacks are particularly dangerous for IoT devices:

- **Security cameras** — An attacker deauths your WiFi camera, creating a blind spot, then physically enters
- **Smart locks** — Connectivity loss may prevent remote locking/unlocking
- **Alarm systems** — WiFi-dependent alarm panels may fail to report to the monitoring station
- **Medical devices** — WiFi-connected health monitors may lose data transmission

{: .warning }
> If your deauth test reveals that critical IoT devices are vulnerable, consider: (1) upgrading to WPA3/PMF-enabled APs, (2) using hardwired Ethernet for security-critical devices, (3) implementing cellular backup for alarm systems, (4) deploying a WIDS to detect deauth floods.

---

## Evil Portal

An evil portal (also called a captive portal attack or rogue AP with credential harvesting) combines AP mode, DNS hijacking, and a phishing page to capture credentials from users who connect to the fake network.

### What Is an Evil Portal?

An evil portal replicates the experience of connecting to a public WiFi network that requires login — like a hotel, airport, or coffee shop. The T-Embed:

1. Creates a WiFi access point with a chosen SSID (e.g., `Free_Airport_WiFi`)
2. Any device that connects receives an IP address via DHCP
3. All DNS queries are redirected to the T-Embed itself (DNS hijacking)
4. The user's browser is redirected to a phishing page hosted on the T-Embed
5. The user enters credentials (email, password, social login)
6. The T-Embed captures and stores the entered credentials

### How It Works — Technical Details

```
┌──────────┐         ┌──────────────────┐
│  Victim  │  WiFi   │   T-Embed AP     │
│  Device  │◄───────►│  (Evil Portal)   │
└────┬─────┘         └────────┬─────────┘
     │                        │
     │  1. Connect to AP      │
     │───────────────────────►│
     │                        │
     │  2. DHCP: Get IP       │
     │◄───────────────────────│
     │                        │
     │  3. DNS: any domain    │
     │───────────────────────►│
     │                        │
     │  4. DNS: → T-Embed IP  │
     │◄───────────────────────│
     │                        │
     │  5. HTTP GET /          │
     │───────────────────────►│
     │                        │
     │  6. Phishing page      │
     │◄───────────────────────│
     │                        │
     │  7. POST credentials   │
     │───────────────────────►│
     │                        │
     │  8. Stored on T-Embed  │
     │                        │
```

### Setting Up Evil Portal on Bruce

1. Navigate to **WiFi** > **Evil Portal** (or **Captive Portal**)
2. **Configure the SSID** — Choose a convincing network name:
   - Match a real nearby network (evil twin)
   - Use a generic public WiFi name (`Guest_WiFi`, `Free_Internet`)
   - Use a location-appropriate name (`Hotel_Lobby_WiFi`)
3. **Select a template** — Bruce includes several default portal templates:
   - Generic login page (email + password)
   - Social media login clone
   - WiFi terms of service acceptance
   - Corporate network login
4. **Start the portal** — The T-Embed creates the AP and begins serving the phishing page
5. The display shows the portal status: SSID, connected clients, and captured credentials
6. **Monitor** — As users connect and enter data, captured credentials appear on screen
7. **Stop** — Press the encoder button to shut down the portal

### Default Portal Templates

Bruce typically includes these templates (varies by firmware version):

| Template | Description | Captures |
|:---------|:------------|:---------|
| **Generic Login** | Simple email + password form | Email, password |
| **WiFi Login** | "Enter credentials to access WiFi" | Username, password |
| **Terms Accept** | "Accept terms to continue" | MAC address, acceptance |
| **Social Login** | Fake Google/Facebook login buttons | Social credentials |
| **Corporate** | Enterprise-style login page | Domain credentials |

### Custom Portal Creation

You can create custom portal templates for authorized testing:

1. Create an HTML file with your phishing page design
2. The form `action` should POST to the T-Embed's capture endpoint
3. Include CSS inline (no external resources — the T-Embed is the only server)
4. Keep the page lightweight — the ESP32-S3 has limited memory and storage
5. Upload the custom HTML to the T-Embed's storage (SD card or SPIFFS)
6. Select the custom template in the Evil Portal menu

```html
<!-- Example minimal evil portal page -->
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>WiFi Login Required</title>
  <style>
    body { font-family: -apple-system, sans-serif; margin: 0; padding: 20px;
           background: #f5f5f5; }
    .container { max-width: 400px; margin: 40px auto; background: white;
                 border-radius: 12px; padding: 32px; box-shadow: 0 2px 12px rgba(0,0,0,0.1); }
    h1 { font-size: 20px; text-align: center; margin-bottom: 24px; }
    input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd;
            border-radius: 6px; box-sizing: border-box; font-size: 16px; }
    button { width: 100%; padding: 14px; background: #007AFF; color: white;
             border: none; border-radius: 6px; font-size: 16px; cursor: pointer;
             margin-top: 16px; }
  </style>
</head>
<body>
  <div class="container">
    <h1>Connect to WiFi</h1>
    <p>Please sign in to access the internet.</p>
    <form method="POST" action="/capture">
      <input type="email" name="email" placeholder="Email address" required>
      <input type="password" name="password" placeholder="Password" required>
      <button type="submit">Sign In</button>
    </form>
  </div>
</body>
</html>
```

{: .note }
> Custom portals should be as minimal as possible. The ESP32-S3's web server has limited concurrent connections and memory. Avoid images, external fonts, or JavaScript frameworks. Inline everything.

### Viewing Captured Data

Captured credentials are stored on the T-Embed and can be accessed:

1. **On-device** — Scroll through captured entries on the display
2. **Via USB** — Connect to a computer and access the filesystem to read the capture log file
3. **Via web interface** — Some Bruce builds expose a web interface when connected to WiFi where you can download captured data

### Use Case: Authorized Security Awareness Testing

Evil portals are a powerful tool for testing your organization's security awareness — **with proper authorization**:

1. **Get written authorization** from management/IT leadership
2. **Define scope** — Which SSID, where, when, and for how long
3. **Notify incident response** — So they don't investigate the rogue AP as a real attack
4. **Deploy the portal** in a controlled area of your organization
5. **Track how many employees** enter credentials vs. recognize the phishing attempt
6. **Report findings** without naming individuals — the goal is to improve training, not to shame people
7. **Follow up** with security awareness training focused on WiFi security

### Detection and Defense Against Evil Portals

Teach your organization to recognize and defend against evil portals:

- **Verify network names** — Confirm the official WiFi SSID with staff before connecting
- **Check for HTTPS** — Legitimate login portals use HTTPS with valid certificates. Evil portals typically use HTTP or show certificate warnings.
- **Use a VPN** — Encrypt all traffic, making credential capture useless even on a compromised network
- **802.1X Enterprise authentication** — Real corporate networks use certificate-based authentication, not web portals
- **WIDS/WIPS** — Deploy wireless intrusion detection to alert on rogue APs matching your SSID
- **Never enter corporate credentials** on a WiFi login page — legitimate corporate networks use 802.1X, not web portals

{: .danger }
> Deploying an evil portal without explicit written authorization is illegal. Even in your own organization, unauthorized phishing tests can result in termination and criminal charges. Always get sign-off from legal, IT leadership, and HR before conducting any social engineering test.

---

## WiFi Password / Credential Attacks

Understanding what the T-Embed can and cannot do regarding WiFi password attacks is crucial for setting realistic expectations.

### What the T-Embed CAN Do

| Capability | Feasibility | Notes |
|:-----------|:------------|:------|
| Identify encryption type (Open/WEP/WPA2/WPA3) | Full | Standard scan functionality |
| Detect WPS-enabled networks | Partial | Some Bruce builds show WPS status |
| Capture EAPOL frames (handshake) | Very Limited | ESP32 buffer limitations cause frequent drops |
| PMKID capture | Very Limited | Requires specific frame timing that ESP32 often misses |
| Evil portal credential capture | Full | Social engineering, not protocol attack |
| Dictionary attack on captured hash | No | Insufficient CPU power |
| Brute force attack | No | Would take centuries on ESP32 |

### PMKID Capture Concept

PMKID (Pairwise Master Key Identifier) is a hash found in the first EAPOL frame of the WPA2 4-way handshake. Unlike a traditional handshake capture (which requires a client to connect), PMKID can be obtained by sending a single association request to the AP.

**In theory**, the T-Embed could:
1. Send an association request to the target AP
2. Capture the first EAPOL frame containing the PMKID
3. Save the PMKID hash for offline cracking

**In practice**, the ESP32-S3's WiFi stack has significant limitations:
- The EAPOL frame handling is not optimized for security testing
- Timing issues cause missed frames
- Buffer overflows under high traffic conditions
- Success rate is much lower than dedicated tools like `hcxdumptool`

### Why Full WPA Cracking Needs Kali

WPA2/WPA3 password cracking requires:

1. **Reliable handshake/PMKID capture** — Need a card with proper monitor mode and packet injection (`hcxdumptool` + Alfa adapter)
2. **Massive computational power** — A dictionary attack against a WPA2 hash tests thousands to millions of passwords per second using `hashcat` with GPU acceleration
3. **Large wordlists** — `rockyou.txt` (14 million passwords), custom wordlists, rule-based mutations
4. **GPU acceleration** — A modern GPU tests billions of WPA2 hashes per second. The ESP32-S3 at 240 MHz could test maybe 10-50 per second.

### Complementary Workflow: T-Embed + Kali

The T-Embed and Kali Linux are complementary, not competing tools:

```
T-Embed (Field Recon)              Kali Linux (Exploitation)
─────────────────────              ──────────────────────────
1. Scan all networks         ───►  4. Target identified APs
2. Identify targets                5. Capture handshake/PMKID
3. Note channel, BSSID,           6. Run hashcat/aircrack-ng
   encryption, client count        7. Crack password (if weak)
                                   8. Test network access
                                   9. Document and report
```

{: .tip }
> The T-Embed excels at **discreet, portable reconnaissance** — walk through an area, scan networks, capture probe requests, and document findings in seconds. Then return to your Kali workstation with target intelligence to conduct the actual penetration test. This two-phase approach is how professional wireless pentesters work.

---

## Network Reconnaissance Workflow

This section provides a complete, step-by-step methodology for using the T-Embed as a portable wireless reconnaissance tool during an **authorized** assessment.

### Prerequisites

- Written authorization to perform wireless assessment
- Defined scope (physical area, network targets, time window)
- T-Embed with latest Bruce firmware
- Fully charged battery pack
- Notebook or phone for documentation

### Phase 1: Passive Scan (5 minutes)

**Objective:** Identify all visible networks and build an initial target inventory.

1. Power on the T-Embed
2. Navigate to **WiFi** > **Scan Networks**
3. Run a full scan — note all discovered networks
4. For each network, record:
   - SSID
   - BSSID (first 3 octets identify the manufacturer — look these up later)
   - Channel
   - RSSI (signal strength)
   - Encryption type
5. Identify networks of interest:
   - Open networks (no encryption) — **critical finding**
   - WEP networks — **critical finding** (trivially crackable)
   - WPA2 without PMF — vulnerable to deauth
   - Hidden networks — note the BSSID for further investigation
   - Unexpected SSIDs — potential rogue APs

### Phase 2: Targeted Monitoring (10-15 minutes)

**Objective:** Observe traffic patterns on channels of interest.

1. Navigate to **WiFi** > **Packet Monitor**
2. Set the channel to match your target network
3. Observe for 3-5 minutes per channel:
   - Management frame count (beacons, probes, auth frames)
   - Data frame volume (indicates active usage)
   - Control frame patterns
   - Any deauth frames (indicates existing attacks or network issues)
4. Note the busiest channels and the most active BSSIDs

### Phase 3: Probe Request Capture (10-15 minutes)

**Objective:** Understand what devices are present and what networks they seek.

1. Navigate to **WiFi** > **Probe Monitor**
2. Capture for 10-15 minutes in areas of interest
3. Document:
   - Unique MAC addresses seen (device count)
   - SSIDs being probed (reveals device network history)
   - Devices probing for your organization's SSID (confirms employee presence)
   - Devices probing for open/insecure network names (susceptible to evil twin)

### Phase 4: Rogue AP Detection (5 minutes)

**Objective:** Compare scan results against the authorized AP inventory.

1. Re-run a WiFi scan
2. Compare every BSSID against the organization's authorized AP list
3. Flag any unknown BSSIDs broadcasting the organization's SSID (evil twin)
4. Flag any unknown BSSIDs on unexpected channels
5. Flag any APs with weaker encryption than policy requires

### Phase 5: Document Findings

**Objective:** Create an actionable report.

1. Export or photograph all scan results
2. Compile findings into categories:
   - **Critical** — Open/WEP networks, rogue APs, evil twins
   - **High** — WPA2 without PMF, devices probing for insecure networks
   - **Medium** — Hidden network reliance, excessive SSID broadcasting
   - **Low** — Channel congestion, suboptimal AP placement
3. Hand off target intelligence to the Kali Linux phase for deeper testing

{: .note }
> This entire reconnaissance workflow can be completed in 30-45 minutes. The T-Embed's pocket-sized form factor means you can perform this assessment walking through a building without drawing attention — which is exactly how a real adversary would do it. Understanding this is key to improving your defenses.

---

## Connecting the T-Embed to WiFi

Beyond offensive operations, the T-Embed itself can connect to WiFi networks for utility functions.

### Joining a Network from Bruce Settings

1. Navigate to **Settings** (or **Config**) > **WiFi** > **Connect**
2. The T-Embed scans for available networks
3. Select your network from the list using the rotary encoder
4. Enter the password using the on-screen keyboard:
   - Rotate to select characters
   - Press to confirm each character
   - Navigate to the submit/enter button when done
5. The T-Embed connects — connection status appears on the display
6. Some Bruce builds remember the last connected network for auto-reconnect

{: .tip }
> Entering a long WiFi password on the T-Embed's rotary encoder is tedious. If your Bruce build supports it, configure WiFi credentials via the web interface or by editing the configuration file on the SD card/SPIFFS directly through USB.

### Using WiFi for OTA Firmware Updates

Once connected to WiFi, the T-Embed can update its firmware over-the-air:

1. Connect to a WiFi network with internet access
2. Navigate to **Settings** > **Update** (or **OTA Update**)
3. The T-Embed checks for the latest Bruce firmware version
4. If an update is available, confirm to download and install
5. The device will download the firmware, flash it, and reboot

{: .warning }
> OTA updates download firmware from the internet. Only perform OTA updates on trusted networks. On a compromised or untrusted network, a man-in-the-middle could serve a malicious firmware image. For maximum security, flash firmware via USB using verified files.

### Web Interface Access

Some Bruce firmware versions expose a web interface when connected to WiFi:

1. Connect the T-Embed to your WiFi network
2. Note the IP address displayed on screen (e.g., `192.168.1.42`)
3. Open a browser on a device on the same network
4. Navigate to `http://192.168.1.42`
5. The web interface allows:
   - File management (upload/download from storage)
   - Configuration changes
   - Viewing captured data
   - Firmware upload

---

## WiFi as a Tool for Other Features

The ESP32-S3's WiFi capability enables several non-WiFi features of the T-Embed.

### Downloading Payloads for BadUSB

When connected to WiFi, the T-Embed can download Ducky Script payloads or BadUSB scripts from the internet:

- Download payloads from GitHub repositories
- Fetch updated scripts without USB connection to a computer
- Store downloaded payloads to the SD card or SPIFFS for later execution

### OTA Firmware Updates

As described above, WiFi enables over-the-air firmware updates, keeping your Bruce installation current with the latest features and bug fixes.

### Time Synchronization (NTP)

When connected to WiFi with internet access, the T-Embed can synchronize its clock via NTP (Network Time Protocol):

- Accurate timestamps on captured data
- Correct time display on the device
- Proper timing for scheduled operations

{: .note }
> Without NTP sync, the ESP32-S3's internal clock drifts over time. If accurate timestamps matter for your captured data (and they do for forensic purposes), connect to WiFi and sync time before starting a capture session.

### Data Exfiltration Channel

In authorized red team scenarios, WiFi provides a channel to exfiltrate captured data:

- Connect to a WiFi network and send captured RF signals, credentials, or scan data to a remote server
- Use as a covert channel during physical penetration tests
- Stream captured data in real-time to a C2 (command and control) server

{: .warning }
> Data exfiltration capabilities should only be used during authorized red team engagements with explicit scope definitions. Unauthorized data exfiltration is a criminal offense.

---

## Comparing T-Embed WiFi vs Dedicated WiFi Hacking

Understanding where the T-Embed fits in your toolkit prevents frustration and sets proper expectations.

| Capability | T-Embed (ESP32-S3) | Alfa AWUS036AXML + Kali |
|:-----------|:-------------------|:------------------------|
| **Network Scanning** | Basic — 2.4 GHz, SSID/BSSID/channel/RSSI/encryption | Advanced — 2.4 + 5 + 6 GHz, detailed capabilities, vendor ID |
| **Monitor Mode** | Promiscuous (limited frame types, single channel) | Full monitor mode (all frames, channel hopping) |
| **Packet Injection** | Limited — management frames only, unreliable timing | Full — reliable injection, precise timing, all frame types |
| **WPA Handshake Capture** | Very limited — frequent buffer overruns, low success rate | Full — `airodump-ng` captures 4-way handshakes reliably |
| **PMKID Capture** | Very limited — timing issues | Full — `hcxdumptool` captures PMKID reliably |
| **Deauthentication** | Basic — works but limited frame rate | Advanced — `aireplay-ng` with precise targeting and high frame rate |
| **Beacon Flood** | Yes — effective within PCB antenna range | Yes — more powerful with external antenna |
| **Evil Portal** | Yes — standalone, no laptop needed | Yes — more customizable with `hostapd` + `dnsmasq` |
| **WPA Cracking** | No — insufficient CPU/GPU | Full — `hashcat` with GPU, `aircrack-ng` with CPU |
| **Band Support** | 2.4 GHz only | 2.4 GHz + 5 GHz + 6 GHz (WiFi 6E) |
| **Portability** | Pocket-sized, battery-powered, standalone | Requires laptop, adapter, power |
| **Stealth** | Very discreet — looks like a USB device | Obvious — laptop + antenna |
| **Setup Time** | Instant — power on and go | Minutes — boot laptop, configure adapter, load tools |
| **Cost** | ~$25 | ~$50 (adapter) + laptop + Kali (free OS) |
| **PCAP Export** | No native full PCAP | Full Wireshark-compatible PCAP capture |

{: .tip }
> **Best practice for wireless penetration testing:** Use the T-Embed for initial reconnaissance (walk-through scanning, probe capture, rogue AP detection). Use the Alfa adapter + Kali for targeted exploitation (handshake capture, cracking, advanced packet analysis). The T-Embed is your eyes; Kali is your hands.

---

## Troubleshooting WiFi Operations

### No Networks Found

| Possible Cause | Solution |
|:---------------|:---------|
| WiFi radio not initialized | Restart the T-Embed; navigate to WiFi menu to trigger radio initialization |
| Antenna issue | Ensure no physical damage to the PCB trace antenna area (top of board) |
| All networks are 5 GHz | The ESP32-S3 only sees 2.4 GHz. If your test environment only has 5 GHz APs, you won't see them. |
| RF interference | Move away from sources of 2.4 GHz interference (microwaves, Bluetooth devices, other ESP32s) |
| Firmware bug | Update to the latest Bruce firmware version |
| Region lock | Some firmware builds restrict channels. Ensure your build supports channels 1-11 (US) or 1-13 (EU). |

### Weak Signal / Limited Range

| Possible Cause | Solution |
|:---------------|:---------|
| PCB antenna limitation | The onboard antenna has limited gain. Move closer to target APs. |
| Physical obstructions | Walls, floors, and metal objects attenuate 2.4 GHz signals. Line of sight improves range. |
| Body absorption | Your hand/body absorbs 2.4 GHz. Hold the T-Embed with the antenna area (top) pointed away from your body. |
| Low TX power setting | Check if Bruce has a TX power setting and ensure it's set to maximum (+20 dBm) |
| Battery level | Low battery can reduce TX power. Use a fully charged battery pack. |

{: .tip }
> For maximum WiFi range, hold the T-Embed with the top edge (where the PCB antenna is located) pointing toward your target area, away from your body. Even this simple orientation change can improve range by 20-30%.

### Deauth Not Working

| Possible Cause | Solution |
|:---------------|:---------|
| Target uses WPA3 | WPA3 mandates Protected Management Frames (PMF). Deauth frames are authenticated and cannot be spoofed. This is working as intended — the network is properly protected. |
| Target uses WPA2 with PMF | Some WPA2 networks enable 802.11w (PMF). Same protection as WPA3 for management frames. |
| Wrong channel | Ensure the T-Embed is on the same channel as the target AP. |
| Wrong BSSID | Double-check the target AP's BSSID. |
| Client not connected | You can only deauth clients that are currently associated. If no clients are connected, deauth has no effect. |
| Out of range | Move closer. The deauth frame must reach both the AP and the client. |
| Frame rate too low | The ESP32's frame injection rate may be too low to sustain deauth against aggressive auto-reconnection. |

### Evil Portal Not Loading

| Possible Cause | Solution |
|:---------------|:---------|
| DHCP not working | The victim device is not getting an IP address. Restart the Evil Portal feature. |
| DNS redirect failing | The captive portal detection on modern devices may bypass the DNS redirect. Try different portal templates. |
| HTTPS enforcement | Modern browsers and apps force HTTPS. The T-Embed serves HTTP only, causing connection errors. Some devices will show a "this connection is not private" warning. |
| Memory exhaustion | Too many simultaneous connections. The ESP32-S3 can handle approximately 4-8 concurrent web clients. |
| Template too large | Custom HTML pages exceeding available memory will fail to load. Keep pages under 10 KB. |
| Captive portal detection | iOS, Android, and Windows have captive portal detection that opens a mini-browser. If your page doesn't return the expected response, the device may not show the portal automatically. |

{: .note }
> Modern operating systems (iOS 14+, Android 11+, Windows 11) have increasingly sophisticated captive portal detection that can interfere with Evil Portal attacks. Apple devices, in particular, use a dedicated mini-browser for captive portals that has limited functionality. Test your portal templates against the specific device types you expect to encounter.

### Connection Drops During Operations

| Possible Cause | Solution |
|:---------------|:---------|
| Watchdog timer | Long-running WiFi operations may trigger the ESP32 watchdog. Update firmware — newer Bruce versions handle this better. |
| Memory leak | Extended packet capture can exhaust RAM. Restart the T-Embed periodically during long sessions. |
| Power supply | Unstable USB power or low battery can cause WiFi radio brownouts. Use a quality USB cable and power source. |
| Overheating | Extended TX operations (beacon flood, continuous deauth) can heat the ESP32. Allow the device to cool if it becomes warm to the touch. |
| Conflicting operations | Running WiFi and CC1101 operations simultaneously may cause instability on some firmware versions. Use one radio at a time. |

{: .warning }
> If the T-Embed becomes unresponsive during WiFi operations, hold the boot button and press reset (or disconnect/reconnect power). This forces a reboot without risking filesystem corruption. Avoid simply pulling power during write operations — this can corrupt the SD card or SPIFFS filesystem.

---

## Key Takeaways

{: .note }
> **Remember the hierarchy:** The T-Embed is a **reconnaissance** tool for WiFi, not a full exploitation platform. Use it to scan, enumerate, and perform basic attacks on YOUR OWN networks. For advanced WiFi penetration testing, pair it with Kali Linux and a dedicated WiFi adapter.

{: .danger }
> **Every WiFi operation described in this chapter that transmits frames (deauth, beacon flood, evil portal) is illegal if performed on networks you do not own or have explicit written authorization to test.** The consequences are real: criminal charges under the CFAA (up to 10 years imprisonment), FCC enforcement actions (fines up to $100,000+), civil liability, and termination of employment. Always operate within legal boundaries and with proper authorization.

---

*Next chapter: [06 — Bluetooth / BLE](../06-bluetooth/) — scanning, GATT enumeration, advertisement manipulation, and BLE tracking detection.*
