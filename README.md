<p align="center">
  <img  width="180" src="ghost.png" />
</p>

---

# KillerBee Toolkit for ZigBee Network Penetration Testing

KillerBee is a powerful toolkit designed for sniffing and attacking ZigBee networks. It is crucial to use this tool responsibly and legally, ensuring you have proper authorization before testing any network, even if it's your own. This guide provides instructions on how to install the KillerBee firmware and use it for ethical penetration testing.

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Installing Dependencies](#installing-dependencies)
- [Installing KillerBee](#installing-killerbee)
- [Flashing the Firmware](#flashing-the-firmware)
- [Verify the Installation](#verify-the-installation)
- [Getting Started](#getting-started)

## Hardware Requirements

- A compatible USB dongle (e.g., CC2531 USB dongle).
- A computer with Linux (preferably Kali Linux) installed.

## Installing Dependencies

Before installing KillerBee, ensure you have the necessary dependencies. Open a terminal and run:

```bash
sudo apt-get update
sudo apt-get install python3-pip libusb-1.0-0-dev libpcap-dev
```

## Installing KillerBee

Install KillerBee using pip, the Python package manager. Run the following commands in the terminal:

```bash
sudo pip3 install pyserial pyusb
git clone https://github.com/riverloopsec/killerbee.git
cd killerbee
sudo python3 setup.py install
```

## Flashing the Firmware

1. Connect your compatible USB dongle to your computer.
2. Navigate to the KillerBee tools folder: `cd tools`.
3. Run `sudo ./install-udev-rules` to set up udev rules for the USB dongle.
4. Run `kb_install_firmware` to install the appropriate firmware for your device. Follow the prompts to complete the installation.

Here are the complete steps for flashing the firmware on a wireless device using KillerBee tools:

1. Connect your compatible USB wireless dongle (such as the Alpha AWUS036H) to your computer.

2. Open a terminal and navigate to the KillerBee tools directory:

```bash
cd path/to/killerbee/tools
```

3. Set up the udev rules for the dongle:

```bash 
sudo ./install-udev-rules
```

4. Flash the appropriate firmware for your device:

```bash
sudo ./kb_install_firmware
```

5. Follow the prompts to select your device and firmware. For the AWUS036H, you would likely select `1` for the RTL8187L chipset.

6. When prompted, press `I` to confirm flashing the firmware. 

7. Once complete, disconnect and reconnect the device for the new firmware to take effect.

8. Your wireless dongle should now be ready with the appropriate firmware for using KillerBee and other wireless tools.

## Verify the Installation

To verify that KillerBee is installed correctly, run the following command in the terminal:

```bash
kb --version
```

This command should display the version of KillerBee you installed, indicating a successful installation.

## Getting Started

You can now use KillerBee tools to perform various tasks related to ZigBee network penetration testing. Always ensure you have proper authorization and comply with all laws and regulations while using this tool.

