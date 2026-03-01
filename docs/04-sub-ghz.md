---
layout: default
title: "04 — Sub-GHz RF Operations"
nav_order: 5
---

# 04 — Sub-GHz RF Operations
{: .no_toc }

The CC1101 transceiver is the beating heart of the T-Embed CC1101 Plus. This chapter is an exhaustive reference covering every aspect of sub-GHz radio frequency operations --- from foundational theory through advanced protocol reverse engineering. Master this chapter and you will understand not just *how* to use the device, but *why* every setting, every waveform, and every protocol behaves the way it does.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## RF Fundamentals for Sub-GHz

### What Is Sub-GHz and Why It Matters

"Sub-GHz" refers to radio frequencies below 1 GHz --- typically the ISM (Industrial, Scientific, and Medical) bands at **315 MHz**, **433.92 MHz**, **868 MHz**, and **915 MHz**. These frequencies are the backbone of an enormous ecosystem of wireless devices: garage door openers, car key fobs, weather stations, smart home sensors, utility meters, and countless other devices that surround us every day.

Sub-GHz matters for security research because:

- **Ubiquity**: Billions of devices operate in these bands worldwide
- **Simplicity**: Many sub-GHz protocols are unencrypted and trivially decoded
- **Range**: Sub-GHz signals penetrate walls and travel much farther than WiFi or Bluetooth
- **Legacy**: Many protocols were designed decades ago with no security considerations
- **Persistence**: Devices using these frequencies remain in service for 10--20+ years

{: .note }
The CC1101 on the T-Embed CC1101 Plus covers **300--348 MHz**, **387--464 MHz**, and **779--928 MHz**. This spans all major sub-GHz ISM bands used globally.

### Frequency vs Wavelength vs Propagation

Radio waves travel at the speed of light. The relationship between frequency and wavelength is:

```
wavelength (m) = speed of light (m/s) / frequency (Hz)

At 433.92 MHz:  wavelength = 299,792,458 / 433,920,000 = 0.691 m (69.1 cm)
At 315 MHz:     wavelength = 299,792,458 / 315,000,000 = 0.952 m (95.2 cm)
At 868 MHz:     wavelength = 299,792,458 / 868,000,000 = 0.345 m (34.5 cm)
At 915 MHz:     wavelength = 299,792,458 / 915,000,000 = 0.328 m (32.8 cm)
```

| Frequency | Wavelength | Quarter-Wave Antenna | Typical Use Region |
|:----------|:-----------|:---------------------|:-------------------|
| 300 MHz | 100.0 cm | 25.0 cm | Military, some legacy devices |
| 315 MHz | 95.2 cm | 23.8 cm | Americas (key fobs, sensors) |
| 390 MHz | 76.9 cm | 19.2 cm | Americas (garage doors) |
| 433.92 MHz | 69.1 cm | 17.3 cm | Worldwide ISM band |
| 868 MHz | 34.5 cm | 8.6 cm | Europe ISM band |
| 915 MHz | 32.8 cm | 8.2 cm | Americas ISM band |

### Why Sub-GHz Travels Farther Than WiFi/Bluetooth

Lower frequencies have fundamentally better propagation characteristics:

1. **Free-Space Path Loss (FSPL)** increases with frequency. At the same distance, a 433 MHz signal loses ~20 dB less than a 2.4 GHz WiFi signal --- that is a 100x difference in power.

```
FSPL (dB) = 20 * log10(d) + 20 * log10(f) + 20 * log10(4pi/c)

At 100 meters:
  433 MHz:  FSPL = 53.2 dB
  868 MHz:  FSPL = 59.2 dB
  2.4 GHz:  FSPL = 68.0 dB
  5.8 GHz:  FSPL = 75.7 dB
```

2. **Diffraction**: Lower frequencies bend around obstacles more effectively. A 433 MHz signal diffracts around building corners and through foliage far better than 2.4 GHz or 5 GHz signals.

3. **Penetration**: Sub-GHz signals pass through walls, floors, and other building materials with significantly less attenuation. A single interior wall may attenuate a 2.4 GHz signal by 3--6 dB but only 1--2 dB at 433 MHz.

4. **Fresnel Zone**: The required clearance around the signal path is smaller at higher frequencies, but the wider Fresnel zone at sub-GHz frequencies actually helps in cluttered environments by allowing more multipath energy to arrive constructively.

### Line-of-Sight vs Non-Line-of-Sight

| Condition | 433 MHz Range | 2.4 GHz Range |
|:----------|:-------------|:---------------|
| Open field (LOS) | 500 m -- 2 km | 100 -- 300 m |
| Suburban (NLOS) | 200 -- 800 m | 50 -- 150 m |
| Indoor (through walls) | 50 -- 200 m | 20 -- 50 m |
| Dense urban | 100 -- 400 m | 30 -- 80 m |

{: .tip }
These ranges assume typical transmit power levels (10 dBm for sub-GHz devices, 20 dBm for WiFi). The T-Embed CC1101 Plus transmits at up to +12 dBm (about 16 mW) on its internal antenna.

### Antenna Polarization and Orientation Effects

The CC1101's onboard antenna and most sub-GHz device antennas are **linearly polarized**. This means orientation matters:

- **Matched polarization** (both antennas oriented the same way): Maximum signal transfer
- **Cross-polarized** (antennas at 90 degrees): Up to 20 dB signal loss --- signals may be undetectable
- **Circular polarization** (used by some commercial systems): 3 dB loss with any linear antenna but avoids orientation problems

**Practical rules:**

1. Hold the T-Embed in the same orientation as the target device's antenna when possible
2. If unsure, slowly rotate the T-Embed while monitoring RSSI
3. The T-Embed's antenna is oriented along the PCB --- holding the device vertically means a vertical antenna

{: .warning }
Antenna orientation mismatch is one of the most common reasons for failed captures and weak replays. If you are getting poor results, try rotating the device 90 degrees before troubleshooting anything else.

---

## Modulation Types Explained

The CC1101 supports several modulation types. Understanding these is critical for both capturing and transmitting signals. Every wireless protocol uses one of these to encode data onto a radio carrier wave.

### OOK (On-Off Keying)

**The simplest modulation.** The carrier is either ON (transmitting) or OFF (silent). A binary "1" is represented by the presence of the carrier, and a binary "0" by its absence.

```
        OOK Modulation (On-Off Keying)
        ================================

 Data:    1     0     1     1     0     0     1     0

         ___         ___   ___                ___
        |   |       |   | |   |              |   |
        |   |       |   | |   |              |   |
 RF:    |   |       |   | |   |              |   |
        |   |       |   | |   |              |   |
 _______|   |_______|   | |   |______________|   |________

 Carrier: ON  OFF   ON    ON    OFF   OFF    ON   OFF
```

**Characteristics:**
- Simplest to implement --- very cheap transmitters and receivers
- No frequency accuracy needed (just ON or OFF)
- Susceptible to noise (any signal spike can look like a "1")
- Most common modulation for cheap consumer devices

**Devices using OOK:**
- Older garage door openers
- Wireless doorbells
- Weather stations (most)
- Power outlet remotes
- Ceiling fan remotes
- Door/window sensors
- Car alarm remotes (older)

{: .tip }
When in doubt about what modulation a device uses, try OOK/AM first. The majority of simple sub-GHz consumer devices use OOK at 433.92 MHz.

### ASK (Amplitude Shift Keying)

ASK encodes data by varying the **amplitude** (power level) of the carrier. OOK is technically a special case of ASK where the amplitude switches between zero and full power.

```
        ASK Modulation (Amplitude Shift Keying)
        ==========================================

 Data:     1       0       1       1       0       1

          ~~~~    ~       ~~~~    ~~~~    ~       ~~~~
         ~~~~~~   ~~     ~~~~~~  ~~~~~~   ~~     ~~~~~~
 RF:    ~~~~~~~~  ~~~   ~~~~~~~~~~~~~~~~  ~~~   ~~~~~~~~
        ~~~~~~~~  ~~~   ~~~~~~~~~~~~~~~~  ~~~   ~~~~~~~~
        ~~~~~~~~  ~~~   ~~~~~~~~^^^^^^^^  ~~~   ~~~~~~~~

        High     Low    High    High     Low    High
        Amp      Amp    Amp     Amp      Amp    Amp
```

**Characteristics:**
- More general form of OOK
- Can have multiple amplitude levels (M-ASK)
- Slightly better noise resistance than pure OOK when using intermediate levels
- Rarely used in its multi-level form in sub-GHz consumer devices

{: .note }
In Bruce firmware and most sub-GHz tools, "AM" (Amplitude Modulation) refers to ASK/OOK modes collectively. When you select "AM" for capture or replay, the tool is using an OOK/ASK demodulator.

### 2-FSK (Frequency Shift Keying)

Instead of turning the carrier on and off, FSK encodes data by **shifting the frequency**. A binary "1" transmits at one frequency, and a binary "0" at a slightly different frequency.

```
        2-FSK Modulation (Frequency Shift Keying)
        ============================================

 Data:    1       0       1       1       0       0       1

         ^^^^    vvvv    ^^^^    ^^^^    vvvv    vvvv    ^^^^
        ^ ^ ^  v    v  ^ ^ ^  ^ ^ ^  v    v  v    v  ^ ^ ^
 RF:   ^   ^ ^v      v^   ^ ^^   ^ ^v      vv      v^   ^ ^
       ^   ^  v      v ^  ^  ^   ^  v      v v      v^   ^
        ^ ^    v    v   ^ ^   ^ ^    v    v   v    v   ^ ^
         v      vvvv     v     v      vvvv     vvvv     v

       f_high  f_low   f_high f_high f_low   f_low   f_high

       f_high = carrier + deviation
       f_low  = carrier - deviation
```

**Characteristics:**
- More robust against noise and amplitude variations
- Constant-envelope signal (transmitter can be more efficient)
- Requires more precise frequency control
- Used by more sophisticated protocols

**Devices using 2-FSK:**
- TPMS (Tire Pressure Monitoring Systems)
- Some smart home sensors
- LoRa (uses a variant --- CSS)
- Utility meters (AMR/AMI)
- Many modern IoT devices

### 4-FSK (4-Level Frequency Shift Keying)

Uses four distinct frequencies to encode two bits per symbol, doubling the data rate compared to 2-FSK at the same symbol rate.

```
        4-FSK Modulation (4-Level FSK)
        ================================

 Data:    11      01      10      00      11      10

         ^^^^                                    ^^^^
        ^    ^           ----                   ^    ^
 RF:   ^      ^  ----  -    -          ^^^^    ^      ^
              ^ -    - ^      ^       ^    ^  ^
               -      -        ----  ^      ^ ^
                                    ^        -

       f+3d   f+d    f-d    f-3d    f+3d    f-d

       Symbol:  11=f+3d  01=f+d  10=f-d  00=f-3d
       (d = frequency deviation)
```

**Characteristics:**
- Higher data rate for the same bandwidth
- More complex demodulation
- Used in some advanced protocols
- Less common in consumer sub-GHz devices

{: .note }
The CC1101 natively supports 4-FSK. While uncommon, some industrial protocols use it for higher throughput in narrow channels.

### GFSK (Gaussian Frequency Shift Keying)

GFSK is FSK with a **Gaussian filter** applied to the data before modulation. This smooths the frequency transitions, reducing spectral splatter (out-of-band emissions) and making the signal occupy less bandwidth.

```
        FSK vs GFSK Frequency Transitions
        ====================================

        Standard 2-FSK:          GFSK:
        (abrupt transitions)     (smooth transitions)

 freq   ____                     ____
 high  |    |         freq      /    \
       |    |         high    /      \
       |    |                /        \
 ______|    |______   ______/          \______

       |<-->|                 |<------>|
       Sharp                  Gaussian
       edge                   smoothed
```

**Characteristics:**
- Reduced bandwidth compared to plain FSK
- Better spectral efficiency
- Slightly more complex to implement
- Used by Bluetooth (at 2.4 GHz) and many modern sub-GHz protocols

**Devices using GFSK:**
- Some modern smart home hubs
- Industrial telemetry systems
- Medical devices (ISM band)
- Higher-end weather stations

### MSK (Minimum Shift Keying)

MSK is a special case of FSK where the frequency deviation is exactly one quarter of the bit rate. This produces the **minimum bandwidth** possible for an FSK signal while maintaining continuous phase.

```
        MSK Modulation (Minimum Shift Keying)
        ========================================

 Data:    1       0       1       1       0

 RF:     ___     ___     ___     ___     ___
        /   \   /   \   /   \   /   \   /   \
       / f+  \ / f-  \ / f+  \ / f+  \ / f-  \
      /       X       X       X       X       \
             / \     / \     / \     / \
            /   \   /   \   /   \   /   \
           /     \_/     \_/     \_/     \

        Phase is ALWAYS continuous (no jumps)
        Deviation = 1/(4 * bit_period)
```

**Characteristics:**
- Most spectrally efficient FSK variant
- Constant envelope (efficient power amplifiers)
- Continuous phase prevents spectral regrowth
- Used in some satellite and telemetry systems
- CC1101 supports MSK natively

### Modulation Comparison Summary

| Modulation | Noise Resistance | Bandwidth | Complexity | Common Use Cases |
|:-----------|:----------------|:----------|:-----------|:-----------------|
| OOK | Low | Moderate | Very Low | Cheap remotes, doorbells |
| ASK | Low--Medium | Moderate | Low | Similar to OOK devices |
| 2-FSK | Medium--High | Moderate | Medium | TPMS, meters, IoT |
| 4-FSK | Medium--High | Lower | Higher | Industrial protocols |
| GFSK | High | Lower | Higher | Modern IoT, medical |
| MSK | High | Lowest | Higher | Telemetry, specialized |

{: .tip }
In Bruce firmware, you typically only need to choose between **AM** (which covers OOK/ASK) and **FM** (which covers FSK/GFSK). The firmware handles the fine details. When in doubt, try AM first for simple consumer devices and FM for more modern or industrial devices.

---

## Encoding Schemes

Modulation determines *how* bits are carried on the radio wave. **Encoding** determines *how* the original data is transformed into those bits. Understanding encoding is essential for decoding captured signals.

### Manchester Encoding

Each bit is represented by a **transition** rather than a level. A "1" is a low-to-high transition, and a "0" is a high-to-low transition (IEEE convention; some systems reverse this).

```
        Manchester Encoding
        =====================

 Data:    1       0       1       1       0       0

         _____         _____   _____
        |     |       |     | |     |
        |     |       |     | |     |
 Man:   |     |       |     | |     |
        |     |_______|     | |     |_______________
              |       |     | |     |       |       |
              |       |     |_|     |       |       |
                                    |_______|_______|

        1 = rising edge at center of bit period
        0 = falling edge at center of bit period
```

**Key properties:**
- **Self-clocking**: The receiver can extract timing from the transitions themselves, no separate clock needed
- **DC balanced**: Equal number of high and low periods
- **Doubled bandwidth**: Requires twice the bandwidth of NRZ for the same data rate
- **Used by**: Many older remote controls, some building access systems, Oregon Scientific weather stations

### PWM (Pulse Width Modulation) Encoding

The bit value is encoded in the **width** of the pulse. A wide pulse represents one bit value, a narrow pulse represents the other.

```
        PWM Encoding
        ==============

 Data:    1         0       1       0         0

         _______   ___     _______   ___     ___
        |       | |   |   |       | |   |   |   |
        |       | |   |   |       | |   |   |   |
 PWM:   |       | |   |   |       | |   |   |   |
        |       | |   |   |       | |   |   |   |
        |       |_|   |___|       |_|   |___|   |___

        1 = long pulse (e.g., 1000 us)
        0 = short pulse (e.g., 350 us)
        Gap between pulses is constant
```

**Key properties:**
- Very common in cheap OOK transmitters
- Easy to decode --- just measure pulse widths
- PT2262 and EV1527 both use PWM encoding
- Pulse widths vary by protocol (typical: 350 us short, 1050 us long)

### NRZ (Non-Return-to-Zero)

The simplest encoding: a "1" is a high level, a "0" is a low level. No transitions are guaranteed, so clock recovery requires a separate mechanism (preamble, sync word, or known data rate).

```
        NRZ Encoding
        ==============

 Data:    1     0     1     1     0     0     1     0

         ___         ___   ___                ___
        |   |       |   | |   |              |   |
        |   |       |   | |   |              |   |
 NRZ:   |   |       |   | |   |              |   |
        |   |_______|   | |   |______________|   |______
                    |   |_|                  |   |

        Level directly represents bit value
        No guaranteed transitions for clock recovery
```

**Key properties:**
- Maximum bandwidth efficiency
- Requires accurate clock synchronization
- Used with preamble + sync word in most FSK protocols
- Common in CC1101 packet mode

### Princeton PT2262/PT2272

The **PT2262** encoder IC is one of the most ubiquitous chips in cheap 433 MHz devices. Understanding its protocol is essential for sub-GHz work.

```
        PT2262 Protocol Format
        ========================

        |<-- Address (A0-A7) -->|<-- Data (D0-D3) -->|  Sync  |
        | 8 address tri-state   | 4 data bits        |  Bit   |
        | bits (0, 1, or float) | (0 or 1)           |        |

        Bit encoding (PWM):

        "0" bit:    ___     ___
                   |   |   |   |             (1 short + 1 short)
                   | 1T|   | 1T|
                ___|   |___|   |___
                   | 1T    | 3T  |

                   1T high, 3T low, 1T high, 3T low

        "1" bit:    _______   _______
                   |       | |       |       (1 long + 1 long)
                   |  3T   | |  3T   |
                ___|       |_|       |___
                   |  3T     |  1T  |

                   3T high, 1T low, 3T high, 1T low

        "F" (float):___     _______
                   |   |   |       |         (1 short + 1 long)
                   | 1T|   |  3T   |
                ___|   |___|       |___
                   | 1T    |  1T  |

                   1T high, 3T low, 3T high, 1T low

        Sync bit:   ___
                   |   |                     (1 short + 31 long)
                   | 1T|
                ___|   |_______...______
                   | 1T      31T       |

        T = basic time unit (typically ~350 us for PT2262)
        Complete frame = 12 tri-state bits + 1 sync bit
        Total = 25 pulse pairs per frame
        Frame is repeated 4+ times
```

**Example transmission (doorbell with address 01F1F0FF, data 1000):**

```
Address:  0    1    F    1    F    0    F    F
Data:     1    0    0    0
Sync:     S

Tri-state values: 0, 1, F, 1, F, 0, F, F, 1, 0, 0, 0, S
```

{: .note }
"Float" (F) is a third state unique to PT2262. Many clones treat it as a distinct value. When setting DIP switches on a device, the middle/disconnected position creates a float state. This effectively gives each address position three values (0, 1, F), making the address space 3^8 = 6561 possible addresses.

### EV1527/HS1527

A more modern variant that replaced PT2262 in many devices. Uses a 20-bit address and 4-bit data field, with simpler two-state encoding (no float).

```
        EV1527 Protocol Format
        ========================

        |<------ Address (20 bits) ------>|<- Data (4 bits) ->|  Sync  |
        | Fixed at manufacturing          | Button code       |  Bit   |
        | (cannot be changed by user)     |                   |        |

        Bit encoding:

        "0" bit:    ___               "1" bit:    _________
                   |   |                         |         |
                   | 1T|                         |   3T    |
                ___|   |_________             ___|         |___
                       |   3T   |                         | 1T|

        Sync:    ___
                |   |
                | 1T|
             ___|   |__________________________
                    |          31T             |

        T = ~350 us typically
        20-bit address = 1,048,576 unique codes
        Frame repeated 4+ times
```

{: .tip }
EV1527 devices have their address burned in at the factory. Unlike PT2262 (where you set DIP switches), you cannot change the address. To "pair" an EV1527 remote to a receiver, you put the receiver in learning mode and press the remote button.

### HCS301 (KeeLoq) Rolling Code

The **HCS301** (by Microchip) implements the **KeeLoq** rolling code algorithm. This is the most common rolling code system in garage door openers, car alarms, and gate remotes.

```
        HCS301 KeeLoq Transmission Format
        =====================================

        |<--- Preamble --->|<--------- Encrypted (32 bits) --------->|
        |  12+ "1" bits    | Rolling code portion (changes each TX)  |
        |                  | OVR(2) | BTN(4) | DISC(10) | CNT(16)   |
        |                  |                                         |
        |                  |<---------- Fixed (34 bits) ------------>|
        |                  | Serial Number (28) | BTN(4) | STS(2)   |
        |                  |                                         |
        |                  |<---- Guard (1 bit) --->|                |

        Total transmission: ~67 bits + preamble

        Rolling Code Mechanism:
        =======================

        TX #1:  Encrypted counter = 00001  -->  [encrypted] = A7F2...
        TX #2:  Encrypted counter = 00002  -->  [encrypted] = 3B91...
        TX #3:  Encrypted counter = 00003  -->  [encrypted] = D4E8...
        TX #4:  Encrypted counter = 00004  -->  [encrypted] = 6C0A...

        Each transmission uses the NEXT counter value.
        The receiver tracks the counter and only accepts values
        within a forward window (typically 16 ahead).

        Recording TX #3 and replaying it later FAILS because
        the receiver has already moved past counter 00003.
```

**Why replay attacks fail:**

1. The counter increments with every button press
2. The receiver stores the last accepted counter value
3. The receiver only accepts counter values *ahead* of the stored value (within a window)
4. A replayed old counter is always *behind* the stored value and is rejected
5. The encryption key is unique to each transmitter-receiver pair

{: .danger }
**KeeLoq is NOT unbreakable.** Cryptographic weaknesses have been published (slide attacks, algebraic attacks). However, exploiting these requires specialized equipment, captured data, and significant computational resources. The T-Embed CC1101 Plus cannot break KeeLoq encryption. Simple replay attacks will **never** work against KeeLoq.

### Nice FLO/FLOR

**Nice FLO** (fixed code) and **Nice FLOR** (rolling code) are common in European gate and barrier systems.

```
        Nice FLO (Fixed Code) Format
        ===============================

        Sync: 36T low

        |<-- Sync -->|<-- Button (4 bits) -->|<-- Serial (12 bits) -->|
        | 36T low    | Encoded button ID     | Unique serial number  |
        | 1T high    |                       |                       |

        Nice FLO encoding: Each bit is PWM
        "0" = 1T high, 3T low
        "1" = 3T high, 1T low
        T = ~700 us

        Nice FLOR (Rolling Code):
        Similar to KeeLoq - uses encrypted rolling counter
        NOT vulnerable to simple replay
```

{: .note }
Nice FLO (without the R) uses fixed codes and IS vulnerable to replay attacks. Nice FLOR uses rolling codes and is NOT vulnerable. Check the receiver model to determine which protocol is in use.

### CAME

CAME remotes are widely used for gates and barriers, primarily in Europe.

```
        CAME Protocol Format
        ======================

        |<-- Start -->|<------- Code (12 bits) ------->|
        | 1 start bit | Address/command data           |
        | (short high)| (PWM encoded)                  |

        Bit encoding:
        "0" = 320us high + 640us low
        "1" = 640us high + 320us low

        Total code: 12 bits = 4096 combinations
        Frequency: 433.92 MHz
        Modulation: OOK
```

{: .warning }
CAME fixed-code remotes have only 4096 possible codes (12 bits). A brute-force attack cycling through all codes takes only a few minutes. CAME Atomo and newer systems use rolling codes. Always verify which version you are testing.

### Linear (Garage Door Openers)

Linear brand garage door openers are common in the Americas.

```
        Linear Protocol Format
        ========================

        Encoding: Binary, 10-bit code
        DIP switches on remote set the code
        Frequency: 300-310 MHz (common) or 315 MHz
        Modulation: OOK

        |<--- 10-bit code (DIP switches) --->|<-- Sync -->|
        | Each bit = 1 DIP switch position   | Gap        |

        "0" = short pulse
        "1" = long pulse
        Total combinations: 1024 (10 bits)
```

### Chamberlain/LiftMaster Security+ and Security+ 2.0

```
        Security+ (1.0):
        - 40-bit rolling code
        - Trinary encoding (three states per position)
        - Fixed portion + rolling portion
        - Frequency: 315 MHz (Americas)
        - CANNOT be replayed

        Security+ 2.0:
        - 40-bit rolling code with enhanced encryption
        - Dual-frequency operation (315 MHz + 390 MHz)
        - IntelliCode technology
        - Even stronger replay protection
        - Frequency hopping between transmissions
        - CANNOT be replayed
```

{: .danger }
Chamberlain/LiftMaster Security+ and Security+ 2.0 are among the most secure consumer rolling code implementations. Do not waste time attempting replay attacks against these systems. They use proprietary encryption and frequency-hopping techniques that the CC1101 cannot defeat.

---

## Common Sub-GHz Devices and Their Frequencies

This comprehensive reference table covers the most common sub-GHz devices you will encounter. Use it to determine the likely frequency, protocol, and vulnerability of a target device.

### Americas (FCC Region)

| Device Type | Frequency | Protocol | Modulation | Security | Replay Vulnerable? |
|:------------|:----------|:---------|:-----------|:---------|:-------------------|
| Garage door (pre-1993) | 300--390 MHz | Fixed code | OOK | None | **YES** |
| Garage door (1993--2005) | 315/390 MHz | KeeLoq | OOK | Rolling code | NO |
| Garage door (2005+) | 315/390 MHz | Security+ 2.0 | OOK | Rolling + encrypted | NO |
| Car key fob (pre-2000) | 315 MHz | Fixed code | OOK | None/weak | **YES** |
| Car key fob (modern) | 315 MHz | AUT64/KeeLoq/AES | OOK/FSK | Rolling + encrypted | NO |
| Wireless doorbell | 315/433 MHz | PT2262/EV1527 | OOK | None | **YES** |
| Weather station | 433.92 MHz | Various (Acurite, OS) | OOK/FSK | None | N/A (RX only) |
| TPMS sensors | 315 MHz | Proprietary | FSK | None | N/A (monitoring) |
| Wireless thermometer | 433.92 MHz | Various | OOK | None | N/A (RX only) |
| Gate/barrier remote (fixed) | 433.92 MHz | PT2262/EV1527 | OOK | None | **YES** |
| Gate/barrier remote (rolling) | 433.92 MHz | KeeLoq/proprietary | OOK | Rolling code | NO |
| Ceiling fan remote | 303/315 MHz | Fixed DIP code | OOK | None | **YES** |
| Power outlet remote | 315/433 MHz | PT2262/EV1527 | OOK | None | **YES** |
| Pet collar/invisible fence | 433 MHz | Proprietary | OOK | None | **YES** |
| Baby monitor (analog) | 49/433 MHz | FM analog | FM | None | N/A (analog audio) |
| AMR utility meters | 900--920 MHz | Various (ERT, ITRON) | FSK | Varies | N/A (monitoring) |
| LoRa devices | 915 MHz | LoRaWAN | CSS (chirp) | AES-128 | NO |
| Smoke/CO detectors | 315/433 MHz | Various | OOK/FSK | Varies | Varies |
| Smart home sensors (Z-Wave) | 908.42 MHz | Z-Wave | FSK | S2 (AES-128) | NO |
| Pager systems | 929--932 MHz | POCSAG/FLEX | FSK | None | N/A (monitoring) |
| Railroad crossing signals | 452 MHz | Proprietary | Various | Yes | N/A (do not interfere) |
| Keyless entry (commercial) | 315/433 MHz | Various | OOK/FSK | Varies | Varies |

### Europe (ETSI Region)

| Device Type | Frequency | Protocol | Modulation | Security | Replay Vulnerable? |
|:------------|:----------|:---------|:-----------|:---------|:-------------------|
| Car key fob | 433.92 MHz | KeeLoq/Hitag/AES | OOK/FSK | Rolling + encrypted | NO |
| Wireless doorbell | 433.92 MHz | PT2262/EV1527 | OOK | None | **YES** |
| Weather station | 433.92 MHz | Various | OOK/FSK | None | N/A (RX only) |
| TPMS sensors | 433.92 MHz | Proprietary | FSK | None | N/A (monitoring) |
| Gate/barrier (Nice FLO) | 433.92 MHz | Nice FLO | OOK | None (fixed) | **YES** |
| Gate/barrier (Nice FLOR) | 433.92 MHz | Nice FLOR | OOK | Rolling code | NO |
| Gate/barrier (CAME) | 433.92 MHz | CAME fixed | OOK | None | **YES** |
| Gate/barrier (CAME Atomo) | 433.92 MHz | CAME rolling | OOK | Rolling code | NO |
| Power outlet remote | 433.92 MHz | PT2262/EV1527 | OOK | None | **YES** |
| Smart home sensors | 433/868 MHz | Various | OOK/FSK | Varies | Varies |
| Smoke detectors | 433/868 MHz | Various | OOK/FSK | Varies | Varies |
| LoRa devices | 868 MHz | LoRaWAN | CSS (chirp) | AES-128 | NO |
| AMR utility meters | 868 MHz | wM-Bus | FSK | AES-128 | NO |
| Z-Wave devices | 868.42 MHz | Z-Wave | FSK | S2 (AES-128) | NO |
| Industrial telemetry | 868 MHz | Various | FSK/GFSK | Varies | Varies |

{: .warning }
This table is a general reference. Manufacturers may change protocols between product generations. A device sold as "rolling code" in marketing materials may actually use fixed codes, or vice versa. **Always verify by capturing and analyzing the actual signals** rather than relying on assumptions.

---

## Frequency Analysis

The frequency analyzer is your first tool when investigating an unknown device. It sweeps the spectrum and shows you where energy is present, helping you identify the exact operating frequency of any sub-GHz transmitter.

### Step-by-Step Frequency Analysis

**Step 1: Navigate to the Frequency Analyzer**

On Bruce firmware:
```
Main Menu → Sub-GHz → Frequency Analyzer
```

**Step 2: Understanding the Display**

The frequency analyzer screen shows:

| Element | Description |
|:--------|:------------|
| **Current Frequency** | The frequency where the strongest signal is detected |
| **RSSI** | Received Signal Strength Indicator in dBm (typical: -90 dBm weak to -30 dBm strong) |
| **Signal Bar/Graph** | Visual representation of signal strength |
| **Threshold Line** | Minimum signal level to register a detection |
| **Peak Frequency** | Frequency of the strongest signal seen during the scan |

**Step 3: Performing the Analysis**

1. **Position the T-Embed** within 1--3 meters of the target device
2. **Start the analyzer** --- it begins sweeping across its frequency range
3. **Trigger the target device** (press the button on the remote, ring the doorbell, etc.)
4. **Observe the spike** on the display --- the frequency where the signal peaks is your target
5. **Note the frequency** precisely (e.g., 433.920 MHz, not just "433 MHz")
6. **Trigger multiple times** to confirm --- the same frequency should spike each time

**Step 4: Interpreting Results**

| RSSI Reading | Signal Quality | Typical Scenario |
|:-------------|:---------------|:-----------------|
| -30 to -40 dBm | Excellent | Device within 1 meter |
| -40 to -60 dBm | Good | Device within 5 meters |
| -60 to -75 dBm | Fair | Device within 20 meters |
| -75 to -85 dBm | Weak | Device far away or obstructed |
| -85 to -95 dBm | Very weak | At the edge of detection |
| Below -95 dBm | Noise floor | No usable signal |

{: .tip }
If you see a signal but it is weak (below -75 dBm), move closer to the target device. You need a clean capture for reliable analysis and replay. Ideally, capture signals at -60 dBm or stronger.

### Common Frequencies to Expect

In a typical home or office environment, you will likely detect signals at:

| Frequency | Likely Source |
|:----------|:-------------|
| 315.000 MHz | Garage door opener, car key fob (Americas) |
| 390.000 MHz | Older garage doors (Americas) |
| 433.920 MHz | General ISM band: doorbells, weather stations, remotes |
| 868.000--868.600 MHz | European ISM band: smart home, LoRa |
| 915.000 MHz | Americas ISM band: LoRa, utility meters |

{: .note }
The frequency analyzer is a **passive** operation --- it only receives. It does not transmit anything and is completely legal to use everywhere. Think of it as a radio scanner.

---

## Signal Capture (Recording)

Once you have identified the frequency of your target device, the next step is to **capture** (record) its signal for analysis or replay.

### Basic Capture

**Navigate to the capture function:**
```
Main Menu → Sub-GHz → Record RAW
```

**Configure parameters before capture:**

| Parameter | Description | How to Choose |
|:----------|:------------|:--------------|
| **Frequency** | Carrier frequency in MHz | Use the value from frequency analysis (e.g., 433.92) |
| **Modulation** | AM (OOK/ASK) or FM (FSK) | AM for simple remotes, FM for modern IoT. Try AM first. |
| **Bandwidth** | Receive filter bandwidth | Wider = more noise but catches off-frequency signals. Start at 270 kHz. |
| **Data Rate** | Bits per second | If unknown, leave at default. Common: 2400--10000 bps for OOK. |

**Capture procedure:**

1. Set the frequency to the value identified by the frequency analyzer
2. Select modulation type (AM for most consumer devices)
3. Press the capture/record button on the T-Embed
4. Within the first few seconds, trigger the target device (press the remote button)
5. Wait for the full transmission (most devices transmit for 1--3 seconds)
6. Stop the capture
7. Save the file

{: .warning }
**Timing is critical.** Start recording *before* you trigger the target device. If you start recording too late, you will miss the beginning of the transmission, which often contains critical sync and preamble data.

### RAW Signal Capture

RAW capture records the signal as a series of **timing durations** --- alternating positive (signal present) and negative (signal absent) values measured in microseconds.

**When to use RAW capture:**
- Unknown protocol
- Complex or multi-part signals
- When decoded capture fails or produces garbage
- When you want the most faithful representation of the signal

**RAW data format example:**

```
RAW_Data: 356 -1072 356 -1072 1072 -356 1072 -356 356 -1072
          1072 -356 356 -1072 356 -1072 356 -1072 1072 -356
          356 -1072 356 -10692
```

How to read this:
- **Positive values** (e.g., `356`): Signal present (carrier ON) for that many microseconds
- **Negative values** (e.g., `-1072`): Signal absent (carrier OFF) for that many microseconds
- The pattern alternates: ON, OFF, ON, OFF...
- Large negative values (e.g., `-10692`) indicate the gap between repeated frames

**Decoding the example above:**

```
356 us ON,  1072 us OFF  →  Short pulse  →  "0" (PT2262 encoding)
356 us ON,  1072 us OFF  →  Short pulse  →  (second half of "0" bit)
1072 us ON, 356 us OFF   →  Long pulse   →  "1" (PT2262 encoding)
1072 us ON, 356 us OFF   →  Long pulse   →  (second half of "1" bit)
...
356 us ON, 10692 us OFF  →  Sync pulse   →  End of frame
```

{: .tip }
RAW data is the "ground truth" of a captured signal. Even if you cannot decode it immediately, save the RAW data. You can always analyze it later with more powerful tools like Universal Radio Hacker on a computer.

### Multi-Capture Strategy

For proper analysis, **always capture multiple transmissions** from the same device:

1. **Capture 1**: Normal single button press
2. **Capture 2**: Another single button press (compare with Capture 1)
3. **Capture 3**: Third button press

**Why multiple captures matter:**

| Comparison Result | Meaning |
|:------------------|:--------|
| All captures identical | Fixed code protocol --- replay will work |
| Some bits change, some stay same | Rolling code --- the changing part is the counter/encrypted section |
| Completely different each time | Encrypted or challenge-response --- replay will NOT work |
| Mostly same with minor timing variations | Fixed code with noise --- captures are valid for replay |

{: .note }
Most fixed-code transmitters repeat their frame 4--8 times in a single button press. A good capture will contain multiple repetitions of the same frame, which helps confirm you have a clean recording.

---

## Signal Replay (Transmission)

Replay is the act of **retransmitting** a previously captured signal. This is one of the most powerful capabilities of the T-Embed CC1101 Plus, and also one that requires the most responsibility.

### Basic Replay

**Navigate to the replay function:**
```
Main Menu → Sub-GHz → Saved Signals → [select file] → Send/Replay
```

**Replay procedure:**

1. Navigate to your saved signals
2. Select the capture file you want to replay
3. Position the T-Embed within operational range of the target receiver
4. Press the transmit/replay button
5. The T-Embed will retransmit the exact captured signal

**Distance and power considerations:**

| Factor | Guidance |
|:-------|:---------|
| **Range** | CC1101 max output is +12 dBm. Typical effective range: 10--50 meters for most receivers. |
| **Orientation** | Match the antenna orientation of the original transmitter if possible |
| **Obstacles** | Walls and metal objects reduce range significantly |
| **Interference** | Other devices on the same frequency can prevent reception |
| **Receiver sensitivity** | Some receivers require stronger signals than others |

### When Replay Works

Replay attacks are effective against **fixed-code** systems:

- **Older garage door openers** (pre-1993, fixed DIP switch codes)
- **Wireless doorbells** (nearly all use fixed codes)
- **Power outlet remotes** (PT2262, EV1527 based)
- **Ceiling fan remotes** (fixed DIP switch codes)
- **Cheap alarm system remotes** (some older systems)
- **Toy remote controls**
- **Simple gate remotes** (Nice FLO, CAME fixed, older Linear)
- **Some restaurant pagers** (fixed code based)
- **Some older hotel key card systems** (RFID is separate --- sub-GHz room controls)

### When Replay Does NOT Work

| System | Why Replay Fails |
|:-------|:-----------------|
| **KeeLoq rolling code** | Counter has moved forward; old codes rejected |
| **Chamberlain Security+** | Encrypted rolling code with trinary encoding |
| **Chamberlain Security+ 2.0** | Encrypted rolling + frequency hopping |
| **Nice FLOR** | Rolling code variant |
| **CAME Atomo** | Rolling code variant |
| **Modern car key fobs** | AES-encrypted rolling codes |
| **LoRaWAN** | AES-128 encrypted with session keys |
| **Z-Wave S2** | AES-128 with ECDH key exchange |
| **Any challenge-response system** | Device must respond to a unique challenge |
| **Timestamped protocols** | Code includes current time; old timestamps rejected |

### Replay Attack Mitigations

Understanding **why** rolling codes defeat replay attacks:

```
        Rolling Code Operation
        ========================

        Transmitter                              Receiver
        ==========                               ========

        Counter: 1000                            Stored: 999
        Key: SECRET                              Key: SECRET

        [Button Press]
        Counter → 1001
        Encrypt(1001, SECRET) → 0xA7F2
        Transmit: 0xA7F2 + serial ──────────→   Receive: 0xA7F2
                                                  Decrypt → 1001
                                                  1001 > 999? YES ✓
                                                  Stored: 1001
                                                  [OPEN GATE]

        [Button Press]
        Counter → 1002
        Encrypt(1002, SECRET) → 0x3B91
        Transmit: 0x3B91 + serial ──────────→   Receive: 0x3B91
                                                  Decrypt → 1002
                                                  1002 > 1001? YES ✓
                                                  Stored: 1002
                                                  [OPEN GATE]

        [Attacker replays 0xA7F2]
        Replay: 0xA7F2 + serial ────────────→   Receive: 0xA7F2
                                                  Decrypt → 1001
                                                  1001 > 1002? NO ✗
                                                  [REJECTED]
```

**Advanced rolling code attacks (theoretical):**

**RollJam Attack:**
1. Attacker jams the receiver while capturing the first transmission
2. User presses button again (thinks first press failed)
3. Attacker jams again while capturing the second transmission
4. Attacker replays the first transmission (user's door opens)
5. Attacker now holds an unused second code for later use

{: .danger }
The RollJam attack requires **simultaneous jamming and receiving** on the same frequency, plus precise timing. While theoretically demonstrated by Samy Kamkar, it requires specialized hardware beyond the T-Embed CC1101 Plus. The CC1101 cannot simultaneously transmit (jam) and receive. This information is provided for educational understanding of security vulnerabilities, not as an attack guide.

**De Bruijn Sequence Attack:**
- For systems with short code spaces, a De Bruijn sequence transmits every possible code with minimal redundancy
- A 10-bit code space (1024 codes) requires only 1033 bits in a De Bruijn sequence instead of 10,240 bits for brute force
- Only relevant for very short fixed codes, not rolling codes

---

## Signal Analysis

Capturing a signal is only the beginning. **Analysis** is where you decode the protocol, understand the data, and gain actionable intelligence.

### Using Universal Radio Hacker (URH) with T-Embed Captures

[Universal Radio Hacker](https://github.com/jopohl/urh) is an open-source tool for wireless protocol investigation. It runs on Linux, macOS, and Windows.

**Exporting captures from T-Embed:**

1. Connect the T-Embed to your computer via USB
2. Navigate to the SD card or internal storage
3. Copy the `.sub` or `.raw` capture files to your computer
4. Open URH and import the file

**URH Workflow:**

```
Step 1: INTERPRETATION
========================
- Load the captured signal
- URH displays the raw waveform
- Select the modulation type (ASK, FSK, PSK)
- URH auto-detects in many cases
- Adjust the noise threshold to filter out background

Step 2: DEMODULATION
========================
- URH converts the analog waveform to digital bits
- Choose the bit length / data rate
- URH shows the demodulated bitstream
- Verify by checking that repeated frames are identical

Step 3: ANALYSIS
========================
- URH shows the bit patterns
- Identify preamble (usually alternating 10101010...)
- Find sync word (unique pattern marking start of data)
- Extract payload data
- Compare multiple captures to find fixed vs variable fields

Step 4: DECODING
========================
- Try different decodings: NRZ, Manchester, differential
- URH auto-detects encoding in many cases
- Apply the correct decoding to get the actual data bits

Step 5: PROTOCOL REVERSE ENGINEERING
======================================
- Assign meaning to bit fields
- Identify address, command, status, checksum fields
- Build a protocol description
- Generate custom payloads
```

### Manual Signal Analysis

When you do not have access to URH, you can analyze signals directly from RAW capture data.

**Step 1: Identify the bit timing**

Look at the RAW data and find the shortest recurring pulse:

```
RAW_Data: 350 -1050 350 -1050 1050 -350 350 -1050 350 -31500

Shortest pulse: ~350 us  →  This is the basic time unit "T"
Long pulse: ~1050 us     →  This is 3T
Very long gap: ~31500 us →  This is the sync/frame gap
```

**Step 2: Identify the encoding**

Compare pulse patterns to known encodings:

```
350 ON, 1050 OFF = 1T ON, 3T OFF  →  PT2262 "0" (half bit)
1050 ON, 350 OFF = 3T ON, 1T OFF  →  PT2262 "1" (half bit)

Reading in pairs (each PT2262 bit = 2 pulse pairs):
(350,-1050, 350,-1050)  = Short-Short  →  "0"
(1050,-350, 1050,-350)  = Long-Long    →  "1"
(350,-1050, 1050,-350)  = Short-Long   →  "F" (float)
```

**Step 3: Extract the payload**

```
Raw:  350 -1050  350 -1050  1050 -350  1050 -350  350 -1050  1050 -350 ...
Pairs:  (SS)         (LL)          (SL)        ...
Decode:  "0"          "1"           "F"         ...

Address: 01F1F0FF
Data:    1000
```

**Step 4: Verify with multiple captures**

If the decoded data matches across multiple captures, you have a fixed code.
If certain fields change, identify which fields are variable (counter, timestamp, encrypted section).

**Step 5: Calculate checksums**

Many protocols include a checksum or CRC at the end. Common types:

| Checksum | Method | Example |
|:---------|:-------|:--------|
| XOR | XOR all data bytes | `0x12 ^ 0x34 ^ 0x56 = 0x70` |
| Sum | Add all bytes modulo 256 | `(0x12 + 0x34 + 0x56) & 0xFF` |
| CRC-8 | Polynomial division (8-bit) | Various polynomials |
| CRC-16 | Polynomial division (16-bit) | CRC-16/CCITT common |
| Nibble sum | Sum of all 4-bit nibbles | Common in weather stations |

---

## Custom Transmission

Beyond capturing and replaying, the T-Embed CC1101 Plus can transmit **custom-crafted signals**. This is useful for building your own RF devices, testing receivers, and simulating protocols.

### Creating Custom Sub-GHz Payloads

**Setting up a custom transmission:**

```
Main Menu → Sub-GHz → Custom TX  (or via Bruce scripting)
```

**Parameters to configure:**

| Parameter | Description | Example Values |
|:----------|:------------|:---------------|
| Frequency | Carrier frequency | 433920000 (Hz) |
| Modulation | OOK, 2-FSK, GFSK, MSK | OOK for most simple devices |
| Data Rate | Bits per second | 2000--10000 for OOK |
| Deviation | FSK frequency offset | 47607 Hz (typical for FSK) |
| Payload | Data bytes to transmit | Hex: `AA 55 01 23 45` |
| Repeat | Number of transmissions | 5--10 for reliability |

### Building a Protocol From Scratch

To build a custom remote control system:

1. **Define your protocol:**
   ```
   Preamble:   0xAA 0xAA 0xAA 0xAA  (alternating bits for clock sync)
   Sync Word:  0x2D 0xD4             (unique pattern)
   Address:    0x01                   (device address)
   Command:    0x01 = ON, 0x02 = OFF (your command set)
   Checksum:   XOR of address + command
   ```

2. **Configure the CC1101** (via firmware settings):
   - Frequency: 433.92 MHz
   - Modulation: 2-FSK
   - Data rate: 9600 bps
   - Deviation: 47.6 kHz
   - Preamble: 4 bytes
   - Sync word: 0x2DD4

3. **Transmit the packet:**
   ```
   [Preamble][Sync][0x01][0x01][0x00]  →  Device 1, ON
   [Preamble][Sync][0x01][0x02][0x03]  →  Device 1, OFF
   ```

4. **Build a receiver** (another ESP32 with CC1101 or a simple OOK receiver module):
   - Configure to the same frequency and modulation
   - Listen for the sync word
   - Parse address and command bytes
   - Execute the commanded action

{: .tip }
This is an excellent learning project. Build a custom remote that controls an LED on another ESP32. You will gain deep understanding of RF protocols by designing one from scratch.

### Use Cases for Custom Transmission

- **IoT development**: Prototype wireless sensors and actuators
- **RF range testing**: Measure signal strength at various distances
- **Protocol simulation**: Simulate a weather station or sensor for testing
- **Receiver testing**: Verify that your security system properly rejects invalid codes
- **Education**: Demonstrate wireless communication concepts

---

## Spectrum Scanning

Spectrum scanning gives you a broad view of the RF environment, showing all active frequencies in a range.

### Performing a Spectrum Sweep

```
Main Menu → Sub-GHz → Scanner  (or Spectrum Analyzer if available)
```

**How it works:**

1. The CC1101 tunes to the start frequency
2. Measures RSSI (signal strength) at that frequency
3. Steps to the next frequency (step size depends on bandwidth setting)
4. Repeats across the entire range
5. Displays the results as a graph of frequency vs. signal strength

**Interpreting the spectrum:**

```
RSSI
(dBm)
 -30 |                              *
 -40 |                             ***
 -50 |                            *****
 -60 |     *                     *******          *
 -70 |    ***       *           *********        ***
 -80 |   *****     ***         ***********      *****
 -90 |__***********+***+++++++*************++++*******+++___
      300    350    400    433   450    500    868  900  928
                        Frequency (MHz)

      Peak at 433 MHz = Active device transmitting
      Peak at 868 MHz = European ISM band device
      Small peak at 350 MHz = Possible interference or device
```

### Building an RF Audit

Create a systematic map of your RF environment:

1. **Start a sweep** from 300 MHz to 928 MHz
2. **Document each peak** with:
   - Frequency
   - Approximate RSSI
   - Time of observation
   - Your location
3. **Identify sources** by correlating peaks with known devices
4. **Repeat at different times** to catch intermittent transmitters
5. **Map the results** in a spreadsheet or notebook

| Frequency | RSSI | Time | Source (identified) |
|:----------|:-----|:-----|:--------------------|
| 315.00 MHz | -45 dBm | 14:00 | Car key fob (pressed during scan) |
| 433.92 MHz | -55 dBm | 14:02 | Weather station (periodic TX) |
| 433.92 MHz | -40 dBm | 14:05 | Wireless doorbell (triggered) |
| 868.35 MHz | -60 dBm | 14:10 | Smart home sensor (periodic) |
| 915.00 MHz | -70 dBm | 14:15 | Neighbor's device (unknown) |

---

## Sub-GHz with External SDR Companion

The T-Embed CC1101 Plus is excellent for targeted capture and transmission, but its receive bandwidth is narrow. Pairing it with a **Software-Defined Radio (SDR)** unlocks wideband monitoring.

### Recommended SDR: RTL-SDR

The RTL-SDR v3 or v4 is an inexpensive (~$30) USB dongle that covers 24 MHz to 1.766 GHz with up to 3.2 MHz bandwidth. It is **receive-only** but provides a wideband view that the CC1101 cannot.

### Workflow: T-Embed TX + RTL-SDR RX

| Role | Device | Purpose |
|:-----|:-------|:--------|
| Transmitter | T-Embed CC1101 Plus | Send captured/custom signals |
| Wideband Receiver | RTL-SDR + GQRX/SDR# | Monitor the full spectrum during transmission |

**Setup:**

1. Connect RTL-SDR to your computer (Kali Linux or macOS)
2. Open GQRX (or SDR# on Windows)
3. Tune to the frequency of interest (e.g., 433.92 MHz)
4. Set bandwidth to 1--2 MHz to see the signal in context
5. Use the T-Embed to transmit
6. Observe the transmission on the waterfall display

**Why this is useful:**
- **Verify transmission quality**: See exactly what the T-Embed is sending
- **Check for spurious emissions**: Ensure clean transmission
- **Monitor receiver response**: Some receivers transmit an acknowledgment
- **Wideband capture**: Record signals the CC1101 might miss due to narrow bandwidth
- **Spectrogram analysis**: The waterfall display reveals modulation type visually

### GNU Radio for Advanced Processing

GNU Radio is a free signal processing framework for advanced analysis:

- **Create custom demodulators** for unknown protocols
- **Build flowgraphs** that process signals in real-time
- **Decode complex protocols** that simple tools cannot handle
- **Record and playback** IQ data at full bandwidth

{: .note }
GNU Radio has a steep learning curve but is the ultimate tool for RF protocol analysis. Start with GQRX for visualization, then graduate to GNU Radio when you need custom processing.

---

## Practical Tutorials

### Tutorial 1: Decode a Weather Station

Weather stations are perfect first targets: they transmit frequently, use simple protocols, and there are no legal or ethical concerns about receiving their data.

**Equipment needed:**
- T-Embed CC1101 Plus with Bruce firmware
- A weather station transmitting on 433.92 MHz (most do)

**Step 1: Identify the frequency**

```
Sub-GHz → Frequency Analyzer
```

Wait for the weather station to transmit (most send data every 30--60 seconds). You should see a spike at or near 433.92 MHz.

**Step 2: Capture the signal**

```
Sub-GHz → Record RAW
Frequency: 433.92 MHz
Modulation: AM (OOK)
```

Wait for a transmission. Weather stations typically transmit a burst of data lasting 100--500 ms, repeated 2--3 times with short gaps.

**Step 3: Analyze the RAW data**

Look for patterns in the pulse timing:

```
Example Acurite 06002M weather station RAW data:

Preamble: Four long pulses (~600 us ON, ~600 us OFF)
Sync:     One very long pulse (~600 us ON, ~2400 us OFF)
Data:     PWM encoded
          - "1" = ~400 us ON, ~200 us OFF
          - "0" = ~200 us ON, ~400 us OFF
```

**Step 4: Decode the data**

Common weather station protocols and their data fields:

**Acurite protocol (common in Americas):**

```
|Channel|ID (7 bits)|Battery|Temperature (11 bits)|Humidity (8 bits)|CRC|
|  2b   |   7b      |  1b   |      11b            |      8b         |8b |

Temperature: Raw value / 10 - 100 (in Fahrenheit)
             e.g., Raw = 1250 → (1250/10) - 100 = 25.0°F
Humidity:    Direct percentage (0-99)
Channel:     00=A, 01=B, 10=C
```

**Oregon Scientific v2.1 protocol:**

```
|Preamble|Sync|Sensor ID|Channel|Rolling Code|Flags|Temp|Humidity|CRC|

Manchester encoded
Temperature in BCD (Binary Coded Decimal)
Nibble order may be reversed
```

**LaCrosse TX protocol:**

```
|Preamble|Type|ID|Data|Checksum|

Type: 0x00 = Temperature, 0x0E = Humidity
Temperature: (Data / 10) - 50 (Celsius)
```

**Step 5: Continuous monitoring**

Set up the T-Embed to continuously capture on 433.92 MHz and decode incoming weather data. Some Bruce firmware builds include weather station decoders that do this automatically.

{: .tip }
The `rtl_433` project (github.com/merbanan/rtl_433) documents over 200 weather station and sensor protocols. Even though it is designed for RTL-SDR, the protocol documentation is invaluable for understanding signals captured with the T-Embed.

---

### Tutorial 2: Clone a Wireless Doorbell

{: .warning }
**Only perform this on your own doorbell.** Replaying signals to other people's devices without permission is illegal in most jurisdictions.

**Equipment needed:**
- T-Embed CC1101 Plus with Bruce firmware
- Your own wireless doorbell (fixed-code type)

**Step 1: Identify the frequency**

```
Sub-GHz → Frequency Analyzer
```

Press the doorbell button. Most doorbells operate at 315 MHz or 433.92 MHz. Note the exact frequency.

**Step 2: Capture the signal**

```
Sub-GHz → Record RAW
Frequency: [from Step 1]
Modulation: AM (OOK)
```

Press the doorbell button while recording. Capture at least 2--3 separate button presses.

**Step 3: Verify it is a fixed code**

Compare your captures. If the RAW data patterns are identical (accounting for minor timing jitter), the doorbell uses a fixed code.

```
Capture 1: 350 -1050 350 -1050 1050 -350 ...
Capture 2: 348 -1055 352 -1048 1045 -358 ...
                                    ↑
            Minor timing differences = normal jitter
            Pattern is the same = fixed code ✓
```

**Step 4: Decode the protocol**

Most doorbells use PT2262 or EV1527 encoding:

```
Measure pulse widths:
  Short: ~350 us
  Long:  ~1050 us (3x short)
  Sync gap: ~10000-12000 us

This matches PT2262 encoding with T ≈ 350 us

Decode the address and data bits:
  Address: [8 tri-state bits from the DIP switches or IC]
  Data:    [4 bits identifying the button/melody]
```

**Step 5: Replay**

```
Sub-GHz → Saved Signals → [your capture] → Send
```

Point the T-Embed at the doorbell receiver and transmit. The doorbell should ring.

**Step 6: Understand the implications**

Your doorbell has no security. Anyone with a $30 device can ring your doorbell (or silence it). If your doorbell controls anything beyond making noise (such as a smart home automation), consider upgrading to a system with encryption.

---

### Tutorial 3: Map Your RF Environment

Build a comprehensive picture of all sub-GHz activity in your environment.

**Step 1: Systematic frequency sweep**

Create a scanning schedule:

| Time | Duration | Location | Notes |
|:-----|:---------|:---------|:------|
| Morning | 15 min | Living room | Catch morning routines |
| Midday | 15 min | Garage | Catch car/door activity |
| Evening | 15 min | Kitchen | Catch dinner-time sensors |
| Night | 15 min | Bedroom | Catch security system activity |

**Step 2: Scan each ISM band**

For each session, sweep these ranges:

```
Scan 1: 300-350 MHz (garage doors, older devices)
Scan 2: 390-440 MHz (garage doors, 433 ISM band)
Scan 3: 860-870 MHz (European ISM, if applicable)
Scan 4: 900-930 MHz (US ISM, utility meters, LoRa)
```

**Step 3: Document findings**

For each detected signal, record:

```
Signal Report #1
================
Date/Time:    2026-03-01, 14:30
Frequency:    433.920 MHz
RSSI:         -52 dBm
Modulation:   OOK (AM)
Duration:     ~300 ms burst, repeats 3x
Interval:     Every 60 seconds
Source:        Acurite weather station (outdoor sensor)
Protocol:     Acurite 06002M
Security:     None (unencrypted)
Notes:        Transmits temperature + humidity + battery status
```

**Step 4: Build your RF map**

Organize all findings into a reference document. This gives you a baseline understanding of your RF environment, which is valuable for:

- Identifying unauthorized devices
- Troubleshooting RF interference
- Understanding your wireless attack surface
- Planning security improvements

{: .note }
This exercise is the sub-GHz equivalent of a WiFi site survey. Security professionals perform RF audits to identify vulnerabilities in an organization's wireless infrastructure.

---

### Tutorial 4: Build a Custom RF Remote

Create your own wireless remote control system using two ESP32 boards with CC1101 modules.

**Concept:**

```
[T-Embed CC1101 Plus]  ════433.92 MHz════>  [ESP32 + CC1101 Receiver]
     (Transmitter)                                (Receiver)
     Custom protocol                          Decodes and acts
     Button press → TX                        RX → Control LED/relay
```

**Step 1: Design the protocol**

```
Frequency:     433.92 MHz
Modulation:    OOK
Data Rate:     2400 bps
Encoding:      PWM (350 us short, 1050 us long)

Packet format:
[Preamble: 8 short pulses]
[Sync: 1 short pulse + 31T gap]
[Address: 16 bits (your unique ID)]
[Command: 8 bits]
[Checksum: 8-bit XOR of address + command bytes]

Commands:
  0x01 = LED ON
  0x02 = LED OFF
  0x03 = Toggle
  0x04 = Blink pattern 1
  0x05 = Blink pattern 2
```

**Step 2: Program the T-Embed to transmit**

Using Bruce firmware's custom transmission feature (or by modifying the firmware source), configure the T-Embed to send your protocol when a button is pressed.

**Step 3: Program the receiver ESP32**

```cpp
// Pseudo-code for receiver (Arduino framework)
#include <ELECHOUSE_CC1101_SRC_DRV.h>

void setup() {
    ELECHOUSE_cc1101.Init();
    ELECHOUSE_cc1101.setMHZ(433.92);
    ELECHOUSE_cc1101.SetRx();
    pinMode(LED_PIN, OUTPUT);
}

void loop() {
    if (ELECHOUSE_cc1101.CheckRxFifo(100)) {
        byte buffer[4];
        ELECHOUSE_cc1101.ReceiveData(buffer);

        uint16_t address = (buffer[0] << 8) | buffer[1];
        uint8_t command = buffer[2];
        uint8_t checksum = buffer[3];

        if (address == MY_ADDRESS &&
            checksum == (buffer[0] ^ buffer[1] ^ buffer[2])) {
            executeCommand(command);
        }
    }
}
```

**Step 4: Test and iterate**

- Verify transmission at close range (1 meter)
- Test at increasing distances
- Add error handling (retry logic, acknowledgments)
- Expand the command set

{: .tip }
This project teaches you more about RF protocols than any amount of reading. By building both the transmitter and receiver, you understand every layer of the communication stack.

---

### Tutorial 5: TPMS Monitoring

Tire Pressure Monitoring System (TPMS) sensors transmit pressure and temperature data wirelessly from each tire. These signals are unencrypted and can be received by the T-Embed.

**Background:**

TPMS has been mandatory in the US since 2007 and in the EU since 2014. Each sensor has a unique ID and transmits:
- Tire pressure (typically in kPa or PSI)
- Temperature (Celsius)
- Battery status
- Sensor ID (unique 28--32 bit identifier)

**Frequency:**
- Americas: 315 MHz
- Europe/Asia: 433.92 MHz

**Modulation:** FSK (most modern sensors) or OOK (some older sensors)

**Step 1: Set up to receive**

```
Sub-GHz → Record RAW
Frequency: 315.000 MHz (Americas) or 433.920 MHz (Europe)
Modulation: FM (FSK)
```

**Step 2: Trigger a transmission**

TPMS sensors transmit:
- Periodically (every 30--60 seconds while driving)
- On pressure change (rapid deflation triggers immediate TX)
- When requested by the car's receiver (some systems)

To capture at rest, you may need to wait or drive the vehicle past the T-Embed.

**Step 3: Decode the data**

Common TPMS protocols:

```
Typical TPMS packet:
[Preamble][Sync][Sensor ID (32 bits)][Pressure (8 bits)][Temp (8 bits)][Flags][CRC]

Pressure:  Raw value * 2.5 = kPa  (or Raw * 0.363 = PSI)
Temperature: Raw value - 50 = Celsius
Flags:     Battery low, rapid deflation, sensor fault
```

**Step 4: Identify vehicles**

Since each TPMS sensor has a unique ID, you can:
- Identify your own vehicle's sensors
- Track which sensor belongs to which tire position
- Monitor pressure trends over time

{: .warning }
TPMS data is broadcast in the clear, which means it is a **privacy concern**. An adversary could track a vehicle by its unique TPMS sensor IDs. This is a known vulnerability documented in automotive security research. Use this knowledge defensively to understand your own vehicle's wireless footprint.

---

## CC1101 Register Configuration

For advanced users who want to understand or customize the CC1101's behavior at the register level.

### How the CC1101 Works

The CC1101 is controlled via **SPI** (Serial Peripheral Interface) by the ESP32-S3 on the T-Embed. All configuration is done by writing values to **47 configuration registers** and an 8-byte **PATABLE** (power amplifier table).

### Key Registers

#### Frequency Configuration (FREQ2, FREQ1, FREQ0)

The carrier frequency is set by three registers that form a 24-bit value:

```
Frequency (Hz) = (f_XOSC / 2^16) * FREQ[23:0]

Where:
  f_XOSC = 26 MHz (CC1101 crystal oscillator)
  FREQ[23:0] = (FREQ2 << 16) | (FREQ1 << 8) | FREQ0

Example: 433.92 MHz
  FREQ = 433920000 * 2^16 / 26000000 = 1094006.73 ≈ 1094007
  FREQ2 = 0x10  (16)
  FREQ1 = 0xB0  (176)
  FREQ0 = 0x71  (113)
  Verify: 26000000 * 1093745 / 65536 = 433,919,982 Hz ✓ (close enough)
```

#### Modulation Format (MDMCFG2)

```
Register 0x12 (MDMCFG2):
  Bits [6:4] = MOD_FORMAT
    000 = 2-FSK
    001 = GFSK
    011 = ASK/OOK
    100 = 4-FSK
    111 = MSK

  Bits [2:0] = SYNC_MODE
    000 = No preamble/sync
    001 = 15/16 sync word bits detected
    010 = 16/16 sync word bits detected
    011 = 30/32 sync word bits detected
    110 = Carrier sense above threshold
    111 = Carrier sense + 30/32 sync word bits
```

#### Data Rate (MDMCFG3, MDMCFG4)

```
Data Rate (baud) = (256 + DRATE_M) * 2^DRATE_E * f_XOSC / 2^28

Where:
  DRATE_M = MDMCFG3[7:0]  (mantissa)
  DRATE_E = MDMCFG4[3:0]  (exponent)

Example: 9600 baud
  DRATE_E = 8,  DRATE_M = 131
  Rate = (256 + 131) * 2^8 * 26000000 / 2^28 = 9596 baud ✓
```

#### Channel Bandwidth (MDMCFG4)

```
Bandwidth (Hz) = f_XOSC / (8 * (4 + CHANBW_M) * 2^CHANBW_E)

Where:
  CHANBW_M = MDMCFG4[5:4]  (mantissa, 0-3)
  CHANBW_E = MDMCFG4[7:6]  (exponent, 0-3)

Example: 270 kHz bandwidth
  CHANBW_E = 1, CHANBW_M = 1
  BW = 26000000 / (8 * (4+1) * 2^1) = 325000 Hz

Minimum bandwidth: 58 kHz (CHANBW_E=3, CHANBW_M=3)
Maximum bandwidth: 812 kHz (CHANBW_E=0, CHANBW_M=0)
```

#### Output Power (PATABLE)

The PATABLE register controls transmit power. For OOK, two entries are used (index 0 = "0" power, index 1 = "1" power). For other modulations, only the first entry is used.

```
Common PATABLE values for 433 MHz:

Value   Output Power
0x00    -30 dBm (minimum)
0x12    -20 dBm
0x0E    -15 dBm
0x1D    -10 dBm
0x34     -5 dBm
0x60      0 dBm
0x84     +5 dBm
0x8E     +7 dBm
0xC5    +10 dBm
0xC0    +12 dBm (maximum for CC1101)
```

{: .warning }
Transmitting at maximum power (+12 dBm) is legal in most ISM bands but may exceed limits in some regions or on some frequencies. Check your local regulations. In the EU, 433 MHz ISM is limited to +10 dBm ERP.

#### Packet Configuration (PKTCTRL0, PKTCTRL1, PKTLEN)

```
PKTCTRL0 (0x08):
  Bits [1:0] = LENGTH_CONFIG
    00 = Fixed packet length (set by PKTLEN)
    01 = Variable packet length (first byte = length)
    10 = Infinite packet length

  Bit [2] = CRC_EN
    0 = CRC disabled
    1 = CRC enabled (auto-appended on TX, auto-checked on RX)

  Bits [5:4] = PKT_FORMAT
    00 = Normal mode (FIFO)
    01 = Synchronous serial mode
    10 = Random TX mode
    11 = Asynchronous serial mode (used for RAW capture)

PKTLEN (0x06):
  Packet length in bytes (for fixed length mode)
  Maximum: 255 bytes

PKTCTRL1 (0x07):
  Bits [2:0] = ADR_CHK (address checking mode)
    00 = No address check
    01 = Check address, no broadcast
    10 = Check address, 0x00 broadcast
    11 = Check address, 0x00 and 0xFF broadcast
```

### How Firmware Translates Settings

When you select a frequency and modulation in Bruce firmware, the firmware performs these register calculations automatically:

```
User selects:        Firmware writes:
──────────────       ─────────────────
Freq: 433.92 MHz  →  FREQ2=0x10, FREQ1=0xB0, FREQ0=0x71
Modulation: OOK   →  MDMCFG2[6:4] = 011
Data rate: 4800    →  MDMCFG3=0x83, MDMCFG4[3:0]=0x07
Bandwidth: 270 kHz →  MDMCFG4[7:4] = 0x6_
Power: +10 dBm    →  PATABLE[0] = 0xC5
```

{: .note }
You rarely need to set registers manually. The firmware abstracts this away. But understanding registers is essential for: (1) troubleshooting reception issues, (2) optimizing for specific protocols, (3) modifying firmware for custom applications, and (4) understanding why certain settings work better than others.

---

## Troubleshooting Sub-GHz

### No Signal Detected

| Possible Cause | Solution |
|:---------------|:---------|
| Wrong frequency | Use frequency analyzer first; try nearby frequencies |
| Wrong modulation | Try both AM and FM modes |
| Out of range | Move closer to the target (within 3 meters for testing) |
| Antenna issue | Verify the CC1101 antenna connector is secure; try different orientation |
| Device not transmitting | Verify the target device works (test with known-good receiver) |
| Frequency out of CC1101 range | CC1101 has gaps: 349--387 MHz and 465--778 MHz are NOT covered |
| Interference | Move to a quieter location; check for nearby strong transmitters |
| Threshold too high | Lower the RSSI detection threshold in settings |

### Captured Signal Won't Replay

| Possible Cause | Solution |
|:---------------|:---------|
| Rolling code system | Compare multiple captures; if data changes, replay is not viable |
| Wrong modulation for TX | Ensure TX modulation matches the original signal |
| Insufficient TX power | Move closer to the target receiver |
| Incomplete capture | Recapture, ensuring you get the full transmission including preamble |
| Wrong frequency (slightly off) | Fine-tune frequency in increments of 10--50 kHz |
| Timing errors in playback | Try RAW replay instead of decoded replay |
| Receiver in wrong state | Some receivers need to be "awake" or in listening mode |
| Protocol requires handshake | Some systems require a bidirectional exchange before accepting commands |

### Weak Signal

| Possible Cause | Solution |
|:---------------|:---------|
| Distance too great | Move closer; sub-GHz has good range but the CC1101 antenna is small |
| Antenna polarization mismatch | Rotate the T-Embed 90 degrees |
| Obstacles blocking signal | Move to line-of-sight if possible |
| Battery low on target device | Replace batteries in the target transmitter |
| Receiver bandwidth too narrow | Increase RX bandwidth in settings (wider catches more energy) |
| Metal enclosure nearby | Move away from metal surfaces that create reflections and nulls |

### False Captures (Noise)

| Possible Cause | Solution |
|:---------------|:---------|
| RSSI threshold too low | Raise the detection threshold |
| Nearby interference sources | Move away from computers, switching power supplies, LED drivers |
| Broadband noise | Try different frequencies; noise may be localized |
| Multiple devices on same frequency | Wait for the target to be the only active transmitter |
| Harmonic interference | Strong signals at half the frequency can create harmonics; use narrower bandwidth |

{: .tip }
**The number one troubleshooting step is to get closer.** Most failed captures and replays are simply range problems. Start at 1 meter and work your way out. If it works at 1 meter but not at 10 meters, you have a power/antenna/obstacle issue, not a protocol issue.

---

## Legal and Ethical Reminders

{: .danger }
**Transmitting sub-GHz signals carries legal responsibility.** In all jurisdictions:

- **Only transmit on ISM bands** where your device is legally permitted to operate
- **Only interact with devices you own** or have explicit written permission to test
- **Never jam or interfere** with any radio communications
- **Never replay signals to devices you do not own** --- this is illegal regardless of the simplicity of the protocol
- **Never interfere with safety-critical systems** (fire alarms, medical devices, emergency communications, aviation, railroad)
- **Observe duty cycle limits** --- in the EU, 433 MHz ISM band has a 10% duty cycle limit; in the US, Part 15 rules apply
- **Understand that "I was just testing" is not a legal defense** if you interfere with someone else's property or safety systems

The T-Embed CC1101 Plus is a powerful tool. Power demands responsibility. Use it to learn, to secure your own systems, and to advance your understanding of wireless technology.

---

## Chapter Summary

| Topic | Key Takeaway |
|:------|:-------------|
| RF Fundamentals | Sub-GHz signals travel farther and penetrate better than WiFi/Bluetooth |
| Modulation | OOK/AM for simple devices, FSK/FM for modern protocols |
| Encoding | PT2262 and EV1527 are the most common fixed-code schemes |
| Rolling Codes | KeeLoq, Security+, and similar systems defeat simple replay attacks |
| Frequency Analysis | Always start with the frequency analyzer before attempting capture |
| Signal Capture | RAW capture is the most reliable method for unknown protocols |
| Signal Replay | Only works against fixed-code systems; always verify before attempting |
| Signal Analysis | Multiple captures + known encoding schemes = decoded protocol |
| Custom TX | Build your own protocols for learning and IoT development |
| CC1101 Registers | Understanding registers enables optimization and troubleshooting |

{: .tip }
This chapter is your primary reference for all sub-GHz operations. Bookmark it and return to it often. The combination of theoretical knowledge (how modulation and encoding work) with practical skills (capture, analyze, replay) is what separates effective RF practitioners from people who just press buttons.

---

*Next chapter: [05 --- WiFi & Bluetooth Operations](05-wifi-bluetooth.md) --- Moving to 2.4 GHz and 5 GHz for network reconnaissance.*
