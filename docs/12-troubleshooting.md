---
layout: default
title: "12 — Troubleshooting & FAQ"
nav_order: 13
---

# 12 --- Troubleshooting & FAQ
{: .fs-9 }

Every common problem, its cause, and its fix. If your T-Embed is not doing what you expect, the answer is here.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc }

1. TOC
{:toc}

---

## Connection Issues

### Device Not Recognized by Computer

**Symptoms:** You plug in the T-Embed via USB-C and nothing happens. No new serial port appears. No sounds or notifications from your OS.

**Causes and Fixes:**

| Cause | Fix |
|:------|:----|
| **Charge-only USB cable** | This is the #1 cause. Many USB-C cables only carry power, not data. Use the cable that came with the device, or verify your cable supports data transfer by testing with a phone or another device. |
| **Faulty USB port** | Try a different USB port on your computer. Avoid USB hubs --- connect directly to the computer. |
| **USB-C orientation** | Try flipping the USB-C connector. Despite being reversible for power, some cables have intermittent data connections on one side. |
| **Driver not installed** | The ESP32-S3 uses a built-in USB-CDC driver on most modern operating systems. If it does not appear, install the CP210x or CH340 driver (depending on the USB-to-serial chip on your specific board revision). |

{: .tip }
> **Quick test:** A data-capable USB cable will allow you to transfer files to your phone. If the cable only charges your phone but does not show up as a media device, it is charge-only and will not work with the T-Embed.

### No Serial Port Appearing

**Symptoms:** The device powers on (screen lights up) but no COM port (Windows) or `/dev/ttyACM*` / `/dev/cu.usbmodem*` (macOS/Linux) appears.

**Fixes:**

1. **Enter boot mode manually:** Hold the **BOOT** button while plugging in USB. This forces the ESP32-S3 into download mode, which uses a different USB descriptor that is more widely recognized.

2. **Check Device Manager / System Information:**
   - **Windows:** Device Manager → Ports (COM & LPT). If you see a device with a yellow warning icon, right-click and update the driver.
   - **macOS:** Terminal → `ls /dev/cu.usb*` — look for new entries when you plug in.
   - **Linux:** Terminal → `ls /dev/ttyACM*` or `dmesg | tail` after plugging in.

3. **Install drivers explicitly:**
   - ESP32-S3 native USB: No driver needed on Windows 10+, macOS 12+, Linux 5.x+
   - If your board has a CH340G chip: [Download CH340 driver](http://www.wch-ic.com/downloads/CH341SER_ZIP.html)
   - If your board has a CP2102 chip: [Download CP210x driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

{: .note }
> The ESP32-S3 supports native USB (USB-CDC) without any external USB-to-serial chip. Most T-Embed CC1101 Plus boards use this native USB mode. If yours has an external chip, it will be visible as a small IC near the USB connector.

### Flashing Fails

**Symptoms:** You start a firmware flash and it fails with errors like "Failed to connect," "Timed out waiting for packet header," or "Wrong boot mode detected."

**Fixes:**

| Error | Solution |
|:------|:---------|
| "Failed to connect" | Hold BOOT button while clicking flash. Release BOOT after "Connecting..." appears. |
| "Timed out" | Reduce baud rate to 115200 (slower but more reliable). Ensure no other program has the serial port open. |
| "Wrong boot mode" | Device is not in download mode. Power cycle while holding BOOT. |
| "MD5 mismatch" | Corrupted firmware file. Re-download from the official source. |
| "Compressed data error" | Try a different USB cable or port. Data corruption during transfer. |

**Nuclear option:** If nothing works, erase the entire flash and start clean:

```bash
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* erase_flash
```

Then flash the firmware again from scratch.

{: .warning }
> Erasing flash deletes everything on the device --- firmware, saved signals, settings, everything. Only do this as a last resort.

### Device Keeps Rebooting (Boot Loop)

**Symptoms:** The screen flashes on and off repeatedly. You may see a brief splash screen before it resets. Serial monitor shows repeated crash dumps.

**Causes and Fixes:**

1. **Corrupted firmware:** Flash a known-good firmware version. Use the web flasher if available (e.g., [Bruce firmware web flasher](https://bruce.computer)).
2. **Power issue:** If running on battery, the battery may be too low to sustain operation. Plug in USB power.
3. **Incompatible firmware:** Make sure you are flashing firmware compiled for the T-Embed CC1101 **Plus** (ESP32-S3), not the original T-Embed (ESP32-S2) or a different board variant.
4. **Stack overflow / memory corruption:** If this started after custom firmware development, check for large stack allocations and buffer overflows. Increase task stack size in `sdkconfig`.

### USB Port Not Providing Enough Power

**Symptoms:** Device resets when transmitting, screen dims or flickers, erratic behavior under load.

**Fixes:**

- Use a USB port that provides at least 500 mA (USB 2.0 spec). USB 3.0 ports provide 900 mA and are preferable.
- Avoid unpowered USB hubs.
- Connect an external LiPo battery via the JST connector to supplement USB power.
- Reduce TX power in firmware settings if the issue only occurs during transmission.

---

## Sub-GHz Issues

### No Signals Detected

**Symptoms:** The frequency analyzer shows a flat line. Sub-GHz capture mode never triggers. You see nothing on any band.

**Fixes:**

| Check | What to Do |
|:------|:-----------|
| **Antenna** | Is the antenna attached? The CC1101 has essentially zero range without an antenna. Use the antenna that came with the device or a proper 433/868/915 MHz antenna. |
| **Frequency** | Are you scanning the right band? Consumer devices in the US primarily use 315 MHz and 915 MHz. In Europe, 433 MHz and 868 MHz are more common. |
| **Range** | Move closer to the transmitting device. Start at 1--2 meters and work outward. |
| **Modulation** | Try both AM (OOK/ASK) and FM (FSK) modes. If you are looking for a specific device, research its modulation type first. |
| **Interference** | Move away from computers, monitors, and switching power supplies. These can generate broadband noise that raises the noise floor. |
| **Dead CC1101** | Very rare, but possible. If the frequency analyzer shows no noise floor variation at all (perfectly flat), the CC1101 may not be communicating with the ESP32-S3. Check SPI connections. |

{: .tip }
> To verify your CC1101 is working, find a known-active frequency. A car key fob pressed within 5 meters should produce a clear signal spike on 315 MHz (US) or 433 MHz (EU). If even this does not work, suspect a hardware or antenna issue.

### Weak Signal Strength

**Symptoms:** Signals are detected but very faint. RSSI values are close to the noise floor. Captures are noisy or incomplete.

**Fixes:**

- **Antenna orientation:** The CC1101 antenna is directional. Rotate the device to find the strongest orientation. Vertical polarization is most common for consumer devices.
- **Antenna matching:** Use an antenna designed for your target frequency. A 433 MHz antenna will perform poorly at 915 MHz and vice versa.
- **Distance:** Move closer. RF signal strength drops with the square of the distance (inverse square law). Halving the distance quadruples the signal power.
- **Obstructions:** Metal walls, concrete, and water-containing objects (humans, plants) attenuate RF significantly. Move to line-of-sight if possible.
- **Battery power:** When running on a low battery, the CC1101's sensitivity may decrease. Charge or connect USB.

### Captured Signal Won't Replay

**Symptoms:** You successfully captured a signal, but when you replay it, the target device does not respond.

This is the most common frustration. There are several possible causes:

| Cause | How to Identify | Fix |
|:------|:----------------|:----|
| **Rolling code** | Each capture of the same button produces different data | Cannot replay. The device is properly secured. This is by design. |
| **Wrong modulation** | Captured in AM but device uses FM (or vice versa) | Try the other modulation type. Re-capture with the correct setting. |
| **Insufficient TX power** | Replay works at 1 meter but not at 5 meters | Move closer, or check TX power setting in firmware. The CC1101 maxes out at +12 dBm. |
| **Frequency drift** | Replay is on a slightly different frequency than the device expects | Some devices have tight frequency filters. Verify exact frequency with the analyzer and set it precisely. |
| **Incomplete capture** | Capture cut off before the full transmission completed | Extend the capture duration. Some transmissions are several hundred milliseconds long. |
| **Timing issues** | Replay timing does not exactly match the original | Some devices are sensitive to bit timing. Verify the capture's sample rate matches the replay rate. |

{: .note }
> Rolling codes are the most common reason replays fail on security-relevant devices. This is **not a bug** --- it is a feature. Garage doors, car key fobs, and modern alarm systems are designed to reject replayed signals. See [Chapter 4 --- Sub-GHz RF Operations](04-sub-ghz/) for details on rolling code systems.

### Frequency Analyzer Shows Nothing

**Symptoms:** Frequency analyzer runs but the display shows a flat noise floor with no peaks.

This may be **completely normal**. If there are no devices actively transmitting nearby, there will be no signals to see. The RF spectrum is not always active.

**Try this:**
1. Press a wireless doorbell, car key fob, or any known RF remote within a few meters
2. If the signal appears, your device is working --- there just are no ambient signals
3. If even this does not produce a spike, see "No Signals Detected" above

### Interference from Other Devices

**Symptoms:** The noise floor is elevated, signals are hard to distinguish, captures contain extra noise.

**Common interference sources:**
- Switching power supplies and chargers
- Computer monitors and displays
- Microwave ovens (2.4 GHz but can affect wideband)
- LED light dimmers
- Other radios and software-defined radios

**Fix:** Move to a different room, away from electronics. Outdoor scanning with battery power provides the cleanest results.

### Out-of-Range Frequency (Band Gaps)

**Symptoms:** You need to scan a frequency but the CC1101 will not tune to it.

The CC1101 has three supported bands with **gaps between them**:

```
Band 1: 300 — 348 MHz
         ╳ GAP: 349 — 386 MHz ╳
Band 2: 387 — 464 MHz
         ╳ GAP: 465 — 778 MHz ╳
Band 3: 779 — 928 MHz
```

{: .warning }
> Frequencies in the gaps (349--386 MHz and 465--778 MHz) are **physically impossible** for the CC1101 to receive or transmit on. No firmware update can change this --- it is a hardware limitation of the CC1101 chip. If your target device operates in these gaps, you need a different radio (RTL-SDR, HackRF, etc.).

---

## WiFi Issues

### No Networks Found

**Symptoms:** WiFi scan returns zero results.

**Fixes:**

- The ESP32-S3 only supports **2.4 GHz WiFi**. It cannot see or interact with 5 GHz networks. If all your networks are 5 GHz only, there will be nothing to see.
- Ensure the WiFi antenna is not damaged. On the T-Embed, the WiFi antenna is a trace on the PCB --- usually reliable unless the board is physically damaged.
- Try scanning in a location where you know WiFi networks exist (most urban areas have dozens).

### Deauth Not Working

**Symptoms:** You send deauthentication frames to your own device on your own network, but the device stays connected.

| Cause | Fix |
|:------|:----|
| **WPA3 / PMF enabled** | Protected Management Frames (802.11w) make deauth frames ineffective. This means the network is **properly secured**. You cannot bypass this. |
| **Wrong channel** | Deauth must be sent on the same channel as the target AP. Scan first to verify the channel. |
| **Out of range** | Deauth requires proximity to both the AP and the client. Move closer. |
| **Target is on 5 GHz** | If the client is connected on the 5 GHz band, the ESP32-S3 cannot reach it. |

{: .tip }
> If deauth does not work on your own network, that is good news --- your network is resistant to this attack. Confirm PMF is enabled in your router settings.

### Evil Portal Not Loading

**Symptoms:** You start an evil portal, but connected clients do not see the captive portal page.

**Fixes:**

1. **DHCP issue:** The T-Embed must act as both the access point and the DHCP server. Verify the DHCP server is running and assigning IP addresses.
2. **DNS issue:** The captive portal works by intercepting DNS queries and redirecting to the local portal page. Verify the DNS server is running and resolving all domains to the portal IP.
3. **Browser issues:** Modern browsers (Chrome, Firefox) use DNS-over-HTTPS (DoH), which bypasses the local DNS interceptor. The portal may not trigger automatically on these browsers.
4. **HTTPS redirect:** Many sites force HTTPS. The portal cannot intercept HTTPS connections without triggering browser security warnings. Target HTTP-only captive portal detection URLs.

### Can't Connect to WiFi

**Symptoms:** The T-Embed cannot join a WiFi network you specify.

**Fixes:**

- Double-check the password. The on-device keyboard can make it easy to mistype.
- Ensure the network is 2.4 GHz. The ESP32-S3 does not support 5 GHz.
- Check for MAC address filtering on the router. Add the T-Embed's MAC address to the allow list.
- Some enterprise networks (802.1X / WPA-Enterprise) require certificate-based authentication that the basic firmware does not support.

---

## Bluetooth Issues

### No BLE Devices Found

**Symptoms:** BLE scan returns zero devices.

**Causes:**

- Most BLE devices only advertise when in **pairing mode**. Your Bluetooth headphones, for example, may only advertise when actively seeking a connection.
- Some devices use directed advertising (advertising to a specific address), which passive scans cannot see.
- BLE range is relatively short indoors (5--15 meters through walls).

**Fix:** Put a known BLE device into pairing mode and scan. If it appears, your BLE radio is working and other devices are simply not advertising.

### BLE Spam Not Working on Target

**Symptoms:** You send BLE advertisement spam (Apple notification spam, etc.) but the target device does not react.

| Cause | Fix |
|:------|:----|
| **iOS version** | Apple has patched many BLE spam vulnerabilities in recent iOS updates. Devices on iOS 17.2+ are largely immune. |
| **Android settings** | Many Android devices allow disabling notification pop-ups for nearby devices. |
| **Distance** | BLE spam requires close proximity (typically within 5 meters). |
| **Not an Apple/Samsung device** | The spam payloads are vendor-specific. Apple payloads do not affect Samsung devices and vice versa. |

### Connection Drops During GATT Enumeration

**Symptoms:** You connect to a BLE device and start reading services/characteristics, but the connection drops partway through.

**Causes:**
- The target device has a short connection timeout
- Too many rapid GATT read requests overwhelm the target
- The target device intentionally disconnects unknown clients

**Fix:** Add delays between GATT read operations. Some devices need 100--500 ms between requests to maintain a stable connection.

---

## IR Issues

### IR Capture Doesn't Decode

**Symptoms:** The IR receiver detects a signal (the raw waveform appears) but the firmware cannot identify the protocol.

**Fixes:**

- **Unknown protocol:** Not all IR protocols are in the firmware's database. The signal may still work as a raw capture.
- **Weak signal:** Move the remote closer (5--10 cm) and point it directly at the receiver. Ensure the remote has fresh batteries.
- **Ambient IR interference:** Bright sunlight, fluorescent lights, and CFL bulbs emit IR noise that can interfere with capture. Move to a dimmer room.
- **Wrong angle:** IR is line-of-sight. Point the remote directly at the IR receiver on the T-Embed.

{: .tip }
> Even when a protocol is not recognized, the raw capture usually replays perfectly. Save it as a raw signal and test the replay. Protocol identification is nice to have but not required for functionality.

### IR Replay Doesn't Work

**Symptoms:** You captured a signal successfully, but replaying it does not control the target device.

**Fixes:**

| Issue | Solution |
|:------|:---------|
| **Distance** | IR replay range is typically 3--5 meters. Move closer to the target. |
| **Angle** | Point the T-Embed's IR LED directly at the device's IR receiver. Most devices have the receiver behind a dark plastic window on the front. |
| **IR LED power** | The T-Embed's IR LED is less powerful than most dedicated remotes. You need to be closer and more precisely aimed. |
| **Wrong protocol parameters** | If the signal was captured as raw, verify the carrier frequency (usually 38 kHz but some devices use 36 or 40 kHz). |
| **Toggle bit** | Some protocols (RC-5, RC-6) include a toggle bit that alternates on each button press. If you capture one press, the next replay may need the opposite toggle. Capture two consecutive presses to get both states. |

### TV-B-Gone Only Works on Some TVs

**Symptoms:** The TV-B-Gone function turns off some TVs but not others.

This is expected. TV-B-Gone works by cycling through a large database of known IR power codes. It covers the majority of TV brands but not all. Newer or less common brands may not be in the database.

**To improve coverage:**
- Capture the missing TV's power code manually and add it to your library
- Contribute the captured code to the firmware's open-source database

---

## BadUSB Issues

### Keyboard Not Recognized by Target

**Symptoms:** You plug the T-Embed into a target computer (yours) and the BadUSB payload does not execute. The computer does not recognize a keyboard.

**Fixes:**

1. **USB enumeration delay:** Add a `DELAY 2000` (2 seconds) at the start of your payload. The target OS needs time to detect and install the USB HID device.
2. **USB mode:** Ensure the firmware is configured for HID mode, not serial/CDC mode. Some firmware requires you to select BadUSB mode before connecting.
3. **Target OS restrictions:** Some corporate machines have USB device control policies that block new HID devices. This is a security feature working as intended.

### Wrong Characters Typed

**Symptoms:** The payload types, but the wrong characters appear. For example, `Z` appears instead of `Y`, or special characters are wrong.

**Cause:** Keyboard layout mismatch. The T-Embed sends USB HID scancodes based on a US QWERTY layout. If the target computer uses a different layout (UK, German QWERTZ, French AZERTY, etc.), the characters will be wrong.

**Fix:**

- Set the keyboard layout in the firmware settings to match the target OS layout
- For payloads that must work across layouts, use the `STRING` command sparingly and prefer `GUI`, `ALT`, and scancode-based commands that are layout-independent

### Script Runs Too Fast

**Symptoms:** The payload runs but skips characters, misses keystrokes, or triggers the wrong UI elements because the OS has not caught up.

**Fix:** Add `DELAY` commands liberally:

```
DELAY 2000          // Wait for USB enumeration
GUI r               // Open Run dialog
DELAY 500           // Wait for Run dialog to appear
STRING powershell   // Type command
DELAY 200           // Wait for text to appear
ENTER               // Execute
DELAY 1500          // Wait for PowerShell to open
```

{: .note }
> Slower computers need longer delays. A script that works on a fast SSD-equipped machine may fail on an older HDD-based system because the OS takes longer to respond.

### UAC/Sudo Blocks Execution

**Symptoms:** The payload runs but a UAC prompt (Windows) or sudo password prompt (macOS/Linux) appears and blocks further execution.

This is **expected behavior**. Modern operating systems require explicit permission for elevated actions. Your payload options:

1. **Bypass UAC (Windows):** Use techniques like `fodhelper.exe` bypass (works on many Windows versions but may be patched).
2. **Avoid elevation:** Design payloads that work within user-level permissions (data exfiltration, reverse shells as user).
3. **Social engineering:** Include a convincing UAC prompt message in the payload (only for authorized testing).
4. **Accept the limitation:** Some actions simply require elevated privileges and cannot be automated without them.

---

## Display Issues

### Screen Flicker

**Symptoms:** The TFT display flickers, strobes, or has inconsistent brightness.

**Fixes:**

- **Power supply:** Ensure stable power delivery. Flickering often indicates voltage drops. Try a different USB cable or power source.
- **Loose connection:** If the display flex cable is loose (rare on factory-assembled units), gentle pressure on the connector may help. Do not force anything.
- **PWM backlight:** Some brightness levels cause visible flicker due to PWM frequency. Try maximum brightness, which bypasses PWM entirely.

### White or Black Screen

**Symptoms:** The display shows solid white, solid black, or does not display anything at all.

**Fixes:**

1. **Firmware corruption:** Reflash the firmware. This is the most common cause.
2. **Wrong firmware:** Make sure you are using firmware compiled for the correct display driver (ST7789, 170x320 resolution). Using firmware for a different board will produce a blank screen.
3. **Reset:** Try holding the reset button for 10 seconds, or disconnect all power (USB and battery) for 30 seconds.

### Display Artifacts

**Symptoms:** Random colored pixels, garbled text, partial screen updates, or corrupted graphics.

**Fixes:**

- Reset the device (press reset button or power cycle)
- Reflash the firmware
- If this happens only during heavy radio TX, it may be EMI from the CC1101 affecting the display's SPI bus. Reduce TX power.

---

## Hardware Issues

### Overheating During Heavy TX

**Symptoms:** The device feels hot to the touch, especially near the CC1101 chip, during continuous transmission.

This is **normal behavior** for continuous TX at high power. The CC1101 draws significant current during transmission.

**Mitigations:**

- Reduce TX duty cycle (do not transmit continuously)
- Lower TX power setting
- Allow cooling breaks between transmissions
- Do not operate in direct sunlight

{: .warning }
> If the device becomes too hot to hold comfortably, stop transmitting immediately and let it cool. Sustained overheating can reduce the lifespan of the CC1101 and other components.

### Battery Not Charging

**Symptoms:** You connect a LiPo battery via JST, but it does not charge when USB is connected, or the battery percentage does not increase.

**Fixes:**

- **JST polarity:** LiPo JST connectors do not have a universal polarity standard. Verify that your battery's red (+) and black (-) wires match the T-Embed's JST pinout. **Reversed polarity can damage the board.**
- **Battery voltage:** If the battery is deeply discharged (below 3.0V), the charging circuit may not start. Try a different battery.
- **Charge IC:** The T-Embed uses a TP4054 or similar linear charger. Charging is slow (around 500 mA max) and may take several hours for a fully depleted battery.

{: .danger }
> **Never force a JST connector that does not fit easily.** If you need to swap polarity, carefully swap the wires in the JST connector housing using a pin extraction tool. Do not cut and splice near the battery.

### Rotary Encoder Skipping

**Symptoms:** The rotary encoder skips steps, registers double inputs, or moves erratically.

**Fixes:**

- **Debouncing:** If you are writing custom firmware, add software debouncing (10--20 ms debounce time).
- **Dirty contacts:** Rapidly rotate the encoder back and forth 20--30 times. This can clean oxidation from the contacts.
- **Firmware update:** Some firmware versions have better encoder handling than others. Update to the latest version.

### Speaker Too Quiet or No Sound

**Symptoms:** RTTTL tones or alert sounds are very quiet or completely silent.

**Fixes:**

- Check volume/sound settings in the firmware menu
- The built-in speaker is small and not very loud by design. It is intended for alerts, not audio playback.
- If there is no sound at all, the speaker GPIO pin may not be configured correctly in the firmware. Check pin definitions.

---

## Firmware Issues

### OTA Update Fails

**Symptoms:** Over-the-air firmware update starts but fails partway through.

**Fixes:**

- **WiFi connection:** Ensure a stable WiFi connection. OTA updates require downloading the full firmware image. If WiFi drops during transfer, the update fails.
- **File size:** Some OTA implementations have size limits. If the firmware binary exceeds the OTA partition size, the update will fail. Use web flasher instead.
- **Timeout:** Large firmware files on slow connections may time out. Move closer to the WiFi access point.
- **Power:** Ensure the device has sufficient power. Do not rely on a low battery during OTA updates. Connect USB power.

{: .warning }
> If an OTA update fails partway through, the device should boot back to the previous firmware. If it does not (boot loop), you will need to flash via USB.

### Features Missing After Update

**Symptoms:** After updating firmware, some features or menu items that were present before are now missing.

**Causes:**

- **Different firmware:** You may have switched between different firmware projects (e.g., Bruce to ESP-Flipper or vice versa). Different firmware has different feature sets.
- **Clean flash vs. OTA:** A clean flash erases all saved data (signals, settings, payloads). An OTA update preserves data but may have different defaults.
- **Feature flags:** Some firmware builds include different features based on compile-time options. Verify you are using the correct build variant.

### Custom Firmware Won't Compile

**Symptoms:** You are trying to build custom firmware and the compiler throws errors.

**Common Fixes:**

| Error Type | Likely Cause | Fix |
|:-----------|:-------------|:----|
| Library not found | Missing dependency | Install the required library via Arduino Library Manager or `platformio lib install` |
| Board not found | Board package not installed | Add the ESP32 board package URL to Arduino IDE preferences or `platformio.ini` |
| "multiple definition" | Library version conflict | Pin specific library versions in your project configuration |
| "PSRAM not found" | Wrong board config | Select the correct board variant with PSRAM (ESP32-S3 with OPI PSRAM) |
| Stack overflow / watchdog | Insufficient task stack | Increase stack sizes in `sdkconfig` or `menuconfig` |

**Environment setup checklist:**

1. Arduino IDE 2.x or PlatformIO with ESP32 board package v2.0.14+
2. ESP32-S3 board selected with correct flash size (16MB) and PSRAM (8MB OPI)
3. Partition scheme: "16MB Flash (3MB APP)" or custom
4. All required libraries at compatible versions

### Partition Table Errors

**Symptoms:** Flashing fails with partition-related errors, or the device has less storage than expected.

**Fix:** Erase the entire flash before flashing new firmware that uses a different partition layout:

```bash
esptool.py --chip esp32s3 --port /dev/cu.usbmodem* erase_flash
```

Then flash the new firmware including the partition table, bootloader, and application binary.

---

## FAQ
{: .fs-8 }

### General

<details><summary>Q: Can the T-Embed CC1101 open my garage door?</summary>

<strong>A:</strong> Only if your garage door uses a <strong>fixed-code</strong> system. The T-Embed can capture and replay fixed codes. However, the vast majority of garage door openers manufactured after 2000 use <strong>rolling codes</strong> (Security+, Security+ 2.0, KeeLoq, etc.), which change with every button press and cannot be replayed. If your garage door uses rolling codes, the T-Embed cannot open it --- and that is exactly how it should be.
</details>

<details><summary>Q: Is this device legal to own?</summary>

<strong>A:</strong> Yes. Owning the T-Embed CC1101 Plus is completely legal in the United States and most countries. It is a general-purpose development board with a radio transceiver. The legality depends on <strong>how you use it</strong>, not whether you own it. See <a href="13-legal-reference/">Chapter 13 — Legal Reference</a> for details on legal and illegal uses.
</details>

<details><summary>Q: How does this compare to the Flipper Zero?</summary>

<strong>A:</strong> The T-Embed CC1101 Plus provides approximately 90% of the Flipper Zero's core functionality at about 15% of the price. Key differences:

<ul>
<li><strong>T-Embed advantages:</strong> WiFi support, color TFT display, full Arduino/ESP-IDF programmability, lower cost (~$25 vs ~$170)</li>
<li><strong>Flipper Zero advantages:</strong> Built-in battery, 125 kHz RFID/NFC (iButton, LFRFID), more polished UI, larger community, dedicated mobile app, sub-1 GHz antenna is better tuned</li>
<li>The T-Embed lacks NFC/RFID capabilities entirely, which is the biggest functional gap</li>
</ul>
</details>

<details><summary>Q: Can I use this for penetration testing professionally?</summary>

<strong>A:</strong> Yes, but with important caveats. The T-Embed is a capable tool for RF, WiFi, and BLE security assessments. However, for professional engagements you must have <strong>written authorization</strong> from the client, a defined scope, and appropriate insurance. Most professional pentesters use this as one tool among many. Consider certifications like CEH, OSCP, or GPEN to formalize your skills and demonstrate competence to clients.
</details>

<details><summary>Q: What frequencies can it transmit on?</summary>

<strong>A:</strong> The CC1101 supports three bands: 300–348 MHz, 387–464 MHz, and 779–928 MHz. However, <strong>legal</strong> transmission is limited to ISM bands within these ranges: 315 MHz (limited in US), 433.92 MHz (ISM worldwide, limited power in US), 868 MHz (EU ISM), and 902–928 MHz (US ISM). Always check your country's regulations before transmitting.
</details>

<details><summary>Q: Does it receive outside those three bands?</summary>

<strong>A:</strong> No. The CC1101's frequency synthesizer physically cannot tune outside those three bands. The gaps (349–386 MHz and 465–778 MHz) are inaccessible. For broadband reception, consider an RTL-SDR v4 (~$30) which covers 24–1766 MHz as a receive-only device.
</details>

### Range and Antenna

<details><summary>Q: What is the maximum range?</summary>

<strong>A:</strong> At maximum TX power (+12 dBm) with the stock antenna, expect:
<ul>
<li><strong>Indoors:</strong> 10–30 meters through walls</li>
<li><strong>Outdoors (line of sight):</strong> 50–200 meters depending on frequency and antenna</li>
<li><strong>With a better antenna:</strong> Range can increase significantly (a proper 1/4 wave whip or directional antenna can double or triple range)</li>
</ul>
Range varies enormously based on frequency, antenna, environment, and target device sensitivity.
</details>

<details><summary>Q: Can I use a different antenna?</summary>

<strong>A:</strong> Yes. The T-Embed CC1101 Plus has an SMA connector for the Sub-GHz antenna. You can use any antenna with the correct SMA connector type. For best results, use an antenna tuned to your target frequency. A 433 MHz 1/4 wave whip (~17 cm) is a good general-purpose upgrade for Band 2. For 915 MHz, use a shorter antenna (~8 cm).
</details>

<details><summary>Q: Can I add an external WiFi antenna?</summary>

<strong>A:</strong> The ESP32-S3's WiFi uses a PCB trace antenna that is not designed to be replaced. Some advanced users have added a U.FL connector and pigtail to an external 2.4 GHz antenna, but this requires soldering to the PCB and is not officially supported. For most use cases, the built-in antenna is sufficient.
</details>

### Battery and Power

<details><summary>Q: What battery should I use?</summary>

<strong>A:</strong> Any single-cell 3.7V LiPo battery with a JST-PH 2.0mm connector will work. Recommended capacities:
<ul>
<li><strong>500 mAh:</strong> 1–2 hours of active use, very compact</li>
<li><strong>1000 mAh:</strong> 3–4 hours, good balance of size and life</li>
<li><strong>2000+ mAh:</strong> 6+ hours, but physically larger than the device itself</li>
</ul>
<strong>Critical:</strong> Verify JST connector polarity before connecting. There is no universal standard and reversed polarity will damage the board.
</details>

<details><summary>Q: How long does the battery last?</summary>

<strong>A:</strong> It depends heavily on what you are doing:
<ul>
<li><strong>Idle with display on:</strong> 6–10 hours (1000 mAh)</li>
<li><strong>Active WiFi scanning:</strong> 3–4 hours</li>
<li><strong>Continuous Sub-GHz TX:</strong> 2–3 hours</li>
<li><strong>BLE scanning:</strong> 4–6 hours</li>
<li><strong>Display off, passive RX:</strong> 8–12 hours</li>
</ul>
</details>

<details><summary>Q: Can I charge while using the device?</summary>

<strong>A:</strong> Yes. When USB is connected, the device runs on USB power and charges the battery simultaneously. However, heavy operations (continuous TX) may draw more current than USB provides, causing the device to supplement from the battery even while "charging."
</details>

### Safety and Ethics

<details><summary>Q: Can I get in trouble for owning this?</summary>

<strong>A:</strong> No. The device itself is legal. It is equivalent to owning a screwdriver — perfectly legal to own, but illegal to use for burglary. What matters is how you use it. Do not transmit on frequencies you are not authorized to use, do not access networks or devices you do not own, and always comply with local regulations.
</details>

<details><summary>Q: Can this hack my neighbor's WiFi password?</summary>

<strong>A:</strong> The T-Embed cannot crack WiFi passwords directly. It can capture WPA handshakes (deauth + monitor), but the actual password cracking requires a powerful computer running hashcat or aircrack-ng. More importantly, <strong>accessing someone else's WiFi network without permission is a federal crime</strong> under the CFAA, regardless of how easy or difficult it is technically.
</details>

<details><summary>Q: Is it safe to use around medical devices?</summary>

<strong>A:</strong> Exercise extreme caution. <strong>Never transmit RF near medical devices</strong> such as pacemakers, insulin pumps, or infusion pumps. While the T-Embed's power output is low (+12 dBm max), any RF transmission near sensitive medical equipment carries risk. Keep the device in receive-only mode in hospitals and medical facilities.
</details>

<details><summary>Q: Can this jam signals?</summary>

<strong>A:</strong> Technically, the CC1101 can transmit continuously on a frequency, which would cause interference. However, <strong>jamming is a serious federal crime</strong> that the FCC aggressively prosecutes. Penalties include fines up to $100,000+ and imprisonment. Do not jam any radio communications, ever, for any reason. The penalties for jamming are among the most severe in FCC enforcement.
</details>

<details><summary>Q: What certifications should I pursue to do this professionally?</summary>

<strong>A:</strong> Recommended certifications for RF and wireless security testing:
<ul>
<li><strong>CEH (Certified Ethical Hacker)</strong> — Good starting point, covers wireless testing basics</li>
<li><strong>OSCP (Offensive Security Certified Professional)</strong> — Hands-on pentest skills, industry gold standard</li>
<li><strong>GPEN (GIAC Penetration Tester)</strong> — Comprehensive network penetration testing</li>
<li><strong>CWSP (Certified Wireless Security Professional)</strong> — WiFi-specific security</li>
<li><strong>Amateur Radio License (FCC Technician)</strong> — Required knowledge for legal RF operation and testing, expands your legal frequency access</li>
</ul>
</details>

### Firmware

<details><summary>Q: Which firmware should I use?</summary>

<strong>A:</strong> For most users, <strong>Bruce firmware</strong> is the best starting point. It has the most comprehensive feature set, active development, good documentation, and supports the T-Embed CC1101 Plus natively. Advanced users may want to try ESP-Flipper or write custom firmware using Arduino or ESP-IDF.
</details>

<details><summary>Q: Can I brick the device?</summary>

<strong>A:</strong> It is very difficult to permanently brick an ESP32-S3. Even if you flash corrupt firmware, you can always recover by:
<ol>
<li>Holding the BOOT button while connecting USB (enters download mode)</li>
<li>Erasing the flash with esptool</li>
<li>Flashing known-good firmware</li>
</ol>
The only way to truly brick the device is physical damage to the hardware.
</details>

<details><summary>Q: Can I run multiple firmware versions?</summary>

<strong>A:</strong> Not simultaneously, but you can flash different firmware as needed. The process takes about 60 seconds per flash. Some advanced users set up custom partition tables with multiple firmware images and a bootloader menu, but this requires significant embedded development experience.
</details>

<details><summary>Q: How do I update the firmware?</summary>

<strong>A:</strong> Three methods, from easiest to most flexible:
<ol>
<li><strong>Web flasher:</strong> Visit the firmware's web flasher page, connect via USB, click flash. Works in Chrome/Edge.</li>
<li><strong>OTA update:</strong> Some firmware supports over-the-air updates via WiFi. Check the firmware's settings menu.</li>
<li><strong>Manual flash:</strong> Use esptool.py or Arduino IDE to flash the binary directly. Most control, most complex.</li>
</ol>
</details>

### Capabilities

<details><summary>Q: Can this read NFC cards or RFID badges?</summary>

<strong>A:</strong> No. The T-Embed CC1101 Plus has no NFC (13.56 MHz) or LFRFID (125 kHz) hardware. These require dedicated reader modules (PN532 for NFC, or a 125 kHz reader for LFRFID). You could potentially add these via the GPIO expansion header, but it is not built in. This is the primary functional gap compared to the Flipper Zero.
</details>

<details><summary>Q: Can this receive FM radio or broadcast TV?</summary>

<strong>A:</strong> No. FM broadcast radio operates at 88–108 MHz, which is outside all three CC1101 bands. Broadcast TV operates at various frequencies also outside the CC1101's range. For these, you need a broadband receiver like an RTL-SDR.
</details>

<details><summary>Q: Can this work with LoRa devices?</summary>

<strong>A:</strong> Partially. LoRa operates on frequencies within the CC1101's bands (433 MHz or 868/915 MHz), but LoRa uses a proprietary chirp spread spectrum modulation that the CC1101 cannot natively demodulate. You can detect LoRa transmissions in the frequency analyzer and see their signal strength, but you cannot decode the data. For LoRa interaction, you would need a dedicated LoRa module (SX1276/SX1278) connected via GPIO.
</details>

<details><summary>Q: Can I use this to clone hotel key cards?</summary>

<strong>A:</strong> No. Hotel key cards typically use either magnetic stripe, 125 kHz RFID (LFRFID), or 13.56 MHz NFC (MIFARE). The T-Embed has none of these interfaces. Additionally, cloning hotel key cards you are not authorized to duplicate is illegal.
</details>

<details><summary>Q: What modulations does the CC1101 support?</summary>

<strong>A:</strong> The CC1101 supports:
<ul>
<li><strong>OOK (On-Off Keying)</strong> — Most consumer RF devices (doorbells, remotes, sensors)</li>
<li><strong>ASK (Amplitude Shift Keying)</strong> — Similar to OOK, used in some industrial sensors</li>
<li><strong>2-FSK (Frequency Shift Keying)</strong> — Digital devices, some smart home sensors</li>
<li><strong>GFSK (Gaussian FSK)</strong> — Bluetooth-adjacent modulations, some advanced sensors</li>
<li><strong>4-FSK</strong> — Higher data rate applications</li>
<li><strong>MSK (Minimum Shift Keying)</strong> — Spectrally efficient digital communications</li>
</ul>
It does <strong>not</strong> support: AM/FM broadcast, LoRa, Zigbee (natively), or any wideband modulation.
</details>

<details><summary>Q: What is the maximum data rate?</summary>

<strong>A:</strong> The CC1101 supports data rates from 0.6 kbps to 500 kbps, depending on modulation and configuration. Practical data rates for reliable communication with consumer devices are typically 1–20 kbps. The higher rates (200–500 kbps) require good signal quality and are used primarily in custom point-to-point links.
</details>

---

{: .tip }
> **Still stuck?** Check the firmware's GitHub Issues page for your specific problem. The community is active and most issues have been encountered and solved by someone before you. If you find a new bug, file an issue with your device info, firmware version, and steps to reproduce.
