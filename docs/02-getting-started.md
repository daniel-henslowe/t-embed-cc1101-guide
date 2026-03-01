---
layout: default
title: "02 — Getting Started"
nav_order: 3
---

# 02 — Getting Started
{: .no_toc }

From unboxing to your first signal capture in under an hour. This chapter walks you through every step — connecting to your computer, flashing firmware, navigating the menus, and performing your first five hands-on operations.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Unboxing and First Inspection

### What's in the Box

When your LILYGO T-Embed CC1101 Plus arrives, you should find:

| Item | Description |
|:-----|:------------|
| **T-Embed CC1101 Plus board** | The main device with integrated display, rotary encoder, and CC1101 radio |
| **USB-C cable** | Standard USB-C data cable (some packages may not include one — have your own ready) |
| **Pin headers** (optional) | Some revisions ship with unsoldered GPIO headers |
| **Antenna** | Sub-GHz antenna, typically an SMA or IPEX stub antenna |
| **Documentation card** | Basic pinout reference or QR code to LILYGO wiki |

> Not every seller includes the same accessories. AliExpress listings may differ from the official LILYGO store. Verify you have at minimum the board and an antenna.
{: .warning }

### Physical Inspection Checklist

Before powering on, visually inspect the board:

1. **Display** — The 1.9-inch ST7789 TFT should be clean, no cracks or dead pixels (you will verify on first boot)
2. **Rotary encoder** — Press and rotate it. It should click when pressed and have smooth detents when rotated. No wobble or looseness
3. **USB-C port** — Inspect for bent pins or debris. Wiggle a cable gently to confirm solid connection
4. **CC1101 radio module** — Located on the board near the antenna connector. Verify the shielding can is intact and not dislodged
5. **Antenna connector** — SMA or IPEX depending on revision. Ensure the connector is firmly soldered
6. **MicroSD slot** — If present, test with a card to confirm the spring mechanism works
7. **ESP32-S3 chip** — The main processor should be cleanly soldered with no bridged pins
8. **Battery connector** — JST connector for LiPo battery (if your revision supports it)
9. **Boot and Reset buttons** — Both should have tactile click feedback
10. **IR LED and receiver** — Small components near the top edge of the board

### Identifying Your Board Revision

LILYGO has produced several revisions. Check the silkscreen on the back of the PCB:

| Marking | Revision | Key Differences |
|:--------|:---------|:----------------|
| `T-Embed V1.0` | Original T-Embed | No CC1101 — different product entirely |
| `T-Embed CC1101 V1.0` | First CC1101 version | Separate CC1101 breakout, older GPIO mapping |
| `T-Embed CC1101 V1.2` | Revised CC1101 | Improved antenna trace, updated pin mapping |
| `T-Embed CC1101 Plus` | Latest (recommended) | Integrated CC1101, improved RF performance, expanded GPIO |

> If your board says just "T-Embed" without "CC1101," you have the base model without the Sub-GHz radio. The Sub-GHz sections of this guide will not apply to you.
{: .danger }

To confirm programmatically after flashing firmware:

```
Board Info → Hardware → Board: T-Embed CC1101
ESP32-S3 Chip: ESP32-S3 (revision v0.2)
Flash: 16MB QD
PSRAM: 8MB OPI
```

---

## Connecting to Your Computer

The ESP32-S3 supports native USB CDC (Communications Device Class), meaning it shows up as a serial device without requiring third-party drivers on most operating systems.

### macOS

**Step 1: Plug in via USB-C**

Use a USB-C data cable (not a charge-only cable). Connect the T-Embed to your Mac.

> Charge-only cables are the number one cause of "my device isn't detected." If `ls /dev/cu.usb*` returns nothing, try a different cable before troubleshooting further.
{: .warning }

**Step 2: Verify the device appears**

Open Terminal and run:

```bash
ls /dev/cu.usb*
```

Expected output:

```
/dev/cu.usbmodem1101
```

Or alternatively:

```bash
ls /dev/tty.usb*
```

```
/dev/tty.usbmodem1101
```

The exact number (`1101`, `2101`, `14101`, etc.) varies by USB port and hub. Both `cu.*` and `tty.*` work — use `cu.*` for outgoing connections (which is what you want).

> On Apple Silicon Macs (M1/M2/M3/M4), the ESP32-S3 native USB works out of the box. No drivers needed. If using an older Intel Mac with the CP2102 USB-UART bridge variant, you may need the [Silicon Labs VCP driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers).
{: .note }

**Step 3: Connect via serial terminal**

Using the built-in `screen` command:

```bash
screen /dev/cu.usbmodem1101 115200
```

To exit `screen`: press `Ctrl+A`, then `K`, then `Y` to confirm.

Using `picocom` (install via Homebrew):

```bash
brew install picocom
picocom -b 115200 /dev/cu.usbmodem1101
```

To exit `picocom`: press `Ctrl+A`, then `Ctrl+X`.

Using `minicom`:

```bash
brew install minicom
minicom -D /dev/cu.usbmodem1101 -b 115200
```

To exit `minicom`: press `Ctrl+A`, then `X`, then confirm.

> `picocom` is the recommended serial terminal for macOS. It is lightweight, reliable, and does not hijack your terminal session the way `screen` can.
{: .tip }

**Step 4: Verify communication**

Once connected, press the Reset button on the T-Embed. You should see boot messages:

```
ESP-ROM:esp32s3-20210327
Build:Mar 27 2021
rst:0x1 (POWERON),boot:0x8 (SPI_FAST_FLASH_BOOT)
...
```

If you see garbled characters, you likely have a baud rate mismatch. Try `115200`, `9600`, or `921600`.

### Linux (including Kali)

**Step 1: Plug in the device**

The ESP32-S3 will appear as an ACM device:

```bash
ls /dev/ttyACM*
```

Expected output:

```
/dev/ttyACM0
```

Verify with `dmesg`:

```bash
dmesg | tail -20
```

Expected output:

```
[12345.678] usb 1-1: new full-speed USB device number 5 using xhci_hcd
[12345.789] usb 1-1: New USB device found, idVendor=303a, idProduct=1001
[12345.790] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[12345.791] usb 1-1: Product: USB JTAG/serial debug unit
[12345.792] usb 1-1: Manufacturer: Espressif
[12345.800] cdc_acm 1-1:1.0: ttyACM0: USB ACM device
```

**Step 2: Fix permissions**

By default, `/dev/ttyACM0` requires root access. You have three options:

**Option A — Quick fix (temporary, resets on replug):**

```bash
sudo chmod 666 /dev/ttyACM0
```

**Option B — Add your user to the `dialout` group (persistent, recommended):**

```bash
sudo usermod -aG dialout $USER
```

Log out and log back in (or reboot) for the group change to take effect. Verify:

```bash
groups
```

Should include `dialout` in the output.

**Option C — udev rule (persistent device naming):**

Create a udev rule so the device always gets the same name:

```bash
sudo nano /etc/udev/rules.d/99-t-embed.rules
```

Add:

```
SUBSYSTEM=="tty", ATTRS{idVendor}=="303a", ATTRS{idProduct}=="1001", SYMLINK+="t-embed", MODE="0666"
```

Reload udev:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Now the device will always be available at `/dev/t-embed` regardless of which USB port you use.

**Step 3: Connect via serial terminal**

```bash
# Using screen
screen /dev/ttyACM0 115200

# Using picocom (install: sudo apt install picocom)
picocom -b 115200 /dev/ttyACM0

# Using minicom (install: sudo apt install minicom)
minicom -D /dev/ttyACM0 -b 115200
```

> On Kali Linux, you may need to install serial tools first: `sudo apt update && sudo apt install -y screen picocom minicom`. Kali's default installation does not always include them.
{: .tip }

### Windows

**Step 1: Plug in the device**

Connect the T-Embed via USB-C. Windows 10/11 should automatically install the USB serial driver.

**Step 2: Find the COM port**

Open **Device Manager** (`devmgmt.msc`):

1. Expand **Ports (COM & LPT)**
2. Look for **USB Serial Device (COM3)** or **USB JTAG/serial debug unit (COM3)**
3. Note the COM port number (e.g., `COM3`)

If the device does not appear:
- Try a different USB cable (data cable, not charge-only)
- Try a different USB port
- Check **Universal Serial Bus controllers** for unknown devices
- Download and install the [Espressif USB-Serial-JTAG driver](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/jtag-debugging/configure-builtin-jtag.html) if needed

**Step 3: Connect via serial terminal**

**Using PuTTY:**

1. Download [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
2. Select **Serial** connection type
3. Set **Serial line** to `COM3` (your COM port)
4. Set **Speed** to `115200`
5. Click **Open**

**Using Tera Term:**

1. Download [Tera Term](https://github.com/TeraTermProject/teraterm/releases)
2. Select **Serial** → choose your COM port
3. Go to **Setup → Serial port** → set baud rate to `115200`

**Using Windows Terminal with PowerShell:**

```powershell
# Install PuTTY via winget
winget install PuTTY.PuTTY

# Or use the built-in mode command to verify the port
mode COM3
```

> Windows users: if you see the device appear and disappear repeatedly in Device Manager, your USB cable may be defective or charge-only. This is the most common Windows-specific issue.
{: .warning }

---

## Installing Bruce Firmware (Recommended)

[Bruce](https://github.com/pr3y/Bruce) is a multi-tool firmware for ESP32-based devices that unlocks the full potential of the T-Embed CC1101 Plus. It provides WiFi, Bluetooth, Sub-GHz, IR, NFC, BadUSB, and dozens of other tools through an intuitive menu system.

> Bruce is the recommended firmware throughout this guide. All operations, menu references, and screenshots assume Bruce firmware. Other firmware options (Marauder, custom builds) are covered in a later chapter.
{: .note }

### Method 1: Web Flasher (Easiest — No Tools Needed)

This is the fastest way to get Bruce running. No software installation required.

**Requirements:**
- Google Chrome, Microsoft Edge, or another Chromium-based browser (Firefox does NOT support WebSerial)
- USB-C data cable
- Internet connection

**Step-by-step:**

1. **Open the Bruce web flasher** — Navigate to [https://bruce.computer/flasher](https://bruce.computer/flasher) in Chrome or Edge

2. **Connect your T-Embed** — Plug the USB-C cable into your T-Embed and your computer

3. **Put the device in download mode** (if required):
   - Hold the **BOOT** button
   - While holding BOOT, briefly press and release the **RST** (Reset) button
   - Release the **BOOT** button
   - The screen should go blank or show nothing — this is normal

   > Some newer revisions of Bruce's web flasher can flash without entering download mode manually. Try without download mode first. If the flasher cannot connect, then use the BOOT+RST method above.
   {: .tip }

4. **Select your board** — From the board dropdown list, select **"T-Embed CC1101"** or **"T-Embed CC1101 Plus"** (match your specific hardware)

5. **Click "Connect"** — A browser dialog will appear listing serial ports. Select the one that matches your device:
   - macOS: `cu.usbmodem1101` or similar
   - Linux: `ttyACM0`
   - Windows: `COM3` or similar

6. **Click "Install" or "Flash"** — The web flasher will:
   - Download the latest Bruce release binary
   - Erase the flash memory
   - Write the new firmware
   - Verify the write

7. **Monitor progress** — You will see a progress bar. The process takes approximately 60-90 seconds. Expected output in the log:

   ```
   Connecting...
   Chip type: ESP32-S3
   Flash size: 16MB
   Erasing flash...
   Writing at 0x00000000... (1%)
   Writing at 0x00010000... (5%)
   ...
   Writing at 0x00F00000... (98%)
   Writing at 0x00FF0000... (100%)
   Verifying...
   Hash verified.
   Done! Resetting device...
   ```

8. **Device reboots** — After flashing, the T-Embed will automatically reboot and you will see the Bruce boot screen with the Bruce logo and version number

> If the web flasher says "No compatible devices found" when you click Connect, you are either using Firefox (not supported), your cable is charge-only, or the device is not in download mode. Try the BOOT+RST sequence again.
{: .warning }

### Method 2: esptool (Command Line)

For those who prefer the command line or need more control over the flashing process.

**Step 1: Install esptool**

```bash
# Using pip (all platforms)
pip install esptool

# Using Homebrew (macOS)
brew install esptool

# Using apt (Debian/Ubuntu/Kali)
sudo apt install esptool

# Verify installation
esptool.py version
```

Expected output:

```
esptool.py v4.7.0
Serial port /dev/cu.usbmodem1101
```

**Step 2: Download the latest Bruce release**

Go to [https://github.com/pr3y/Bruce/releases](https://github.com/pr3y/Bruce/releases) and download the binary for your board. Look for a file named something like:

```
Bruce_t-embed-cc1101_vX.X.X.bin
```

Or download via command line:

```bash
# Create a working directory
mkdir -p ~/t-embed && cd ~/t-embed

# Download the latest release (check GitHub for current version)
# Replace the URL with the actual latest release URL
curl -L -o bruce_firmware.bin \
  "https://github.com/pr3y/Bruce/releases/latest/download/Bruce_t-embed-cc1101.bin"
```

> Always download firmware from the official GitHub releases page. Never flash firmware from untrusted sources — a malicious firmware could brick your device or exfiltrate data.
{: .danger }

**Step 3: Put the device in download mode**

Two methods:

**Method A — BOOT + RST buttons:**
1. Hold the **BOOT** button
2. While holding BOOT, press and release the **RST** button
3. Release the **BOOT** button
4. The device is now in download mode (screen blank)

**Method B — BOOT + USB plug:**
1. Disconnect the USB cable
2. Hold the **BOOT** button
3. While holding BOOT, plug in the USB cable
4. Release the **BOOT** button

Verify download mode:

```bash
esptool.py --chip esp32s3 chip_id
```

Expected output:

```
esptool.py v4.7.0
Serial port /dev/cu.usbmodem1101
Connecting...
Chip is ESP32-S3 (revision v0.2)
Features: WiFi, BLE, Embedded Flash 16MB (GD), Embedded PSRAM 8MB (AP_3v3)
Crystal is 40MHz
MAC: xx:xx:xx:xx:xx:xx
Uploading stub...
Running stub...
Stub running...
Chip ID: 0x00000009
```

**Step 4: Erase flash (clean install recommended)**

```bash
esptool.py --chip esp32s3 --port /dev/cu.usbmodem1101 erase_flash
```

Expected output:

```
esptool.py v4.7.0
Serial port /dev/cu.usbmodem1101
Connecting...
Chip is ESP32-S3 (revision v0.2)
Features: WiFi, BLE, Embedded Flash 16MB (GD), Embedded PSRAM 8MB (AP_3v3)
Uploading stub...
Running stub...
Erasing flash (this may take a while)...
Chip erase completed successfully in 8.2s
Hard resetting via RTS pin...
```

> Erasing flash wipes all saved settings, captured signals, and stored data. If you are upgrading and want to keep your data, skip the erase step — but be aware that stale configuration can sometimes cause issues after an upgrade.
{: .warning }

**Step 5: Flash the firmware**

```bash
esptool.py --chip esp32s3 --port /dev/cu.usbmodem1101 \
  --baud 921600 write_flash -z 0x0 bruce_firmware.bin
```

Expected output:

```
esptool.py v4.7.0
Serial port /dev/cu.usbmodem1101
Connecting...
Chip is ESP32-S3 (revision v0.2)
Features: WiFi, BLE, Embedded Flash 16MB (GD), Embedded PSRAM 8MB (AP_3v3)
Crystal is 40MHz
Uploading stub...
Running stub...
Changing baud rate to 921600
Changed.
Configuring flash size...
Flash will be erased from 0x00000000 to 0x00ffffff...
Compressed 16777216 bytes to 8234512...
Wrote 16777216 bytes (8234512 compressed) at 0x00000000 in 21.3 seconds...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

**Baud rate optimization:**

| Baud Rate | Flash Time (16MB) | Reliability |
|:----------|:-------------------|:------------|
| `115200` | ~150 seconds | Most reliable |
| `460800` | ~45 seconds | Very reliable |
| `921600` | ~22 seconds | Reliable on most setups |
| `1500000` | ~15 seconds | May fail on some USB hubs |

> Start with `921600`. If you get write errors or timeouts, drop to `460800` or `115200`. USB hubs and long cables reduce reliable baud rates.
{: .tip }

**Step 6: Verify successful flash**

After flashing, the device should automatically reset and boot. If it does not:

1. Press the **RST** button
2. If still nothing, unplug and replug the USB cable

You should see the Bruce boot screen on the display.

**Troubleshooting failed flashes:**

| Symptom | Cause | Fix |
|:--------|:------|:----|
| `Failed to connect` | Not in download mode | Redo BOOT+RST sequence |
| `A fatal error occurred: Could not open port` | Permission denied | Run with `sudo` or fix permissions |
| `Invalid head of packet` | Bad cable or baud rate too high | Lower baud rate, try different cable |
| `MD5 of file does not match` | Corrupted download | Re-download the firmware binary |
| `Timed out waiting for packet header` | USB hub issue | Connect directly to computer, no hub |
| Screen stays blank after flash | Wrong firmware variant | Verify you downloaded the T-Embed CC1101 variant |

### Method 3: PlatformIO (Build from Source)

Building from source gives you the latest features, the ability to modify code, and deeper understanding of the firmware.

**Step 1: Install prerequisites**

```bash
# Install PlatformIO CLI
pip install platformio

# Or install PlatformIO as a VS Code extension:
# 1. Open VS Code
# 2. Go to Extensions (Cmd+Shift+X / Ctrl+Shift+X)
# 3. Search "PlatformIO IDE"
# 4. Install it

# Verify PlatformIO
pio --version
```

**Step 2: Clone the Bruce repository**

```bash
git clone https://github.com/pr3y/Bruce.git
cd Bruce
```

**Step 3: Identify the correct build environment**

```bash
# List available build environments
pio project config | grep "env:"
```

Look for the environment matching your board. It may be listed as:

- `t-embed-cc1101`
- `t-embed-cc1101-plus`
- `lilygo-t-embed-cc1101`

Check `platformio.ini` for the exact environment name:

```bash
grep -A 5 "\[env:.*embed.*cc1101\]" platformio.ini
```

**Step 4: Build the firmware**

```bash
pio run -e t-embed-cc1101
```

This will:
- Download the ESP32-S3 toolchain (first run only, ~500MB)
- Download all library dependencies
- Compile the entire firmware
- Output the binary to `.pio/build/t-embed-cc1101/firmware.bin`

Expected output (end):

```
Linking .pio/build/t-embed-cc1101/firmware.elf
Building .pio/build/t-embed-cc1101/firmware.bin
Calculating checksum...
============================ [SUCCESS] Took 45.23 seconds ============================
```

**Step 5: Upload to the device**

```bash
pio run -e t-embed-cc1101 -t upload
```

PlatformIO will automatically detect the serial port, enter download mode (if supported), flash, and reset the device.

**Step 6: Monitor serial output (optional)**

```bash
pio device monitor -b 115200
```

> Building from source requires approximately 2GB of disk space for the toolchain and dependencies. The first build takes 2-5 minutes. Subsequent builds are faster thanks to incremental compilation.
{: .note }

---

## First Boot

### What You See on First Boot

When Bruce firmware boots for the first time, you will see:

1. **Bruce logo** — The stylized Bruce logo appears for 1-2 seconds
2. **Version info** — Firmware version, build date, and board type flash briefly
3. **Main menu** — The home screen appears with the top-level menu

If the screen stays blank or shows garbage pixels:
- Press RST to reboot
- If persistent, re-flash the firmware (you may have flashed the wrong board variant)

### Navigating the Bruce Menu System

The T-Embed CC1101 Plus uses a rotary encoder with a built-in push button as its primary input:

| Input | Action |
|:------|:-------|
| **Rotate clockwise** | Scroll down / next item |
| **Rotate counter-clockwise** | Scroll up / previous item |
| **Short press** (click encoder) | Select / Enter / Confirm |
| **Long press** (~1 second hold) | Back / Return to previous menu |
| **Double press** (two quick clicks) | Context action (varies by screen; some menus use this for quick settings) |

> The rotary encoder is your primary interface. Get comfortable with it — rotate to browse, click to select, long-press to go back. Within 5 minutes it will feel natural.
{: .tip }

### Menu Structure Walkthrough

Below is the complete top-level menu structure with submenus. Your exact menu may vary slightly depending on Bruce version.

#### 1. WiFi

| Submenu | Description |
|:--------|:------------|
| **Scan Networks** | Scan for nearby WiFi access points, showing SSID, BSSID, channel, RSSI, and encryption |
| **Beacon Spam** | Broadcast fake WiFi network names (for demonstration/testing) |
| **Deauth** | Send deauthentication frames to disconnect clients from a network (test your own network only) |
| **Probe Request** | Monitor and capture probe request frames from nearby devices |
| **Evil Portal** | Create a captive portal for social engineering testing |
| **Packet Monitor** | Real-time packet capture and analysis |
| **Connect to WiFi** | Join a WiFi network (needed for some features like NTP time sync) |

#### 2. Bluetooth

| Submenu | Description |
|:--------|:------------|
| **BLE Scan** | Scan for Bluetooth Low Energy devices |
| **BLE Spam** | Send BLE advertisement spam (Apple/Samsung/Windows device notifications) |
| **Apple Juice** | Send spoofed Apple device pairing notifications |
| **Samsung BLE** | Send spoofed Samsung device notifications |
| **Swift Pair** | Send spoofed Windows Swift Pair notifications |
| **BLE Keyboard** | Use the T-Embed as a BLE keyboard |

#### 3. Sub-GHz (RF)

| Submenu | Description |
|:--------|:------------|
| **Frequency Analyzer** | Sweep the spectrum and display signal strength across frequencies |
| **Receive Raw** | Capture raw Sub-GHz signals |
| **Receive/Decode** | Capture and decode known protocols (e.g., weather stations, doorbells) |
| **Transmit** | Replay captured signals or transmit saved ones |
| **Jammer** | RF signal jammer (ILLEGAL in most jurisdictions — for educational reference only) |
| **Saved Signals** | Browse and manage captured signals |
| **Frequency Presets** | Quick access to common frequencies (315, 433.92, 868, 915 MHz) |

> The Sub-GHz Jammer option exists in the firmware for completeness but using it is ILLEGAL under FCC (USA), ETSI (EU), and equivalent regulations worldwide. Jamming radio frequencies can interfere with emergency communications and carries severe criminal penalties. Never use it outside of a properly shielded RF enclosure.
{: .danger }

#### 4. Infrared (IR)

| Submenu | Description |
|:--------|:------------|
| **Receive** | Capture IR signals from remote controls |
| **Transmit** | Replay captured IR signals or send from library |
| **TV-B-Gone** | Cycle through power-off codes for hundreds of TV brands |
| **Custom IR** | Create and store custom IR sequences |
| **Saved Signals** | Browse and manage captured IR data |

#### 5. RFID / NFC

| Submenu | Description |
|:--------|:------------|
| **Read Tag** | Read RFID/NFC tags (requires external module for some tag types) |
| **Emulate** | Emulate a saved tag |
| **Saved Tags** | Browse and manage saved tag data |
| **Write Tag** | Write data to a writable tag |

#### 6. BadUSB

| Submenu | Description |
|:--------|:------------|
| **Run Script** | Execute a Ducky Script payload |
| **Select Script** | Browse saved scripts on MicroSD |
| **USB Keyboard** | Manual keyboard input mode |
| **USB Mass Storage** | Present the MicroSD as a USB drive |

#### 7. Tools

| Submenu | Description |
|:--------|:------------|
| **File Manager** | Browse MicroSD card contents |
| **Flashlight** | Use the screen as a white light |
| **Battery Info** | Display battery voltage, percentage, and charging status |
| **GPS** | GPS module data display (requires external GPS module) |
| **Clock** | Display current time (set via NTP or manual) |
| **QR Code** | Generate and display QR codes |

#### 8. Settings

| Submenu | Description |
|:--------|:------------|
| **Display Brightness** | Adjust backlight (0-100%) |
| **Screen Timeout** | Auto-off timer (15s, 30s, 1m, 5m, never) |
| **WiFi Settings** | Saved networks, auto-connect |
| **Time Zone** | UTC offset for clock |
| **Sound** | Toggle buzzer on/off, adjust volume |
| **UI Theme** | Color scheme selection |
| **LED** | Control onboard RGB LED color and patterns |
| **About** | Firmware version, board info, MAC addresses |
| **Reboot** | Restart the device |
| **Power Off** | Turn off (if battery-powered) |

### Settings to Configure First

Before doing anything else, configure these settings:

**1. Display Brightness** — Navigate to `Settings → Display Brightness`. Set to 50-70% for indoor use. Full brightness drains battery quickly.

**2. Screen Timeout** — Navigate to `Settings → Screen Timeout`. Set to `1 minute` or `5 minutes` to prevent burn-in and save battery. Set to `Never` when actively working.

**3. WiFi Connection** — Navigate to `WiFi → Connect to WiFi`. Select your network and enter the password using the rotary encoder character input:
   - Rotate to select character
   - Press to confirm character
   - Navigate to "Done" or a checkmark to submit

**4. Time Zone** — Navigate to `Settings → Time Zone`. Set your UTC offset (e.g., UTC-5 for EST, UTC+0 for GMT, UTC+10 for AEST). Time syncs via NTP once WiFi is connected.

**5. Sound** — Navigate to `Settings → Sound`. Turn it on or off based on your preference. On by default — the buzzer provides audio feedback for button presses and alerts.

**6. UI Theme** — Navigate to `Settings → UI Theme`. Choose a color scheme you like. This is purely cosmetic but makes extended use more pleasant.

> After configuring WiFi and time zone, reboot the device once. Some settings take effect immediately, but a clean reboot ensures everything initializes properly.
{: .tip }

---

## Your First 5 Operations (Hands-On)

Time to put the T-Embed to work. These five operations introduce you to the core capabilities of the device. Each one should take under 5 minutes.

### Operation 1: Scan for WiFi Networks

**What you will learn:** How your local RF environment looks and what information is broadcast by every WiFi access point around you.

**Step-by-step:**

1. From the main menu, rotate the encoder to **WiFi**
2. Press the encoder to enter the WiFi menu
3. Select **Scan Networks**
4. The scan runs automatically for 3-5 seconds
5. Results appear as a scrollable list

**Understanding the output:**

```
SSID: MyHomeNetwork
BSSID: A4:CF:12:B3:22:F1
Channel: 6
RSSI: -42 dBm
Encryption: WPA2
---
SSID: Neighbor_5G
BSSID: 00:1A:2B:3C:4D:5E
Channel: 149
RSSI: -71 dBm
Encryption: WPA3
---
SSID: [Hidden]
BSSID: D8:EC:5E:12:34:56
Channel: 11
RSSI: -65 dBm
Encryption: WPA2
```

| Field | Meaning |
|:------|:--------|
| **SSID** | Network name. `[Hidden]` means the AP is not broadcasting its name |
| **BSSID** | MAC address of the access point (unique hardware identifier) |
| **Channel** | WiFi channel (1-13 for 2.4 GHz, 36-165 for 5 GHz). The ESP32-S3 scans 2.4 GHz only |
| **RSSI** | Signal strength in dBm. -30 is excellent, -50 is good, -70 is weak, -90 is barely detectable |
| **Encryption** | Security protocol: Open, WEP, WPA, WPA2, WPA3 |

**What you can learn from a WiFi scan:**

- How many networks are in range and on which channels (useful for finding the least congested channel for your own router)
- Which networks use outdated security (WEP or Open)
- Hidden networks that are broadcasting but not advertising their name
- Approximate location of access points based on signal strength
- Manufacturer of access points (the first 3 octets of BSSID identify the manufacturer via OUI lookup)

> This scan is entirely passive — your device is only listening to beacons that access points are already broadcasting. This is legal everywhere and no different from what your phone does constantly.
{: .note }

### Operation 2: Scan for Bluetooth Devices

**What you will learn:** How many Bluetooth devices are active around you, what types they are, and what information they broadcast.

**Step-by-step:**

1. From the main menu, navigate to **Bluetooth**
2. Select **BLE Scan**
3. The scan runs for 5-10 seconds, discovering devices
4. Results appear as a scrollable list

**Understanding the output:**

```
Name: iPhone (John)
MAC: A1:B2:C3:D4:E5:F6
RSSI: -55 dBm
Type: BLE
---
Name: [Unknown]
MAC: 11:22:33:44:55:66
RSSI: -78 dBm
Type: BLE
---
Name: JBL Flip 6
MAC: AA:BB:CC:DD:EE:FF
RSSI: -62 dBm
Type: BLE
```

**BLE vs Classic Bluetooth:**

| Feature | Bluetooth Low Energy (BLE) | Classic Bluetooth |
|:--------|:---------------------------|:------------------|
| Power | Ultra-low | Higher |
| Range | ~100m | ~100m |
| Data rate | 1-2 Mbps | 1-3 Mbps |
| Use cases | Fitness trackers, beacons, sensors, AirTags | Audio streaming, file transfer, keyboards |
| ESP32-S3 support | Full scan + connect | Scan only (limited) |

**Device types you will commonly see:**

- **Phones** — iPhones, Android devices broadcasting BLE advertisements
- **Wearables** — Apple Watch, Fitbit, Garmin devices
- **Headphones** — AirPods, JBL, Sony, Bose speakers and earbuds
- **Smart home** — Tile trackers, AirTags, smart locks, thermometers
- **Beacons** — iBeacon/Eddystone from retail stores
- **[Unknown]** — Devices not broadcasting a name (common for trackers and IoT sensors)

> In a typical home environment, you may see 5-15 BLE devices. In a coffee shop or office, expect 30-100+. Most of these are phones and wearables silently advertising in the background.
{: .note }

### Operation 3: Capture an IR Remote Signal

**What you will learn:** How infrared remote control works, and how trivially simple it is to capture and replay any IR signal.

**What you need:** Any infrared remote control — TV remote, air conditioner remote, soundbar remote, fan remote, etc.

**Step-by-step:**

1. From the main menu, navigate to **Infrared (IR)**
2. Select **Receive**
3. The screen displays "Waiting for IR signal..." with a receiver icon
4. Point your remote control directly at the **top edge** of the T-Embed (where the IR receiver is located), at a distance of 5-15 cm
5. Press any button on your remote (e.g., Power, Volume Up)
6. The T-Embed captures the signal and displays:

```
Protocol: NEC
Address: 0x04
Command: 0x08
Bits: 32
Raw: 9000 4500 560 560 560 1690 560 560...
```

| Field | Meaning |
|:------|:--------|
| **Protocol** | The IR encoding protocol (NEC, Sony, Samsung, RC5, RC6, etc.) |
| **Address** | Device identifier (which device the remote is talking to) |
| **Command** | The specific command (power, volume, channel, etc.) |
| **Bits** | Number of bits in the message |
| **Raw** | Raw timing data in microseconds (mark/space pairs) |

**Replaying the signal:**

1. After capture, you will see options: **Save**, **Replay**, **Discard**
2. Select **Replay**
3. Point the T-Embed at the device (TV, AC unit, etc.)
4. The T-Embed transmits the captured IR signal
5. The device should respond exactly as if you pressed the original remote

> IR signals travel in a straight line and require line-of-sight. Point the T-Embed directly at the device's IR receiver (usually near the front panel). Maximum effective range is typically 3-8 meters depending on ambient light conditions.
{: .tip }

**Save the signal for later:**

1. Select **Save** after capture
2. Choose a filename (e.g., "TV_Power", "AC_Cool_22")
3. The signal is stored to the MicroSD card or internal storage
4. Access it later via `IR → Saved Signals`

### Operation 4: Sub-GHz Frequency Scan

**What you will learn:** What the radio spectrum looks like around your home on Sub-GHz frequencies, and what devices are actively transmitting.

**Step-by-step:**

1. **Ensure the Sub-GHz antenna is connected** to the T-Embed. Without it, you will see almost no signals.
2. From the main menu, navigate to **Sub-GHz (RF)**
3. Select **Frequency Analyzer**
4. The display shows a real-time spectrum view:
   - X-axis: frequency range (typically 300-928 MHz)
   - Y-axis: signal strength (RSSI)
   - Peaks indicate active transmissions

> ALWAYS connect the antenna before using Sub-GHz features. Transmitting without an antenna can damage the CC1101 radio module due to reflected RF energy. Receiving without an antenna simply gives poor results, but transmitting without one is harmful to the hardware.
{: .danger }

**What you are looking at:**

The frequency analyzer sweeps through the Sub-GHz spectrum and displays signal activity in real time. You will see:

- A relatively flat noise floor (the baseline, usually around -90 to -100 dBm)
- Occasional spikes when a device transmits
- Persistent signals from always-on devices (some weather stations transmit every 30-60 seconds)

**Common signals you will detect in your home:**

| Frequency | Common Source |
|:----------|:-------------|
| **315 MHz** | Older garage door openers, some car key fobs (North America) |
| **433.92 MHz** | Weather stations, wireless doorbells, tire pressure sensors, cheap IoT devices, some car key fobs (worldwide) |
| **868 MHz** | Smart home devices, LoRa sensors (Europe) |
| **915 MHz** | LoRa devices, smart meters (North America) |

**Tips for observing signals:**

- Press a wireless doorbell button and watch for a spike at 433.92 MHz
- Check if you have a weather station — it transmits periodically, creating regular spikes
- Walk near a car and press the key fob — you may see a brief spike at 315 or 433 MHz
- Smart home devices (Zigbee, Z-Wave) operate on different frequencies but some use Sub-GHz bands

> The frequency analyzer is entirely passive (receive-only). You are not transmitting anything. This is equivalent to an AM/FM radio scanning for stations.
{: .note }

### Operation 5: Sub-GHz Signal Capture

**What you will learn:** How to capture a specific Sub-GHz radio signal, decode it, and replay it.

**What you need:** A device that transmits on Sub-GHz frequencies. The easiest to test with:
- A wireless doorbell (433.92 MHz, widely available for under $10)
- A wireless weather station
- A wireless outlet switch
- Your own garage door opener (be cautious — see warning below)

> ONLY capture and replay signals from devices YOU OWN. Replaying someone else's garage door opener, car key fob, or security system signal without authorization is illegal. Modern car key fobs use rolling codes that cannot be simply replayed anyway — but the legal issue applies regardless.
{: .warning }

**Step-by-step:**

1. From the main menu, navigate to **Sub-GHz (RF)**
2. Select **Receive Raw** or **Receive/Decode**
3. Set the frequency to **433.92 MHz** (the most common frequency for consumer devices):
   - Rotate the encoder to adjust the frequency
   - Or select from **Frequency Presets** → **433.92 MHz**
4. The screen shows "Listening on 433.920 MHz..."
5. Press the button on your wireless doorbell (or trigger your test device)
6. The T-Embed captures the signal and displays:

```
Signal captured!
Frequency: 433.920 MHz
Modulation: ASK/OOK
Protocol: Princeton (or Generic)
Data: 0xABCDEF
Bit length: 24
Repeat: 3
```

| Field | Meaning |
|:------|:--------|
| **Frequency** | The exact frequency the signal was received on |
| **Modulation** | How the data is encoded on the carrier wave (ASK/OOK is most common for simple devices) |
| **Protocol** | Named protocol if recognized, or "Generic/Raw" |
| **Data** | The actual data payload (the "code" the device sent) |
| **Bit length** | Number of bits in the data |
| **Repeat** | How many times the device repeated the transmission |

**Replaying the signal:**

1. After capture, select **Replay** or **Transmit**
2. The T-Embed transmits the captured signal
3. Your doorbell receiver should chime (or your test device should respond)

**Saving the signal:**

1. Select **Save**
2. Name it something descriptive (e.g., "Front_Doorbell")
3. It saves to MicroSD or internal storage
4. Access later via `Sub-GHz → Saved Signals`

> If capture fails (no signal detected), try these fixes: (1) Move the T-Embed closer to the transmitter, (2) Verify the antenna is connected, (3) Try a slightly different frequency (e.g., 433.50 or 434.00), (4) Make sure the device actually transmits on Sub-GHz (some modern doorbells use WiFi instead).
{: .tip }

---

## Connecting to Kali Linux VM (UTM on M1/M2/M3/M4 Mac)

If you are running Kali Linux in a virtual machine using UTM on Apple Silicon, you need to pass the T-Embed USB device through to the VM.

### UTM USB Device Sharing

**Step 1: Connect the T-Embed to your Mac**

Plug in the USB-C cable. Verify the device appears on the Mac side first:

```bash
ls /dev/cu.usb*
```

**Step 2: Share the device with UTM**

1. Start your Kali Linux VM in UTM
2. In the UTM toolbar, click the **USB** icon (looks like a USB plug, or find it under the VM window toolbar)
3. You will see a list of connected USB devices
4. Check/select **Espressif USB JTAG/serial debug unit** (or similar name matching your T-Embed)
5. The device will disconnect from macOS and attach to the Kali VM

> When you share a USB device with UTM, it is no longer available on the host macOS. To get it back, uncheck the device in UTM's USB menu or shut down the VM.
{: .note }

**Step 3: Verify connection inside Kali**

Open a terminal in Kali and run:

```bash
dmesg | tail -20
```

Expected output:

```
[  234.567890] usb 2-1: new full-speed USB device number 3 using xhci_hcd
[  234.678901] usb 2-1: New USB device found, idVendor=303a, idProduct=1001
[  234.679012] cdc_acm 2-1:1.0: ttyACM0: USB ACM device
```

Verify the device node:

```bash
ls -la /dev/ttyACM*
```

Expected:

```
crw-rw---- 1 root dialout 166, 0 Feb 28 12:00 /dev/ttyACM0
```

**Step 4: Fix permissions in Kali**

```bash
# Add your user to dialout group
sudo usermod -aG dialout $USER

# Apply immediately (or log out/in)
newgrp dialout

# Verify
groups
```

**Step 5: Connect via serial**

```bash
picocom -b 115200 /dev/ttyACM0
```

### Installing Development Tools in Kali for the T-Embed

```bash
# Update package lists
sudo apt update

# Install esptool for flashing
sudo apt install -y esptool

# Install Python and pip (if not present)
sudo apt install -y python3 python3-pip

# Install picocom for serial communication
sudo apt install -y picocom minicom

# Install PlatformIO (for building from source)
pip install platformio

# Verify all tools
esptool.py version
picocom --help
pio --version
```

> UTM on Apple Silicon uses QEMU under the hood with USB passthrough via SPICE. If the device does not appear in UTM's USB menu, try: (1) Unplug and replug the device, (2) Restart the VM, (3) Update UTM to the latest version. USB passthrough can be finicky with certain USB-C hubs — connect directly to the Mac if possible.
{: .tip }

---

## MicroSD Card Setup

The T-Embed CC1101 Plus has a MicroSD card slot for storing captured signals, Ducky Script payloads, IR databases, and configuration files.

### Supported Cards and Formatting

| Specification | Requirement |
|:-------------|:------------|
| **Capacity** | 1GB to 32GB recommended (64GB+ may require FAT32 formatting) |
| **Format** | FAT32 (required — exFAT and NTFS are NOT supported) |
| **Speed class** | Class 10 or higher recommended |
| **Brand** | SanDisk, Samsung, Kingston recommended for reliability |

**Formatting on macOS:**

```bash
# Identify the MicroSD card
diskutil list

# Format as FAT32 (replace diskX with your disk number)
# CAUTION: This erases all data on the card
sudo diskutil eraseDisk FAT32 TEMBED MBRFormat /dev/diskX
```

**Formatting on Linux:**

```bash
# Identify the card
lsblk

# Format as FAT32 (replace sdX1 with your partition)
sudo mkfs.vfat -F 32 /dev/sdX1
```

**Formatting on Windows:**

1. Open File Explorer
2. Right-click the SD card drive
3. Select **Format**
4. Choose **FAT32** as File system
5. Click **Start**

> For cards 64GB or larger, Windows may not offer FAT32 as an option in the format dialog. Use a tool like [Rufus](https://rufus.ie/) or [guiformat](http://ridgecrop.co.uk/index.htm?guiformat.htm) to force FAT32 formatting on larger cards.
{: .note }

### Directory Structure for Saved Signals

Bruce firmware creates and uses the following directory structure on the MicroSD card:

```
/
├── bruce/
│   ├── ir/                  # Saved IR signals
│   │   ├── TV_Power.ir
│   │   ├── AC_Cool_22.ir
│   │   └── ...
│   ├── subghz/              # Saved Sub-GHz signals
│   │   ├── Doorbell.sub
│   │   ├── Weather_Station.sub
│   │   └── ...
│   ├── rfid/                # Saved RFID/NFC data
│   │   ├── Office_Badge.rfid
│   │   └── ...
│   ├── badusb/              # Ducky Script payloads
│   │   ├── hello_world.txt
│   │   ├── reverse_shell.txt
│   │   └── ...
│   ├── wifi/                # WiFi-related files
│   │   └── ...
│   └── config/              # Configuration files
│       └── ...
```

### Organizing Captured Data

Adopt a consistent naming convention from the start:

```
[Category]_[Device]_[Action]_[Date]
```

Examples:

```
IR_SamsungTV_Power_20260228.ir
SubGHz_Doorbell_Ring_20260228.sub
RFID_OfficeBadge_Read_20260228.rfid
```

> Back up your MicroSD card regularly. Captured signals, custom scripts, and configuration files represent hours of work. Copy the entire card to your computer periodically.
{: .tip }

---

## Accessories and Add-ons

### External Antennas (SMA Pigtail Adapters)

The stock antenna works for basic operations, but an external antenna dramatically improves Sub-GHz range.

| Antenna Type | Use Case | Frequency Range | Improvement |
|:-------------|:---------|:----------------|:------------|
| **433 MHz whip antenna** | General Sub-GHz work | 430-440 MHz | 2-3x range |
| **Dual-band 433/868 MHz** | European IoT devices | 430-440 / 860-870 MHz | Multi-band flexibility |
| **915 MHz antenna** | LoRa / US ISM band | 900-930 MHz | Required for 915 MHz work |
| **Wideband discone** | Frequency analysis | 100-1000 MHz | Best for scanning |

**Connection type:** The T-Embed uses an IPEX (U.FL) connector on-board. To connect an external SMA antenna, you need an **IPEX to SMA pigtail cable** (typically 10-15 cm long).

```
T-Embed [IPEX connector] → [IPEX-to-SMA pigtail] → [SMA antenna]
```

> When connecting/disconnecting IPEX cables, use a fingernail or small plastic tool to carefully unclip the connector. IPEX connectors are delicate — they are rated for approximately 30 mating cycles. Do not pull on the cable itself.
{: .warning }

### Cases and Enclosures

The T-Embed does not come with a case. Options:

- **3D Printed** — Search Thingiverse or Printables for "T-Embed CC1101 case." Multiple community designs exist ranging from minimal bumper cases to full enclosures with belt clips
- **Generic project enclosures** — A small ABS project box (approximately 80x50x25mm) can be modified to fit
- **Silicone sleeve** — Some makers sell flexible silicone covers that protect without adding bulk

### Battery Options

The T-Embed CC1101 Plus has a JST 1.25mm 2-pin battery connector for LiPo batteries.

| Battery | Capacity | Size | Runtime Estimate |
|:--------|:---------|:-----|:-----------------|
| **3.7V 500mAh LiPo** | Small | Fits behind board | ~2-3 hours active use |
| **3.7V 1000mAh LiPo** | Medium | Slight overhang | ~5-6 hours active use |
| **3.7V 2000mAh LiPo** | Large | Requires larger case | ~10-12 hours active use |

> Use ONLY 3.7V single-cell LiPo batteries with appropriate JST connectors. Verify the polarity of the connector before plugging in — reversed polarity will permanently destroy the charging circuit and possibly the ESP32-S3. Check with a multimeter if unsure.
{: .danger }

### Expansion Modules

The T-Embed's GPIO pins allow connecting additional hardware:

| Module | Purpose | Connection | Notes |
|:-------|:--------|:-----------|:------|
| **NRF24L01+** | 2.4 GHz radio (MouseJack, sniffing) | SPI (4 wires + power) | Requires soldering to GPIO pads |
| **RFID RC522** | 13.56 MHz RFID read/write | SPI | For MIFARE Classic, NTAG cards |
| **PN532 NFC** | Full NFC support | I2C or SPI | Better NFC compatibility than RC522 |
| **GPS module (NEO-6M)** | Location tagging, wardriving | UART (2 wires + power) | Adds GPS coordinates to captures |
| **External OLED** | Secondary display | I2C | For status info while main screen shows another tool |

### OTG Adapter for BadUSB

To use BadUSB features (where the T-Embed acts as a keyboard), you simply connect the T-Embed directly to the target computer via USB-C. No OTG adapter is needed — the ESP32-S3 supports USB Device mode natively.

However, if you want to connect a USB device TO the T-Embed (USB Host mode), you would need a USB-C OTG adapter. Host mode support depends on firmware implementation.

---

## Safety Checklist Before You Begin

Before using the T-Embed CC1101 Plus for any testing, internalize these rules.

### Legal Awareness

Radio frequency transmission, WiFi deauthentication, and signal replay are regulated by law in every country. Ignorance is not a defense.

| Activity | Legal Status | Notes |
|:---------|:-------------|:------|
| **Passive WiFi scanning** | Legal everywhere | Your phone does this constantly |
| **Passive BLE scanning** | Legal everywhere | Same as above |
| **Passive Sub-GHz listening** | Legal everywhere | Same as AM/FM radio reception |
| **IR capture and replay** | Legal (your own devices) | No RF regulations apply to IR |
| **Sub-GHz signal replay** | Legal only on YOUR devices | Replaying someone else's garage door = illegal |
| **WiFi deauthentication** | Illegal in most jurisdictions | Even on your own network, legality is gray |
| **RF jamming** | Illegal everywhere | Criminal offense in USA, EU, UK, AU, and most countries |
| **Unauthorized network access** | Illegal everywhere | Computer fraud laws apply |

> In the United States, the Computer Fraud and Abuse Act (CFAA), the Wiretap Act, and FCC regulations (47 USC 333) all apply to radio and network hacking. Violations can result in fines up to $100,000 and imprisonment. Similar laws exist in the EU (GDPR, Computer Misuse Act), UK, Canada, Australia, and virtually every other country.
{: .danger }

### What NOT To Do

1. **Never jam radio frequencies** — Jamming interferes with emergency services (police, fire, EMS), aviation, and cellular communications. It is a federal crime in the US and carries up to $112,500 in fines per violation plus imprisonment
2. **Never replay signals on devices you do not own** — Even if "just testing," unauthorized signal replay on someone else's property is illegal
3. **Never deauthenticate clients on networks you do not own** — WiFi deauth attacks disrupt service and may violate computer fraud and wiretapping laws
4. **Never intercept communications without authorization** — Capturing and reading other people's network traffic is wiretapping
5. **Never use the device near airports, military installations, or government buildings** — Even passive scanning may attract unwanted attention and potential legal trouble
6. **Never transmit on frequencies you are not authorized to use** — Amateur radio licenses cover some bands but not all, and the T-Embed is not a certified amateur radio transmitter

### FCC / ETSI Compliance Basics

| Region | Regulatory Body | Key Sub-GHz Frequencies | Power Limits |
|:-------|:----------------|:------------------------|:-------------|
| **USA** | FCC | 315 MHz, 433 MHz, 902-928 MHz ISM | 1 mW (non-licensed) |
| **EU** | ETSI | 433.05-434.79 MHz, 868-870 MHz | 25 mW (433), 25 mW (868) |
| **UK** | Ofcom | Same as EU | Same as EU |
| **Australia** | ACMA | 433 MHz, 915-928 MHz | 25 mW |

The T-Embed CC1101 Plus is NOT FCC certified as a standalone transmitter. Using it to transmit is technically illegal unless:
- You are operating within ISM band power limits
- You are testing your own devices in a controlled environment
- You have an appropriate amateur radio license (for some bands)

### Testing Only Your Own Devices

The golden rule: **if you do not own it, do not touch it.**

Acceptable testing targets:
- Your own WiFi router
- Your own wireless doorbell
- Your own TV/AC remote
- Your own car (parked in your own garage, with rolling code awareness)
- Your own smart home devices
- Devices in a dedicated RF-shielded lab environment

### Documentation Habits

Start a testing log from day one. For every operation:

```
Date: 2026-02-28
Time: 14:30 UTC
Operation: Sub-GHz capture
Target: My wireless doorbell (Brand X, Model Y)
Frequency: 433.92 MHz
Location: My home, indoors
Result: Captured signal, successful replay
Authorization: Own device
Notes: Signal uses simple ASK/OOK, no rolling code
```

This log protects you legally and helps you track what you have learned. Store it on the MicroSD card or in a dedicated notebook.

> A testing log may seem excessive for hobby use, but in a professional penetration testing context, it is mandatory. Building the habit now makes you a better, more disciplined security researcher.
{: .tip }

---

## Summary

At this point you should have:

- [x] Inspected your T-Embed CC1101 Plus and identified the board revision
- [x] Connected it to your computer and established serial communication
- [x] Flashed Bruce firmware using your preferred method
- [x] Configured essential settings (brightness, WiFi, timezone, sound)
- [x] Completed all 5 hands-on operations (WiFi scan, BLE scan, IR capture, frequency analysis, Sub-GHz capture)
- [x] Optionally connected to a Kali Linux VM and set up a MicroSD card
- [x] Read and understood the safety and legal guidelines

You are now ready to dive deep into each capability. The following chapters cover WiFi attacks, Bluetooth exploitation, Sub-GHz radio hacking, infrared systems, BadUSB, and more — each with the same step-by-step depth you have seen here.

---

**Next:** [03 — WiFi Attacks and Analysis](03-wifi-attacks) -- Deep dive into WiFi scanning, deauthentication, beacon spam, evil portals, and packet analysis.
{: .fs-5 }
