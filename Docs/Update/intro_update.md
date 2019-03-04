# Firmware updates in Mbed Linux OS

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

You can perform a **firmware over the air** (FOTA) update for:

* The MBL root file system.
* Any application running on the MBL device.

## How software is updated

MBL uses Pelion Device Management to update firmware. <!--this is clunky-->After you upload a new version of your firmware (for one or more components) to your Device Management account, you can start the update process: Device Management sends the device an update request with a manifest. If the device accepts the request, Device Management sends your uploaded payload file.

The payload is a `.tar` file that can contain either:

* An application update: One or more OPKG packages (`.ipk` files).
* A `rootfs` update: A compressed `tar` file called `rootfs.tar.xz` that contains root file system content.

Currently, MBL does **not** support updating both an application and a root file system in a single FOTA update.

<span class="tips">**Tip:** Refer to the Pelion Device Management documentation for a [full review of the update process](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).</span>

### Application updates

After receiving a payload file containing application updates, for each application update, MBL:

* Installs the version of the application contained in the update payload
* Stops the version of the application previously running on the system.
* Runs the newly installed version of the application.
* Removes the previously installed version of the application.

Note: The previously installed version of an application is removed only if the new version can be successfully installed and run. In case of the update payload containing multiple applications, the previously installed versions are removed only if and only if all applications are successfully installed and run. In case of failure to run or install any application in the update payload, all newly installed application versions are removed and the previously installed versions are restarted. 


### Root file system updates

To support `rootfs` updates, MBL devices have two root partitions called:

* **Active**: a partition containing the root file system for the running system.
* **Inactive**: a partition available to receive a `rootfs` update payload.

After receiving a payload file containing a `rootfs` update, MBL:

* Writes the contents of `rootfs.tar.xz` to the inactive partition.
* Flips a flag indicating which root file system partition is active, so that after a reboot the previously inactive partition becomes the active one, and vice versa.
* Reboots the device.
* Mounts the root file system from the update payload.

The previously active (now inactive) root file system partition is now ready to receive the next `rootfs` update.
