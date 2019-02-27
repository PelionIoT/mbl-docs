# Raspberry Pi 3 devices

1. Connect a micro-SD card to your PC. You now see:

    * The SD card device file in `/dev`, probably as `/dev/sdX` for some letter `X` (for example, `/dev/sdd`).
    * Device files for its partitions. `/dev/sdXN` for the same letter `X` and some numbers `N` (for example, `/dev/sdd1` and `/dev/sdd2`).

    <span class="notes">In the commands below, replace `/dev/sdX` with the device file name for the SD card _without_ a number at the end. You can use `lsblk` to identify the name of the SD card device.</span>

1. Ensure none of the micro-SD card's partitions are mounted (replace `/dev/sdX` as explained above):

    ```
    sudo umount /dev/sdX*
    ```

1. Write the disk image to the SD card device - not a partition on it (replace `/dev/sdX` as explained above):

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.gz /dev/sdX`</span>

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. Detach the micro-SD card from your PC, and plug it into the Raspberry Pi 3.
1. You need access to the device's console, so before powering it on, either:

    * Connect it to a monitor and keyboard (using its HDMI and USB sockets).
    * Connect it to your PC. For example, if you're using a [C232HD-DDHSP-0](http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_C232HD_UART_CABLE.pdf) cable, use [this pin numbering reference](https://www.element14.com/community/servlet/JiveServlet/previewBody/73950-102-10-339300/pi3_gpio.png) and connect USB-UART colored wires:
       * **Grounds** wire (usually black) to pin **06**.
       * **Tx** wire (usually yellow) to pin **08**.
       * **Rx** wire (usually orange) to pin **10**.

    Connect the other end of the C232HD-DDHSP-0 cable to the USB port on your PC.

    The cable's TX and RX are used to communicate with the board.

1. After connecting the Raspberry Pi 3, from your PC, run the following command to connect to the device's serial console:

    ```
    minicom -D /dev/ttyUSB0
    ```

    Use the following settings:

    * Baud rate: 115200.
    * Encoding: [8N1](https://en.wikipedia.org/wiki/8-N-1).
    * No hardware flow control (enabled by default).

1. Connect the Raspberry Pi 3's micro-USB socket to a USB power supply.

    The device now boots into MBL.
