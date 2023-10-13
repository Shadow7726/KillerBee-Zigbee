<p align="center">
  <img  width="180" src="ghost.png" />
</p>

--- 

## Here are the compatible USB dongles and hardware options for the Attify Zigbee Framework (AZF)

| Hardware | Description | Buy |
|-|-|-|  
| [Texas Instruments CC2531 USB Dongle](https://www.ti.com/tool/CC2531EMK) | The recommended and most widely used option. Provides an IEEE 802.15.4 compatible radio and USB interface. | [Amazon](https://www.amazon.com/gp/product/B07J681QQN) |
| [Texas Instruments CC2530 Evaluation Module Kit](https://www.ti.com/tool/CC2530EMK) | More advanced Zigbee development board with CC2530 SOC. | [Mouser](https://www.mouser.com/ProductDetail/Texas-Instruments/CC2530EMK?qs=vfohlfHL41H4F6%2FsribqxA%3D%3D) |
| [Texas Instruments CC26X2R1 LaunchPad](https://www.ti.com/tool/LAUNCHXL-CC26X2R1) | Newer LAUNCHXL-CC26X2R1 kit with CC26x2R1 SOC and Zigbee support.| [Digi-Key](https://www.digikey.com/en/products/detail/texas-instruments/LAUNCHXL-CC26X2R1/9695668) |
| [Great Scott Gadgets Yard Stick One](https://greatscottgadgets.com/yardstickone/) | Lower cost 802.15.4 transceiver, limited AZF support. | [Hacker Warehouse](https://hackerwarehouse.com/product/yard-stick-one-ys1/) |
| [Hak5 WiFi Pineapple Tetra](https://shop.hak5.org/products/wifi-pineapple?variant=81044992) | Add-on module for WiFi Pineapple with 802.15.4 radio, needs custom firmware.| [Hak5 Store](https://shop.hak5.org/products/wifi-pineapple?variant=81044992) |
| [ubertooth.org Ubertooth One](https://ubertooth.github.io/index.html) | Primarily BLE focused but can sniff some 2.4GHz signals. Limited AZF functionality. | [Hacker Warehouse](https://hackerwarehouse.com/product/ubertooth-one/) |
| [Atmel/Microchip ATUSB IEEE 802.15.4 Dongle](https://www.microchip.com/DevelopmentTools/ProductDetails/ATAVRRZUSB) | Similar to CC2531, based on AT86RF230 transceiver. | [Digi-Key](https://www.digikey.com/en/products/detail/microchip-technology/ATAVRRZUSB-XPRO/9262180) |

In general, any USB dongle or development board containing an 802.15.4 compatible transceiver chip like the CC2520, CC2530, CC2531, or compatible RF chip should potentially work with AZF. The CC2531 dongle has the best out-of-the-box compatibility and requires no extra configuration. Always refer to the AZF documentation for compatibility details.


Here is a more detailed description and references for each Zigbee hardware tool to aid further research:

## Hardware Tools

### USB Sniffers

- [USB Zigbee Sniffer](https://www.amazon.com/USB-Zigbee-Sniffer-Devices-CC2531/dp/B07Q6YPFR8) - Low-cost USB sniffer using the CC2531 chipset to intercept and analyze Zigbee traffic. Useful for reconnaissance and sniffing.

- [TI CC2531 USB Sniffer](https://www.txinstruments.com/product/cc2531-usb-dongle/) - USB sniffer from Texas Instruments using the CC2531 chip. Can sniff non-encrypted Zigbee networks. Refer [documentation](http://www.ti.com/lit/ds/symlink/cc2531.pdf).

- [Sysmocom Flossy SDR](https://sysmocom.de/products/flossy/) - Software radio peripheral capable of sniffing, analyzing, and transmitting on sub-GHz frequencies including Zigbee. Details [here](https://osmocom.org/projects/flossy/wiki/ZigBee).

### Development Kits

- [Silicon Labs EFR32MG](https://www.silabs.com/wireless/zigbee/development-tools/efr32mg-zigbee-kits) - Wireless SoC kits for developing Zigbee solutions using Silicon Labs chips. Refer [datasheet](https://www.silabs.com/documents/public/data-sheets/efr32mg-datasheet.pdf).

- [NXP JN516x](https://www.nxp.com/products/wireless-connectivity/thread-zigbee/zigbee-3-0-modules-and-tooling:JN-SW-TOOL) - Zigbee 3.0 toolkit with NXP JN516x chips. Overview [here](https://www.nxp.com/products/wireless-connectivity/thread-zigbee/zigbee-3-0-modules-and-tooling:JN-SW-TOOL).

- [Atmel/Microchip Zigbee](https://www.microchip.com/DesignCenters/Wireless-Connectivity/Embedded-Zigbee) - Atmel/Microchip Zigbee solutions like Zigbit kits. Documentation [here](https://www.microchip.com/wwwproducts/en/ATZB-24-A2).

### Gateways/Coordinators

- [CC2531 USB Stick](https://www.zigbee2mqtt.io/information/supported_adapters.html#cc2531) - CC2531 based Zigbee coordinator. Interfaces with tools like Zigbee2MQTT.

- [ConBee II](https://www.zigbee2mqtt.io/information/supported_adapters.html#conbee_ii) - Zigbee coordinator by Dresden Elektronik. Supports [OTA firmware updates](https://github.com/dresden-elektronik/conbee2-firmware).

- [HubZ Zigbee](https://www.zigbee2mqtt.io/information/supported_adapters.html#hubz_zb) - Zigbee hub coordinators by Hubitat. Enables integration with Zigbee networks.  

### Misc Hardware

- [WiFi Pineapple](https://shop.hak5.org/products/wifi-pineapple) - Penetration testing device that can analyze and attack Zigbee with modules like Nexmon. Refer [documentation](https://docs.hak5.org/hc/en-us/categories/360002173534-WiFi-Pineapple).

- [YARD Stick One](https://greatscottgadgets.com/yardstickone/) - RF analysis tool capable of analyzing Zigbee networks. Details [here](https://github.com/greatscottgadgets/yardstick).  

- [HackRF](https://greatscottgadgets.com/hackrf/) - Software defined radio platform that can transmit/receive signals up to 6GHz. [Zigbee sniffing guide](https://github.com/mossmann/hackrf/wiki/ sniffzigbee).

- [Ubertooth](https://github.com/greatscottgadgets/ubertooth) - RF analysis tool focused on Bluetooth. Can be used for some Zigbee assessment. Overview [here](https://en.wikipedia.org/wiki/Ubertooth).

---

