<h2 id="mbl-pelion-connect">Building an Mbed Linux OS image and connecting to Pelion Device Management</h2>

This tutorial builds and installs an Mbed Linux OS (MBL) image on a device. This image includes credentials to connect to your Pelion Device Management account, but does not include a user application (that is covered [in the Hello World tutorial](../getting-started/tutorial-user-application.html)).

<span class="notes">Each release has its own branch, such as `mbl-os-0.5`. Throughout this guide, the release branch is referred to as `mbl-XXX`. Replace it with the name of the branch you're working with.</span>

The full process is:

* [Setting up your development environment]().
* [Prepare Device Management sources](preparing-device-management-sources.html).
* [Building the image](building-an-mbl-image.html) with a build script.
* [Writing and booting the disk image](writing-and-booting-the-disk-image.html).
* [Setting up a network connection](setting-up-a-network-connection.html) over Ethernet or Wi-Fi.
* [Verifying that the device is connected to Device Management](verifying-that-the-device-is-connected-to-device-management.html).

<span class="notes">**Note:** These instructions use the developer workflow and certificates as connectivity credentials. Do not use these credentials for production devices.</span>
