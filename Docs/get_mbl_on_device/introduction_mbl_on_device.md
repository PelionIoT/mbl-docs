# Installing MBL on a new device

<!--this is the same text as the app developers intro...-->

MBL applications are not compiled together with the MBL codebase or with any Pelion Device Management credentials (unlike Mbed OS, where the codebase, credentials and application form a single binary). This method allows you to deploy and manage multiple applications on a single device.

1. Review our list of [supported development boards]().
1. Get an MBL image:
    * You can use [our evaluation image]().
    * You can [build your own](). You will need to [set up a full development environment for this]().
1. [Flash the image to the device]().
1. Provision the device with [Pelion Device Management credentials and an API key]() so that it can connect to your Device Management account.<!--I think this now happens after the flashing, but I could be wrong-->
1. [Set up your network connection]() and [test your Device Management connectivity]().
