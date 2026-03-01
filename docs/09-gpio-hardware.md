---
layout: default
title: "09 — GPIO & Hardware Hacking"
nav_order: 10
---

# 09 — GPIO & Hardware Hacking
{: .no_toc }

The T-Embed CC1101 Plus is more than a self-contained tool. Its exposed GPIO pins transform it into a hardware hacking platform capable of interfacing with external modules, dumping firmware from target devices, sniffing communication buses, and building custom pentest rigs. This chapter covers every pin, every protocol, and every expansion path available to you.
{: .fs-5 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Available GPIO Pins

### Complete Pin Mapping

The LILYGO T-Embed CC1101 Plus is built around the **ESP32-S3-WROOM-1** module. Many GPIO pins are consumed by onboard peripherals. Understanding which pins are free is critical before connecting anything external.

#### Internally Used Pins (DO NOT reassign)

| Function | Pin(s) | Notes |
|:---------|:-------|:------|
| **CC1101 SPI** | GPIO 36 (SCK), GPIO 35 (MOSI), GPIO 37 (MISO), GPIO 34 (CS) | Main Sub-GHz radio SPI bus |
| **CC1101 GDO0** | GPIO 38 | CC1101 interrupt / data ready |
| **CC1101 GDO2** | GPIO 39 | CC1101 status / sync word |
| **TFT Display SPI** | GPIO 12 (SCK), GPIO 11 (MOSI), GPIO 42 (CS), GPIO 16 (DC), GPIO 15 (RST), GPIO 40 (BL) | ST7789 320x170 display |
| **Rotary Encoder** | GPIO 2 (A), GPIO 1 (B), GPIO 0 (Button) | Navigation input |
| **IR Transmitter** | GPIO 44 | Infrared LED output |
| **IR Receiver** | GPIO 4 | TSOP-style IR sensor input |
| **Speaker / Buzzer** | GPIO 46 | PWM audio output |
| **Battery ADC** | GPIO 14 | Battery voltage monitoring |
| **USB D+/D-** | GPIO 19 (D-), GPIO 20 (D+) | USB OTG / CDC / HID |
| **PSRAM** | GPIO 33, GPIO 34 (shared) | OPI PSRAM (internal) |
| **Flash** | GPIO 26-32 | Internal SPI flash (do not touch) |

{: .warning }
> Never attempt to use internally assigned pins for external hardware. Doing so can cause bus conflicts, data corruption, firmware crashes, or permanent hardware damage.

#### Exposed and Available GPIO Pins

The T-Embed CC1101 Plus exposes a limited number of GPIO pins on its headers/pads. The exact availability depends on your board revision, but the following pins are generally accessible:

| GPIO | Capabilities | Notes |
|:-----|:-------------|:------|
| **GPIO 3** | Digital I/O, ADC1_CH2, Touch | General purpose, safe to use |
| **GPIO 5** | Digital I/O, ADC1_CH4 | General purpose |
| **GPIO 6** | Digital I/O, ADC1_CH5 | General purpose |
| **GPIO 7** | Digital I/O, ADC1_CH6 | General purpose |
| **GPIO 8** | Digital I/O, ADC1_CH7 | Commonly used for I2C SDA |
| **GPIO 9** | Digital I/O, ADC1_CH8 | Commonly used for I2C SCL |
| **GPIO 10** | Digital I/O, ADC1_CH9, Touch | General purpose |
| **GPIO 13** | Digital I/O, ADC2_CH2, Touch | General purpose |
| **GPIO 17** | Digital I/O, ADC2_CH6 | UART1 TX candidate |
| **GPIO 18** | Digital I/O, ADC2_CH7 | UART1 RX candidate |
| **GPIO 43** | Digital I/O, UART0 TX | Default serial TX (used by USB CDC) |
| **GPIO 44** | Digital I/O, UART0 RX | Shared with IR TX on some revisions |
| **3V3** | Power | 3.3V regulated output |
| **GND** | Ground | Common ground |
| **5V (VBUS)** | Power | USB 5V passthrough (when USB connected) |

{: .note }
> Pin availability varies between board revisions. Always verify with a multimeter in continuity mode against the schematic before wiring.

### Pin Capabilities

The ESP32-S3 GPIO pins support multiple functions. Here is what each capability means for your projects:

| Capability | Description | Available On |
|:-----------|:------------|:-------------|
| **Digital I/O** | Standard HIGH (3.3V) / LOW (0V) | All GPIO pins |
| **ADC (12-bit)** | Analog-to-digital converter, 0-3.3V range | ADC1: GPIO 1-10; ADC2: GPIO 11-20 |
| **DAC** | Digital-to-analog converter | Not available on ESP32-S3 (use PWM + filter) |
| **Touch** | Capacitive touch sensing | GPIO 1-14 |
| **PWM (LEDC)** | Pulse-width modulation, up to 40 MHz | Any GPIO via LEDC peripheral |
| **I2C** | Two-wire serial (SDA + SCL) | Any GPIO (software-assignable) |
| **SPI** | Four-wire serial (MOSI, MISO, SCK, CS) | Any GPIO (software-assignable) |
| **UART** | Serial communication (TX, RX) | Any GPIO (software-assignable) |
| **RMT** | Remote control transceiver (IR, WS2812B) | Any GPIO via RMT peripheral |

### Voltage Levels and Electrical Limits

{: .warning }
> **The ESP32-S3 is a 3.3V device. Its GPIO pins are NOT 5V tolerant.** Applying 5V to any GPIO pin will damage the chip permanently. Always use a level shifter when interfacing with 5V devices.

| Parameter | Value |
|:----------|:------|
| Logic HIGH | 3.3V |
| Logic LOW | 0V |
| Maximum voltage on any GPIO | 3.6V absolute maximum |
| Maximum source current per pin | 40 mA |
| Maximum sink current per pin | 40 mA |
| Recommended operating current | 20 mA or less |
| Total GPIO current (all pins combined) | 1200 mA max (ESP32-S3 datasheet) |
| Internal pull-up resistor | ~45 kohm |
| Internal pull-down resistor | ~45 kohm |

To enable internal pull-ups or pull-downs in code:

```cpp
pinMode(GPIO_NUM, INPUT_PULLUP);   // Enable internal pull-up
pinMode(GPIO_NUM, INPUT_PULLDOWN); // Enable internal pull-down
```

{: .tip }
> For reliable I2C communication, external 4.7 kohm pull-up resistors to 3.3V on SDA and SCL are strongly recommended. The internal pull-ups are too weak for fast I2C speeds.

---

## Expansion Modules

### NRF24L01+ Module (2.4 GHz Wireless)

The NRF24L01+ adds **2.4 GHz wireless communication** that operates on a completely different protocol layer than WiFi or Bluetooth. It uses the Nordic Semiconductor proprietary ShockBurst protocol and can also be configured for raw 2.4 GHz packet sniffing.

**What it enables:**
- **MouseJack attacks** -- exploit unencrypted wireless mice/keyboards (Logitech Unifying, Microsoft, Dell)
- **2.4 GHz protocol sniffing** -- capture raw packets from baby monitors, drones, wireless sensors
- **Custom wireless links** -- build your own encrypted data channels
- **Drone communication analysis** -- many toy drones use NRF24-compatible protocols

#### Wiring to T-Embed

| NRF24L01+ Pin | T-Embed GPIO | Function |
|:--------------|:-------------|:---------|
| VCC | 3V3 | Power (3.3V only!) |
| GND | GND | Ground |
| CE | GPIO 5 | Chip Enable (TX/RX mode control) |
| CSN | GPIO 6 | SPI Chip Select |
| SCK | GPIO 7 | SPI Clock |
| MOSI | GPIO 3 | SPI Master Out |
| MISO | GPIO 10 | SPI Master In |
| IRQ | GPIO 13 (optional) | Interrupt (not required) |

{: .warning }
> The NRF24L01+ module MUST be powered at 3.3V. The PA+LNA (long-range) versions draw significant current -- add a 10 uF capacitor between VCC and GND directly at the module to prevent power instability.

#### Arduino Code Example

```cpp
#include <SPI.h>
#include <RF24.h>

#define CE_PIN  5
#define CSN_PIN 6

// Use the second SPI bus to avoid conflicts with CC1101/TFT
SPIClass spi2(HSPI);
RF24 radio(CE_PIN, CSN_PIN);

void setup() {
    Serial.begin(115200);

    // Initialize SPI on custom pins
    spi2.begin(7, 10, 3, CSN_PIN);  // SCK, MISO, MOSI, CS

    if (!radio.begin(&spi2)) {
        Serial.println("NRF24L01+ not detected!");
        while (1);
    }

    Serial.println("NRF24L01+ initialized");

    // Configure for promiscuous sniffing
    radio.setAutoAck(false);
    radio.setDataRate(RF24_2MBPS);
    radio.setCRCLength(RF24_CRC_DISABLED);
    radio.setPayloadSize(32);
    radio.setAddressWidth(3);

    // Open a sniffing pipe
    uint8_t addr[] = {0xAA, 0xAA, 0xAA};
    radio.openReadingPipe(0, addr);
    radio.startListening();

    Serial.println("Listening on 2.4 GHz...");
}

void loop() {
    if (radio.available()) {
        uint8_t buf[32];
        radio.read(buf, 32);

        Serial.print("Packet: ");
        for (int i = 0; i < 32; i++) {
            if (buf[i] < 0x10) Serial.print("0");
            Serial.print(buf[i], HEX);
            Serial.print(" ");
        }
        Serial.println();
    }
}
```

#### MouseJack Scanning Example

```cpp
// Sweep channels 0-125 looking for wireless keyboard/mouse traffic
void mousejackScan() {
    for (uint8_t channel = 0; channel <= 125; channel++) {
        radio.setChannel(channel);
        radio.startListening();
        delayMicroseconds(250);

        if (radio.available()) {
            uint8_t buf[32];
            radio.read(buf, 32);

            Serial.printf("CH %3d (2.%03d GHz): ", channel, 400 + channel);
            for (int i = 0; i < 32; i++) {
                Serial.printf("%02X ", buf[i]);
            }
            Serial.println();
        }
    }
}
```

{: .note }
> Bruce firmware includes NRF24 scanning features when the module is connected. Check the Bruce documentation for supported pin configurations.

---

### RFID RC522 Module (13.56 MHz)

The MFRC522 reads and writes **13.56 MHz RFID tags**, primarily **MIFARE Classic** (1K and 4K) and **MIFARE Ultralight** cards. These are the most common access cards, transit cards, and payment tokens worldwide.

**What it enables:**
- Read UIDs from access cards, transit passes, hotel keys
- Dump MIFARE Classic sector data (with known keys)
- Clone card UIDs to writable "magic" cards (UID-changeable cards like Gen1a/Gen2)
- Key recovery attacks on MIFARE Classic (darkside, nested, hardnested)

#### Wiring to T-Embed

| RC522 Pin | T-Embed GPIO | Function |
|:----------|:-------------|:---------|
| VCC | 3V3 | Power (3.3V) |
| GND | GND | Ground |
| SDA (CS) | GPIO 6 | SPI Chip Select |
| SCK | GPIO 7 | SPI Clock |
| MOSI | GPIO 3 | SPI Master Out |
| MISO | GPIO 10 | SPI Master In |
| RST | GPIO 5 | Reset |
| IRQ | Not connected | Not used |

#### Arduino Code Example

```cpp
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN  6
#define RST_PIN 5

SPIClass spi2(HSPI);
MFRC522 mfrc522(SS_PIN, RST_PIN);

void setup() {
    Serial.begin(115200);
    spi2.begin(7, 10, 3, SS_PIN);  // SCK, MISO, MOSI, CS
    mfrc522.PCD_Init();
    Serial.println("RC522 RFID Reader initialized");
    Serial.println("Place a card near the reader...");
}

void loop() {
    // Look for new cards
    if (!mfrc522.PICC_IsNewCardPresent()) return;
    if (!mfrc522.PICC_ReadCardSerial()) return;

    // Print UID
    Serial.print("Card UID: ");
    for (byte i = 0; i < mfrc522.uid.size; i++) {
        if (mfrc522.uid.uidByte[i] < 0x10) Serial.print("0");
        Serial.print(mfrc522.uid.uidByte[i], HEX);
        if (i < mfrc522.uid.size - 1) Serial.print(":");
    }
    Serial.println();

    // Print card type
    MFRC522::PICC_Type piccType = mfrc522.PICC_GetType(mfrc522.uid.sak);
    Serial.print("Card Type: ");
    Serial.println(mfrc522.PICC_GetTypeName(piccType));

    // Dump MIFARE Classic sector 0 with default key
    if (piccType == MFRC522::PICC_TYPE_MIFARE_1K ||
        piccType == MFRC522::PICC_TYPE_MIFARE_4K) {
        dumpSector(0);
    }

    mfrc522.PICC_HaltA();
    mfrc522.PCD_StopCrypto1();

    delay(1000);
}

void dumpSector(byte sector) {
    MFRC522::MIFARE_Key key;
    // Default MIFARE key: FF FF FF FF FF FF
    for (byte i = 0; i < 6; i++) key.keyByte[i] = 0xFF;

    byte firstBlock = sector * 4;

    // Authenticate with Key A
    MFRC522::StatusCode status = mfrc522.PCD_Authenticate(
        MFRC522::PICC_CMD_MF_AUTH_KEY_A, firstBlock, &key, &(mfrc522.uid));

    if (status != MFRC522::STATUS_OK) {
        Serial.print("Auth failed: ");
        Serial.println(mfrc522.GetStatusCodeName(status));
        return;
    }

    Serial.printf("--- Sector %d ---\n", sector);

    // Read all 4 blocks in the sector
    for (byte block = firstBlock; block < firstBlock + 4; block++) {
        byte buffer[18];
        byte size = sizeof(buffer);

        status = mfrc522.MIFARE_Read(block, buffer, &size);
        if (status == MFRC522::STATUS_OK) {
            Serial.printf("Block %2d: ", block);
            for (byte i = 0; i < 16; i++) {
                Serial.printf("%02X ", buffer[i]);
            }
            // Print ASCII representation
            Serial.print(" | ");
            for (byte i = 0; i < 16; i++) {
                Serial.print(buffer[i] >= 0x20 && buffer[i] < 0x7F
                    ? (char)buffer[i] : '.');
            }
            Serial.println();
        }
    }
}
```

#### MIFARE Classic Key Recovery Concepts

MIFARE Classic uses a proprietary cipher called **Crypto-1** which has been thoroughly broken. Common attack vectors:

1. **Default keys** -- Many cards ship with factory default keys (`FF FF FF FF FF FF`, `A0 A1 A2 A3 A4 A5`, etc.). Always try these first.
2. **Darkside attack** -- Exploits a weakness in the PRNG when authentication fails. Works on MIFARE Classic cards without any known keys.
3. **Nested authentication** -- Once you have one key, exploit the PRNG to recover other sector keys.
4. **Hardnested attack** -- Works on "fixed nonce" cards where the darkside attack fails.

{: .warning }
> Cloning access cards you do not own is illegal in most jurisdictions. Only test on cards you own or have explicit written authorization to test.

---

### PN532 NFC Module (Full NFC)

The PN532 is a more capable NFC controller than the RC522. It supports **read, write, and card emulation** across multiple NFC types.

**What it adds over RC522:**
- **Card emulation** -- make the T-Embed appear as an NFC card
- **Peer-to-peer** -- NFC data exchange between devices
- **NDEF support** -- read and write standard NFC Data Exchange Format records
- **FeliCa** -- Japanese transit card support (Suica, Pasmo)
- **Multiple interfaces** -- I2C, SPI, or UART (I2C recommended to save SPI pins)

#### Wiring (I2C Mode -- Recommended)

| PN532 Pin | T-Embed GPIO | Function |
|:----------|:-------------|:---------|
| VCC | 3V3 or 5V | Power (check module) |
| GND | GND | Ground |
| SDA | GPIO 8 | I2C Data |
| SCL | GPIO 9 | I2C Clock |
| IRQ | GPIO 13 (optional) | Interrupt |
| RST | GPIO 5 (optional) | Reset |

Set the PN532 interface DIP switches to I2C mode (usually both switches OFF, but check your module's documentation).

#### NFC Card Emulation Example

```cpp
#include <Wire.h>
#include <Adafruit_PN532.h>

#define PN532_IRQ   13
#define PN532_RESET  5

Adafruit_PN532 nfc(PN532_IRQ, PN532_RESET);

void setup() {
    Serial.begin(115200);
    nfc.begin();

    uint32_t versiondata = nfc.getFirmwareVersion();
    if (!versiondata) {
        Serial.println("PN532 not found!");
        while (1);
    }

    Serial.printf("PN532 Firmware: %d.%d\n",
        (versiondata >> 16) & 0xFF,
        (versiondata >> 8) & 0xFF);

    nfc.SAMConfig();
    Serial.println("PN532 ready");
}

void loop() {
    uint8_t uid[7];
    uint8_t uidLength;

    Serial.println("Waiting for NFC card...");

    if (nfc.readPassiveTargetID(PN532_MIFARE_ISO14443A,
                                uid, &uidLength, 5000)) {
        Serial.print("UID: ");
        for (uint8_t i = 0; i < uidLength; i++) {
            Serial.printf("%02X", uid[i]);
            if (i < uidLength - 1) Serial.print(":");
        }
        Serial.printf(" (%d bytes)\n", uidLength);

        // Read NDEF message if present
        if (uidLength == 4) {
            readNDEF();
        }
    }
}

void readNDEF() {
    // Read block 4 (start of NDEF data on NTAG/Ultralight)
    uint8_t data[32];

    for (uint8_t page = 4; page < 16; page++) {
        if (nfc.mifareclassic_ReadDataBlock(page, data)) {
            Serial.printf("Page %2d: ", page);
            for (int i = 0; i < 4; i++) {
                Serial.printf("%02X ", data[i]);
            }
            Serial.println();
        }
    }
}

// Write NDEF URL record to NTAG213/215/216
void writeNDEFUrl(const char* url) {
    uint8_t ndefBuf[64];
    uint8_t len = strlen(url);

    // NDEF message header
    ndefBuf[0] = 0x03;           // NDEF Message TLV
    ndefBuf[1] = len + 5;       // Length
    ndefBuf[2] = 0xD1;          // MB=1, ME=1, SR=1, TNF=1 (Well-known)
    ndefBuf[3] = 0x01;          // Type length = 1
    ndefBuf[4] = len + 1;       // Payload length
    ndefBuf[5] = 0x55;          // Type: 'U' (URI)
    ndefBuf[6] = 0x04;          // URI prefix: https://
    memcpy(&ndefBuf[7], url, len);
    ndefBuf[7 + len] = 0xFE;   // Terminator TLV

    // Write 4 bytes per page starting at page 4
    uint8_t totalBytes = 8 + len;
    for (uint8_t i = 0; i < totalBytes; i += 4) {
        uint8_t page = 4 + (i / 4);
        nfc.mifareclassic_WriteDataBlock(page, &ndefBuf[i]);
    }

    Serial.println("NDEF URL written successfully");
}
```

---

### GPS Module (NEO-6M / NEO-7M)

Adding a GPS module enables **wardriving** -- combining WiFi or Bluetooth scanning with precise geographic coordinates. The NEO-6M and NEO-7M modules communicate over UART and output standard NMEA sentences.

#### Wiring (UART)

| GPS Pin | T-Embed GPIO | Function |
|:--------|:-------------|:---------|
| VCC | 3V3 | Power |
| GND | GND | Ground |
| TX | GPIO 18 | GPS transmits to ESP32 RX |
| RX | GPIO 17 | ESP32 transmits to GPS (optional) |

{: .note }
> GPS TX connects to your ESP32 RX pin, and vice versa. This is a common wiring mistake.

#### NMEA Parsing and Wardriving Logger

```cpp
#include <TinyGPS++.h>
#include <WiFi.h>
#include <FS.h>
#include <SPIFFS.h>

TinyGPSPlus gps;
HardwareSerial gpsSerial(1);  // UART1

#define GPS_TX 18  // GPS module TX -> ESP32 RX
#define GPS_RX 17  // ESP32 TX -> GPS module RX

File logFile;

void setup() {
    Serial.begin(115200);
    gpsSerial.begin(9600, SERIAL_8N1, GPS_TX, GPS_RX);

    SPIFFS.begin(true);
    logFile = SPIFFS.open("/wardriving.csv", FILE_APPEND);

    if (logFile) {
        // WiGLE CSV format header
        logFile.println("MAC,SSID,AuthMode,FirstSeen,Channel,"
                        "RSSI,CurrentLatitude,CurrentLongitude,"
                        "AltitudeMeters,AccuracyMeters,Type");
    }

    WiFi.mode(WIFI_STA);
    WiFi.disconnect();

    Serial.println("Wardriving logger started");
    Serial.println("Waiting for GPS fix...");
}

void loop() {
    // Feed GPS data
    while (gpsSerial.available()) {
        gps.encode(gpsSerial.read());
    }

    if (gps.location.isValid()) {
        // Scan WiFi networks
        int networks = WiFi.scanNetworks(false, true);  // async=false, hidden=true

        for (int i = 0; i < networks; i++) {
            String line = "";
            line += WiFi.BSSIDstr(i) + ",";
            line += WiFi.SSID(i) + ",";
            line += getAuthType(WiFi.encryptionType(i)) + ",";
            line += getTimestamp() + ",";
            line += String(WiFi.channel(i)) + ",";
            line += String(WiFi.RSSI(i)) + ",";
            line += String(gps.location.lat(), 6) + ",";
            line += String(gps.location.lng(), 6) + ",";
            line += String(gps.altitude.meters(), 1) + ",";
            line += String(gps.hdop.hdop(), 1) + ",";
            line += "WIFI";

            logFile.println(line);
            Serial.println(line);
        }

        logFile.flush();
        Serial.printf("Logged %d networks at %.6f, %.6f\n",
            networks, gps.location.lat(), gps.location.lng());
    } else {
        Serial.printf("GPS: Satellites=%d, waiting for fix...\n",
            gps.satellites.value());
    }

    delay(5000);  // Scan every 5 seconds
}

String getAuthType(wifi_auth_mode_t auth) {
    switch (auth) {
        case WIFI_AUTH_OPEN:          return "OPEN";
        case WIFI_AUTH_WEP:           return "WEP";
        case WIFI_AUTH_WPA_PSK:       return "WPA_PSK";
        case WIFI_AUTH_WPA2_PSK:      return "WPA2_PSK";
        case WIFI_AUTH_WPA_WPA2_PSK:  return "WPA_WPA2_PSK";
        case WIFI_AUTH_WPA3_PSK:      return "WPA3_PSK";
        default:                      return "UNKNOWN";
    }
}

String getTimestamp() {
    char buf[20];
    snprintf(buf, sizeof(buf), "%04d-%02d-%02d %02d:%02d:%02d",
        gps.date.year(), gps.date.month(), gps.date.day(),
        gps.time.hour(), gps.time.minute(), gps.time.second());
    return String(buf);
}
```

{: .tip }
> The output CSV is compatible with [WiGLE.net](https://wigle.net) for uploading and visualizing your wardriving data on a global map.

---

### External Antenna (SMA)

The onboard CC1101 antenna works for short range, but an external antenna dramatically improves range and sensitivity.

#### IPEX to SMA Pigtail

You need an **IPEX (U.FL) to SMA female** pigtail cable. The CC1101 module on the T-Embed has an IPEX connector (sometimes labeled U.FL). Connect the pigtail, then attach any SMA antenna.

**When you need an external antenna:**
- Receiving signals beyond 10-20 meters
- Transmitting to devices more than a few meters away
- Operating in environments with walls, interference, or noise
- Working with very weak signals
- When directionality matters (Yagi antennas)

#### Quarter-Wave Antenna Cutting Guide

A quarter-wave monopole is the simplest and most effective antenna you can build. The formula is:

```
Length (mm) = 71250 / Frequency (MHz)
```

| Frequency | Band | Quarter-Wave Length | Use Case |
|:----------|:-----|:-------------------|:---------|
| 315 MHz | Sub-GHz | 226.2 mm (8.9 in) | Garage doors, older remotes |
| 433.92 MHz | Sub-GHz | 164.2 mm (6.5 in) | Most common: remotes, sensors, key fobs |
| 868 MHz | Sub-GHz | 82.1 mm (3.2 in) | European IoT, LoRa |
| 915 MHz | Sub-GHz | 77.9 mm (3.1 in) | US ISM band, LoRa |

**Build instructions:**
1. Get a piece of solid copper wire (14-18 AWG)
2. Cut to the length above
3. Solder to the center pin of an SMA male connector
4. The SMA ground shell acts as the ground plane

{: .tip }
> For even better performance, build a ground plane antenna by adding 4 radial wires (same length) angled 45 degrees down from the base. This creates an effective ground plane for the monopole.

#### Antenna Gain and Directionality

| Antenna Type | Gain | Pattern | Use Case |
|:-------------|:-----|:--------|:---------|
| Rubber duck (stock) | 2-3 dBi | Omnidirectional | General use |
| Quarter-wave monopole | 2-5 dBi | Omnidirectional | All-around reception |
| Half-wave dipole | 2-5 dBi | Omnidirectional | Balanced performance |
| Yagi (3-element) | 7-10 dBi | Directional | Long-range targeted |
| Yagi (5+ element) | 10-15 dBi | Highly directional | Maximum range, precise aiming |

---

### SD Card Module

Adding an SD card provides virtually unlimited storage for captures, logs, firmware dumps, and signal recordings.

#### Wiring (SPI)

| SD Card Pin | T-Embed GPIO | Function |
|:------------|:-------------|:---------|
| VCC | 3V3 | Power |
| GND | GND | Ground |
| CS | GPIO 6 | Chip Select |
| SCK | GPIO 7 | SPI Clock |
| MOSI | GPIO 3 | SPI Data In |
| MISO | GPIO 10 | SPI Data Out |

```cpp
#include <SPI.h>
#include <SD.h>

#define SD_CS 6

SPIClass sdSPI(HSPI);

void setup() {
    Serial.begin(115200);
    sdSPI.begin(7, 10, 3, SD_CS);

    if (!SD.begin(SD_CS, sdSPI)) {
        Serial.println("SD card mount failed!");
        return;
    }

    uint8_t cardType = SD.cardType();
    Serial.printf("SD Card Type: %s\n",
        cardType == CARD_SD ? "SD" :
        cardType == CARD_SDHC ? "SDHC" :
        cardType == CARD_MMC ? "MMC" : "Unknown");

    Serial.printf("SD Card Size: %llu MB\n", SD.cardSize() / (1024 * 1024));

    // Write a test file
    File file = SD.open("/test.txt", FILE_WRITE);
    if (file) {
        file.println("T-Embed CC1101 SD Card Test");
        file.close();
        Serial.println("Test file written");
    }
}
```

{: .note }
> When sharing the SPI bus with other devices (NRF24, RC522), ensure each device has its own CS pin and that only one device is selected at a time. Pull all unused CS pins HIGH.

---

### OLED / LCD Additional Displays

A secondary I2C OLED display (SSD1306 128x64 or 128x32) is useful for displaying status information without consuming the main TFT for your primary application.

#### Wiring (I2C)

| OLED Pin | T-Embed GPIO | Function |
|:---------|:-------------|:---------|
| VCC | 3V3 | Power |
| GND | GND | Ground |
| SDA | GPIO 8 | I2C Data |
| SCL | GPIO 9 | I2C Clock |

```cpp
#include <Wire.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT  64
#define OLED_SDA  8
#define OLED_SCL  9

Adafruit_SSD1306 oled(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

void setup() {
    Wire.begin(OLED_SDA, OLED_SCL);

    if (!oled.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
        Serial.println("SSD1306 OLED not found!");
        return;
    }

    oled.clearDisplay();
    oled.setTextSize(1);
    oled.setTextColor(SSD1306_WHITE);
    oled.setCursor(0, 0);
    oled.println("T-Embed CC1101+");
    oled.println("Status: Ready");
    oled.printf("Freq: 433.92 MHz\n");
    oled.printf("RSSI: -45 dBm\n");
    oled.display();
}
```

---

## Hardware Communication Protocols

Understanding SPI, I2C, and UART is fundamental to hardware hacking. The T-Embed uses all three internally, and you will encounter them on nearly every target device.

### SPI (Serial Peripheral Interface)

SPI is a **synchronous, full-duplex, master-slave** protocol. It is the fastest common serial protocol and is used by the CC1101 radio and the TFT display on the T-Embed.

#### How SPI Works

```
         Master (ESP32)              Slave (CC1101)
        ┌──────────────┐           ┌──────────────┐
        │         MOSI ├──────────>│ MOSI         │
        │         MISO │<──────────┤ MISO         │
        │          SCK ├──────────>│ SCK          │
        │           CS ├──────────>│ CS           │
        └──────────────┘           └──────────────┘

MOSI = Master Out, Slave In   (data TO the slave)
MISO = Master In, Slave Out   (data FROM the slave)
SCK  = Serial Clock           (master controls timing)
CS   = Chip Select             (LOW = selected, HIGH = deselected)
```

**Key characteristics:**
- **Full duplex** -- data flows both directions simultaneously
- **No addressing** -- devices selected by individual CS lines
- **Fast** -- up to 80 MHz on ESP32-S3
- **4 SPI modes** -- defined by clock polarity (CPOL) and phase (CPHA)

| SPI Mode | CPOL | CPHA | Clock Idle | Data Sampled |
|:---------|:-----|:-----|:-----------|:-------------|
| Mode 0 | 0 | 0 | LOW | Rising edge |
| Mode 1 | 0 | 1 | LOW | Falling edge |
| Mode 2 | 1 | 0 | HIGH | Falling edge |
| Mode 3 | 1 | 1 | HIGH | Rising edge |

#### SPI Bus Sharing Considerations

The T-Embed CC1101 Plus already uses SPI for both the CC1101 radio (FSPI/SPI2) and the TFT display (FSPI/SPI2 shared or SPI3). Adding more SPI devices requires careful management:

1. **Use the second SPI bus** -- ESP32-S3 has two usable SPI peripherals (SPI2/HSPI and SPI3/VSPI). Use the one not occupied by the display and CC1101.
2. **Separate CS pins** -- Every SPI device needs its own CS pin. Pull all CS pins HIGH at startup.
3. **Mutex / semaphore** -- If sharing a bus, use a FreeRTOS mutex to prevent simultaneous access.

```cpp
SemaphoreHandle_t spiMutex = xSemaphoreCreateMutex();

void readCC1101() {
    if (xSemaphoreTake(spiMutex, portMAX_DELAY)) {
        digitalWrite(CC1101_CS, LOW);
        // ... SPI transaction ...
        digitalWrite(CC1101_CS, HIGH);
        xSemaphoreGive(spiMutex);
    }
}

void readSD() {
    if (xSemaphoreTake(spiMutex, portMAX_DELAY)) {
        digitalWrite(SD_CS, LOW);
        // ... SPI transaction ...
        digitalWrite(SD_CS, HIGH);
        xSemaphoreGive(spiMutex);
    }
}
```

#### Arduino SPI Example

```cpp
#include <SPI.h>

SPIClass hspi(HSPI);

void setup() {
    hspi.begin(7, 10, 3);  // SCK, MISO, MOSI
    pinMode(6, OUTPUT);
    digitalWrite(6, HIGH);  // CS inactive
}

// Read a register from an SPI device
uint8_t spiReadRegister(uint8_t reg) {
    uint8_t result;
    digitalWrite(6, LOW);                    // Select device
    hspi.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
    hspi.transfer(reg | 0x80);              // Set read bit
    result = hspi.transfer(0x00);            // Clock out response
    hspi.endTransaction();
    digitalWrite(6, HIGH);                   // Deselect device
    return result;
}

// Write a register on an SPI device
void spiWriteRegister(uint8_t reg, uint8_t value) {
    digitalWrite(6, LOW);
    hspi.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
    hspi.transfer(reg & 0x7F);              // Clear read bit
    hspi.transfer(value);
    hspi.endTransaction();
    digitalWrite(6, HIGH);
}
```

---

### I2C (Inter-Integrated Circuit)

I2C is a **synchronous, half-duplex, multi-master** two-wire protocol. It is slower than SPI but uses only two wires and supports up to 127 devices on a single bus (each with a unique address).

#### How I2C Works

```
              ┌──────┐     ┌──────┐     ┌──────┐
   3.3V ──┬──┤ 4.7k ├──┬──┤ 4.7k ├──┬──┤      │
           │  └──────┘  │  └──────┘  │  │      │
           │            │            │  │      │
    SDA ───┼────────────┼────────────┼──┤ Dev3 │
           │            │            │  │      │
    SCL ───┼────────────┼────────────┼──┤      │
           │            │            │  └──────┘
       ┌───┴──┐     ┌───┴──┐        │
       │ Dev1 │     │ Dev2 │        │
       └──────┘     └──────┘        │
                                    │
                               ┌────┴───┐
                               │ Master │
                               │(ESP32) │
                               └────────┘
```

**Key characteristics:**
- **Two wires only** -- SDA (data) and SCL (clock)
- **Addressed** -- each device has a 7-bit address (0x00-0x7F)
- **Multi-device** -- up to 127 devices on one bus
- **Pull-up resistors required** -- 4.7 kohm to 3.3V on both SDA and SCL
- **Speeds** -- Standard (100 kHz), Fast (400 kHz), Fast+ (1 MHz)

#### I2C Address Scanning

This is one of the most useful diagnostic tools. Scan the bus to discover what devices are connected and their addresses:

```cpp
#include <Wire.h>

#define I2C_SDA 8
#define I2C_SCL 9

void setup() {
    Serial.begin(115200);
    Wire.begin(I2C_SDA, I2C_SCL);

    Serial.println("I2C Scanner");
    Serial.println("Scanning...\n");

    int devicesFound = 0;

    for (uint8_t addr = 1; addr < 127; addr++) {
        Wire.beginTransmission(addr);
        uint8_t error = Wire.endTransmission();

        if (error == 0) {
            Serial.printf("Device found at address 0x%02X", addr);

            // Identify common devices
            switch (addr) {
                case 0x3C: case 0x3D:
                    Serial.print(" (SSD1306 OLED)"); break;
                case 0x24:
                    Serial.print(" (PN532 NFC)"); break;
                case 0x27: case 0x3F:
                    Serial.print(" (PCF8574 I/O Expander)"); break;
                case 0x50: case 0x51: case 0x52: case 0x53:
                case 0x54: case 0x55: case 0x56: case 0x57:
                    Serial.print(" (AT24Cxx EEPROM)"); break;
                case 0x68:
                    Serial.print(" (DS3231 RTC / MPU6050)"); break;
                case 0x76: case 0x77:
                    Serial.print(" (BME280/BMP280 Sensor)"); break;
            }

            Serial.println();
            devicesFound++;
        }
    }

    Serial.printf("\nScan complete. %d device(s) found.\n", devicesFound);
}

void loop() {}
```

#### Arduino I2C Examples

**Reading an EEPROM (AT24C32):**

```cpp
// Read a byte from EEPROM at given address
uint8_t eepromRead(uint8_t devAddr, uint16_t memAddr) {
    Wire.beginTransmission(devAddr);
    Wire.write((uint8_t)(memAddr >> 8));   // MSB of memory address
    Wire.write((uint8_t)(memAddr & 0xFF)); // LSB of memory address
    Wire.endTransmission();

    Wire.requestFrom(devAddr, (uint8_t)1);
    return Wire.available() ? Wire.read() : 0xFF;
}

// Dump first 256 bytes of EEPROM
void dumpEEPROM(uint8_t devAddr) {
    Serial.println("EEPROM Dump:");
    Serial.println("     00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F");

    for (uint16_t addr = 0; addr < 256; addr += 16) {
        Serial.printf("%04X ", addr);
        for (uint8_t i = 0; i < 16; i++) {
            Serial.printf("%02X ", eepromRead(devAddr, addr + i));
        }
        Serial.println();
    }
}
```

---

### UART (Universal Asynchronous Receiver/Transmitter)

UART is the simplest serial protocol -- **asynchronous, full-duplex, point-to-point**. No clock wire needed; both sides agree on a baud rate. UART is the most important protocol for hardware hacking because nearly every embedded device has a UART debug console.

#### How UART Works

```
    Device A                    Device B
   ┌────────┐                 ┌────────┐
   │     TX ├────────────────>│ RX     │
   │     RX │<────────────────┤ TX     │
   │    GND ├─────────────────┤ GND    │
   └────────┘                 └────────┘

TX crosses to RX. Always connect GND between devices.
```

**UART frame format:**

```
    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬────┐
    │ S │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ P │ ST │
    └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴────┘
     Start  ────── Data Bits ──────  Par  Stop
      Bit        (LSB first)        Bit   Bit
```

**Common baud rates:** 9600, 19200, 38400, 57600, 115200, 230400, 460800, 921600

**Common configurations:** 8N1 (8 data bits, No parity, 1 stop bit) is used by ~95% of devices.

#### Arduino UART Examples

**Using hardware serial ports:**

```cpp
// ESP32-S3 has 3 UART peripherals: UART0, UART1, UART2
// UART0 is typically used for USB CDC serial monitor

HardwareSerial targetUART(1);  // UART1

#define TARGET_RX  18  // GPIO to receive from target TX
#define TARGET_TX  17  // GPIO to transmit to target RX

void setup() {
    Serial.begin(115200);  // USB serial to PC
    targetUART.begin(115200, SERIAL_8N1, TARGET_RX, TARGET_TX);

    Serial.println("UART bridge ready");
    Serial.println("Type commands to send to target device:");
}

void loop() {
    // Forward data from USB serial to target
    while (Serial.available()) {
        targetUART.write(Serial.read());
    }

    // Forward data from target to USB serial
    while (targetUART.available()) {
        Serial.write(targetUART.read());
    }
}
```

**Auto baud rate detection:**

```cpp
// Try common baud rates and look for printable ASCII
uint32_t baudRates[] = {9600, 19200, 38400, 57600, 115200, 230400, 460800, 921600};

void detectBaudRate() {
    for (int i = 0; i < 8; i++) {
        targetUART.begin(baudRates[i], SERIAL_8N1, TARGET_RX, TARGET_TX);
        Serial.printf("Trying %d baud... ", baudRates[i]);

        unsigned long start = millis();
        int printableCount = 0;
        int totalCount = 0;

        while (millis() - start < 2000) {  // Listen for 2 seconds
            if (targetUART.available()) {
                char c = targetUART.read();
                totalCount++;
                if (c >= 0x20 && c <= 0x7E) printableCount++;
            }
        }

        if (totalCount > 0) {
            float ratio = (float)printableCount / totalCount * 100;
            Serial.printf("Got %d bytes, %.0f%% printable", totalCount, ratio);
            if (ratio > 70) {
                Serial.println(" <-- LIKELY MATCH");
            } else {
                Serial.println();
            }
        } else {
            Serial.println("No data");
        }

        targetUART.end();
    }
}
```

---

## Hardware Hacking Use Cases

### UART Console Access

This is the single most common hardware hacking technique. Nearly every embedded device -- routers, IoT cameras, smart speakers, printers -- has UART test pads on its PCB that give access to a boot console and often a root shell.

#### Step 1: Identify UART Pads on Target Hardware

1. **Open the device** and examine the PCB
2. **Look for** a row of 3-4 pads/pins labeled TX, RX, GND (and sometimes VCC)
3. **Common indicators:**
   - 4 pads in a row near the processor
   - Pads labeled J1, J2, J3, JP1, or TP (test point)
   - Unpopulated pin headers
4. **Use a multimeter:**
   - GND: continuity with any ground point (shield, USB connector shell)
   - VCC: steady 3.3V or 5V
   - TX: fluctuating voltage (the device is transmitting data at boot)
   - RX: pulled high (3.3V) or floating

{: .warning }
> **Check voltage levels first!** If the target UART operates at 5V, you MUST use a level shifter. Connecting 5V TX directly to the ESP32 GPIO will destroy it.

#### Step 2: Connect T-Embed GPIO to Target UART

```
Target Device              T-Embed CC1101+
┌──────────┐              ┌──────────────┐
│   TX ────┼──────────────┤ GPIO 18 (RX) │
│   RX ────┼──────────────┤ GPIO 17 (TX) │
│  GND ────┼──────────────┤ GND          │
└──────────┘              └──────────────┘

DO NOT connect VCC between devices unless you know what you are doing.
```

#### Step 3: Use T-Embed as a Serial Terminal

```cpp
// Simple UART pass-through: T-Embed acts as USB-to-UART bridge
HardwareSerial target(1);

void setup() {
    Serial.begin(115200);      // USB to PC
    target.begin(115200, SERIAL_8N1, 18, 17);  // To target device
    Serial.println("=== T-Embed UART Terminal ===");
    Serial.println("Connected. Type to interact with target.");
}

void loop() {
    while (Serial.available())  target.write(Serial.read());
    while (target.available())  Serial.write(target.read());
}
```

#### What You Can Typically Extract

- **Boot logs** -- processor type, firmware version, memory layout, boot arguments
- **Linux shell** -- many devices drop to a root shell with no authentication
- **U-Boot prompt** -- interrupt boot with key press, modify boot arguments, dump flash
- **Debug info** -- error messages, stack traces, configuration dumps
- **Credentials** -- hardcoded passwords, API keys, WiFi credentials in boot logs

---

### SPI Flash Dumping

Many embedded devices store their firmware on an external SPI flash chip. You can read (dump) the entire chip contents for offline analysis.

#### Common SPI Flash Chips

| Chip | Capacity | Package | Common In |
|:-----|:---------|:--------|:----------|
| W25Q32 | 4 MB | SOIC-8 | IoT devices, cameras |
| W25Q64 | 8 MB | SOIC-8 | Routers, access points |
| W25Q128 | 16 MB | SOIC-8 | Smart devices, NAS |
| MX25L6405 | 8 MB | SOIC-8 | Various embedded |

#### Wiring T-Embed to SPI Flash

```
SPI Flash (SOIC-8)            T-Embed
┌───────────────┐           ┌──────────┐
│ 1  CS   VCC 8 ├───── 3V3 ─┤ 3V3      │
│ 2  MISO  /H 7 ├───── 3V3 ─┤ 3V3      │   (HOLD pin high)
│ 3  /WP  SCK 6 ├───── 3V3 ─┤ GPIO 7   │
│ 4  GND MOSI 5 │           │          │
└───┬───────┬───┘           │          │
    │       │               │          │
 GND ─── GND ──────────────┤ GND      │
                            │          │
Pin 1 (CS)   ──────────────┤ GPIO 6   │
Pin 2 (MISO) ──────────────┤ GPIO 10  │
Pin 5 (MOSI) ──────────────┤ GPIO 3   │
Pin 6 (SCK)  ──────────────┤ GPIO 7   │
                            └──────────┘
```

{: .warning }
> **Desolder or isolate the flash chip** from the target board before reading. The target's processor may interfere with SPI communication if still connected. Alternatively, use a SOIC-8 clip for in-circuit reading (less reliable but non-destructive).

#### Dumping with Custom Code

```cpp
#include <SPI.h>

#define FLASH_CS 6
SPIClass flashSPI(HSPI);

void setup() {
    Serial.begin(921600);  // High baud for fast transfer
    flashSPI.begin(7, 10, 3, FLASH_CS);
    pinMode(FLASH_CS, OUTPUT);
    digitalWrite(FLASH_CS, HIGH);

    // Read JEDEC ID
    uint32_t jedec = readJEDECID();
    Serial.printf("JEDEC ID: %06X\n", jedec);
    Serial.printf("Manufacturer: %02X\n", (jedec >> 16) & 0xFF);
    Serial.printf("Device Type:  %02X\n", (jedec >> 8) & 0xFF);
    Serial.printf("Capacity:     %02X\n", jedec & 0xFF);

    // Calculate size from capacity byte
    uint32_t sizeBytes = 1 << (jedec & 0xFF);
    Serial.printf("Flash Size:   %d bytes (%d MB)\n", sizeBytes, sizeBytes / 1048576);

    Serial.println("\nSending firmware dump via serial...");
    Serial.println("Use: screen -L /dev/ttyUSB0 921600 to capture");
    delay(2000);

    // Dump entire flash
    dumpFlash(sizeBytes);
}

uint32_t readJEDECID() {
    digitalWrite(FLASH_CS, LOW);
    flashSPI.transfer(0x9F);  // JEDEC ID command
    uint8_t mfr = flashSPI.transfer(0);
    uint8_t type = flashSPI.transfer(0);
    uint8_t cap = flashSPI.transfer(0);
    digitalWrite(FLASH_CS, HIGH);
    return ((uint32_t)mfr << 16) | ((uint32_t)type << 8) | cap;
}

void dumpFlash(uint32_t size) {
    uint8_t buf[256];

    for (uint32_t addr = 0; addr < size; addr += 256) {
        // Read 256 bytes at current address
        digitalWrite(FLASH_CS, LOW);
        flashSPI.transfer(0x03);            // Read command
        flashSPI.transfer((addr >> 16) & 0xFF);
        flashSPI.transfer((addr >> 8) & 0xFF);
        flashSPI.transfer(addr & 0xFF);

        for (int i = 0; i < 256; i++) {
            buf[i] = flashSPI.transfer(0);
        }
        digitalWrite(FLASH_CS, HIGH);

        // Send raw bytes over serial
        Serial.write(buf, 256);

        // Progress indicator (sent to stderr if available)
        if (addr % 65536 == 0) {
            // Print progress to USB CDC debug
        }
    }
}

void loop() {}
```

**Analyzing the dump with binwalk:**

```bash
# Install binwalk
pip install binwalk

# Scan for embedded file systems, compressed data, crypto signatures
binwalk firmware.bin

# Extract all recognized file systems
binwalk -e firmware.bin

# Entropy analysis (find encrypted/compressed regions)
binwalk -E firmware.bin
```

---

### I2C EEPROM Reading

Many devices store configuration data, credentials, and calibration data in small I2C EEPROMs (AT24C02, AT24C32, AT24C256).

```cpp
// Dump an I2C EEPROM connected to the T-Embed
#include <Wire.h>

#define EEPROM_ADDR 0x50  // Most common address for AT24Cxx
#define I2C_SDA 8
#define I2C_SCL 9

void setup() {
    Serial.begin(115200);
    Wire.begin(I2C_SDA, I2C_SCL);

    Serial.println("I2C EEPROM Dumper");
    Serial.println("=================\n");

    // Try to detect EEPROM size by reading
    // AT24C02 = 256 bytes, AT24C32 = 4096 bytes, AT24C256 = 32768 bytes

    uint32_t size = detectEEPROMSize();
    Serial.printf("Detected EEPROM size: %d bytes\n\n", size);

    dumpEEPROM(size);
}

uint32_t detectEEPROMSize() {
    // Read byte at increasing addresses until we wrap around
    uint8_t firstByte = readByte(0);

    uint32_t sizes[] = {256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536};
    for (int i = 0; i < 9; i++) {
        // This is a heuristic -- not 100% reliable
        // For definitive sizing, check the chip marking
        Wire.beginTransmission(EEPROM_ADDR);
        Wire.write((uint8_t)(sizes[i] >> 8));
        Wire.write((uint8_t)(sizes[i] & 0xFF));
        if (Wire.endTransmission() != 0) {
            return sizes[i > 0 ? i - 1 : 0];
        }
    }
    return 32768;  // Default to max
}

uint8_t readByte(uint16_t addr) {
    Wire.beginTransmission(EEPROM_ADDR);
    Wire.write((uint8_t)(addr >> 8));
    Wire.write((uint8_t)(addr & 0xFF));
    Wire.endTransmission();
    Wire.requestFrom(EEPROM_ADDR, (uint8_t)1);
    return Wire.available() ? Wire.read() : 0xFF;
}

void dumpEEPROM(uint32_t size) {
    Serial.println("Addr  00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F  ASCII");
    Serial.println("----  -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --  ----------------");

    for (uint32_t addr = 0; addr < size; addr += 16) {
        // Read 16 bytes
        uint8_t buf[16];
        Wire.beginTransmission(EEPROM_ADDR);
        Wire.write((uint8_t)(addr >> 8));
        Wire.write((uint8_t)(addr & 0xFF));
        Wire.endTransmission();
        Wire.requestFrom(EEPROM_ADDR, (uint8_t)16);

        for (int i = 0; i < 16 && Wire.available(); i++) {
            buf[i] = Wire.read();
        }

        // Print hex
        Serial.printf("%04X  ", addr);
        for (int i = 0; i < 16; i++) {
            Serial.printf("%02X ", buf[i]);
        }

        // Print ASCII
        Serial.print(" ");
        for (int i = 0; i < 16; i++) {
            Serial.print(buf[i] >= 0x20 && buf[i] < 0x7F ? (char)buf[i] : '.');
        }
        Serial.println();
    }
}

void loop() {}
```

**What to look for in EEPROM dumps:**
- Readable ASCII strings (SSIDs, passwords, usernames)
- MAC addresses (6 consecutive bytes, often starting with a known OUI)
- IP addresses (4 bytes)
- Cryptographic keys (high-entropy regions)
- Configuration flags and serial numbers

---

### Logic Analyzer (Basic)

You can use the ESP32-S3 GPIO pins as a basic logic analyzer to capture and analyze digital signals from target hardware. While not as capable as dedicated hardware (Saleae, DSLogic), it works for slow protocols.

```cpp
// Basic logic analyzer: capture digital transitions on a GPIO pin
#define CAPTURE_PIN  13
#define MAX_SAMPLES  10000

volatile uint32_t timestamps[MAX_SAMPLES];
volatile uint8_t  levels[MAX_SAMPLES];
volatile uint32_t sampleCount = 0;

void IRAM_ATTR captureISR() {
    if (sampleCount < MAX_SAMPLES) {
        timestamps[sampleCount] = micros();
        levels[sampleCount] = digitalRead(CAPTURE_PIN);
        sampleCount++;
    }
}

void setup() {
    Serial.begin(115200);
    pinMode(CAPTURE_PIN, INPUT);

    Serial.println("Logic Analyzer Ready");
    Serial.println("Waiting for signal on GPIO 13...");

    sampleCount = 0;
    attachInterrupt(digitalPinToInterrupt(CAPTURE_PIN), captureISR, CHANGE);

    delay(5000);  // Capture for 5 seconds

    detachInterrupt(digitalPinToInterrupt(CAPTURE_PIN));

    Serial.printf("Captured %d transitions\n\n", sampleCount);
    Serial.println("Time(us)  Level  Delta(us)");
    Serial.println("--------  -----  ---------");

    for (uint32_t i = 0; i < sampleCount; i++) {
        uint32_t delta = i > 0 ? timestamps[i] - timestamps[i-1] : 0;
        Serial.printf("%8d  %s  %9d\n",
            timestamps[i],
            levels[i] ? "HIGH " : "LOW  ",
            delta);
    }
}

void loop() {}
```

{: .tip }
> For more sophisticated logic analysis, look into **Sigrok** and **PulseView**. Some community projects implement Sigrok-compatible firmware for ESP32 that turns it into a USB logic analyzer.

---

### Signal Injection

Using GPIO pins to inject signals into target hardware allows for active testing rather than passive observation.

**Use cases:**
- Injecting UART commands into a target device
- Toggling reset lines to cause reboots at specific moments
- Providing clock signals to desynchronize processors
- PWM injection for motor/servo control testing

{: .warning }
> **Signal injection can permanently damage target hardware.** Always verify voltage levels and current limits before connecting. Start with passive observation (reading) before attempting active injection (writing). Use current-limiting resistors (330 ohm to 1 kohm) on all output lines.

#### Voltage Glitching Concepts

Voltage glitching is an advanced attack where you briefly disrupt the power supply to a processor, causing it to skip instructions (potentially bypassing security checks).

```
Normal power:     ────────────────────────────
                  3.3V

Glitched power:   ──────┐    ┌───────────────
                        │    │
                        └────┘   <- Brief voltage drop
                  Causes instruction skip
```

This is an advanced technique requiring precise timing (often nanosecond-level) and is typically done with dedicated hardware like a ChipWhisperer. The T-Embed GPIO can provide crude glitching using a MOSFET to momentarily short the target's power rail, but results will be inconsistent.

{: .warning }
> Voltage glitching can permanently destroy the target device. Only attempt on hardware you own and are prepared to lose.

---

## Power Considerations for Expansions

### USB Power Budget

When powered via USB, the T-Embed typically receives **500 mA at 5V** (USB 2.0) or up to **900 mA** (USB 3.0). The onboard 3.3V regulator has its own limits.

| Consumer | Typical Current |
|:---------|:----------------|
| ESP32-S3 (active, WiFi+BLE) | 150-350 mA |
| TFT display (backlight on) | 30-50 mA |
| CC1101 (transmitting) | 30-35 mA |
| IR LED (transmitting) | 20-50 mA |
| NRF24L01+ | 12-15 mA |
| NRF24L01+ PA+LNA | 100-150 mA |
| RC522 RFID | 20-30 mA |
| PN532 NFC | 50-120 mA |
| GPS NEO-6M | 40-50 mA |
| SSD1306 OLED | 10-20 mA |
| SD card (writing) | 50-100 mA |

{: .warning }
> Adding a PA+LNA NRF24 module plus GPS plus the ESP32-S3 at full power can exceed USB 2.0 limits. Use a powered USB hub or external battery with a dedicated 3.3V regulator.

### Battery Power

For portable operation, use a **3.7V LiPo battery** with a TP4056 charge controller or a USB power bank. The T-Embed supports battery operation via its onboard charging circuit (check your specific board revision).

### Level Shifting

When interfacing with 5V devices (Arduino Uno, many sensors, some UART debug ports):

| Method | Speed | Bidirectional | Cost |
|:-------|:------|:--------------|:-----|
| Resistor divider (5V to 3.3V only) | Slow | No (input only) | Cheapest |
| BSS138 MOSFET level shifter | Medium | Yes | ~$1 per module |
| TXS0108E module | Fast | Yes | ~$2 per module |
| 74HC4050 buffer | Fast | No (one direction) | ~$0.50 |

**Simple resistor divider for 5V input to 3.3V GPIO:**

```
5V signal ──── 1kΩ ──┬── GPIO pin (3.3V tolerant input)
                      │
                    2kΩ
                      │
                    GND
```

---

## Enclosures and Physical Builds

### 3D Printed Cases

Search Thingiverse and Printables for "LILYGO T-Embed" cases. Key considerations:

- **Antenna access** -- slot or hole for IPEX pigtail to SMA connector
- **GPIO breakout** -- opening or slot for header wires
- **Encoder access** -- knob must rotate freely
- **Display window** -- clear or open panel for the TFT
- **USB port** -- accessible for power and programming
- **Battery compartment** -- space for 18650 or LiPo cell
- **Ventilation** -- the ESP32-S3 gets warm under heavy WiFi/BLE use

### Portable Pentest Tool Build

A complete portable rig:

1. **T-Embed CC1101 Plus** -- main unit
2. **18650 battery + TP4056 charger** -- portable power
3. **SMA antenna (433 MHz quarter-wave)** -- improved Sub-GHz range
4. **3D printed case** -- protection and mounting
5. **Optional modules** in internal compartments:
   - NRF24L01+ for 2.4 GHz attacks
   - RC522 for RFID reading
   - GPS for wardriving

{: .tip }
> Use a small slide switch between the battery and the T-Embed to add a physical power switch. This prevents accidental battery drain and provides a quick kill switch.

---

## Troubleshooting Hardware Connections

### No Response from Module

| Symptom | Likely Cause | Fix |
|:--------|:-------------|:----|
| Module not detected at all | Wiring error | Verify every connection with multimeter continuity |
| Module detected but no data | Wrong SPI mode or speed | Try all 4 SPI modes; reduce clock speed to 1 MHz |
| Module works intermittently | Loose connection | Solder joints instead of breadboard jumpers |
| Module works then stops | Power issue | Add decoupling capacitors (100nF + 10uF) at module VCC |

### SPI Bus Conflicts

**Symptoms:** CC1101 stops working when SD card module is added. Display garbles when accessing NRF24.

**Fix:**
1. Ensure each device has a unique CS pin
2. Pull all CS pins HIGH at startup before initializing any device
3. Use SPI transactions with `beginTransaction()` / `endTransaction()`
4. Consider using the second SPI peripheral (HSPI) for external devices

```cpp
// Proper multi-device SPI initialization
void setup() {
    // Set all CS pins HIGH first
    pinMode(CC1101_CS, OUTPUT);  digitalWrite(CC1101_CS, HIGH);
    pinMode(TFT_CS, OUTPUT);     digitalWrite(TFT_CS, HIGH);
    pinMode(SD_CS, OUTPUT);      digitalWrite(SD_CS, HIGH);
    pinMode(NRF_CS, OUTPUT);     digitalWrite(NRF_CS, HIGH);

    // Then initialize devices one at a time
    SPI.begin();
    // ...
}
```

### I2C Address Conflicts

**Symptoms:** One device works but another on the same I2C bus does not respond.

**Fix:**
1. Run the I2C scanner to verify addresses
2. Some devices have configurable addresses (A0, A1, A2 pins)
3. Use an I2C multiplexer (TCA9548A) if two devices share the same fixed address

### Power Issues

**Symptoms:** Device resets when module activates. Display dims or flickers. Erratic behavior.

**Fix:**
1. Add bulk capacitance: 100 uF electrolytic near the power input
2. Add decoupling capacitors: 100 nF ceramic at each module VCC/GND
3. Use a separate voltage regulator (AMS1117-3.3) for power-hungry modules
4. Check total current draw with a USB power meter

### Signal Integrity

**Symptoms:** Data corruption, CRC errors, garbled text, intermittent failures at higher speeds.

**Fix:**
1. Keep wires short (under 15 cm for SPI, under 30 cm for I2C)
2. Use twisted pairs or shielded cable for long runs
3. Add series termination resistors (33 ohm) on fast signal lines
4. Reduce clock speed until stable, then gradually increase
5. Use a ground wire for every 2-3 signal wires

{: .note }
> Breadboards are the number one cause of intermittent hardware issues. For anything beyond initial prototyping, solder your connections or use high-quality header connectors.

---

**Next chapter:** [10 -- Custom Firmware Development](10-custom-development.md) -- write your own firmware for the T-Embed CC1101 Plus from scratch.
