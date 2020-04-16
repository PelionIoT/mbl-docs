# Firmware updates in Mbed Linux OS

You can perform a **firmware over the air** (FOTA) update for:

* The MBL root file system.
* Any application running on the MBL device.

<span class="notes">**Note**: Currently, MBL does **not** support updating both an application and a root file system in a single FOTA update.</span>

## How software is updated

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/update_process.png" width="50%" align="right" />

MBL uses Pelion Device Management to manage firmware updates. Before you start, upload a new version of your firmware (for one or more components) to your Device Management account. This is your payload, and it is a `.tar` file that can contain either:

* An application update: One or more OPKG packages (`.ipk` files).
* A `rootfs` update: A compressed `tar` file called `rootfs.tar.xz` that contains root file system content.

Start the process by initiating an update campaign; Device Management then sends the device an update request with a manifest detailing what needs to be updated. If the device accepts the request, Device Management sends your uploaded payload file to the device and monitors the update process.

**Tip:** Refer to the Pelion Device Management documentation for a [full review of the update process](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).


### Application updates

After receiving a payload file containing application updates, for each application update, MBL:

1. Installs the version of the application contained in the update payload.
1. Stops the version of the application already running on the system.
1. Runs the newly installed version of the application.
1. Removes the previously installed version of the application.

The previously installed version of an application is removed only if the new version can be successfully installed and run. If the update payload contains multiple applications, the previously installed versions are removed only if *all* applications are successfully installed and run. If any application in the update payload fails to install or run, all newly installed application versions are removed, and the previously installed versions are restarted.

### Root file system updates

To support `rootfs` updates, MBL devices have two root partitions:

* **Active**: a partition containing the root file system for the running system.
* **Inactive**: a partition available to receive a `rootfs` update payload.

After receiving a payload file containing a `rootfs` update, MBL:

1. Writes the contents of `rootfs.tar.xz` to the inactive partition.
1. Flips a flag indicating which root file system partition is active, so that after a reboot the previously inactive partition becomes the active one, and vice versa.
1. Reboots the device.
1. Mounts the root file system from the update payload.

The previously active (now inactive) root file system partition is now ready to receive the next `rootfs` update.

<span class="notes">**Note:** MBL relies on Device Management to validate updates. For more information, [see the Security in firmware update section of the Device Management documentation](https://www.pelion.com/docs/device-management/latest/updating-firmware/security.html).


***

Copyright © 2020 Arm Limited (or its affiliates)
