## Connecting the device

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
