# Supported hardware

Supported boards:

* TechNexion [PICO-PI-IMX7-IG development kit](https://shop.technexion.com/system-on-modules/pico/pico-evk/pico-pi-imx7-1g.html), which is made up of a PICO-PI-GL baseboard and a [PICO-IMX7D-10-R10-E08-9377 System-on-Module](https://shop.technexion.com/system-on-modules/pico/pico-modules/pico-imx7d-10-r10-e08-9377.html) (SoM). Referred to as the PICO-PI with IMX7D. You may also need one micro-USB cable, and one USB-C cable.

* [NXP i.MX 8M Mini LPDDR4 Evaluation Kit](https://www.nxp.com/support/developer-resources/software-development-tools/i.mx-developer-resources/evaluation-kit-for-the-i.mx-8m-mini-applications-processor:8MMINILPD4-EVK?tid=vanimx8mminievk). Referred to as the NXP 8M Mini EVK. This kit comes with power supply, micro-USB cable and USB-C cable, but you will need a micro-SD card (at least 4GB).

* [NXP Warp7](https://www.nxp.com/support/developer-resources/nxp-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7). You also need two micro-USB cables.
* [Raspberry Pi 3 models B or B+](https://www.raspberrypi.org/products/), with:
    * A micro-SD card (at least 4GB).
    * An C232HD-DDHSP-0 cable to connect the GPIO to a PC's USB (for debugging).
    * If you want to use MBL CLI: a USB to network adaptor and an RJ45 network cable (to convert from RPi3 USB host port to network). We've tested with the TP-Link UE300.

         An optional second USB network adaptor if you want to convert back to USB (to connect to your PC over USB rather than network).

    <span class="warnings">**Warning:** The Raspberry Pi 3 is suitable for development only. Do not use it for production.</span>

## Hardware features

A basic comparison of features of the supported board:

| Name | SoC | CPUs | Memory | Storage | Wi-Fi | Ethernet | Bluetooth | USB | Cellular | Security |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PICO-PI with IMX7D | NXP i.MX7D | Arm 2x Cortex-A7, 1x M4 | 1GB | 8GB eMMC | 802.11 a/b/g/n/ac | gigabit | 4.1* | gadget, 1x USB-A | via USB module | High Assurance Boot, OPTEE |
| NXP 8M Mini EVK | NXP i.MX8M mini | Arm 4x Cortex-A53, 1x M4 | 2GB | 16GB eMMC*, SD-card | 802.11 a/b/g/n/ac* | gigabit | 4.1* | gadget | - | High Assurance Boot, OPTEE |
| NXP Warp7 | NXP i.MX7S | Arm 1x Cortex-A7, 1x M4 | 4GB | 8GB eMMC | 802.11 b/g/n | - | 4.1* | gadget | - | High Assurance Boot, OPTEE |
| Raspberry Pi 3B+ | Broadcom BCM2837B0 | Arm 1x Cortex-A53 | 1GB | SD-card | 802.11 b/g/n/ac | gigabit | 4.2* | 4x USB-A | via USB module | - |

<span class="notes">**Note:** The entries marked with an asterisk (`*`) are not yet fully supported or tested in Mbed Linux OS.</span>
