# Mbed Linux OS Example - QR Scanner

<span class="notes">**Note**: Mbed Linux OS (MBL) is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

This tutorial creates a Python user application that runs on MBL devices and scans QR codes. By default, the application decodes the QR code to UTF-8 values and saves these values to a log. If you want, you can use the application to upload the decoded QR codes to an Arm Treasure Data account or save it to the device's persistent memory. This reference demonstrates how easily a standalone application can be adjusted to work on MBL, and how to use the configuration file to give a dockerized application access to device peripherals.

## Requirements and hardware notes

* This is a standard Python application, and can run on any operating system with Python 3. As an Mbed Linux OS IoT application, it runs on a Raspberry Pi 3.
* Your device must already be running an MBL image. Please [follow the tutorial](../first-image/index.html) if you don't have an MBL image yet.
* You must still have all [tool and accounts prerequisites](../first-image/setting-up-and-supported-hardware.html).
* Cameras:
    * This application does not work with the Raspberry Pi 3 camera.
    * We recommend using the Logitech webcam series.

## License

Please see the [License](https://github.com/ARMmbed/mbl-example-qr/blob/master/LICENSE.md) document on GitHub for more information.

## Contributing

If you want to contribute additional functionality or fixes to this application, please see the [Contribution guidelines](../references/contribution-guidelines.html).
