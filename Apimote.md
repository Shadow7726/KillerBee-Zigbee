<p align="center">
  <img  width="180" src="ghost.png" />
</p>


---

## Zigbee Testing Setup with Apimote and Attify

### Hardware Required:

- Apimote Zigbee Sniffer/Analyzer (USB dongle)
<p align="center">
  <img  width="600" src="apimote-v4b-front.jpg" />
</p>

Here is the complete hardware identification for the Apimote Zigbee sniffer :

| Hardware | Details | 
|-|-|  
| Name | Apimote Zigbee Sniffer/Analyzer |
| Type | USB Dongle |
| Chipset | Texas Instruments CC2531 | 
| RF Protocol | 802.15.4 (Zigbee) |
| Frequency | 2.4 GHz |
| Data Rates | 250 kbps |
| Sensitivity | -97 dBm | 
| TX Power | +4.5 dBm |
| Antenna | PCB Antenna |
| Interface | USB 2.0 |
| Software Support | Wireshark, Attify Zigbee Framework |
| OS Support | Linux, Windows, macOS |
| Power | USB Bus Powered |
| Dimensions | 19 x 14 x 6 mm |
| Certifications | CE, FCC, RoHS |

- Attify Zigbee framework (compatible with Apimote dongle)

### Software Installation:

1. **Clone Attify Zigbee Framework:**
   ```shell
   git clone https://github.com/attify/attify-zigbee.git
   cd attify-zigbee
   sudo python setup.py install
   ```

2. **Install Dependencies:**
   ```shell
   sudo apt-get install python-pip
   sudo pip install pyserial cython jsonpickle
   ```

3. **Plug in Apimote Zigbee Sniffer Dongle:**
   Connect the Apimote dongle into an available USB port on your machine.

4. **Assign Permissions:**
   ```shell
   sudo chmod 666 /dev/ttyACM0
   ```

5. **Verify Installation:**
   ```shell
   zigbee-sniffer --channel 11
   ```

   This command initiates scanning on channel 11. Now you can utilize the Attify Zigbee framework to test Zigbee devices, capture traffic, and more.

---

*Note: Ensure you have the necessary permissions and privileges while performing these operations. For any issues or troubleshooting, refer to the official documentation of Attify Zigbee framework and Apimote Zigbee Sniffer/Analyzer.*

---

Certainly! Here are the detailed steps to configure an Apimote with Zigbee for penetration testing, presented in a Markdown format:

---

# Configuring Apimote with Zigbee for Penetration Testing

1. **Get Apimote Device:**
   - Acquire an Apimote device with a built-in Zigbee radio or support for a USB Zigbee dongle. Ensure compatibility by referring to the KillerBee wiki for a list of compatible USB radios.

2. **Flash OpenWrt Firmware:**
   - If not pre-installed, flash the Apimote with the latest OpenWrt firmware:
   ```shell
   wget http://downloads.openwrt.org/releases/18.06.2/targets/ar71xx/generic/openwrt-18.06.2-ar71xx-generic-apm821xx-squashfs-sysupgrade.bin
   sysupgrade -n -F openwrt-18.06.2-ar71xx-generic-apm821xx-squashfs-sysupgrade.bin
   ```
## Here are the steps to flash OpenWrt firmware on an Apimote device:

1. Download the latest OpenWrt firmware for your Apimote model from the OpenWrt website. For example, for the Apimote EVB:

```bash
wget http://downloads.openwrt.org/releases/18.06.2/targets/ramips/mt7621/openwrt-18.06.2-ramips-mt7621-apm821xx-squashfs-sysupgrade.bin
```

2. Connect to the Apimote via SSH or serial console. You may need to use default credentials if this is your first time flashing it.

3. Use the sysupgrade utility to flash the new firmware: 

```bash
sysupgrade -n -F openwrt-18.06.2-ramips-mt7621-apm821xx-squashfs-sysupgrade.bin
```

4. This will install the new firmware while preserving the existing configuration. 

5. Wait several minutes for the process to complete. The Apimote will reboot into the new OpenWrt firmware.

6. You can check it was successful by verifying the version:

```bash 
cat /etc/openwrt_version
```

Now the Apimote is ready for further configuration and pentesting software installation. Just be sure to select the correct firmware file for your Apimote model when downloading.

3. **Install Prerequisite Packages:**
   ```shell
   opkg update
   opkg install tcpdump usbutils coreutils-hexdump swig python-dev python-pip libusb-1.0
   ```

4. **Install Tools:**
   ```shell
   pip install killerbee rfcat_zigbee
   git clone https://github.com/woz-u-cs-course/z3sec-tool.git
   ```

5. **Configure Zigbee Radio:**
   - Enable promiscuous mode on the Zigbee radio using `iwconfig`, `iwpriv` commands, or KillerBee's `zbsetpromiscuousmode` tool.

6. **Scan for Zigbee Networks:**
   - Utilize KillerBee's `zbstumbler` or `zbfind` tool to scan for nearby Zigbee networks.

7. **Capture and Extract Encryption Keys:**
   ```shell
   tcpdump -i wlan1 -w capture.pcap
   tshark -r capture.pcap -Y zigbee_nwk_key -T fields -e zigbee.nwk.key > nwk_key
   ```

8. **Decrypt Packets:**
   - Use the extracted key to decrypt packets with KillerBee:
   ```shell
   zbdecrypt -k nwk_key -r capture.pcap
   ``` 

9. **Perform Attacks:**
   - Conduct various attacks like replay attacks, DoS, etc., using KillerBee's `zbreplay`, `zbjammer` tools.

10. **Analyze with Wireshark:**
   - Use Wireshark on the controlling computer to analyze packets and attack results throughout testing.

---

*Ensure that you have proper authorization and adhere to ethical guidelines when conducting penetration testing activities.*
