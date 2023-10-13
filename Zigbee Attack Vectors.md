

---

# Zigbee Attack Surfaces and Exploitation Techniques

## Sniffing Attacks

**1. OTA Crypto Key Sniffing**
- Zigbee devices transmit network keys OTA during joining. Use tools like [KillerBee](https://github.com/riverloopsec/killerbee) or [Ubertooth](https://github.com/greatscottgadgets/ubertooth) to capture and extract network keys for decrypting traffic.

**2. RAM Dump Attack**
- Physical access to a Zigbee device allows RAM content extraction. Utilize JTAG and debugging tools to obtain crypto keys and network keys stored in memory.

## Reconnaissance  

**1. Finding All Devices**
- Tools like [Zbfind](https://github.com/nccgroup/zbfind) and [Z3Scanner](https://github.com/IoTsec/Z3Scanner) enumerate Zigbee networks, aiding in device discovery and network mapping.

**2. Open-source/Third-party Zigbee Stacks**
- Analyze open-source Zigbee stacks such as ZBOSS to identify vulnerabilities and security flaws in devices using these stacks.

## Insecure Cryptography

**1. Lack of CRC Verification**
- Some Zigbee devices skip packet CRC verification, enabling bit flipping attacks to manipulate payloads.

**2. Insecure Key Storage**
- Extract hardcoded keys stored in firmware through reverse engineering.

**3. Insecure Key Transport**
- Weak key exchange algorithms like single DES can be exploited to obtain keys by capturing exchanges.

**4. Reusing CCM* and Initialization Vectors (IVs)**
- Poor encryption implementation, such as reusing IVs, weakens ciphers, allowing decryption of payload data.

**5. Sending Security Headers in Cleartext**
- Unencrypted protocol security headers expose network topology and linked keys to attackers.

## Network Layer Attacks

**1. Default Link Key Values**
- Default 'ZigBeeAlliance09' trust key is used by many devices, providing broad network access to attackers.

**2. CSMA/CA Trade-off**
- Vulnerable to brute force attacks for joining new devices and causing network congestion.

**3. Unencrypted Keys**
- Some development kits ship with unencrypted default Trust Center link keys, granting network access.

**4. Predictable PAN IDs and Channels**
- Limited PAN ID and channel space make networks vulnerable to targeted attacks.

**5. Insufficient Replay Protections**
- Absence of replay protection allows attackers to duplicate actions by replaying old frames.

## Physical Layer Attacks 

**1. Signal Interference**
- RF noise injection disrupts signals, denying availability and achieving DoS.

**2. Command Injection**
- Injecting frames during joining can add malicious devices or elevate privileges.

## Availability Attacks

**1. Lack of DDoS Protections**
- Absence of native DDoS prevention exposes Zigbee networks to barrage attacks, disrupting availability.

## Cryptographic Attacks

**1. Re-using Link Keys**
- Reused link keys across networks enable lateral movement between insecure Zigbee networks.

**2. Touchlink Factory Reset** 
- Exploiting Touchlink commissioning to factory reset devices provides a vector for command injection.

---
