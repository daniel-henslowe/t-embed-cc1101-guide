---
layout: default
title: "13 — Legal Reference"
nav_order: 14
---

# 13 --- Legal Reference
{: .fs-9 }

Comprehensive legal guidance for radio frequency operations, wireless security testing, and authorized use of the T-Embed CC1101 Plus. Know the law before you transmit.
{: .fs-6 .fw-300 }

---

{: .danger }
> **This chapter is not legal advice.** It is an educational summary of relevant laws and regulations. Laws vary by jurisdiction and change over time. If you plan to conduct professional security testing or have questions about specific activities, **consult a licensed attorney** in your jurisdiction.

---

## Table of Contents
{: .no_toc }

1. TOC
{:toc}

---

## Radio Frequency Regulations

### United States (FCC)

The Federal Communications Commission (FCC) regulates all radio transmissions in the United States under Title 47 of the Code of Federal Regulations.

#### FCC Part 15: Unlicensed Operation

Part 15 governs unlicensed intentional, unintentional, and incidental radiators. The T-Embed CC1101 Plus falls under **intentional radiator** rules when transmitting.

Key Part 15 rules:

| Rule | Requirement |
|:-----|:------------|
| **15.5** | You must accept interference from other devices and must not cause harmful interference |
| **15.15** | Equipment must be authorized (certified, verified, or exempt) |
| **15.23** | Home-built devices: You may build up to 5 units for personal use without certification |
| **15.209** | General field strength limits for unlicensed devices |
| **15.231** | Periodic operation in 40.66--40.70 MHz and 70--73 MHz bands |
| **15.240** | Operation in 433.5--434.5 MHz band (limited in US) |
| **15.247** | Spread spectrum systems in ISM bands |
| **15.249** | Operation in 902--928 MHz, 2400--2483.5 MHz, and other ISM bands |

{: .note }
> Part 15.23 allows you to build up to 5 units of an unlicensed device for **personal use** without FCC certification. This is why building and using the T-Embed CC1101 Plus for personal, non-commercial experimentation is legal. If you sell or distribute devices, FCC certification requirements apply.

#### ISM Bands in the United States

ISM (Industrial, Scientific, and Medical) bands allow unlicensed operation within specific power limits:

| Band | Frequency Range | Max TX Power (EIRP) | Common Uses | CC1101 Coverage |
|:-----|:---------------|:--------------------|:------------|:---------------|
| 315 MHz | 314--316 MHz | Very limited (field strength limits) | Car key fobs, garage doors | Yes |
| 433 MHz | 433.05--434.79 MHz | Very limited in US (field strength limits) | Some sensors, imports from EU | Yes |
| 902--928 MHz | 902--928 MHz | +30 dBm (1 watt) FHSS, +36 dBm (4 watts) with directional antenna | LoRa, smart meters, Z-Wave | Yes |
| 2.4 GHz | 2400--2483.5 MHz | +30 dBm (1 watt) | WiFi, Bluetooth, Zigbee | ESP32-S3 WiFi/BLE |

{: .warning }
> The 433 MHz band has **very limited** legal use in the United States compared to Europe. US regulations under Part 15.231 and 15.240 allow low-power, periodic transmissions only. Continuous transmission on 433 MHz in the US may violate FCC rules. The 902--928 MHz band is the primary US ISM band for Sub-GHz applications.

#### Maximum Power Limits

The CC1101's maximum output power is +12 dBm (approximately 16 milliwatts). For reference:

| Power Level | Milliwatts | Comparison |
|:------------|:-----------|:-----------|
| +12 dBm (CC1101 max) | ~16 mW | Well below most Part 15 limits |
| +20 dBm | 100 mW | Typical WiFi transmitter |
| +27 dBm | 500 mW | High-power WiFi |
| +30 dBm | 1,000 mW (1W) | Part 15.247 maximum for ISM band |

{: .tip }
> The CC1101's maximum output power (+12 dBm) is well within the power limits for ISM band operation in the 902--928 MHz band. For 315 MHz and 433 MHz in the US, however, the limits are expressed as field strength (measured in microvolts per meter at a specific distance), which makes compliance more nuanced. When in doubt, use the lowest power setting that accomplishes your goal.

#### Prohibited Frequencies

**Never transmit on these frequencies under any circumstances without proper authorization:**

| Frequency Range | Allocated To | Why It Matters |
|:---------------|:------------|:---------------|
| 300--322 MHz | Military (various) | Federal use, severe penalties |
| 328.6--335.4 MHz | Aeronautical radionavigation | Aircraft safety systems |
| 406--406.1 MHz | Emergency beacons (SARSAT/COSPAS) | Search and rescue satellite system |
| 960--1215 MHz | Aeronautical radionavigation | Air traffic control radar |

{: .danger }
> Transmitting on aeronautical or emergency frequencies is a **federal crime** that can result in immediate FCC enforcement action, significant fines, and criminal prosecution. The CC1101's Band 2 (387--464 MHz) includes sensitive allocations near the 406 MHz emergency beacon frequency. **Never transmit on or near 406 MHz.**

#### FCC Enforcement Actions and Penalties

The FCC's Enforcement Bureau actively investigates illegal radio transmissions. They have direction-finding equipment and can locate unauthorized transmitters.

| Violation | Penalty Range |
|:----------|:-------------|
| Unauthorized operation | Warning letter to $100,000+ fine |
| Interference with licensed operations | $10,000--$100,000 fine |
| Jamming any communication | $100,000+ fine and/or criminal prosecution |
| Interference with aviation or emergency services | Criminal prosecution, imprisonment |
| Marketing non-compliant devices | $100,000+ fine per violation |

Recent enforcement trends:

- FCC has increased enforcement against signal jammers (GPS, cellular, WiFi)
- Drone-related interference investigations are increasing
- Amateur radio operators who exceed band limits or power levels face license revocation
- The FCC has issued fines ranging from $25,000 to $144,344 for individual violation cases

#### Amateur Radio License

An FCC Amateur Radio License significantly expands your legal frequency access and power limits. There are three license classes:

| License Class | Exam | Privileges Relevant to T-Embed |
|:-------------|:-----|:------------------------------|
| **Technician** | 35-question multiple choice | Full privileges on 33 cm band (902--928 MHz) with up to 1500W, limited HF access |
| **General** | Additional 35-question exam | Expanded HF privileges, all Technician privileges |
| **Amateur Extra** | Additional 50-question exam | All amateur frequencies and privileges |

{: .tip }
> The **Technician license** is the most relevant. It costs nothing to maintain (no recurring fees), requires passing a single 35-question exam, and gives you legal access to operate on 902--928 MHz at much higher power levels than Part 15 allows. Study materials are freely available, and exams are offered by volunteer examiners nationwide. Visit [arrl.org](https://www.arrl.org/find-an-amateur-radio-license-exam-session) to find an exam session near you.

**Important:** Even with an amateur license, you are still bound by amateur radio rules --- no encryption, no commercial use, and you must identify your transmissions with your callsign.

---

### European Union (ETSI)

The European Telecommunications Standards Institute (ETSI) sets harmonized standards for radio equipment in the EU under the Radio Equipment Directive (RED) 2014/53/EU.

#### EN 300 220: Short Range Devices

EN 300 220 covers Short Range Devices (SRDs) operating below 1 GHz.

| Band | Frequency Range | Max Power (ERP) | Duty Cycle | Common Uses |
|:-----|:---------------|:---------------|:-----------|:------------|
| **g** | 433.05--434.79 MHz | +10 dBm (10 mW) | Up to 10% | Remote controls, sensors, weather stations |
| **g1** | 433.05--434.79 MHz | +10 dBm (10 mW) | Up to 10% | Non-specific SRDs |
| **h1.1** | 863--870 MHz | +25 dBm (500 mW) | Varies by sub-band | Alarms, smart metering |
| **h1.2** | 868.0--868.6 MHz | +14 dBm (25 mW) | 1% | General SRDs |
| **h1.3** | 868.7--869.2 MHz | +14 dBm (25 mW) | 0.1% | Alarms |
| **h1.4** | 869.4--869.65 MHz | +27 dBm (500 mW) | 10% | Alarms, social alarms |
| **h1.6** | 869.7--870.0 MHz | +14 dBm (25 mW) | 1% | General SRDs |

{: .warning }
> **Duty cycle limits are strictly enforced in the EU.** A 1% duty cycle on 868.0--868.6 MHz means you may transmit for a maximum of 36 seconds per hour. Continuous transmission violates ETSI regulations even if you are within the power limit.

#### CE Marking

Any radio equipment sold in the EU must carry a CE mark indicating compliance with the RED. For personal experimentation and development, CE marking is not required, but transmission power and duty cycle limits still apply.

---

### Other Regions

#### United Kingdom (Ofcom)

Post-Brexit, the UK follows its own regulations through Ofcom, but they remain largely aligned with ETSI standards:

- Interface Requirement IR 2030 covers Short Range Devices
- 433 MHz: 10 mW ERP, 10% duty cycle
- 868 MHz: Same sub-band structure as EU
- UKCA marking replaces CE marking for UK market

#### Canada (ISED / Industry Canada)

- RSS-210: Low-power license-exempt radio devices
- ISM bands align closely with FCC Part 15
- 902--928 MHz: Similar power limits to US
- 433 MHz: Very restricted (similar to US)

#### Japan (MIC)

- 315 MHz: Not available for SRDs
- 426 MHz and 429 MHz bands for specific low-power applications
- 920 MHz: Available for IoT/LPWAN (ARIB STD-T108)
- Strict type approval required for imported devices

#### Australia (ACMA)

- Class Licence for Short Range Devices (Radiocommunications Licence)
- 433 MHz: 25 mW EIRP
- 915--928 MHz: 1W EIRP (more permissive than EU, similar to US)
- No duty cycle restrictions (unlike EU)

{: .note }
> If you travel internationally with the T-Embed, the legal frequencies and power limits change with each country. **Configure your device for the destination country's regulations before transmitting.** Receiving (listen-only mode) is legal almost everywhere.

---

## Computer Fraud and Abuse Act (CFAA)

The Computer Fraud and Abuse Act (18 U.S.C. 1030) is the primary US federal law governing unauthorized computer access. It is directly relevant to WiFi, BLE, and BadUSB operations.

### What Constitutes Unauthorized Access

The CFAA prohibits:

1. **Accessing a computer without authorization** --- Connecting to a WiFi network without permission, injecting keystrokes via BadUSB into a computer you do not own
2. **Exceeding authorized access** --- Having some access but going beyond what is permitted (e.g., having guest WiFi access but scanning the internal network)
3. **Intentionally causing damage** --- Deauthenticating devices from a network causes a denial of service, which may constitute "damage" under the CFAA
4. **Trafficking in passwords/access devices** --- Sharing captured WiFi passwords or credentials obtained through evil portal attacks

### How the CFAA Applies to T-Embed Operations

| Operation | CFAA Relevance |
|:----------|:--------------|
| **WiFi deauth** | Potentially violates CFAA as intentional disruption of a computer network. Even if no data is accessed, causing a denial of service may constitute "damage." |
| **Evil portal** | Capturing credentials through a fake access point is fraud. Even testing on your own network with others' devices without their knowledge could create issues. |
| **BadUSB** | Injecting keystrokes into a computer without authorization is unauthorized access, regardless of what the payload does. |
| **RF signal capture** | Generally legal for signals in the public airwaves. However, intercepting certain communications (e.g., cell phone data) violates the Wiretap Act. |
| **BLE enumeration** | Passive scanning is generally legal. Actively connecting to and exploring devices without permission may constitute unauthorized access. |

### Penalties Under the CFAA

| Offense | First Offense | Subsequent Offense |
|:--------|:-------------|:-------------------|
| Unauthorized access (no damage) | Up to 1 year imprisonment | Up to 10 years |
| Unauthorized access with intent to defraud | Up to 5 years | Up to 10 years |
| Intentionally causing damage | Up to 10 years | Up to 20 years |
| Trafficking in passwords | Up to 1 year | Up to 10 years |
| Accessing government computers | Up to 10 years | Up to 20 years |

Additionally, the CFAA provides for civil lawsuits by affected parties seeking damages and injunctive relief.

{: .danger }
> The CFAA has been broadly interpreted by courts. Activities that might seem harmless --- like connecting to an open WiFi network or scanning BLE devices --- can potentially be prosecuted as unauthorized access depending on circumstances and jurisdiction. **When in doubt, do not do it without written permission.**

### Recent Case Law

Key cases that shape how the CFAA applies to security researchers:

- **Van Buren v. United States (2021):** The Supreme Court narrowed the definition of "exceeding authorized access" to mean accessing information one is not entitled to, not using information improperly. This is favorable for security researchers who discover data within their authorized scope.
- **hiQ Labs v. LinkedIn (2022):** Scraping publicly available data was found not to violate the CFAA. This supports the legality of passive RF and WiFi scanning.
- **United States v. Auernheimer (2015):** Accessing publicly exposed data was initially prosecuted under CFAA, conviction later overturned on jurisdictional grounds. Illustrates the risk of broad CFAA interpretation.

---

## Authorized Security Testing

### Getting Written Permission

Before conducting any security testing on systems you do not personally own, you need **explicit written authorization**. Verbal permission is insufficient --- you need a document that can be presented to law enforcement if questions arise.

#### Authorization Template

The following template covers the essential elements. Adapt it to your specific engagement:

```
═══════════════════════════════════════════════════
    SECURITY TESTING AUTHORIZATION AGREEMENT

Date: _______________

AUTHORIZING PARTY (System Owner):
Name: _______________
Title: _______________
Organization: _______________
Contact: _______________

AUTHORIZED TESTER:
Name: _______________
Contact: _______________

SCOPE OF AUTHORIZATION:

The Authorizing Party hereby grants the Authorized
Tester permission to conduct security testing on
the following systems and devices:

Physical Location(s):
□ _______________
□ _______________

Systems/Devices Under Test:
□ WiFi networks: _______________ (SSIDs)
□ RF devices: _______________ (describe)
□ Bluetooth devices: _______________ (describe)
□ Computers/endpoints: _______________ (describe)
□ Other: _______________

Authorized Activities:
□ WiFi network scanning and enumeration
□ WiFi deauthentication testing
□ Captive portal / evil portal testing
□ Sub-GHz RF signal capture and analysis
□ Sub-GHz RF signal replay testing
□ BLE device scanning and enumeration
□ IR signal capture and replay
□ BadUSB/HID testing
□ Other: _______________

RESTRICTIONS:
□ Testing window: ___ to ___ (dates/times)
□ No denial-of-service that disrupts production
□ No data exfiltration of real user data
□ No modification of production systems
□ Other: _______________

The Authorizing Party confirms they have the
legal authority to grant this permission.

Signature (Authorizing Party): _______________
Signature (Authorized Tester): _______________
Date: _______________
═══════════════════════════════════════════════════
```

{: .warning }
> This template is a starting point, not a substitute for legal counsel. For professional engagements, have an attorney review the authorization agreement. Many organizations also require a Master Services Agreement (MSA) and a Statement of Work (SOW) in addition to the authorization letter.

### Scope of Authorization

The scope must be specific and bounded:

- **In scope:** Exactly which systems, networks, and devices can be tested
- **Out of scope:** Explicitly list what must not be touched (production systems, third-party services, neighboring properties)
- **Time window:** When testing is permitted (avoid testing during business-critical hours unless specifically authorized)
- **Geographic boundary:** Where testing can be conducted (important for RF, which does not respect property lines)
- **Escalation procedures:** Who to contact if something goes wrong or if a critical vulnerability is discovered

### Rules of Engagement for RF Testing

RF testing has unique considerations because radio signals propagate beyond physical boundaries:

1. **RF leakage:** Your signals will be received beyond the authorized test area. Keep TX power to the minimum necessary.
2. **Neighbor's devices:** Even when testing your own property, your signals may affect neighboring devices on the same frequency. Be aware and minimize impact.
3. **Interference:** Your testing must not cause interference to licensed radio services.
4. **Duration:** Keep testing sessions as brief as possible to minimize the window of potential interference.
5. **Documentation:** Log all transmissions with timestamp, frequency, power level, and duration.

### Bug Bounty and Responsible Disclosure

If you discover a vulnerability in a commercial product or service during authorized testing:

**Responsible Disclosure Process:**

1. **Document** the vulnerability thoroughly (steps to reproduce, impact, affected versions)
2. **Check** if the vendor has a bug bounty program or vulnerability disclosure policy
3. **Report** to the vendor through their preferred channel (security@company.com, HackerOne, Bugcrowd)
4. **Allow** a reasonable remediation window (typically 90 days)
5. **Coordinate** public disclosure with the vendor if desired
6. **Never** exploit the vulnerability beyond what is necessary to demonstrate it

{: .tip }
> Many IoT and RF device manufacturers do not have bug bounty programs. In these cases, contact the manufacturer's support or engineering team directly. If they are unresponsive after reasonable attempts, organizations like CERT/CC ([cert.org](https://www.cert.org)) can coordinate disclosure.

### Professional Certifications

Relevant certifications for RF and wireless security professionals:

| Certification | Organization | Focus | Prerequisites |
|:-------------|:------------|:------|:-------------|
| **CEH** | EC-Council | Broad ethical hacking, includes wireless | 2 years InfoSec experience or training |
| **OSCP** | OffSec | Hands-on penetration testing | None (self-study or course) |
| **GPEN** | GIAC/SANS | Network penetration testing | None (SEC560 course recommended) |
| **CWSP** | CWNP | WiFi security specifically | CWNA certification |
| **GAWN** | GIAC/SANS | Wireless network assessment | None (SEC617 course recommended) |
| **FCC Amateur Technician** | FCC/ARRL | RF fundamentals, legal operation | Pass 35-question exam |

---

## What You CAN Legally Do

The following activities are generally legal in the United States. Check your specific jurisdiction for local variations.

### Receive Signals on Any Frequency

{: .tip }
> **Receiving is almost universally legal.** In the US, there is no law against receiving radio signals on any frequency with a few narrow exceptions (certain encrypted government/military communications under 18 U.S.C. 2511). Passive listening, scanning, and spectrum analysis are legal activities.

This means you can:

- Run the frequency analyzer on any band the CC1101 supports
- Capture and record signals on any frequency
- Analyze signal characteristics (modulation, timing, encoding)
- Build a spectrum monitor covering all CC1101 bands
- Listen to weather stations, sensors, and other transmissions

{: .warning }
> While receiving is legal, **acting on intercepted information** may not be. For example, receiving a neighbor's baby monitor signal is legal, but using that information for voyeurism or harassment is a crime. The Electronic Communications Privacy Act (ECPA) restricts the use of intercepted communications.

### Transmit on ISM Bands Within Power Limits

You may transmit on ISM bands without a license, provided you:

- Stay within the published power limits for each band
- Follow duty cycle restrictions (EU) or other operational requirements
- Do not cause harmful interference to licensed services
- Accept interference from other unlicensed and licensed users

### Test Your Own Devices

You have broad legal authority to test devices you own:

- Capture and replay signals from your own remotes, sensors, and security devices
- Deauthenticate your own devices from your own WiFi network
- Scan and enumerate your own BLE devices
- Use BadUSB on your own computers
- Probe your own network for vulnerabilities

### Educational Research in Controlled Environments

Academic and educational RF research is generally protected:

- University research labs with proper authorization
- Shielded (Faraday cage) environments that prevent signal leakage
- Simulation and software-defined radio analysis that does not involve actual transmission
- Analysis of publicly documented protocols

### Authorized Penetration Testing

With proper written authorization (see template above):

- Test client systems and devices within the defined scope
- Document and report vulnerabilities
- Demonstrate exploits to prove impact
- Recommend remediation measures

---

## What You CANNOT Legally Do

The following activities are **illegal** under US federal law. Violations carry serious criminal and civil penalties.

### Jam or Interfere with Any Radio Communication

{: .danger }
> **Signal jamming is a federal crime. No exceptions.** Under 47 U.S.C. 333, it is illegal to willfully or maliciously interfere with any radio communication. This applies to all frequencies and all types of communication --- WiFi, cellular, GPS, Bluetooth, emergency services, everything. The FCC considers this one of the most serious violations and pursues enforcement aggressively.

Specific prohibitions:

- Transmitting continuously on a frequency to block other devices
- Using the CC1101 to interfere with wireless security sensors
- Disrupting WiFi networks (even open/public ones)
- GPS jamming (even to prevent your own vehicle from being tracked)
- Any device marketed as a "jammer" is illegal to sell, market, or operate in the US

### Replay Signals on Devices You Don't Own

- Capturing your neighbor's garage door signal and replaying it is unauthorized access to their property
- Replaying car key fob signals to unlock vehicles you do not own is vehicle theft or attempted theft
- Replaying alarm system signals to disable security systems is burglary preparation

### Deauthenticate Networks You Don't Own

- Sending deauthentication frames to WiFi networks you do not own or are not authorized to test constitutes a denial-of-service attack
- This applies even to open/public WiFi networks
- The Marriott hotel chain was fined $600,000 by the FCC in 2014 for deauthenticating personal hotspots on their property

### Use BadUSB on Unauthorized Computers

- Plugging a BadUSB device into any computer without authorization is unauthorized access under the CFAA
- This applies even if the payload is "harmless" (e.g., a Rickroll)
- Physical access to a device does not imply authorization to inject input

### Intercept Communications Not Intended for You

Under the Wiretap Act (18 U.S.C. 2511) and ECPA:

- Intercepting cell phone calls or data
- Capturing and reading WiFi traffic not intended for you
- Monitoring private radio communications (cordless phones, etc.)

{: .note }
> The distinction between "receiving" (legal) and "intercepting" (potentially illegal) is context-dependent. Passively hearing a signal on a scanner is generally legal. Deliberately targeting specific communications, recording them, and using the content is potentially illegal under wiretapping laws.

### Operate Outside Permitted Frequency Bands

- Transmitting on frequencies allocated to licensed services without a license
- Exceeding power limits for unlicensed ISM band operation
- Ignoring duty cycle restrictions (EU)

### Exceed Power Limits

- The CC1101's maximum +12 dBm is within most ISM limits, but adding an external amplifier could push you over legal limits
- Never add an RF power amplifier without understanding and complying with applicable power limits

---

## Penalties Summary

| Violation | Statute | Maximum Penalty |
|:----------|:--------|:---------------|
| Unauthorized computer access | CFAA (18 U.S.C. 1030) | 1--20 years imprisonment + fines |
| Intentional damage to computer | CFAA (18 U.S.C. 1030) | 10--20 years imprisonment + fines |
| Signal jamming | 47 U.S.C. 333 | $100,000+ fine per violation per day |
| Unauthorized radio transmission | 47 U.S.C. 301 | $10,000--$100,000 fine |
| Wiretapping | 18 U.S.C. 2511 | 5 years imprisonment + $250,000 fine |
| Marketing jamming devices | 47 U.S.C. 302a | $100,000+ fine, seizure of equipment |

Additionally:

- **Civil liability:** Victims of unauthorized access can sue for actual damages, attorney's fees, and injunctive relief
- **Asset forfeiture:** The FCC can seize equipment used in violations
- **Professional consequences:** Criminal charges end security careers permanently

{: .danger }
> These are not theoretical penalties. The FCC issues enforcement actions regularly, and CFAA prosecutions are common. A single act of jamming or unauthorized access can result in life-changing legal consequences.

---

## Best Practices for Staying Legal

Follow these ten rules and you will stay on the right side of the law:

### Rule 1: Own It or Have Permission

Never test, access, probe, scan, replay, deauth, or interact with any device, network, or system you do not personally own or have explicit written authorization to test. This is the single most important rule.

### Rule 2: When in Doubt, Don't Transmit

Receiving is almost always legal. Transmitting creates legal risk. If you are uncertain whether a transmission is legal, keep your device in receive-only mode until you have confirmed the legality.

### Rule 3: Use Minimum Necessary Power

Even on ISM bands where transmission is legal, use the lowest TX power that accomplishes your goal. Lower power means less interference with others and smaller legal exposure.

### Rule 4: Document Everything

Keep records of:
- What you tested and when
- Authorization documents
- Findings and results
- Any incidents or unexpected outcomes

Documentation protects you if your activities are ever questioned.

### Rule 5: Know Your Frequencies

Before transmitting on any frequency, know:
- Whether it is an ISM band in your country
- The power limits for that band
- Any duty cycle restrictions
- What licensed services share the band

### Rule 6: Respect Rolling Codes

If a device uses rolling codes, it is designed to resist replay attacks. Attempting to defeat rolling-code security on devices you do not own is illegal, and the tools to do so (RollJam, etc.) are specifically designed to circumvent security measures.

### Rule 7: Never Jam

There is no legitimate reason to jam any radio signal. Jamming is always illegal, always prosecuted, and always harmful. The FCC can locate jammers with direction-finding equipment and has done so repeatedly.

### Rule 8: Separate Research from Exploitation

There is a clear line between security research (understanding how things work) and exploitation (using vulnerabilities for unauthorized gain). Stay on the research side.

### Rule 9: Consider the Impact

Before any action, ask: "If this goes wrong, who is harmed?" If the answer is anyone other than you and your own equipment, reconsider.

### Rule 10: Get Licensed

An FCC Amateur Radio Technician license costs nothing to maintain, takes a few weeks of study, and demonstrates that you understand RF regulations. It also expands your legal frequency access significantly. There is no reason not to get one.

---

## Documentation Templates

### RF Testing Log Template

Use this log for every testing session to maintain a complete record:

```
═══════════════════════════════════════
    RF TESTING SESSION LOG

Date: _______________
Location: _______________
Tester: _______________
Authorization: Self / Written auth ref# ___

Equipment:
  Device: T-Embed CC1101 Plus
  Firmware: _______________ v___
  Antenna: _______________

Session Entries:

Time    | Freq (MHz) | TX/RX | Power  | Activity
--------|------------|-------|--------|------------------
        |            |       |        |
        |            |       |        |
        |            |       |        |

Findings:
_______________________________________________

Incidents:
□ None
□ Describe: ________________________________

Signature: _______________
═══════════════════════════════════════
```

### Vulnerability Disclosure Template

When reporting a vulnerability you have discovered during authorized testing:

```
═══════════════════════════════════════
    VULNERABILITY DISCLOSURE REPORT

Report Date: _______________
Reporter: _______________
Reporter Contact: _______________

DEVICE/SYSTEM INFORMATION:
  Product: _______________
  Manufacturer: _______________
  Model/Version: _______________
  Firmware Version: _______________

VULNERABILITY DETAILS:
  Type: _______________
  Severity: Critical / High / Medium / Low
  CVE (if assigned): _______________

DESCRIPTION:
[Clear, detailed description of the vulnerability]

STEPS TO REPRODUCE:
1. _______________
2. _______________
3. _______________

IMPACT:
[Description of what an attacker could achieve]

PROOF OF CONCEPT:
[Minimal demonstration, no weaponized exploit code]

REMEDIATION RECOMMENDATION:
[Suggested fix or mitigation]

DISCOVERY CONTEXT:
  Authorization: _______________
  Discovery date: _______________
  Methodology: _______________

DISCLOSURE TIMELINE:
  Vendor notified: _______________
  Vendor response: _______________
  Patch available: _______________
  Public disclosure: _______________
═══════════════════════════════════════
```

---

## When in Doubt

{: .warning }
> If you are ever unsure whether an activity is legal, follow this decision tree:

**1. Do you own the device/network/system?**
- Yes: Proceed with caution, stay within RF regulations.
- No: Go to step 2.

**2. Do you have written authorization from the owner?**
- Yes: Proceed within the defined scope.
- No: Go to step 3.

**3. Are you only receiving (not transmitting or connecting)?**
- Yes: Generally legal. Proceed, but do not act on intercepted private communications.
- No: **Stop. Do not proceed.**

**4. If you need authorization:**
- For personal devices: Test them yourself, no authorization needed.
- For employer's devices: Get written authorization from IT leadership or management.
- For client engagement: Formal authorization agreement, SOW, and insurance.
- For research: Work with your institution's legal counsel and IRB if applicable.

---

## Consulting a Lawyer

For professional penetration testing engagements, consider consulting an attorney who specializes in:

- **Cybersecurity law** --- Understands CFAA, ECPA, and state computer crime laws
- **Telecommunications law** --- Understands FCC regulations and enforcement
- **Technology transactions** --- Can draft proper authorization agreements and SOWs

An hour of legal consultation before an engagement can prevent years of legal consequences after one.

{: .tip }
> The Electronic Frontier Foundation (EFF) maintains resources on security research and the law at [eff.org](https://www.eff.org/issues/coders). The EFF has also represented security researchers in CFAA cases and can provide guidance on legal risks.

---

{: .danger }
> **Final reminder:** This guide is for education and authorized security research only. The T-Embed CC1101 Plus is a powerful tool. With power comes responsibility. Every technique in this guide can be used ethically to improve security or unethically to cause harm. Choose to improve security. Always comply with the law. Always get permission. Always consider the impact of your actions.
