# Installing MBL on a new device

<!--this is the same text as the app developers intro...-->

MBL applications are not compiled together with the MBL codebase or with any Pelion Device Management credentials (unlike Mbed OS, where the codebase, credentials and application form a single binary). This method allows you to deploy and manage multiple applications on a single device.

To install MBL on a new device:

1. Review our list of [supported development boards](../first-image/hardware.html) and set up your [development environment](../first-image/development-environment.html).
1. Get an MBL image:
    * You can use [our evaluation image]().<!--when will we have this?-->
    * You can [build your own](). You will need to [set up a full development environment for this](../first-image/building-an-mbl-image.html).
1. [Flash the image to the device](../first-image/writing-and-booting-the-disk-image.html).
1. Provision the device with [Pelion Device Management credentials and an API key](../first-image/pelion-device-management-accounts-and-certificates.html) so that it can connect to your Device Management account.
1. [Set up your network connection](../first-image/connecting-to-a-network-and-pelion-device-management.html) and [test your Device Management connectivity](../first-image/verifying-that-the-device-is-connected-to-device-management.html).
