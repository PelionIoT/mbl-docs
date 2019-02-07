## Firmware updates in Mbed Linux OS

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

You can perform a firmware an over the air (FOTA) update for the following components of Mbed Linux OS (MBL):

* The root file system.
* An application.

### How software is updated

Send software updates to MBL devices using Pelion Device Management Portal. The update process begins when Device Management sends the device a request with a manifest. If the device accepts the request, Device Management sends a payload file containing an update for one or more MBL component.

The payload is a `.tar` file that can contain either:

* An application update: One or more OPKG packages (`.ipk` files).
* A `rootfs` update: A compressed `tar` file called `rootfs.tar.xz` that contains root file system content.

Currently, MBL does **not** support updating both an application and a root file system in a single FOTA update.

<span class="tips">**Tip:** Refer to the Pelion Device Management documentation for a [full review of the update process](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).</span>

#### Application updates

After receiving a payload file containing application updates, for each application update, MBL:

* Stops any existing version of the application from running.
* Removes any existing version of the application.
* Installs the application update from the payload file.
* Starts the updated application.

#### Root file system updates

To support `rootfs` updates, MBL devices have two root partitions called:

* **Active**: a partition containing the root file system for the running system.
* **Inactive**: a partition available to receive a `rootfs` update payload.

After receiving a payload file containing a `rootfs` update, MBL:

* Writes the contents of `rootfs.tar.xz` to the inactive partition.
* Flips a flag indicating which root file system partition is active, so that after a reboot the previously inactive partition becomes the active one, and vice versa.
* Reboots the device.
* Mounts the root file system from the update payload.

The previously active (now inactive) root file system partition is now ready to receive the next `rootfs` update.
