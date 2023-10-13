
---

# Zigbee Penetration Testing Tools & Attack Vectors

## Network Layer Attacks 

### Sniffing

- **Tools:** [KillerBee](https://github.com/riverloopsec/killerbee), [Ubertooth](https://github.com/greatscottgadgets/ubertooth), [Wireshark](https://www.wireshark.org/), [Zigbee2MQTT](https://github.com/Koenkk/zigbee2mqtt)

- **Description:** Sniffing Zigbee traffic using packet sniffers and SDRs to intercept sensitive information.

### Jamming

- **Tools:** [HackRF](https://greatscottgadgets.com/hackrf/), [YARD Stick One](https://greatscottgadgets.com/yardstickone/) 

- **Description:** RF jamming to disrupt and deny service Zigbee networks.

### Replay Attacks

- **Tools:** [Scapy](https://scapy.net/), [KillerBee](https://github.com/riverloopsec/killerbee)

- **Description:** Capturing and replaying Zigbee packets to impersonate devices or replay old commands.

### Key Cracking

- **Tools:** [Z3Decrypt](https://github.com/IoTsec/Z3Decrypt), [Key extraction scripts](https://github.com/cisagov/zigbee-network-key-cracking)

- **Description:** Cracking network or link keys via brute force to decrypt encrypted traffic.

## Application Layer Attacks

### Weak Encryption

- **Tools:** [ECB Decrypt](https://github.com/cisco/ecbdecrypter), [KillerBee](https://github.com/riverloopsec/killerbee)

- **Description:** Exploiting weak ciphers like single DES to decrypt payload data.

### Default/Weak Credentials

- **Tools:** [ZigDiggity](https://github.com/BishopFox/zigdiggity), [ZbeeTool](https://github.com/realcryptonight/ZbeeTool)

- **Description:** Accessing devices using default or weak login credentials.

### Unpatched Vulnerabilities

- **Tools:** [Z3Fuzz](https://github.com/IoTsec/Z3Fuzz), [BeepBeep-NG](https://github.com/virtualabs/beepbeep-ng)

- **Description:** Exploiting known vulnerabilities in Zigbee stack implementations. 

### Buffer Overflows

- **Tools:** [Z3Fuzz](https://github.com/IoTsec/Z3Fuzz), [Scapy](https://scapy.net/)

- **Description:** Fuzzing payloads to crash devices or execute malicious code.

### Insecure EAP

- **Tools:** [Zigdiggity](https://github.com/BishopFox/zigdiggity), [ZbeeTool](https://github.com/realcryptonight/ZbeeTool)

- **Description:** Breaking Zigbee's EAP to authenticate without credentials.

## Physical Access Attacks

### Firmware Extraction  

- **Tools:** [JTAG/Flash Readers](https://www.adafruit.com/category/92), [Bus Pirate](https://hackaday.io/project/182-bus-pirate)

- **Description:** Dumping firmware from physically accessed chips to uncover flaws.

### Flash Memory Access

- **Tools:** [JTAG Readers](https://www.adafruit.com/category/92), [Bus Pirate](https://hackaday.io/project/182-bus-pirate)

- **Description:** Directly reading flash memory to obtain credentials.

## Miscellaneous Attacks

### Intercepting OTA Updates

- **Tools:** [KillerBee](https://github.com/riverloopsec/killerbee), [Ubertooth](https://github.com/greatscottgadgets/ubertooth) 

- **Description:** Intercepting firmware updates over-the-air to analyze or modify. 

### Deactivating Security Controls

- **Tools:** [Zigdiggity](https://github.com/BishopFox/zigdiggity), [Attify Zigbee Framework](https://github.com/attify/zigbee)

- **Description:** Disabling security controls and protections via exploits.

### Reverse Engineering

- **Tools:** [Ghidra](https://ghidra-sre.org/), [IDA Pro](https://hex-rays.com/ida-pro/)

- **Description:** Reverse engineering firmware and systems to uncover vulnerabilities.

