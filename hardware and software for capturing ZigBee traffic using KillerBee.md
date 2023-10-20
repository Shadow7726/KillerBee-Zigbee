<p align="center">
  <img  width="180" src="ghost.png" />
</p>

---

## Hardware Setup for ZigBee Traffic Capture with KillerBee:

### 1. **Obtain a Compatible USB Sniffer Device:**
   - Purchase an RZUSBSTICK, YardStick One, or another compatible 802.15.4 transceiver dongle.

### 2. **Flash with KillerBee Firmware:**
   - Connect the AVR programmer to your computer and the sniffer device.
   - Download the latest KillerBee firmware binary.
   - Use avrdude to flash the firmware:
     ```
     avrdude -p at90usb1287 -P usb -c avrisp -U flash:w:killerbee-firmware.hex
     ```
   - Verify the new firmware version:
     ```
     avrdude -p at90usb1287 -P usb -c avrisp -U flash:r:read.hex:r
     ```

### 3. **Connect Sniffer to Computer's USB Port:**
   - Plug the USB dongle into an available USB port on your computer.
   - Verify it is detected using `lsusb`.

---

## Hardware Connection Steps:

1. Obtain an AVR programmer that supports the AT90USB1287 chip, e.g., Atmel JTAGICE mkII or AVR Dragon.
2. Install required drivers or software for the AVR programmer on your computer.
3. Connect the AVR programmer to your computer using the appropriate interface (USB, Ethernet, etc).
4. Power on the AVR programmer and verify its detection by your computer.
5. Insert the KillerBee sniffer device into the programming socket on the AVR programmer (use adapter boards or cables if needed).
6. Identify correct pinout and connections between the programmer and the sniffer device. Consult datasheets and documentation.
7. Verify the sniffer device is powered on and detected by the programmer (test firmware version reading using avrdude).
8. Use avrdude or other software to erase and write the new KillerBee firmware onto the sniffer device's microcontroller flash memory.
9. After programming, remove the sniffer device from the programmer and connect it directly to your computer's USB port to operate with the new firmware.

Refer to device datasheets and avrdude documentation for low-level details.

---

## Software Setup for ZigBee Traffic Capturing with KillerBee:

### 1. **Install KillerBee:**
   - **From Source Code:**
     ```
     git clone https://github.com/riverloopsec/killerbee
     cd killerbee
     sudo python setup.py install
     ```
   - **Or Using apt on Linux:**
     ```
     sudo apt install killerbee
     ```

### 2. **Install Wireshark:**
   - Download the latest stable version from [Wireshark Official Website](https://www.wireshark.org).
   - Install Wireshark using your package manager or from source.

### 3. **Connect KillerBee Sniffer:**
   - Plug the RZUSBSTICK or another sniffer into a USB port.
   - Verify its detection using:
     ```
     lsusb
     ```
     *or*
     ```
     dmesg | grep USB
     ```

### 4. **Verify Sniffer Operation:**
   - Discover devices:
     ```
     sudo zbid
     ```
   - Scan for networks:
     ```
     sudo zbstumbler
     ```
   - This ensures the tools are correctly installed, and the USB sniffer device is operational.

---

## ZigBee Traffic Capture Steps:

### 1. **Capture Traffic with zbdump:**
   - Use the following command to capture traffic to a PCAP file using KillerBee:
     ```
     sudo zbdump -i 0 -f 15 -w capture.pcap
     ```
   - Replace `-i 0` with the KillerBee device ID and `-f 15` with the Zigbee channel number.

### 2. **Capture in Wireshark:**
   - Open Wireshark and select the KillerBee device as the capture interface.
   - Start capture with a filter similar to:
     ```
     wpan.src64 == 0x1234567890 && wpan.dst64 == 0x0987654321
     ```
   
### 3. **Analyze in Wireshark:**
   - Apply filters like `wpan.src16`, `wpan.dst64`, `zbee_aps.profile_id`, etc.
   - Go to **Preferences > IEEE 802.15.4** to enter the decryption key (in reverse byte order).
   - Right-click on a packet and choose **Decode As** to decrypt Zigbee layers.
   - **Expert Info** will highlight errors, security issues, etc.

---
| Tool Name            | Description                                      | Example Usage                                              |
|----------------------|--------------------------------------------------|------------------------------------------------------------|
| **zbid**             | View compatible interfaces for KillerBee         | `zbid`                                                     |
| **zbwireshark**      | Capture packets to Wireshark in real-time        | `zbwireshark -i wlan0`                                     |
| **zbdump**           | Capture 802.15.4 packets like tcpdump             | `zbdump -i wlan0 -c 11 -w capture.pcap`                     |
| **zbreplay**         | Replay captured packets                          | `zbreplay -i wlan0 -r capture.pcap`                         |
| **zbstumbler**       | Discover Zigbee networks                         | `zbstumbler -i wlan0 -c 11`                                |
| **zbpanidconflictflood** | Attack to destroy Zigbee networks             | `zbpanidconflictflood -i wlan0 -c 11`                       |
| **zborphannotify**   | Spoof orphan notification to coordinator         | `zborphannotify -i wlan0 -s 1122334455667788 -d 1122334455667789` |
| **zbrealign**        | Spoof realignment frame to device                | `zbrealign -i wlan0`                                       |
| **zbfakebeacon**     | Spoof beacon frames                              | `zbfakebeacon -i wlan0 -c 11`                              |
| **zbopenear**        | Capture on multiple channels                     | `zbopenear -i wlan0 -c 11`                                 |
| **zbassocflood**     | Flood associations to crash device               | `zbassocflood -i wlan0 -c 11`                              |
| **zbconvert**        | Convert between pcap and SNA                      | `zbconvert -i capture.pcap -o capture.sna`                 |
| **zbdsniff**         | Sniff network keys from traffic                   | `zbdsniff -i wlan0`                                        |
| **zbfind**           | Locate transmitter via RSSI                      | `zbfind -i wlan0 -t transmitter_mac_address`              |
| **zbgoodfind**       | Find keys in memory dumps                         | `zbgoodfind -b ram.bin -k key`                            |
| **zbwardrive**       | Automated network discovery                       | `zbwardrive -i wlan0 -c 11`                                |
| **zbscapy**          | Scapy shell through KillerBee                    | `zbscapy`                                                 |
| **kbbootloader**     | Put device into bootloader mode                   | `kbbootloader -i wlan0 --target`                          |

---

By following these steps, you can capture and analyze ZigBee traffic using KillerBee and Wireshark, providing a comprehensive view of the captured packets.

Please ensure you adjust commands and parameters according to your specific hardware and network configurations. For detailed information, refer to the respective documentation and datasheets of your devices.
