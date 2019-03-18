# Supported hardware

Supported boards:

* TechNexion [PICO-PI-GL baseboard](https://shop.technexion.com/system-on-modules/pico/pico-baseboards/picopigl.html) with a [PICO-IMX7D-10-R10-E08-9377 System-on-Module](https://shop.technexion.com/system-on-modules/pico/pico-modules/pico-imx7d-10-r10-e08-9377.html) (SoM). Referred to as the PICO-PI with IMX7D. You also need one micro-USB cable, and one USB-C cable.

    <span class="notes">This platform does not yet support WiFi communication.</span>

* [NXP i.MX 8M Mini LPDDR4 Evaluation Kit](https://www.nxp.com/products/processors-and-microcontrollers/arm-based-processors-and-mcus/i.mx-applications-processors/i.mx-8-processors/i.mx-8m-mini-family-arm-cortex-a53-cortex-m4-audio-voice-video:i.MX8MMINI). Referred to as the NXP 8M Mini EVK. This kit comes with power supply, micro-USB cable and USB-C cable, but you will need a micro-SD card (at least 4GB).

    <span class="notes">This platform is still in development and is only partially supported at the moment. The boot partitioning has not been finished and Wi-Fi communication is not working yet.</span>

* [NXP Warp7](https://www.nxp.com/support/developer-resources/nxp-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7). You also need two micro-USB cables.
* [Raspberry Pi 3 models B or B+](https://www.raspberrypi.org/products/), with:
    * A micro-SD card (at least 4GB).
    * An C232HD-DDHSP-0 cable to connect the GPIO to a PC's USB (for debugging).
    * If you want to use MBL CLI: a USB to network adaptor and an RJ45 network cable (to convert from RPi3 USB host port to network). We've tested with the TP-Link UE300.

         An optional second USB network adaptor if you want to convert back to USB (to connect to your PC over USB rather than network).

    <span class="warnings">**Warning:** The Raspberry Pi 3 is suitable for development only. Do not use it for production.</span>
