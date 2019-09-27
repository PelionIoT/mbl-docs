# Provisioning and updating

## Provisioning for Pelion Device Management

MBL CLI can provision your device with all the certificates and keys it requires to connect to your Pelion Device Management account. For detailed instructions, please see [the image installation section](../first-image/provisioning-for-pelion-device-management.html).

Note that you do not need to provision each application on its own; provisioning is done for the device as a whole.

## Device application update

You can use [MBL CLI](../update/updating-an-application.html#using-mbl-cli) to install or update an existing application over a USB connection.

MBL uses `mbl-firmware-update-manager` to manage updates. To see all arguments, use `mbl-firmware-update-manager -h`. For detailed instructions, as well as reference information about device update, please see the [Updating devices section](../update/index.html)
