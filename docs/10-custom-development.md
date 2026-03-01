---
layout: default
title: "10 — Custom Firmware Development"
nav_order: 11
---

# 10 — Custom Firmware Development
{: .no_toc }

This chapter turns you from a firmware user into a firmware author. You will learn how to set up a development environment, control every hardware peripheral on the T-Embed CC1101 Plus, and build complete projects from scratch -- culminating in your own multi-tool framework.
{: .fs-5 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Development Environment Setup

### Arduino IDE

The Arduino IDE is the easiest way to get started. It hides build system complexity and has the largest library ecosystem.

#### Installation

1. Download Arduino IDE 2.x from [arduino.cc/en/software](https://www.arduino.cc/en/software)
2. Install and launch

#### Adding ESP32-S3 Board Support

1. Go to **File > Preferences**
2. In **Additional Board Manager URLs**, add:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Go to **Tools > Board > Boards Manager**
4. Search for **esp32** by Espressif Systems
5. Install the latest version (3.x recommended)

#### Board Configuration

Select **Tools > Board > esp32 > ESP32S3 Dev Module** and configure:

| Setting | Value | Why |
|:--------|:------|:----|
| USB CDC On Boot | **Enabled** | Enables Serial over USB |
| Upload Mode | **UART0 / Hardware CDC** | Upload via USB-C port |
| USB Mode | **Hardware CDC and JTAG** | USB serial + debug |
| Flash Size | **16MB (128Mb)** | T-Embed has 16MB flash |
| Partition Scheme | **16M Flash (3MB APP/9.9MB FATFS)** | Large app + file storage |
| PSRAM | **OPI PSRAM** | T-Embed has octal PSRAM |
| Upload Speed | **921600** | Fastest reliable speed |
| CPU Frequency | **240MHz (WiFi)** | Full speed |

{: .warning }
> If upload fails, hold the **BOOT** button on the T-Embed while clicking Upload. Release after "Connecting..." appears. Some board revisions require this for initial flash.

#### Installing Libraries

Go to **Tools > Manage Libraries** and install:

| Library | Author | Purpose |
|:--------|:-------|:--------|
| TFT_eSPI | Bodmer | ST7789 display driver |
| SmartRC-CC1101-Driver-Lib | LSatan | CC1101 radio control |
| IRremoteESP8266 | crankyoldgit | IR send/receive |
| NimBLE-Arduino | h2zero | Bluetooth LE |
| ArduinoJson | Benoit Blanchon | JSON parsing |
| TinyGPSPlus | Mikal Hart | GPS NMEA parsing |

---

### PlatformIO (Recommended)

PlatformIO provides a professional development experience with proper dependency management, multiple build environments, and integrated debugging.

#### Installation

1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install the **PlatformIO IDE** extension from the VS Code marketplace
3. Restart VS Code

Alternatively, install the CLI:

```bash
pip install platformio
```

#### platformio.ini Configuration

Create a new PlatformIO project and use this configuration:

```ini
[env:t-embed-cc1101]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino

; CPU and memory
board_build.mcu = esp32s3
board_build.f_cpu = 240000000L
board_build.flash_mode = qio
board_build.flash_size = 16MB
board_build.psram_type = opi
board_upload.flash_size = 16MB

; Upload and monitor
monitor_speed = 115200
upload_speed = 921600

; USB CDC serial
build_flags =
    -DARDUINO_USB_MODE=1
    -DARDUINO_USB_CDC_ON_BOOT=1
    -DBOARD_HAS_PSRAM
    -mfix-esp32-psram-cache-issue

; Libraries
lib_deps =
    SPI
    Wire
    bodmer/TFT_eSPI@^2.5.0
    LSatan/SmartRC-CC1101-Driver-Lib@^2.5.7
    crankyoldgit/IRremoteESP8266@^2.8.6
    h2zero/NimBLE-Arduino@^1.4.0
    bblanchon/ArduinoJson@^7.0.0
    mikalhart/TinyGPSPlus@^1.0.3
```

#### Building and Uploading

```bash
# Build the project
pio run

# Upload to T-Embed
pio run --target upload

# Open serial monitor
pio device monitor

# Build, upload, and monitor in one command
pio run --target upload && pio device monitor
```

#### Debugging

PlatformIO supports JTAG debugging on ESP32-S3 via the built-in USB JTAG interface:

```ini
; Add to platformio.ini
debug_tool = esp-builtin
debug_init_break = tbreak setup
```

Then use **Run > Start Debugging (F5)** in VS Code to set breakpoints, inspect variables, and step through code.

---

### ESP-IDF (Advanced)

The Espressif IoT Development Framework is the official SDK. Use it when you need:
- Maximum performance and control
- Direct FreeRTOS access
- Custom partition tables
- Low-level peripheral drivers
- Features not exposed by the Arduino framework

#### When to Use ESP-IDF vs Arduino

| Criteria | Arduino | ESP-IDF |
|:---------|:--------|:--------|
| Learning curve | Gentle | Steep |
| Library ecosystem | Massive | Moderate |
| Performance | Good | Maximum |
| Binary size | Larger | Smaller |
| FreeRTOS control | Limited | Full |
| OTA updates | Basic | Advanced |
| Best for | Prototyping, most projects | Production firmware, advanced features |

#### Setup

```bash
# Clone ESP-IDF
mkdir -p ~/esp
cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32s3

# Activate environment (run in every new terminal)
. ~/esp/esp-idf/export.sh

# Create a new project
idf.py create-project my_project
cd my_project

# Set target
idf.py set-target esp32s3

# Configure (menuconfig)
idf.py menuconfig

# Build
idf.py build

# Flash
idf.py -p /dev/ttyACM0 flash

# Monitor
idf.py -p /dev/ttyACM0 monitor
```

#### FreeRTOS Concepts

ESP-IDF runs FreeRTOS under the hood (Arduino does too, but hides it). Key concepts:

- **Tasks** -- independent threads of execution, each with a priority and stack
- **Queues** -- thread-safe data passing between tasks
- **Semaphores/Mutexes** -- synchronization primitives
- **Dual core** -- ESP32-S3 has two cores; pin tasks to specific cores

```cpp
// FreeRTOS task example (works in Arduino framework too)
void wifiScanTask(void *pvParameters) {
    while (true) {
        // Scan WiFi networks
        int n = WiFi.scanNetworks();
        Serial.printf("[WiFi] Found %d networks\n", n);
        vTaskDelay(pdMS_TO_TICKS(10000));  // Scan every 10s
    }
}

void bleScanTask(void *pvParameters) {
    while (true) {
        // Scan BLE devices
        NimBLEScan *scan = NimBLEDevice::getScan();
        scan->start(5);  // 5 second scan
        Serial.printf("[BLE] Found %d devices\n", scan->getResults().getCount());
        vTaskDelay(pdMS_TO_TICKS(15000));
    }
}

void setup() {
    Serial.begin(115200);

    // Create tasks on different cores
    xTaskCreatePinnedToCore(wifiScanTask, "WiFiScan", 8192, NULL, 1, NULL, 0);
    xTaskCreatePinnedToCore(bleScanTask,  "BLEScan",  8192, NULL, 1, NULL, 1);
}

void loop() {
    // Main loop is free for other work (or can be empty)
    vTaskDelay(pdMS_TO_TICKS(1000));
}
```

---

## Essential Libraries

### TFT_eSPI (Display)

The T-Embed CC1101 Plus uses a **ST7789 320x170** TFT display connected via SPI. The TFT_eSPI library by Bodmer is the standard driver.

#### Configuration (User_Setup.h)

You must configure TFT_eSPI for the T-Embed's specific wiring. Create or edit the `User_Setup.h` file in the TFT_eSPI library directory:

```cpp
// User_Setup.h for LILYGO T-Embed CC1101 Plus

#define USER_SETUP_INFO "T-Embed-CC1101"

// Driver
#define ST7789_DRIVER

// Resolution
#define TFT_WIDTH  170
#define TFT_HEIGHT 320

// Pin assignments
#define TFT_MOSI 11
#define TFT_SCLK 12
#define TFT_CS   42
#define TFT_DC   16
#define TFT_RST  15
#define TFT_BL   40

// SPI frequency
#define SPI_FREQUENCY       40000000  // 40 MHz
#define SPI_READ_FREQUENCY  16000000
#define SPI_TOUCH_FREQUENCY  2500000

// Color order
#define TFT_RGB_ORDER TFT_RGB

// Offsets (may need adjustment for your specific panel)
#define TFT_BACKLIGHT_ON HIGH
```

{: .tip }
> With PlatformIO, you can define these in `platformio.ini` build flags instead of editing the library file directly. Add `-DUSER_SETUP_LOADED` and define each pin with `-DTFT_MOSI=11` etc.

#### Drawing Primitives

```cpp
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI();

void setup() {
    tft.init();
    tft.setRotation(3);  // Landscape, USB on right
    tft.fillScreen(TFT_BLACK);

    // Turn on backlight
    pinMode(TFT_BL, OUTPUT);
    digitalWrite(TFT_BL, HIGH);

    // Pixels
    tft.drawPixel(10, 10, TFT_WHITE);

    // Lines
    tft.drawLine(0, 0, 319, 169, TFT_GREEN);
    tft.drawFastHLine(0, 85, 320, TFT_YELLOW);  // Horizontal (fast)
    tft.drawFastVLine(160, 0, 170, TFT_YELLOW);  // Vertical (fast)

    // Rectangles
    tft.drawRect(10, 10, 100, 50, TFT_CYAN);           // Outline
    tft.fillRect(120, 10, 100, 50, TFT_RED);            // Filled
    tft.fillRoundRect(10, 70, 100, 50, 8, TFT_BLUE);   // Rounded

    // Circles
    tft.drawCircle(200, 100, 30, TFT_MAGENTA);          // Outline
    tft.fillCircle(270, 100, 30, TFT_ORANGE);           // Filled

    // Triangles
    tft.fillTriangle(10, 160, 50, 130, 90, 160, TFT_GREEN);
}
```

#### Text Rendering

```cpp
void drawTextExamples() {
    tft.fillScreen(TFT_BLACK);

    // Built-in fonts (1-4, plus 6, 7, 8)
    tft.setTextColor(TFT_WHITE, TFT_BLACK);  // Text color, background color

    tft.setTextFont(1);
    tft.setTextSize(1);
    tft.drawString("Font 1 Size 1", 10, 10);

    tft.setTextFont(2);
    tft.drawString("Font 2", 10, 30);

    tft.setTextFont(4);
    tft.drawString("Font 4", 10, 50);

    // Formatted numbers
    tft.setTextFont(7);  // 7-segment font
    tft.drawNumber(433, 10, 80);

    tft.setTextFont(2);
    tft.drawFloat(433.92, 2, 10, 130);  // 2 decimal places

    // Text alignment
    tft.setTextDatum(MC_DATUM);  // Middle-center
    tft.drawString("CENTERED", 160, 85);

    // Datum options: TL_DATUM, TC_DATUM, TR_DATUM,
    //               ML_DATUM, MC_DATUM, MR_DATUM,
    //               BL_DATUM, BC_DATUM, BR_DATUM
}
```

#### Double Buffering with Sprites

Sprites eliminate flickering by drawing to an off-screen buffer, then pushing the entire buffer to the display in one operation:

```cpp
TFT_eSPI tft = TFT_eSPI();
TFT_eSprite sprite = TFT_eSprite(&tft);

void setup() {
    tft.init();
    tft.setRotation(3);

    // Create sprite in PSRAM (plenty of memory)
    sprite.createSprite(320, 170);
    sprite.setTextDatum(MC_DATUM);
}

void loop() {
    static float rssi = -80.0;
    rssi += random(-5, 6);  // Simulated RSSI

    // Draw everything to sprite (no flicker)
    sprite.fillScreen(TFT_BLACK);

    // Header bar
    sprite.fillRect(0, 0, 320, 24, TFT_NAVY);
    sprite.setTextColor(TFT_WHITE);
    sprite.setTextFont(2);
    sprite.drawString("T-Embed Signal Monitor", 160, 12);

    // RSSI bar graph
    int barWidth = map(constrain(rssi, -100, -20), -100, -20, 0, 280);
    uint16_t barColor = rssi > -50 ? TFT_GREEN : rssi > -70 ? TFT_YELLOW : TFT_RED;

    sprite.drawRect(19, 49, 282, 22, TFT_DARKGREY);
    sprite.fillRect(20, 50, barWidth, 20, barColor);

    // RSSI value
    sprite.setTextFont(4);
    char buf[16];
    snprintf(buf, sizeof(buf), "%.1f dBm", rssi);
    sprite.drawString(buf, 160, 100);

    // Frequency
    sprite.setTextFont(2);
    sprite.setTextColor(TFT_CYAN);
    sprite.drawString("433.920 MHz | ASK/OOK", 160, 140);

    // Push sprite to display in one operation
    sprite.pushSprite(0, 0);

    delay(100);
}
```

#### Full Example: Signal Dashboard

```cpp
#include <TFT_eSPI.h>
#include <ELECHOUSE_CC1101_SRC_DRV.h>

TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);

#define HISTORY_LEN 280
int8_t rssiHistory[HISTORY_LEN];
int historyIdx = 0;

float currentFreq = 433.92;
int packetCount = 0;

void setup() {
    Serial.begin(115200);

    tft.init();
    tft.setRotation(3);
    tft.fillScreen(TFT_BLACK);
    pinMode(TFT_BL, OUTPUT);
    digitalWrite(TFT_BL, HIGH);

    canvas.createSprite(320, 170);

    // Initialize CC1101
    if (ELECHOUSE_cc1101.getCC1101()) {
        ELECHOUSE_cc1101.Init();
        ELECHOUSE_cc1101.setMHZ(currentFreq);
        ELECHOUSE_cc1101.SetRx();
        Serial.println("CC1101 initialized");
    }

    memset(rssiHistory, -100, HISTORY_LEN);
}

void loop() {
    // Read current RSSI
    int8_t rssi = ELECHOUSE_cc1101.getRssi();

    // Store in circular buffer
    rssiHistory[historyIdx] = rssi;
    historyIdx = (historyIdx + 1) % HISTORY_LEN;

    // Check for packet
    if (ELECHOUSE_cc1101.CheckRxFifo(100)) {
        packetCount++;
    }

    // Draw dashboard
    canvas.fillScreen(TFT_BLACK);

    // Title bar
    canvas.fillRect(0, 0, 320, 22, 0x1082);
    canvas.setTextColor(TFT_WHITE);
    canvas.setTextFont(2);
    canvas.setTextDatum(ML_DATUM);
    canvas.drawString("SUB-GHZ MONITOR", 8, 11);
    canvas.setTextDatum(MR_DATUM);
    char freqStr[16];
    snprintf(freqStr, sizeof(freqStr), "%.3f MHz", currentFreq);
    canvas.drawString(freqStr, 312, 11);

    // RSSI waterfall graph
    canvas.drawRect(19, 29, 282, 82, TFT_DARKGREY);
    for (int i = 0; i < HISTORY_LEN; i++) {
        int idx = (historyIdx + i) % HISTORY_LEN;
        int h = map(constrain(rssiHistory[idx], -100, -20), -100, -20, 0, 80);
        if (h > 0) {
            uint16_t color;
            if (h > 60) color = TFT_RED;
            else if (h > 40) color = TFT_YELLOW;
            else if (h > 20) color = TFT_GREEN;
            else color = 0x0320;  // Dark green

            canvas.drawFastVLine(20 + i, 110 - h, h, color);
        }
    }

    // Stats
    canvas.setTextFont(2);
    canvas.setTextDatum(TL_DATUM);
    canvas.setTextColor(TFT_CYAN);
    canvas.drawString("RSSI:", 10, 118);
    canvas.drawString("Packets:", 10, 138);
    canvas.drawString("Mode:", 10, 158);

    canvas.setTextColor(TFT_WHITE);
    char rssiStr[16];
    snprintf(rssiStr, sizeof(rssiStr), "%d dBm", rssi);
    canvas.drawString(rssiStr, 80, 118);
    canvas.drawNumber(packetCount, 80, 138);
    canvas.drawString("ASK/OOK RX", 80, 158);

    // Signal strength indicator
    uint16_t sigColor = rssi > -50 ? TFT_GREEN : rssi > -70 ? TFT_YELLOW : TFT_RED;
    canvas.fillCircle(300, 140, 8, sigColor);

    canvas.pushSprite(0, 0);

    delay(50);
}
```

---

### ELECHOUSE_CC1101_SRC_DRV (CC1101 Control)

This library provides complete control over the CC1101 Sub-GHz transceiver.

#### Initialization

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>

void setup() {
    Serial.begin(115200);

    // Check if CC1101 is connected
    if (ELECHOUSE_cc1101.getCC1101()) {
        Serial.println("CC1101 connected");
    } else {
        Serial.println("CC1101 not found!");
        while (1);
    }

    // Initialize with default settings
    ELECHOUSE_cc1101.Init();

    // Set frequency (MHz)
    ELECHOUSE_cc1101.setMHZ(433.92);

    // Set modulation
    // 0 = 2-FSK
    // 1 = GFSK
    // 2 = ASK/OOK (most common for remotes/sensors)
    // 3 = 4-FSK
    // 4 = MSK
    ELECHOUSE_cc1101.setModulation(2);

    // Set data rate (kBaud)
    ELECHOUSE_cc1101.setDRate(1.2);

    // Set receiver bandwidth (kHz)
    ELECHOUSE_cc1101.setRxBW(58.03);

    // Set output power (dBm)
    // For 433 MHz: -30, -20, -15, -10, 0, 5, 7, 10
    ELECHOUSE_cc1101.setPA(10);

    // Set sync word (2 bytes)
    ELECHOUSE_cc1101.setSyncWord(0xD3, 0x91);

    // Set packet length (fixed)
    ELECHOUSE_cc1101.setPacketLength(32);

    Serial.println("CC1101 configured");
}
```

#### Receiving Data

```cpp
void startReceiving() {
    ELECHOUSE_cc1101.SetRx();  // Put CC1101 in receive mode
    Serial.println("Listening...");
}

void loop() {
    if (ELECHOUSE_cc1101.CheckRxFifo(100)) {
        uint8_t buffer[64];
        int len = ELECHOUSE_cc1101.ReceiveData(buffer);

        Serial.printf("Received %d bytes: ", len);
        for (int i = 0; i < len; i++) {
            Serial.printf("%02X ", buffer[i]);
        }
        Serial.println();

        // Print RSSI and LQI
        int rssi = ELECHOUSE_cc1101.getRssi();
        int lqi = ELECHOUSE_cc1101.getLqi();
        Serial.printf("RSSI: %d dBm | LQI: %d\n", rssi, lqi);
    }
}
```

#### Transmitting Data

```cpp
void transmitData() {
    uint8_t data[] = {0x48, 0x65, 0x6C, 0x6C, 0x6F};  // "Hello"

    ELECHOUSE_cc1101.SetTx();  // Put CC1101 in transmit mode
    ELECHOUSE_cc1101.SendData(data, sizeof(data));

    Serial.println("Data transmitted");

    ELECHOUSE_cc1101.SetRx();  // Return to receive mode
}
```

{: .warning }
> Transmitting on frequencies you are not licensed for is illegal. Ensure you operate within ISM bands (433.05-434.79 MHz, 868.0-868.6 MHz, 902-928 MHz depending on your region) and comply with local power limits.

#### Complete Example: Frequency Scanner

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);

#define START_FREQ 430.0
#define END_FREQ   440.0
#define FREQ_STEP  0.05
#define NUM_STEPS  ((int)((END_FREQ - START_FREQ) / FREQ_STEP))

int8_t scanData[300];

void setup() {
    Serial.begin(115200);
    tft.init();
    tft.setRotation(3);
    canvas.createSprite(320, 170);
    pinMode(TFT_BL, OUTPUT);
    digitalWrite(TFT_BL, HIGH);

    ELECHOUSE_cc1101.Init();
    ELECHOUSE_cc1101.setModulation(2);  // ASK/OOK
    ELECHOUSE_cc1101.setRxBW(58.03);

    Serial.println("Frequency scanner ready");
}

void loop() {
    // Sweep frequencies
    for (int i = 0; i < NUM_STEPS && i < 300; i++) {
        float freq = START_FREQ + (i * FREQ_STEP);
        ELECHOUSE_cc1101.setMHZ(freq);
        ELECHOUSE_cc1101.SetRx();

        delayMicroseconds(500);  // Let RSSI settle

        scanData[i] = ELECHOUSE_cc1101.getRssi();
    }

    // Draw spectrum
    canvas.fillScreen(TFT_BLACK);

    // Title
    canvas.setTextColor(TFT_WHITE);
    canvas.setTextFont(2);
    canvas.setTextDatum(TC_DATUM);
    canvas.drawString("SPECTRUM ANALYZER", 160, 2);

    // Axis labels
    canvas.setTextFont(1);
    canvas.setTextDatum(BL_DATUM);
    char startStr[10], endStr[10];
    snprintf(startStr, sizeof(startStr), "%.1f", START_FREQ);
    snprintf(endStr, sizeof(endStr), "%.1f", END_FREQ);
    canvas.drawString(startStr, 10, 165);
    canvas.setTextDatum(BR_DATUM);
    canvas.drawString(endStr, 310, 165);
    canvas.setTextDatum(BC_DATUM);
    canvas.drawString("MHz", 160, 165);

    // Draw spectrum bars
    int graphWidth = 300;
    int graphHeight = 130;
    int graphX = 10;
    int graphY = 20;

    canvas.drawRect(graphX - 1, graphY - 1, graphWidth + 2, graphHeight + 2, TFT_DARKGREY);

    int peakIdx = 0;
    int8_t peakRSSI = -128;

    for (int i = 0; i < NUM_STEPS && i < graphWidth; i++) {
        int barH = map(constrain(scanData[i], -110, -20), -110, -20, 0, graphHeight);

        uint16_t color;
        if (scanData[i] > -40) color = TFT_RED;
        else if (scanData[i] > -60) color = TFT_YELLOW;
        else if (scanData[i] > -80) color = TFT_GREEN;
        else color = 0x03E0;

        if (barH > 0) {
            canvas.drawFastVLine(graphX + i, graphY + graphHeight - barH, barH, color);
        }

        if (scanData[i] > peakRSSI) {
            peakRSSI = scanData[i];
            peakIdx = i;
        }
    }

    // Mark peak frequency
    float peakFreq = START_FREQ + (peakIdx * FREQ_STEP);
    canvas.drawFastVLine(graphX + peakIdx, graphY, graphHeight, TFT_WHITE);
    char peakStr[32];
    snprintf(peakStr, sizeof(peakStr), "Peak: %.2f MHz (%d dBm)", peakFreq, peakRSSI);
    canvas.setTextFont(1);
    canvas.setTextDatum(TC_DATUM);
    canvas.setTextColor(TFT_CYAN);
    canvas.drawString(peakStr, 160, 155);

    canvas.pushSprite(0, 0);
}
```

#### Complete Example: Signal Recorder and Replayer

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>

#define MAX_SAMPLES 2048

volatile uint32_t signalTimings[MAX_SAMPLES];
volatile int signalCount = 0;
volatile uint32_t lastMicros = 0;

#define GDO0_PIN 38  // CC1101 GDO0

void IRAM_ATTR signalISR() {
    if (signalCount < MAX_SAMPLES) {
        uint32_t now = micros();
        signalTimings[signalCount] = now - lastMicros;
        lastMicros = now;
        signalCount++;
    }
}

void recordSignal(float frequency) {
    ELECHOUSE_cc1101.setMHZ(frequency);
    ELECHOUSE_cc1101.setModulation(2);  // ASK/OOK
    ELECHOUSE_cc1101.SetRx();

    signalCount = 0;
    lastMicros = micros();

    attachInterrupt(digitalPinToInterrupt(GDO0_PIN), signalISR, CHANGE);

    Serial.println("Recording... press button or send 's' to stop");

    while (signalCount < MAX_SAMPLES) {
        if (Serial.available() && Serial.read() == 's') break;
        delay(10);
    }

    detachInterrupt(digitalPinToInterrupt(GDO0_PIN));
    ELECHOUSE_cc1101.setSidle();

    Serial.printf("Recorded %d transitions\n", signalCount);

    // Print RAW timings
    Serial.println("RAW_Data:");
    for (int i = 0; i < signalCount; i++) {
        Serial.printf("%d ", signalTimings[i]);
    }
    Serial.println();
}

void replaySignal(float frequency) {
    if (signalCount == 0) {
        Serial.println("No signal recorded!");
        return;
    }

    ELECHOUSE_cc1101.setMHZ(frequency);
    ELECHOUSE_cc1101.setModulation(2);
    ELECHOUSE_cc1101.SetTx();

    Serial.printf("Replaying %d transitions...\n", signalCount);

    // Replay 3 times for reliability
    for (int repeat = 0; repeat < 3; repeat++) {
        for (int i = 0; i < signalCount; i++) {
            digitalWrite(GDO0_PIN, i % 2 == 0 ? HIGH : LOW);
            delayMicroseconds(signalTimings[i]);
        }
        digitalWrite(GDO0_PIN, LOW);
        delay(50);  // Gap between repeats
    }

    ELECHOUSE_cc1101.setSidle();
    Serial.println("Replay complete");
}

void setup() {
    Serial.begin(115200);
    pinMode(GDO0_PIN, INPUT);

    ELECHOUSE_cc1101.Init();
    Serial.println("Signal Recorder/Replayer");
    Serial.println("Commands: r=record, p=play, d=dump");
}

void loop() {
    if (Serial.available()) {
        char cmd = Serial.read();
        switch (cmd) {
            case 'r': recordSignal(433.92); break;
            case 'p': replaySignal(433.92); break;
            case 'd':
                for (int i = 0; i < signalCount; i++) {
                    Serial.printf("%d ", signalTimings[i]);
                }
                Serial.println();
                break;
        }
    }
}
```

#### Complete Example: RSSI Proximity Detector

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>

float targetFreq = 433.92;
int threshold = -60;  // dBm

void setup() {
    Serial.begin(115200);
    ELECHOUSE_cc1101.Init();
    ELECHOUSE_cc1101.setMHZ(targetFreq);
    ELECHOUSE_cc1101.setModulation(2);
    ELECHOUSE_cc1101.SetRx();

    Serial.printf("Proximity detector on %.2f MHz\n", targetFreq);
    Serial.printf("Threshold: %d dBm\n", threshold);
}

void loop() {
    int rssi = ELECHOUSE_cc1101.getRssi();

    // Simple moving average
    static int readings[10];
    static int idx = 0;
    readings[idx] = rssi;
    idx = (idx + 1) % 10;

    int avgRSSI = 0;
    for (int i = 0; i < 10; i++) avgRSSI += readings[i];
    avgRSSI /= 10;

    if (avgRSSI > threshold) {
        Serial.printf("*** SIGNAL DETECTED *** RSSI: %d dBm (avg: %d)\n", rssi, avgRSSI);
    }

    delay(50);
}
```

---

### IRremoteESP8266 (Infrared)

#### Receiving IR Signals

```cpp
#include <IRrecv.h>
#include <IRutils.h>

#define IR_RECV_PIN 4

IRrecv irrecv(IR_RECV_PIN);
decode_results results;

void setup() {
    Serial.begin(115200);
    irrecv.enableIRIn();
    Serial.println("IR Receiver ready. Point a remote and press a button.");
}

void loop() {
    if (irrecv.decode(&results)) {
        // Print protocol and value
        Serial.printf("Protocol: %s\n", typeToString(results.decode_type).c_str());
        Serial.printf("Value: 0x%llX\n", results.value);
        Serial.printf("Bits: %d\n", results.bits);

        // Print raw timing data
        Serial.println("Raw timings (microseconds):");
        for (uint16_t i = 1; i < results.rawlen; i++) {
            if (i % 2) {
                Serial.printf("+%d ", results.rawbuf[i] * kRawTick);
            } else {
                Serial.printf("-%d ", results.rawbuf[i] * kRawTick);
            }
            if (i % 16 == 0) Serial.println();
        }
        Serial.println("\n");

        irrecv.resume();
    }
}
```

#### Transmitting IR Signals

```cpp
#include <IRsend.h>

#define IR_SEND_PIN 44

IRsend irsend(IR_SEND_PIN);

void setup() {
    Serial.begin(115200);
    irsend.begin();
    Serial.println("IR Transmitter ready");
}

void sendNECCommand(uint32_t code) {
    irsend.sendNEC(code, 32);
    Serial.printf("Sent NEC: 0x%08X\n", code);
}

void sendSamsungTV(uint32_t code) {
    irsend.sendSAMSUNG(code, 32);
    Serial.printf("Sent Samsung: 0x%08X\n", code);
}

void sendRawSignal(uint16_t *rawData, uint16_t len, uint16_t freq) {
    irsend.sendRaw(rawData, len, freq);
    Serial.println("Sent raw IR signal");
}
```

#### Complete Example: IR Remote Clone

```cpp
#include <IRrecv.h>
#include <IRsend.h>
#include <IRutils.h>

#define IR_RECV_PIN 4
#define IR_SEND_PIN 44
#define MAX_SIGNALS 20

IRrecv irrecv(IR_RECV_PIN);
IRsend irsend(IR_SEND_PIN);
decode_results results;

// Storage for captured signals
struct IRSignal {
    decode_type_t protocol;
    uint64_t value;
    uint16_t bits;
    uint16_t rawData[256];
    uint16_t rawLen;
    bool used;
    char name[16];
};

IRSignal savedSignals[MAX_SIGNALS];
int signalCount = 0;

void setup() {
    Serial.begin(115200);
    irrecv.enableIRIn();
    irsend.begin();

    memset(savedSignals, 0, sizeof(savedSignals));

    Serial.println("=== IR Remote Clone ===");
    Serial.println("Commands:");
    Serial.println("  c - Capture new signal");
    Serial.println("  p <n> - Play signal n");
    Serial.println("  l - List saved signals");
}

void captureSignal() {
    if (signalCount >= MAX_SIGNALS) {
        Serial.println("Storage full!");
        return;
    }

    Serial.println("Point remote and press button...");

    unsigned long timeout = millis() + 10000;
    while (millis() < timeout) {
        if (irrecv.decode(&results)) {
            IRSignal &sig = savedSignals[signalCount];
            sig.protocol = results.decode_type;
            sig.value = results.value;
            sig.bits = results.bits;
            sig.used = true;

            // Save raw data as backup
            sig.rawLen = min((int)results.rawlen, 256);
            for (uint16_t i = 0; i < sig.rawLen; i++) {
                sig.rawData[i] = results.rawbuf[i] * kRawTick;
            }

            snprintf(sig.name, sizeof(sig.name), "Signal_%d", signalCount);

            Serial.printf("Captured: %s 0x%llX (%d bits) -> Slot %d\n",
                typeToString(sig.protocol).c_str(), sig.value,
                sig.bits, signalCount);

            signalCount++;
            irrecv.resume();
            return;
        }
    }

    Serial.println("Timeout - no signal captured");
}

void playSignal(int idx) {
    if (idx < 0 || idx >= MAX_SIGNALS || !savedSignals[idx].used) {
        Serial.println("Invalid signal index");
        return;
    }

    IRSignal &sig = savedSignals[idx];

    // Try protocol-specific send first
    if (sig.protocol != decode_type_t::UNKNOWN) {
        irsend.send(sig.protocol, sig.value, sig.bits);
        Serial.printf("Sent %s: 0x%llX\n",
            typeToString(sig.protocol).c_str(), sig.value);
    } else {
        // Fall back to raw
        irsend.sendRaw(sig.rawData, sig.rawLen, 38);
        Serial.println("Sent raw signal");
    }
}

void listSignals() {
    Serial.println("\nSaved Signals:");
    Serial.println("Idx  Name          Protocol    Value");
    Serial.println("---  ----          --------    -----");
    for (int i = 0; i < MAX_SIGNALS; i++) {
        if (savedSignals[i].used) {
            Serial.printf("%3d  %-14s %-11s 0x%llX\n",
                i, savedSignals[i].name,
                typeToString(savedSignals[i].protocol).c_str(),
                savedSignals[i].value);
        }
    }
    Serial.println();
}

void loop() {
    if (Serial.available()) {
        String cmd = Serial.readStringUntil('\n');
        cmd.trim();

        if (cmd == "c") {
            captureSignal();
        } else if (cmd.startsWith("p ")) {
            int idx = cmd.substring(2).toInt();
            playSignal(idx);
        } else if (cmd == "l") {
            listSignals();
        }
    }
}
```

---

### NimBLE-Arduino (Bluetooth LE)

NimBLE is a lightweight BLE stack that uses significantly less memory than the default ESP32 BLE library.

#### BLE Scanner with Device Info

```cpp
#include <NimBLEDevice.h>

NimBLEScan *pScan;

class ScanCallbacks : public NimBLEScanCallbacks {
    void onResult(const NimBLEAdvertisedDevice *device) override {
        Serial.printf("BLE Device: %s", device->getAddress().toString().c_str());

        if (device->haveName()) {
            Serial.printf(" | Name: %s", device->getName().c_str());
        }

        Serial.printf(" | RSSI: %d dBm", device->getRSSI());

        if (device->haveManufacturerData()) {
            std::string mfr = device->getManufacturerData();
            if (mfr.length() >= 2) {
                uint16_t companyId = (uint8_t)mfr[0] | ((uint8_t)mfr[1] << 8);
                Serial.printf(" | Mfr: 0x%04X", companyId);

                // Identify common manufacturers
                switch (companyId) {
                    case 0x004C: Serial.print(" (Apple)"); break;
                    case 0x0006: Serial.print(" (Microsoft)"); break;
                    case 0x0075: Serial.print(" (Samsung)"); break;
                    case 0x00E0: Serial.print(" (Google)"); break;
                    case 0x0059: Serial.print(" (Nordic Semi)"); break;
                }
            }
        }

        if (device->haveServiceUUID()) {
            Serial.printf(" | SVC: %s", device->getServiceUUID().toString().c_str());
        }

        Serial.println();
    }

    void onScanEnd(const NimBLEScanResults &results, int reason) override {
        Serial.printf("\nScan complete: %d devices found (reason: %d)\n",
            results.getCount(), reason);
    }
};

void setup() {
    Serial.begin(115200);
    NimBLEDevice::init("T-Embed-Scanner");

    pScan = NimBLEDevice::getScan();
    pScan->setScanCallbacks(new ScanCallbacks());
    pScan->setActiveScan(true);
    pScan->setInterval(100);
    pScan->setWindow(99);

    Serial.println("=== BLE Scanner ===\n");
}

void loop() {
    Serial.println("Starting BLE scan (10 seconds)...\n");
    pScan->clearResults();
    pScan->start(10, false);

    delay(15000);
}
```

#### BLE Client: Connect and Read Characteristics

```cpp
#include <NimBLEDevice.h>

// Target device address (replace with your target)
NimBLEAddress targetAddr("AA:BB:CC:DD:EE:FF");

void connectAndEnumerate() {
    NimBLEClient *pClient = NimBLEDevice::createClient();

    Serial.printf("Connecting to %s...\n", targetAddr.toString().c_str());

    if (!pClient->connect(targetAddr)) {
        Serial.println("Connection failed");
        NimBLEDevice::deleteClient(pClient);
        return;
    }

    Serial.println("Connected! Enumerating services...\n");

    // Discover all services
    std::vector<NimBLERemoteService *> *services = pClient->getServices(true);

    for (auto &svc : *services) {
        Serial.printf("Service: %s\n", svc->getUUID().toString().c_str());

        // Get all characteristics in this service
        std::vector<NimBLERemoteCharacteristic *> *chars = svc->getCharacteristics(true);

        for (auto &chr : *chars) {
            Serial.printf("  Char: %s", chr->getUUID().toString().c_str());
            Serial.printf(" [%s%s%s%s]",
                chr->canRead()      ? "R" : "",
                chr->canWrite()     ? "W" : "",
                chr->canNotify()    ? "N" : "",
                chr->canIndicate()  ? "I" : "");

            // Try to read the value
            if (chr->canRead()) {
                std::string val = chr->readValue();
                Serial.printf(" = ");
                // Print as hex
                for (char c : val) {
                    Serial.printf("%02X ", (uint8_t)c);
                }
                // Print as ASCII if printable
                bool printable = true;
                for (char c : val) {
                    if (c < 0x20 || c > 0x7E) { printable = false; break; }
                }
                if (printable && val.length() > 0) {
                    Serial.printf("(\"%s\")", val.c_str());
                }
            }
            Serial.println();
        }
        Serial.println();
    }

    pClient->disconnect();
    NimBLEDevice::deleteClient(pClient);
    Serial.println("Disconnected");
}

void setup() {
    Serial.begin(115200);
    NimBLEDevice::init("T-Embed");
    connectAndEnumerate();
}

void loop() {
    delay(60000);
}
```

---

### USB HID (BadUSB)

The ESP32-S3 has a native USB controller that can emulate a keyboard and mouse. This is the basis of "BadUSB" or "Rubber Ducky" style attacks.

{: .warning }
> BadUSB payloads execute with the privileges of the logged-in user on the target computer. Only use on systems you own or have explicit authorization to test. Unauthorized use is a criminal offense.

```cpp
#include <USB.h>
#include <USBHIDKeyboard.h>
#include <USBHIDMouse.h>

USBHIDKeyboard Keyboard;
USBHIDMouse Mouse;

void setup() {
    Serial.begin(115200);
    USB.begin();
    Keyboard.begin();
    Mouse.begin();

    delay(3000);  // Wait for USB enumeration

    Serial.println("BadUSB ready. Send 'r' to run payload.");
}

// Type a string with realistic delays
void typeString(const char *str) {
    while (*str) {
        Keyboard.write(*str++);
        delay(random(30, 80));  // Human-like typing speed
    }
}

// Open Run dialog (Windows)
void openRunDialog() {
    Keyboard.press(KEY_LEFT_GUI);
    Keyboard.press('r');
    delay(100);
    Keyboard.releaseAll();
    delay(500);
}

// Open Terminal (macOS)
void openTerminalMac() {
    Keyboard.press(KEY_LEFT_GUI);
    Keyboard.press(' ');
    delay(100);
    Keyboard.releaseAll();
    delay(500);

    typeString("Terminal");
    delay(300);

    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(1000);
}

// Example payload: Open notepad and type a message (Windows)
void payloadWindowsMessage() {
    openRunDialog();
    typeString("notepad");
    Keyboard.press(KEY_RETURN);
    Keyboard.releaseAll();
    delay(1000);

    typeString("This computer is vulnerable to BadUSB attacks.\n");
    typeString("A malicious device could have:\n");
    typeString("- Exfiltrated your files\n");
    typeString("- Installed malware\n");
    typeString("- Created a backdoor\n\n");
    typeString("Protect yourself:\n");
    typeString("- Never plug in unknown USB devices\n");
    typeString("- Use USB device whitelisting\n");
    typeString("- Enable USB Restricted Mode\n");
}

void loop() {
    if (Serial.available()) {
        char cmd = Serial.read();
        if (cmd == 'r') {
            Serial.println("Executing payload...");
            payloadWindowsMessage();
            Serial.println("Payload complete");
        }
    }
}
```

---

### AceButton (Rotary Encoder)

The T-Embed's rotary encoder is the primary input device. Use interrupts for reliable rotation detection:

```cpp
#define ENCODER_A   2
#define ENCODER_B   1
#define ENCODER_BTN 0

volatile int encoderPos = 0;
volatile bool buttonPressed = false;
unsigned long lastButtonPress = 0;

void IRAM_ATTR encoderISR() {
    if (digitalRead(ENCODER_B)) {
        encoderPos++;
    } else {
        encoderPos--;
    }
}

void IRAM_ATTR buttonISR() {
    unsigned long now = millis();
    if (now - lastButtonPress > 200) {  // Debounce
        buttonPressed = true;
        lastButtonPress = now;
    }
}

void setup() {
    Serial.begin(115200);

    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(ENCODER_BTN, INPUT_PULLUP);

    attachInterrupt(digitalPinToInterrupt(ENCODER_A), encoderISR, FALLING);
    attachInterrupt(digitalPinToInterrupt(ENCODER_BTN), buttonISR, FALLING);
}

// Menu navigation pattern
#define MENU_ITEMS 6
const char *menuItems[] = {
    "Sub-GHz RX", "Sub-GHz TX", "WiFi Scan",
    "BLE Scan", "IR Remote", "Settings"
};
int selectedItem = 0;

void loop() {
    static int lastPos = 0;

    if (encoderPos != lastPos) {
        int delta = encoderPos - lastPos;
        lastPos = encoderPos;

        selectedItem += delta;
        if (selectedItem < 0) selectedItem = MENU_ITEMS - 1;
        if (selectedItem >= MENU_ITEMS) selectedItem = 0;

        Serial.printf("Selected: %s\n", menuItems[selectedItem]);
    }

    if (buttonPressed) {
        buttonPressed = false;
        Serial.printf("Activated: %s\n", menuItems[selectedItem]);
        // Handle menu selection...
    }
}
```

---

## Complete Project Examples

### Project 1: Sub-GHz Signal Analyzer

A complete, self-contained signal analyzer that receives on 433 MHz, visualizes signals on the TFT, shows RSSI, decodes OOK patterns, and saves captures to internal flash.

```cpp
#include <TFT_eSPI.h>
#include <ELECHOUSE_CC1101_SRC_DRV.h>
#include <FS.h>
#include <SPIFFS.h>

// Hardware pins
#define ENCODER_A    2
#define ENCODER_B    1
#define ENCODER_BTN  0
#define TFT_BL_PIN  40
#define GDO0_PIN    38

// Display
TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);

// Signal capture
#define MAX_TRANSITIONS 4096
volatile uint32_t transitions[MAX_TRANSITIONS];
volatile int transCount = 0;
volatile uint32_t lastEdge = 0;

// Settings
float frequency = 433.92;
int modulation = 2;  // ASK/OOK
bool recording = false;
int captureCount = 0;

// Display layout
#define HEADER_H   22
#define GRAPH_Y    28
#define GRAPH_H    90
#define GRAPH_W   300
#define GRAPH_X    10
#define STATS_Y   125

// Signal visualization
#define VIS_SAMPLES 300
uint8_t signalVis[VIS_SAMPLES];
int visIdx = 0;

// Encoder
volatile int encPos = 0;
volatile bool encBtn = false;

void IRAM_ATTR onEncoder() {
    encPos += digitalRead(ENCODER_B) ? 1 : -1;
}

void IRAM_ATTR onButton() {
    static unsigned long last = 0;
    if (millis() - last > 200) { encBtn = true; last = millis(); }
}

void IRAM_ATTR onSignalEdge() {
    uint32_t now = micros();
    if (transCount < MAX_TRANSITIONS) {
        transitions[transCount++] = now - lastEdge;
        lastEdge = now;
    }
}

void setup() {
    Serial.begin(115200);
    SPIFFS.begin(true);

    // Display
    tft.init();
    tft.setRotation(3);
    pinMode(TFT_BL_PIN, OUTPUT);
    digitalWrite(TFT_BL_PIN, HIGH);
    canvas.createSprite(320, 170);

    // Encoder
    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(ENCODER_A), onEncoder, FALLING);
    attachInterrupt(digitalPinToInterrupt(ENCODER_BTN), onButton, FALLING);

    // CC1101
    if (!ELECHOUSE_cc1101.getCC1101()) {
        canvas.fillScreen(TFT_RED);
        canvas.setTextColor(TFT_WHITE);
        canvas.setTextFont(4);
        canvas.setTextDatum(MC_DATUM);
        canvas.drawString("CC1101 NOT FOUND", 160, 85);
        canvas.pushSprite(0, 0);
        while (1) delay(1000);
    }

    ELECHOUSE_cc1101.Init();
    ELECHOUSE_cc1101.setMHZ(frequency);
    ELECHOUSE_cc1101.setModulation(modulation);
    ELECHOUSE_cc1101.setRxBW(58.03);
    ELECHOUSE_cc1101.SetRx();

    memset(signalVis, 0, VIS_SAMPLES);

    Serial.println("Sub-GHz Signal Analyzer Ready");
}

void drawUI() {
    canvas.fillScreen(TFT_BLACK);

    // Header
    canvas.fillRect(0, 0, 320, HEADER_H, 0x1082);
    canvas.setTextColor(TFT_WHITE);
    canvas.setTextFont(2);
    canvas.setTextDatum(ML_DATUM);
    canvas.drawString("SUB-GHZ ANALYZER", 6, HEADER_H / 2);

    canvas.setTextDatum(MR_DATUM);
    char fStr[16];
    snprintf(fStr, sizeof(fStr), "%.2f MHz", frequency);
    canvas.drawString(fStr, 314, HEADER_H / 2);

    // Recording indicator
    if (recording) {
        canvas.fillCircle(155, HEADER_H / 2, 5, TFT_RED);
        canvas.setTextDatum(ML_DATUM);
        canvas.setTextColor(TFT_RED);
        canvas.drawString("REC", 164, HEADER_H / 2);
    }

    // Signal graph border
    canvas.drawRect(GRAPH_X - 1, GRAPH_Y - 1, GRAPH_W + 2, GRAPH_H + 2, TFT_DARKGREY);

    // Draw signal visualization (scrolling waveform)
    for (int i = 0; i < VIS_SAMPLES - 1; i++) {
        int idx = (visIdx + i) % VIS_SAMPLES;
        int nextIdx = (visIdx + i + 1) % VIS_SAMPLES;
        int y1 = GRAPH_Y + GRAPH_H - map(signalVis[idx], 0, 255, 0, GRAPH_H);
        int y2 = GRAPH_Y + GRAPH_H - map(signalVis[nextIdx], 0, 255, 0, GRAPH_H);

        uint16_t color;
        if (signalVis[idx] > 200) color = TFT_RED;
        else if (signalVis[idx] > 150) color = TFT_YELLOW;
        else if (signalVis[idx] > 80) color = TFT_GREEN;
        else color = 0x03E0;

        canvas.drawLine(GRAPH_X + i, y1, GRAPH_X + i + 1, y2, color);
    }

    // RSSI scale labels
    canvas.setTextFont(1);
    canvas.setTextColor(TFT_DARKGREY);
    canvas.setTextDatum(MR_DATUM);
    canvas.drawString("-20", GRAPH_X - 3, GRAPH_Y + 5);
    canvas.drawString("-60", GRAPH_X - 3, GRAPH_Y + GRAPH_H / 2);
    canvas.drawString("-100", GRAPH_X - 3, GRAPH_Y + GRAPH_H - 5);

    // Stats bar
    int rssi = ELECHOUSE_cc1101.getRssi();
    int lqi = ELECHOUSE_cc1101.getLqi();

    canvas.setTextFont(2);
    canvas.setTextDatum(TL_DATUM);
    canvas.setTextColor(TFT_CYAN);
    canvas.drawString("RSSI:", 10, STATS_Y);
    canvas.drawString("LQI:", 10, STATS_Y + 16);
    canvas.drawString("Captures:", 170, STATS_Y);
    canvas.drawString("Mode:", 170, STATS_Y + 16);

    canvas.setTextColor(TFT_WHITE);
    char rssiStr[16], lqiStr[8];
    snprintf(rssiStr, sizeof(rssiStr), "%d dBm", rssi);
    snprintf(lqiStr, sizeof(lqiStr), "%d", lqi);
    canvas.drawString(rssiStr, 65, STATS_Y);
    canvas.drawString(lqiStr, 55, STATS_Y + 16);
    canvas.drawNumber(captureCount, 250, STATS_Y);
    canvas.drawString("ASK/OOK", 220, STATS_Y + 16);

    // RSSI bar
    int barW = map(constrain(rssi, -100, -20), -100, -20, 0, 80);
    uint16_t barColor = rssi > -50 ? TFT_GREEN : rssi > -70 ? TFT_YELLOW : TFT_RED;
    canvas.fillRect(100, STATS_Y + 3, barW, 10, barColor);
    canvas.drawRect(100, STATS_Y + 3, 80, 10, TFT_DARKGREY);

    // Bottom hint
    canvas.setTextFont(1);
    canvas.setTextColor(TFT_DARKGREY);
    canvas.setTextDatum(BC_DATUM);
    canvas.drawString("Rotate:Freq | Press:Record/Stop | Long:Save", 160, 168);

    canvas.pushSprite(0, 0);
}

void saveCapture() {
    if (transCount == 0) return;

    char filename[32];
    snprintf(filename, sizeof(filename), "/capture_%d.txt", captureCount);

    File f = SPIFFS.open(filename, FILE_WRITE);
    if (f) {
        f.printf("Frequency: %.2f MHz\n", frequency);
        f.printf("Modulation: ASK/OOK\n");
        f.printf("Transitions: %d\n", transCount);
        f.printf("RAW_Data: ");
        for (int i = 0; i < transCount; i++) {
            f.printf("%d ", transitions[i]);
        }
        f.println();
        f.close();

        captureCount++;
        Serial.printf("Saved %s (%d transitions)\n", filename, transCount);
    }
}

void loop() {
    // Read RSSI for visualization
    int rssi = ELECHOUSE_cc1101.getRssi();
    signalVis[visIdx] = map(constrain(rssi, -100, -20), -100, -20, 0, 255);
    visIdx = (visIdx + 1) % VIS_SAMPLES;

    // Handle encoder rotation (frequency adjustment)
    static int lastEnc = 0;
    if (encPos != lastEnc) {
        int delta = encPos - lastEnc;
        lastEnc = encPos;

        frequency += delta * 0.01;
        frequency = constrain(frequency, 300.0, 928.0);

        ELECHOUSE_cc1101.setMHZ(frequency);
        ELECHOUSE_cc1101.SetRx();
    }

    // Handle button press (toggle recording)
    if (encBtn) {
        encBtn = false;

        if (!recording) {
            // Start recording
            transCount = 0;
            lastEdge = micros();
            attachInterrupt(digitalPinToInterrupt(GDO0_PIN), onSignalEdge, CHANGE);
            recording = true;
            Serial.println("Recording started");
        } else {
            // Stop recording and save
            detachInterrupt(digitalPinToInterrupt(GDO0_PIN));
            recording = false;
            saveCapture();
            Serial.printf("Recording stopped: %d transitions\n", transCount);
        }
    }

    drawUI();
    delay(30);  // ~30 FPS
}
```

---

### Project 2: WiFi + BLE Scanner Dashboard

```cpp
#include <TFT_eSPI.h>
#include <WiFi.h>
#include <NimBLEDevice.h>

TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);

#define ENCODER_A    2
#define ENCODER_B    1
#define ENCODER_BTN  0

// Scan results
struct WifiNet {
    char ssid[33];
    int rssi;
    uint8_t enc;
    uint8_t channel;
    uint8_t bssid[6];
};

struct BLEDev {
    char name[32];
    char addr[18];
    int rssi;
    uint16_t mfr;
};

#define MAX_WIFI 64
#define MAX_BLE  64

WifiNet wifiNets[MAX_WIFI];
int wifiCount = 0;
BLEDev bleDevs[MAX_BLE];
int bleCount = 0;

// UI state
int scrollOffset = 0;
int activeTab = 0;  // 0=WiFi, 1=BLE
volatile int encPos = 0;
volatile bool encBtn = false;

void IRAM_ATTR onEnc() { encPos += digitalRead(ENCODER_B) ? 1 : -1; }
void IRAM_ATTR onBtn() {
    static unsigned long l = 0;
    if (millis() - l > 200) { encBtn = true; l = millis(); }
}

// BLE scan callback
class BLECb : public NimBLEScanCallbacks {
    void onResult(const NimBLEAdvertisedDevice *dev) override {
        if (bleCount >= MAX_BLE) return;
        BLEDev &d = bleDevs[bleCount];
        strncpy(d.addr, dev->getAddress().toString().c_str(), 17);
        d.addr[17] = 0;
        if (dev->haveName()) {
            strncpy(d.name, dev->getName().c_str(), 31);
        } else {
            strcpy(d.name, "(unknown)");
        }
        d.name[31] = 0;
        d.rssi = dev->getRSSI();
        d.mfr = 0;
        if (dev->haveManufacturerData()) {
            std::string m = dev->getManufacturerData();
            if (m.length() >= 2) d.mfr = (uint8_t)m[0] | ((uint8_t)m[1] << 8);
        }
        bleCount++;
    }
};

void scanWiFi() {
    wifiCount = 0;
    int n = WiFi.scanNetworks(false, true);
    for (int i = 0; i < n && i < MAX_WIFI; i++) {
        strncpy(wifiNets[i].ssid, WiFi.SSID(i).c_str(), 32);
        wifiNets[i].ssid[32] = 0;
        if (strlen(wifiNets[i].ssid) == 0) strcpy(wifiNets[i].ssid, "(hidden)");
        wifiNets[i].rssi = WiFi.RSSI(i);
        wifiNets[i].enc = WiFi.encryptionType(i);
        wifiNets[i].channel = WiFi.channel(i);
        memcpy(wifiNets[i].bssid, WiFi.BSSID(i), 6);
        wifiCount++;
    }
    WiFi.scanDelete();

    // Sort by RSSI (strongest first)
    for (int i = 0; i < wifiCount - 1; i++) {
        for (int j = i + 1; j < wifiCount; j++) {
            if (wifiNets[j].rssi > wifiNets[i].rssi) {
                WifiNet tmp = wifiNets[i];
                wifiNets[i] = wifiNets[j];
                wifiNets[j] = tmp;
            }
        }
    }
}

void scanBLE() {
    bleCount = 0;
    NimBLEScan *s = NimBLEDevice::getScan();
    s->clearResults();
    s->start(5, false);
}

uint16_t encColor(uint8_t enc) {
    switch (enc) {
        case WIFI_AUTH_OPEN: return TFT_RED;
        case WIFI_AUTH_WEP:  return TFT_ORANGE;
        case WIFI_AUTH_WPA_PSK: return TFT_YELLOW;
        default: return TFT_GREEN;
    }
}

const char *encStr(uint8_t enc) {
    switch (enc) {
        case WIFI_AUTH_OPEN: return "OPEN";
        case WIFI_AUTH_WEP:  return "WEP";
        case WIFI_AUTH_WPA_PSK: return "WPA";
        case WIFI_AUTH_WPA2_PSK: return "WPA2";
        case WIFI_AUTH_WPA_WPA2_PSK: return "WPA/2";
        case WIFI_AUTH_WPA3_PSK: return "WPA3";
        default: return "?";
    }
}

uint16_t rssiColor(int rssi) {
    if (rssi > -50) return TFT_GREEN;
    if (rssi > -70) return TFT_YELLOW;
    if (rssi > -85) return TFT_ORANGE;
    return TFT_RED;
}

void drawUI() {
    canvas.fillScreen(TFT_BLACK);

    // Tabs
    uint16_t tabWifi = activeTab == 0 ? TFT_NAVY : 0x1082;
    uint16_t tabBLE  = activeTab == 1 ? TFT_NAVY : 0x1082;
    canvas.fillRect(0, 0, 160, 20, tabWifi);
    canvas.fillRect(160, 0, 160, 20, tabBLE);

    canvas.setTextFont(2);
    canvas.setTextDatum(MC_DATUM);
    canvas.setTextColor(activeTab == 0 ? TFT_WHITE : TFT_DARKGREY);
    char wTab[20];
    snprintf(wTab, sizeof(wTab), "WiFi (%d)", wifiCount);
    canvas.drawString(wTab, 80, 10);

    canvas.setTextColor(activeTab == 1 ? TFT_WHITE : TFT_DARKGREY);
    char bTab[20];
    snprintf(bTab, sizeof(bTab), "BLE (%d)", bleCount);
    canvas.drawString(bTab, 240, 10);

    canvas.drawFastHLine(0, 20, 320, TFT_DARKGREY);

    // List area
    int listY = 24;
    int rowH = 14;
    int maxRows = 10;

    canvas.setTextFont(1);
    canvas.setTextDatum(TL_DATUM);

    if (activeTab == 0) {
        // WiFi list
        for (int i = 0; i < maxRows && (scrollOffset + i) < wifiCount; i++) {
            int idx = scrollOffset + i;
            WifiNet &w = wifiNets[idx];
            int y = listY + i * rowH;

            // RSSI bar
            int barW = map(constrain(w.rssi, -100, -30), -100, -30, 0, 30);
            canvas.fillRect(2, y + 2, barW, 10, rssiColor(w.rssi));

            // Encryption indicator
            canvas.fillCircle(40, y + 6, 3, encColor(w.enc));

            // SSID
            canvas.setTextColor(TFT_WHITE);
            canvas.drawString(w.ssid, 48, y + 2);

            // Details
            canvas.setTextColor(TFT_DARKGREY);
            char det[32];
            snprintf(det, sizeof(det), "CH%d %s %ddBm", w.channel, encStr(w.enc), w.rssi);
            canvas.setTextDatum(TR_DATUM);
            canvas.drawString(det, 318, y + 2);
            canvas.setTextDatum(TL_DATUM);
        }
    } else {
        // BLE list
        for (int i = 0; i < maxRows && (scrollOffset + i) < bleCount; i++) {
            int idx = scrollOffset + i;
            BLEDev &b = bleDevs[idx];
            int y = listY + i * rowH;

            // RSSI bar
            int barW = map(constrain(b.rssi, -100, -30), -100, -30, 0, 30);
            canvas.fillRect(2, y + 2, barW, 10, rssiColor(b.rssi));

            // Name or address
            canvas.setTextColor(TFT_WHITE);
            canvas.drawString(b.name, 36, y + 2);

            // RSSI and manufacturer
            canvas.setTextColor(TFT_DARKGREY);
            char det[24];
            snprintf(det, sizeof(det), "%ddBm", b.rssi);
            canvas.setTextDatum(TR_DATUM);
            canvas.drawString(det, 318, y + 2);
            canvas.setTextDatum(TL_DATUM);
        }
    }

    // Scrollbar
    int totalItems = activeTab == 0 ? wifiCount : bleCount;
    if (totalItems > maxRows) {
        int sbH = (maxRows * (170 - listY)) / totalItems;
        int sbY = listY + (scrollOffset * (170 - listY - sbH)) / (totalItems - maxRows);
        canvas.fillRect(317, sbY, 3, sbH, TFT_DARKGREY);
    }

    canvas.pushSprite(0, 0);
}

void setup() {
    Serial.begin(115200);

    tft.init();
    tft.setRotation(3);
    canvas.createSprite(320, 170);
    pinMode(40, OUTPUT);
    digitalWrite(40, HIGH);

    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(ENCODER_A), onEnc, FALLING);
    attachInterrupt(digitalPinToInterrupt(ENCODER_BTN), onBtn, FALLING);

    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    NimBLEDevice::init("T-Embed");
    NimBLEDevice::getScan()->setScanCallbacks(new BLECb());
    NimBLEDevice::getScan()->setActiveScan(true);

    scanWiFi();
}

unsigned long lastScan = 0;

void loop() {
    // Handle encoder scroll
    static int lastEnc = 0;
    if (encPos != lastEnc) {
        int delta = encPos - lastEnc;
        lastEnc = encPos;
        int total = activeTab == 0 ? wifiCount : bleCount;
        scrollOffset = constrain(scrollOffset + delta, 0, max(0, total - 10));
    }

    // Handle button (switch tabs)
    if (encBtn) {
        encBtn = false;
        activeTab = 1 - activeTab;
        scrollOffset = 0;
    }

    // Auto-rescan every 15 seconds
    if (millis() - lastScan > 15000) {
        lastScan = millis();
        scanWiFi();
        scanBLE();
    }

    drawUI();
    delay(30);
}
```

---

### Project 3: Universal Remote with Menu

```cpp
#include <TFT_eSPI.h>
#include <IRrecv.h>
#include <IRsend.h>
#include <IRutils.h>
#include <FS.h>
#include <SPIFFS.h>
#include <ArduinoJson.h>

TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);

#define IR_RECV_PIN  4
#define IR_SEND_PIN  44
#define ENCODER_A    2
#define ENCODER_B    1
#define ENCODER_BTN  0

IRrecv irrecv(IR_RECV_PIN);
IRsend irsend(IR_SEND_PIN);
decode_results results;

// Signal storage
#define MAX_BUTTONS 32
struct IRButton {
    char label[16];
    decode_type_t protocol;
    uint64_t value;
    uint16_t bits;
    uint16_t rawData[256];
    uint16_t rawLen;
    bool used;
};

#define MAX_PROFILES 8
struct RemoteProfile {
    char name[20];
    IRButton buttons[MAX_BUTTONS];
    int buttonCount;
};

RemoteProfile profiles[MAX_PROFILES];
int profileCount = 0;
int currentProfile = 0;

// UI state
enum UIState { MENU_MAIN, MENU_PROFILE, CAPTURE_MODE, SEND_MODE };
UIState uiState = MENU_MAIN;
int menuSelection = 0;
volatile int encPos = 0;
volatile bool encBtn = false;
unsigned long encBtnTime = 0;
bool longPress = false;

void IRAM_ATTR onEnc() { encPos += digitalRead(ENCODER_B) ? 1 : -1; }
void IRAM_ATTR onBtn() {
    static unsigned long l = 0;
    if (millis() - l > 200) { encBtn = true; encBtnTime = millis(); l = millis(); }
}

void saveProfiles() {
    File f = SPIFFS.open("/remotes.json", FILE_WRITE);
    if (!f) return;

    JsonDocument doc;
    JsonArray arr = doc["profiles"].to<JsonArray>();

    for (int p = 0; p < profileCount; p++) {
        JsonObject prof = arr.add<JsonObject>();
        prof["name"] = profiles[p].name;
        JsonArray btns = prof["buttons"].to<JsonArray>();

        for (int b = 0; b < profiles[p].buttonCount; b++) {
            if (!profiles[p].buttons[b].used) continue;
            JsonObject btn = btns.add<JsonObject>();
            btn["label"] = profiles[p].buttons[b].label;
            btn["proto"] = (int)profiles[p].buttons[b].protocol;
            btn["value"] = (double)profiles[p].buttons[b].value;
            btn["bits"] = profiles[p].buttons[b].bits;
        }
    }

    serializeJson(doc, f);
    f.close();
}

void loadProfiles() {
    File f = SPIFFS.open("/remotes.json", FILE_READ);
    if (!f) return;

    JsonDocument doc;
    deserializeJson(doc, f);
    f.close();

    JsonArray arr = doc["profiles"].as<JsonArray>();
    profileCount = 0;

    for (JsonObject prof : arr) {
        if (profileCount >= MAX_PROFILES) break;
        strncpy(profiles[profileCount].name, prof["name"] | "Remote", 19);
        profiles[profileCount].buttonCount = 0;

        JsonArray btns = prof["buttons"].as<JsonArray>();
        for (JsonObject btn : btns) {
            int b = profiles[profileCount].buttonCount;
            if (b >= MAX_BUTTONS) break;
            strncpy(profiles[profileCount].buttons[b].label, btn["label"] | "Button", 15);
            profiles[profileCount].buttons[b].protocol = (decode_type_t)(int)btn["proto"];
            profiles[profileCount].buttons[b].value = (uint64_t)(double)btn["value"];
            profiles[profileCount].buttons[b].bits = btn["bits"];
            profiles[profileCount].buttons[b].used = true;
            profiles[profileCount].buttonCount++;
        }
        profileCount++;
    }
}

void drawMainMenu() {
    canvas.fillScreen(TFT_BLACK);

    // Title
    canvas.fillRect(0, 0, 320, 24, TFT_NAVY);
    canvas.setTextFont(2);
    canvas.setTextDatum(MC_DATUM);
    canvas.setTextColor(TFT_WHITE);
    canvas.drawString("UNIVERSAL REMOTE", 160, 12);

    // Menu items
    const char *items[] = {"My Remotes", "Capture New Signal", "Quick Send", "Manage Profiles"};
    int itemCount = 4;

    for (int i = 0; i < itemCount; i++) {
        int y = 34 + i * 30;
        if (i == menuSelection) {
            canvas.fillRoundRect(10, y, 300, 26, 4, 0x1082);
            canvas.drawRoundRect(10, y, 300, 26, 4, TFT_CYAN);
            canvas.setTextColor(TFT_WHITE);
        } else {
            canvas.setTextColor(TFT_DARKGREY);
        }
        canvas.setTextDatum(ML_DATUM);
        canvas.drawString(items[i], 24, y + 13);
    }

    // Footer
    canvas.setTextFont(1);
    canvas.setTextColor(0x4208);
    canvas.setTextDatum(BC_DATUM);
    canvas.drawString("Rotate: Navigate | Press: Select | Long: Back", 160, 168);

    canvas.pushSprite(0, 0);
}

void drawProfileMenu() {
    canvas.fillScreen(TFT_BLACK);

    RemoteProfile &prof = profiles[currentProfile];

    // Header
    canvas.fillRect(0, 0, 320, 24, 0x4000);
    canvas.setTextFont(2);
    canvas.setTextDatum(MC_DATUM);
    canvas.setTextColor(TFT_WHITE);
    canvas.drawString(prof.name, 160, 12);

    if (prof.buttonCount == 0) {
        canvas.setTextColor(TFT_DARKGREY);
        canvas.setTextFont(2);
        canvas.drawString("No buttons saved", 160, 85);
        canvas.drawString("Press to capture", 160, 105);
    } else {
        for (int i = 0; i < prof.buttonCount && i < 8; i++) {
            int col = i % 2;
            int row = i / 2;
            int x = 10 + col * 155;
            int y = 30 + row * 32;

            if (i == menuSelection) {
                canvas.fillRoundRect(x, y, 148, 28, 4, TFT_NAVY);
                canvas.drawRoundRect(x, y, 148, 28, 4, TFT_CYAN);
            } else {
                canvas.drawRoundRect(x, y, 148, 28, 4, TFT_DARKGREY);
            }

            canvas.setTextColor(TFT_WHITE);
            canvas.setTextDatum(MC_DATUM);
            canvas.drawString(prof.buttons[i].label, x + 74, y + 14);
        }
    }

    canvas.pushSprite(0, 0);
}

void setup() {
    Serial.begin(115200);
    SPIFFS.begin(true);

    tft.init();
    tft.setRotation(3);
    canvas.createSprite(320, 170);
    pinMode(40, OUTPUT);
    digitalWrite(40, HIGH);

    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(ENCODER_A), onEnc, FALLING);
    attachInterrupt(digitalPinToInterrupt(ENCODER_BTN), onBtn, FALLING);

    irrecv.enableIRIn();
    irsend.begin();

    loadProfiles();

    if (profileCount == 0) {
        strcpy(profiles[0].name, "Living Room TV");
        profiles[0].buttonCount = 0;
        profileCount = 1;
    }
}

void loop() {
    // Handle encoder
    static int lastEnc = 0;
    if (encPos != lastEnc) {
        int delta = encPos - lastEnc;
        lastEnc = encPos;

        int maxItems = 0;
        if (uiState == MENU_MAIN) maxItems = 4;
        else if (uiState == MENU_PROFILE) maxItems = profiles[currentProfile].buttonCount;

        menuSelection += delta;
        if (menuSelection < 0) menuSelection = max(0, maxItems - 1);
        if (menuSelection >= maxItems) menuSelection = 0;
    }

    // Handle button
    if (encBtn) {
        encBtn = false;

        if (uiState == MENU_MAIN) {
            if (menuSelection == 0) {
                uiState = MENU_PROFILE;
                currentProfile = 0;
                menuSelection = 0;
            } else if (menuSelection == 1) {
                uiState = CAPTURE_MODE;
            }
        } else if (uiState == MENU_PROFILE) {
            // Send the selected button's IR signal
            RemoteProfile &prof = profiles[currentProfile];
            if (menuSelection < prof.buttonCount) {
                IRButton &btn = prof.buttons[menuSelection];
                if (btn.protocol != decode_type_t::UNKNOWN) {
                    irsend.send(btn.protocol, btn.value, btn.bits);
                }
                // Flash feedback
                canvas.fillScreen(TFT_GREEN);
                canvas.setTextColor(TFT_BLACK);
                canvas.setTextFont(4);
                canvas.setTextDatum(MC_DATUM);
                canvas.drawString("SENT!", 160, 85);
                canvas.pushSprite(0, 0);
                delay(300);
            }
        } else if (uiState == CAPTURE_MODE) {
            uiState = MENU_MAIN;
            menuSelection = 0;
        }
    }

    // Capture mode: listen for IR
    if (uiState == CAPTURE_MODE) {
        canvas.fillScreen(TFT_BLACK);
        canvas.fillRect(0, 0, 320, 24, 0x4800);
        canvas.setTextFont(2);
        canvas.setTextDatum(MC_DATUM);
        canvas.setTextColor(TFT_WHITE);
        canvas.drawString("CAPTURE MODE", 160, 12);
        canvas.setTextColor(TFT_YELLOW);
        canvas.drawString("Point remote at receiver", 160, 60);
        canvas.drawString("and press a button...", 160, 80);

        // Check for IR
        if (irrecv.decode(&results)) {
            RemoteProfile &prof = profiles[currentProfile];
            int b = prof.buttonCount;
            if (b < MAX_BUTTONS) {
                snprintf(prof.buttons[b].label, 16, "Button %d", b + 1);
                prof.buttons[b].protocol = results.decode_type;
                prof.buttons[b].value = results.value;
                prof.buttons[b].bits = results.bits;
                prof.buttons[b].used = true;
                prof.buttonCount++;

                saveProfiles();

                canvas.fillScreen(TFT_GREEN);
                canvas.setTextColor(TFT_BLACK);
                canvas.setTextFont(2);
                canvas.drawString("CAPTURED!", 160, 70);
                char info[48];
                snprintf(info, sizeof(info), "%s: 0x%llX",
                    typeToString(results.decode_type).c_str(), results.value);
                canvas.drawString(info, 160, 95);
                canvas.pushSprite(0, 0);
                delay(1500);
            }
            irrecv.resume();
        }

        canvas.setTextColor(TFT_DARKGREY);
        canvas.setTextFont(1);
        canvas.drawString("Press encoder to exit", 160, 150);
        canvas.pushSprite(0, 0);
        delay(50);
        return;
    }

    // Draw appropriate menu
    switch (uiState) {
        case MENU_MAIN: drawMainMenu(); break;
        case MENU_PROFILE: drawProfileMenu(); break;
        default: break;
    }

    delay(30);
}
```

---

### Project 4: Portable Spectrum Analyzer

This project sweeps the CC1101 across a frequency range and renders a color-mapped waterfall display on the TFT, showing signal activity over time.

```cpp
#include <TFT_eSPI.h>
#include <ELECHOUSE_CC1101_SRC_DRV.h>

TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);

#define ENCODER_A    2
#define ENCODER_B    1
#define ENCODER_BTN  0

// Spectrum settings
float centerFreq = 433.92;
float spanMHz = 10.0;  // Total sweep width
int sweepPoints = 300; // Horizontal resolution

// Waterfall buffer (each row = one sweep)
#define WATERFALL_ROWS 120
#define WATERFALL_COLS 300
uint8_t waterfall[WATERFALL_ROWS][WATERFALL_COLS];
int waterfallRow = 0;

// Current sweep data
int8_t currentSweep[300];
float peakFreq = 0;
int8_t peakRSSI = -128;

volatile int encPos = 0;
volatile bool encBtn = false;
void IRAM_ATTR onEnc() { encPos += digitalRead(ENCODER_B) ? 1 : -1; }
void IRAM_ATTR onBtn() {
    static unsigned long l = 0;
    if (millis() - l > 200) { encBtn = true; l = millis(); }
}

// RSSI to color mapping (blue -> green -> yellow -> red -> white)
uint16_t rssiToColor(int8_t rssi) {
    // Map -110 to -20 dBm into 0-255 range
    int v = constrain(map(rssi, -110, -20, 0, 255), 0, 255);

    uint8_t r, g, b;
    if (v < 64) {        // Black to blue
        r = 0; g = 0; b = v * 4;
    } else if (v < 128) { // Blue to green
        r = 0; g = (v - 64) * 4; b = 255 - (v - 64) * 4;
    } else if (v < 192) { // Green to yellow
        r = (v - 128) * 4; g = 255; b = 0;
    } else {               // Yellow to red
        r = 255; g = 255 - (v - 192) * 4; b = 0;
    }

    return ((r & 0xF8) << 8) | ((g & 0xFC) << 3) | (b >> 3);
}

void performSweep() {
    float startFreq = centerFreq - spanMHz / 2.0;
    float step = spanMHz / sweepPoints;

    peakRSSI = -128;

    for (int i = 0; i < sweepPoints; i++) {
        float f = startFreq + i * step;
        ELECHOUSE_cc1101.setMHZ(f);
        ELECHOUSE_cc1101.SetRx();
        delayMicroseconds(300);

        int8_t rssi = ELECHOUSE_cc1101.getRssi();
        currentSweep[i] = rssi;

        // Store in waterfall (normalized to 0-255)
        waterfall[waterfallRow][i] = constrain(map(rssi, -110, -20, 0, 255), 0, 255);

        if (rssi > peakRSSI) {
            peakRSSI = rssi;
            peakFreq = f;
        }
    }

    waterfallRow = (waterfallRow + 1) % WATERFALL_ROWS;
}

void drawSpectrum() {
    canvas.fillScreen(TFT_BLACK);

    // Header
    canvas.setTextFont(1);
    canvas.setTextColor(TFT_WHITE);
    canvas.setTextDatum(TL_DATUM);
    canvas.drawString("SPECTRUM ANALYZER", 4, 2);

    canvas.setTextDatum(TR_DATUM);
    char cStr[24];
    snprintf(cStr, sizeof(cStr), "CF:%.2f Span:%.1f", centerFreq, spanMHz);
    canvas.drawString(cStr, 316, 2);

    // Draw waterfall (scrolling, newest at bottom)
    int waterfallY = 14;
    int waterfallH = WATERFALL_ROWS;

    for (int row = 0; row < WATERFALL_ROWS; row++) {
        int srcRow = (waterfallRow + row) % WATERFALL_ROWS;
        int y = waterfallY + row;

        for (int col = 0; col < WATERFALL_COLS; col++) {
            uint8_t v = waterfall[srcRow][col];
            if (v > 10) {  // Skip near-black pixels for speed
                canvas.drawPixel(10 + col, y, rssiToColor(-110 + v * 90 / 255));
            }
        }
    }

    // Current sweep line overlay (top of waterfall)
    int lineY = waterfallY + WATERFALL_ROWS + 2;
    int graphH = 30;

    canvas.drawFastHLine(10, lineY + graphH, 300, TFT_DARKGREY);

    for (int i = 0; i < sweepPoints - 1; i++) {
        int y1 = lineY + graphH - map(constrain(currentSweep[i], -110, -20), -110, -20, 0, graphH);
        int y2 = lineY + graphH - map(constrain(currentSweep[i+1], -110, -20), -110, -20, 0, graphH);
        canvas.drawLine(10 + i, y1, 11 + i, y2, TFT_GREEN);
    }

    // Peak marker
    if (peakRSSI > -100) {
        int peakX = 10 + (int)((peakFreq - (centerFreq - spanMHz / 2.0)) / spanMHz * sweepPoints);
        peakX = constrain(peakX, 10, 310);
        canvas.drawFastVLine(peakX, waterfallY, WATERFALL_ROWS, TFT_WHITE);

        canvas.setTextFont(1);
        canvas.setTextColor(TFT_CYAN);
        canvas.setTextDatum(BC_DATUM);
        char pStr[32];
        snprintf(pStr, sizeof(pStr), "%.2f MHz %d dBm", peakFreq, peakRSSI);
        canvas.drawString(pStr, 160, 168);
    }

    // Frequency axis
    canvas.setTextFont(1);
    canvas.setTextColor(TFT_DARKGREY);
    float startF = centerFreq - spanMHz / 2.0;
    float endF = centerFreq + spanMHz / 2.0;
    canvas.setTextDatum(TC_DATUM);

    char fLabel[8];
    snprintf(fLabel, sizeof(fLabel), "%.1f", startF);
    canvas.drawString(fLabel, 10, lineY + graphH + 2);
    snprintf(fLabel, sizeof(fLabel), "%.1f", centerFreq);
    canvas.drawString(fLabel, 160, lineY + graphH + 2);
    snprintf(fLabel, sizeof(fLabel), "%.1f", endF);
    canvas.drawString(fLabel, 310, lineY + graphH + 2);

    canvas.pushSprite(0, 0);
}

void setup() {
    Serial.begin(115200);

    tft.init();
    tft.setRotation(3);
    canvas.createSprite(320, 170);
    pinMode(40, OUTPUT);
    digitalWrite(40, HIGH);

    pinMode(ENCODER_A, INPUT_PULLUP);
    pinMode(ENCODER_B, INPUT_PULLUP);
    pinMode(ENCODER_BTN, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(ENCODER_A), onEnc, FALLING);
    attachInterrupt(digitalPinToInterrupt(ENCODER_BTN), onBtn, FALLING);

    ELECHOUSE_cc1101.Init();
    ELECHOUSE_cc1101.setModulation(2);
    ELECHOUSE_cc1101.setRxBW(58.03);

    memset(waterfall, 0, sizeof(waterfall));
}

void loop() {
    // Encoder: adjust center frequency
    static int lastEnc = 0;
    if (encPos != lastEnc) {
        int delta = encPos - lastEnc;
        lastEnc = encPos;
        centerFreq += delta * (spanMHz / 100.0);
        centerFreq = constrain(centerFreq, 300.0 + spanMHz / 2, 928.0 - spanMHz / 2);
    }

    // Button: cycle span
    if (encBtn) {
        encBtn = false;
        if (spanMHz == 2.0) spanMHz = 5.0;
        else if (spanMHz == 5.0) spanMHz = 10.0;
        else if (spanMHz == 10.0) spanMHz = 20.0;
        else if (spanMHz == 20.0) spanMHz = 50.0;
        else spanMHz = 2.0;
        memset(waterfall, 0, sizeof(waterfall));
    }

    performSweep();
    drawSpectrum();
}
```

---

### Project 5: Multi-Tool Framework

This is a skeleton framework that integrates Sub-GHz, WiFi, BLE, IR, and BadUSB into a single firmware with a unified menu system. This is essentially the architecture behind tools like Bruce firmware -- built from scratch so you understand every line.

```cpp
#include <TFT_eSPI.h>
#include <WiFi.h>
#include <NimBLEDevice.h>
#include <ELECHOUSE_CC1101_SRC_DRV.h>
#include <IRrecv.h>
#include <IRsend.h>
#include <USB.h>
#include <USBHIDKeyboard.h>
#include <FS.h>
#include <SPIFFS.h>
#include <Preferences.h>

// ============================================================
//  HARDWARE ABSTRACTION
// ============================================================

TFT_eSPI tft = TFT_eSPI();
TFT_eSprite canvas = TFT_eSprite(&tft);
IRrecv irrecv(4);
IRsend irsend(44);
USBHIDKeyboard keyboard;
Preferences prefs;

#define ENC_A   2
#define ENC_B   1
#define ENC_BTN 0

volatile int encDelta = 0;
volatile bool encPressed = false;
volatile unsigned long encPressTime = 0;

void IRAM_ATTR encISR() { encDelta += digitalRead(ENC_B) ? 1 : -1; }
void IRAM_ATTR btnISR() {
    static unsigned long last = 0;
    unsigned long now = millis();
    if (now - last > 200) {
        encPressed = true;
        encPressTime = now;
        last = now;
    }
}

// ============================================================
//  APP FRAMEWORK
// ============================================================

// Forward declarations
class App;
void setActiveApp(App *app);

class App {
public:
    virtual const char *name() = 0;
    virtual void onEnter() {}
    virtual void onExit() {}
    virtual void onTick() = 0;
    virtual void onScroll(int delta) {}
    virtual void onPress() {}
    virtual void onLongPress() {}
    virtual ~App() {}
};

// Global state
App *activeApp = nullptr;
App *mainMenu = nullptr;

void setActiveApp(App *app) {
    if (activeApp) activeApp->onExit();
    activeApp = app;
    if (activeApp) activeApp->onEnter();
}

// ============================================================
//  DRAWING HELPERS
// ============================================================

void drawHeader(const char *title, uint16_t color = TFT_NAVY) {
    canvas.fillRect(0, 0, 320, 22, color);
    canvas.setTextFont(2);
    canvas.setTextDatum(MC_DATUM);
    canvas.setTextColor(TFT_WHITE);
    canvas.drawString(title, 160, 11);
}

void drawFooter(const char *hint) {
    canvas.setTextFont(1);
    canvas.setTextColor(0x4208);
    canvas.setTextDatum(BC_DATUM);
    canvas.drawString(hint, 160, 168);
}

void drawMenuItem(int index, const char *label, bool selected, int startY = 28) {
    int y = startY + index * 28;
    if (selected) {
        canvas.fillRoundRect(8, y, 304, 24, 4, 0x1082);
        canvas.drawRoundRect(8, y, 304, 24, 4, TFT_CYAN);
        canvas.setTextColor(TFT_WHITE);
    } else {
        canvas.setTextColor(TFT_DARKGREY);
    }
    canvas.setTextFont(2);
    canvas.setTextDatum(ML_DATUM);
    canvas.drawString(label, 20, y + 12);
}

// ============================================================
//  MAIN MENU
// ============================================================

class SubGhzApp : public App {
    int sel = 0;
    const char *items[4] = {"Receiver", "Transmitter", "Scanner", "Signal Replay"};
public:
    const char *name() override { return "Sub-GHz"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);
        drawHeader("SUB-GHZ", 0x4000);
        for (int i = 0; i < 4; i++) drawMenuItem(i, items[i], i == sel);
        drawFooter("Scroll:Nav | Press:Select | Long:Back");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + 4) % 4; }
    void onPress() override { /* Launch sub-app */ }
    void onLongPress() override { setActiveApp(mainMenu); }
};

class WifiApp : public App {
    int sel = 0;
    const char *items[4] = {"Scan Networks", "Deauth Detector", "Beacon Spammer", "Packet Monitor"};
public:
    const char *name() override { return "WiFi"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);
        drawHeader("WIFI TOOLS", 0x0300);
        for (int i = 0; i < 4; i++) drawMenuItem(i, items[i], i == sel);
        drawFooter("Scroll:Nav | Press:Select | Long:Back");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + 4) % 4; }
    void onLongPress() override { setActiveApp(mainMenu); }
};

class BleApp : public App {
    int sel = 0;
    const char *items[3] = {"BLE Scanner", "BLE Spam", "Apple BLE Spam"};
public:
    const char *name() override { return "Bluetooth"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);
        drawHeader("BLUETOOTH", 0x0018);
        for (int i = 0; i < 3; i++) drawMenuItem(i, items[i], i == sel);
        drawFooter("Scroll:Nav | Press:Select | Long:Back");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + 3) % 3; }
    void onLongPress() override { setActiveApp(mainMenu); }
};

class IrApp : public App {
    int sel = 0;
    const char *items[3] = {"IR Receiver", "IR Transmitter", "TV-B-Gone"};
public:
    const char *name() override { return "Infrared"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);
        drawHeader("INFRARED", 0x4800);
        for (int i = 0; i < 3; i++) drawMenuItem(i, items[i], i == sel);
        drawFooter("Scroll:Nav | Press:Select | Long:Back");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + 3) % 3; }
    void onLongPress() override { setActiveApp(mainMenu); }
};

class BadUsbApp : public App {
    int sel = 0;
    const char *items[3] = {"Run Payload", "Edit Payload", "USB Keyboard"};
public:
    const char *name() override { return "BadUSB"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);
        drawHeader("BAD USB", 0x4010);
        for (int i = 0; i < 3; i++) drawMenuItem(i, items[i], i == sel);
        drawFooter("Scroll:Nav | Press:Select | Long:Back");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + 3) % 3; }
    void onLongPress() override { setActiveApp(mainMenu); }
};

class SettingsApp : public App {
    int sel = 0;
    const char *items[4] = {"Brightness", "Sound", "About", "Reboot"};
public:
    const char *name() override { return "Settings"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);
        drawHeader("SETTINGS", TFT_DARKGREY);
        for (int i = 0; i < 4; i++) drawMenuItem(i, items[i], i == sel);
        drawFooter("Scroll:Nav | Press:Select | Long:Back");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + 4) % 4; }
    void onPress() override {
        if (sel == 3) ESP.restart();
    }
    void onLongPress() override { setActiveApp(mainMenu); }
};

// App instances
SubGhzApp subGhzApp;
WifiApp wifiApp;
BleApp bleApp;
IrApp irApp;
BadUsbApp badUsbApp;
SettingsApp settingsApp;

App *apps[] = { &subGhzApp, &wifiApp, &bleApp, &irApp, &badUsbApp, &settingsApp };
const int APP_COUNT = 6;
const char *appIcons[] = { "RF", "Wi", "BT", "IR", "US", "ST" };
const uint16_t appColors[] = { 0x4000, 0x0300, 0x0018, 0x4800, 0x4010, TFT_DARKGREY };

class MainMenuApp : public App {
    int sel = 0;
public:
    const char *name() override { return "Main Menu"; }
    void onEnter() override { sel = 0; }
    void onTick() override {
        canvas.fillScreen(TFT_BLACK);

        // Title
        canvas.setTextFont(2);
        canvas.setTextDatum(MC_DATUM);
        canvas.setTextColor(TFT_WHITE);
        canvas.drawString("T-EMBED MULTI-TOOL", 160, 12);
        canvas.drawFastHLine(20, 24, 280, TFT_DARKGREY);

        // Grid layout (2x3)
        for (int i = 0; i < APP_COUNT; i++) {
            int col = i % 3;
            int row = i / 3;
            int x = 15 + col * 100;
            int y = 32 + row * 66;
            int w = 90;
            int h = 58;

            if (i == sel) {
                canvas.fillRoundRect(x, y, w, h, 6, appColors[i]);
                canvas.drawRoundRect(x, y, w, h, 6, TFT_WHITE);
            } else {
                canvas.drawRoundRect(x, y, w, h, 6, appColors[i]);
            }

            // Icon text
            canvas.setTextFont(4);
            canvas.setTextColor(i == sel ? TFT_WHITE : appColors[i]);
            canvas.drawString(appIcons[i], x + w / 2, y + 22);

            // Label
            canvas.setTextFont(1);
            canvas.setTextColor(i == sel ? TFT_WHITE : TFT_DARKGREY);
            canvas.drawString(apps[i]->name(), x + w / 2, y + h - 10);
        }

        drawFooter("Scroll:Navigate | Press:Open");
        canvas.pushSprite(0, 0);
    }
    void onScroll(int d) override { sel = (sel + d + APP_COUNT) % APP_COUNT; }
    void onPress() override { setActiveApp(apps[sel]); }
};

MainMenuApp mainMenuApp;

// ============================================================
//  SETUP & LOOP
// ============================================================

void setup() {
    Serial.begin(115200);
    SPIFFS.begin(true);
    prefs.begin("multitool", false);

    // Display
    tft.init();
    tft.setRotation(3);
    tft.fillScreen(TFT_BLACK);
    canvas.createSprite(320, 170);
    canvas.setSwapBytes(true);
    pinMode(40, OUTPUT);
    digitalWrite(40, HIGH);

    // Encoder
    pinMode(ENC_A, INPUT_PULLUP);
    pinMode(ENC_B, INPUT_PULLUP);
    pinMode(ENC_BTN, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(ENC_A), encISR, FALLING);
    attachInterrupt(digitalPinToInterrupt(ENC_BTN), btnISR, FALLING);

    // CC1101
    if (ELECHOUSE_cc1101.getCC1101()) {
        ELECHOUSE_cc1101.Init();
        Serial.println("CC1101 OK");
    }

    // BLE
    NimBLEDevice::init("T-Embed");

    // IR
    irrecv.enableIRIn();
    irsend.begin();

    // USB HID
    USB.begin();
    keyboard.begin();

    // WiFi in station mode (not connected)
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();

    // Boot splash
    canvas.fillScreen(TFT_BLACK);
    canvas.setTextFont(4);
    canvas.setTextDatum(MC_DATUM);
    canvas.setTextColor(TFT_CYAN);
    canvas.drawString("T-EMBED", 160, 60);
    canvas.setTextFont(2);
    canvas.setTextColor(TFT_DARKGREY);
    canvas.drawString("MULTI-TOOL v1.0", 160, 95);
    canvas.drawString("Custom Firmware", 160, 115);
    canvas.pushSprite(0, 0);
    delay(1500);

    // Start at main menu
    mainMenu = &mainMenuApp;
    setActiveApp(mainMenu);

    Serial.println("Multi-tool ready");
}

void loop() {
    // Process encoder rotation
    if (encDelta != 0) {
        int d = encDelta;
        encDelta = 0;
        if (activeApp) activeApp->onScroll(d);
    }

    // Process button press
    if (encPressed) {
        encPressed = false;

        // Detect long press (hold > 800ms)
        unsigned long pressStart = encPressTime;
        delay(50);  // Brief debounce

        if (!digitalRead(ENC_BTN)) {
            // Button still held, wait for release or long press threshold
            while (!digitalRead(ENC_BTN) && (millis() - pressStart < 800)) {
                delay(10);
            }

            if (millis() - pressStart >= 800) {
                if (activeApp) activeApp->onLongPress();
            } else {
                if (activeApp) activeApp->onPress();
            }
        } else {
            if (activeApp) activeApp->onPress();
        }
    }

    // Run active app
    if (activeApp) activeApp->onTick();

    delay(16);  // ~60 FPS target
}
```

{: .tip }
> This framework uses an object-oriented app pattern. Each feature module is a self-contained `App` class. To add a new feature, create a new class inheriting from `App`, implement the virtual methods, and add it to the `apps[]` array. This is the exact same architectural pattern used by professional multi-tool firmwares.

---

## Code Architecture Patterns

### FreeRTOS Tasks for Concurrent Operations

```cpp
// Run WiFi scanning and BLE scanning simultaneously on different cores
TaskHandle_t wifiTaskHandle;
TaskHandle_t bleTaskHandle;

void wifiTask(void *param) {
    while (true) {
        int n = WiFi.scanNetworks(false, true);
        // Process results...
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}

void bleTask(void *param) {
    while (true) {
        NimBLEDevice::getScan()->start(5, false);
        // Process results...
        vTaskDelay(pdMS_TO_TICKS(15000));
    }
}

void setup() {
    xTaskCreatePinnedToCore(wifiTask, "WiFi", 8192, NULL, 1, &wifiTaskHandle, 0);
    xTaskCreatePinnedToCore(bleTask, "BLE", 8192, NULL, 1, &bleTaskHandle, 1);
}
```

### State Machine Pattern for Menus

```cpp
enum State { STATE_IDLE, STATE_SCANNING, STATE_RESULTS, STATE_DETAIL };
State currentState = STATE_IDLE;

void handleState() {
    switch (currentState) {
        case STATE_IDLE:
            drawIdleScreen();
            if (buttonPressed) currentState = STATE_SCANNING;
            break;
        case STATE_SCANNING:
            drawScanningScreen();
            performScan();
            currentState = STATE_RESULTS;
            break;
        case STATE_RESULTS:
            drawResultsScreen();
            if (buttonPressed) currentState = STATE_DETAIL;
            if (backPressed) currentState = STATE_IDLE;
            break;
        case STATE_DETAIL:
            drawDetailScreen();
            if (backPressed) currentState = STATE_RESULTS;
            break;
    }
}
```

### Memory Management on ESP32-S3

```cpp
void printMemoryInfo() {
    Serial.printf("Free heap:      %d bytes\n", ESP.getFreeHeap());
    Serial.printf("Free PSRAM:     %d bytes\n", ESP.getFreePsram());
    Serial.printf("Heap size:      %d bytes\n", ESP.getHeapSize());
    Serial.printf("PSRAM size:     %d bytes\n", ESP.getPsramSize());
    Serial.printf("Min free heap:  %d bytes\n", ESP.getMinFreeHeap());
}

// Allocate large buffers in PSRAM
uint8_t *bigBuffer = (uint8_t *)ps_malloc(1024 * 1024);  // 1MB in PSRAM

// Check if PSRAM allocation succeeded
if (bigBuffer == nullptr) {
    Serial.println("PSRAM allocation failed!");
}
```

{: .note }
> The ESP32-S3 on the T-Embed has 8MB of PSRAM (OPI). Use `ps_malloc()` for large allocations (display buffers, signal captures, scan results) and leave the regular heap for stack and small objects.

### Non-Blocking Design

```cpp
// BAD: blocking delay
void badExample() {
    scanNetworks();
    delay(5000);        // Nothing else can run for 5 seconds
    displayResults();
}

// GOOD: non-blocking with millis()
unsigned long lastScan = 0;
bool scanPending = false;

void goodExample() {
    if (!scanPending && millis() - lastScan > 5000) {
        scanNetworks();
        scanPending = true;
    }

    if (scanPending && scanComplete()) {
        displayResults();
        scanPending = false;
        lastScan = millis();
    }

    // UI remains responsive
    handleEncoder();
    updateDisplay();
}
```

---

## Debugging Techniques

### Serial Debugging

```cpp
// Use different log levels
#define LOG_ERROR   0
#define LOG_WARN    1
#define LOG_INFO    2
#define LOG_DEBUG   3
#define LOG_VERBOSE 4

int logLevel = LOG_DEBUG;

void logMsg(int level, const char *tag, const char *fmt, ...) {
    if (level > logLevel) return;

    const char *labels[] = {"ERROR", "WARN", "INFO", "DEBUG", "VERB"};
    char buf[256];
    va_list args;
    va_start(args, fmt);
    vsnprintf(buf, sizeof(buf), fmt, args);
    va_end(args);

    Serial.printf("[%s][%s] %s\n", labels[level], tag, buf);
}

// Usage
logMsg(LOG_INFO, "CC1101", "Frequency set to %.2f MHz", freq);
logMsg(LOG_ERROR, "SPI", "Device not responding on CS pin %d", csPin);
logMsg(LOG_DEBUG, "BLE", "Found device: %s RSSI: %d", name, rssi);
```

### ESP32 Exception Decoder

When the ESP32 crashes, it prints a stack trace with memory addresses. To decode these:

```bash
# Install the decoder
pip install esp-exception-decoder

# Decode a crash (paste the backtrace)
# Or use the PlatformIO "ESP Exception Decoder" extension in VS Code
```

### Memory Leak Detection

```cpp
// Monitor heap usage over time
void heapWatchdog(void *param) {
    int lastHeap = ESP.getFreeHeap();

    while (true) {
        int currentHeap = ESP.getFreeHeap();
        int delta = currentHeap - lastHeap;

        if (delta < -1000) {  // More than 1KB lost
            Serial.printf("HEAP WARNING: Lost %d bytes (now %d free)\n",
                -delta, currentHeap);
        }

        lastHeap = currentHeap;
        vTaskDelay(pdMS_TO_TICKS(5000));
    }
}

// Start watchdog in setup()
xTaskCreate(heapWatchdog, "HeapWatch", 2048, NULL, 0, NULL);
```

---

## Publishing Your Firmware

### Creating a Binary Release

```bash
# Build in PlatformIO
pio run

# The binary is at:
# .pio/build/t-embed-cc1101/firmware.bin

# For Arduino IDE, enable verbose output in preferences
# The binary path is shown in the build output
```

### Partition Table Customization

Create `partitions.csv` in your project root:

```csv
# Name,    Type, SubType, Offset,   Size,    Flags
nvs,       data, nvs,     0x9000,   0x5000,
otadata,   data, ota,     0xe000,   0x2000,
app0,      app,  ota_0,   0x10000,  0x300000,
app1,      app,  ota_1,   0x310000, 0x300000,
spiffs,    data, spiffs,  0x610000, 0x9F0000,
```

Reference in `platformio.ini`:

```ini
board_build.partitions = partitions.csv
```

### OTA Update Implementation

```cpp
#include <ArduinoOTA.h>
#include <WiFi.h>

void setupOTA() {
    WiFi.begin("YourSSID", "YourPassword");
    while (WiFi.status() != WL_CONNECTED) delay(500);

    ArduinoOTA.setHostname("t-embed-cc1101");
    ArduinoOTA.setPassword("update123");

    ArduinoOTA.onStart([]() {
        Serial.println("OTA update starting...");
    });

    ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) {
        Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
    });

    ArduinoOTA.onEnd([]() {
        Serial.println("\nOTA complete. Rebooting...");
    });

    ArduinoOTA.onError([](ota_error_t error) {
        Serial.printf("OTA Error: %u\n", error);
    });

    ArduinoOTA.begin();
    Serial.printf("OTA ready at %s\n", WiFi.localIP().toString().c_str());
}

void loop() {
    ArduinoOTA.handle();  // Must call in loop
    // ... rest of your code
}
```

### GitHub Actions CI/CD

Create `.github/workflows/build.yml`:

```yaml
name: Build Firmware

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install PlatformIO
        run: pip install platformio

      - name: Build firmware
        run: pio run

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: firmware
          path: .pio/build/t-embed-cc1101/firmware.bin

      - name: Create Release
        if: startsWith(github.ref, 'refs/tags/v')
        uses: softprops/action-gh-release@v1
        with:
          files: .pio/build/t-embed-cc1101/firmware.bin
          generate_release_notes: true
```

{: .tip }
> Tag a commit with `git tag v1.0.0 && git push --tags` to automatically trigger a GitHub Release with your compiled firmware binary attached. Users can then flash it directly without setting up a development environment.

---

**Previous chapter:** [09 -- GPIO & Hardware Hacking](09-gpio-hardware.md) -- pin mapping, expansion modules, and hardware communication protocols.
