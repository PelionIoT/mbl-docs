# Building an Mbed Linux OS image

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

This tutorial builds and installs an Mbed Linux OS (MBL) image on a device. This image includes credentials to connect to your Pelion Device Management account, but does not include a user application (which is covered [in the Hello World tutorial](../develop-apps/hello-world-application.html)).

The full process is:

* [Setting up your development environment](../first-image/setting-up-and-supported-hardware.html), if you haven't already done so.
* [Preparing Device Management sources](../first-image/provisioning-for-pelion-device-management.html). You will need these later, when you try to connect to your Device Management account.
* [Building the Mbed Linux OS image](../first-image/building-an-mbl-image.html) with a build script.
* [Writing the image to the device](../first-image/writing-and-booting-the-disk-image.html).

You should then continue to the next tutorial, where you will:

* [Set up a network connection](../first-image/connecting-to-a-network-and-pelion-device-management.html) over Ethernet or Wi-Fi.
* [Verify that the device is connected to Device Management](../first-image/verifying-that-the-device-is-connected-to-device-management.html).
