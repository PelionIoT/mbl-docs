<!-- assuming this happened?-->
# Firmware updates in Mbed Linux OS

MBL can perform a **firmware over the air** (FOTA) update of:
<!--we call the process "updating the image", so I'm wondering whether this list should be split into image (with three sub-areas) and app--><!--I think that would fit nicely with the rest of the docs.-->
* The MBL boot components - including Trusted Firmware, OP-TEE and U-Boot.
* The Linux kernel image, Linux Device Tree, U-Boot boot script and initramfs.
* The MBL root file system.
* Any application running on the device.
<!--Are there places we can link these? For example, I'm not sure "the MBL boot components" is immediately clear.-->

<span class="notes">**Note**: Currently, MBL does **not** support updating multiple components in a single FOTA update. To update multiple components, you need to run multiple updates.</span>

## How software is updated

<!-- Needs to be updated with new update components, separated into 2 like now, apps on one side, everything else on another -->

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/update_process.png" width="50%" align="right" />

MBL uses Pelion Device Management to manage firmware updates. Before you start, use the `create-update-payload` tool to create a new version of your firmware to your Device Management account. This is your payload, and it is a `.tar` file that can contain either:
<!--Is the "either" clear enough that the list below is one or the other? Or should we add an "Or" between the top two bullets?-->

* An application update: one or more OPKG packages (`.ipk` files), as explained in [Updating an application](../update/updating_an_application.html).
* An MBL image, as explained in [updating an MBL image](../update/updating-an-mbl-image.html), which can contain an updated version of one of:
    * A `rootfs` update: a compressed `tar` file that contains the root file system content.
    * A boot component update: either of the two boot component binaries. The first boot component usually contains the TF-A bootloader stage two (BL2). The second boot component usually contains the TF-A bootloader stage three (BL3), OP-TEE and U-boot.
    * A kernel update: a binary containing the Linux kernel image, the associated device tree blob, U-Boot boot script and initramfs.
<!--this mostly repeats the opening paragraph; should find a way to merge--><!-- +1. I'll play with it.-->
<!--at what point did I upload my payload, and what tool did I use for that?-->
Start the update process by initiating an update campaign; Device Management then sends the device an update request with a manifest detailing what needs to be updated. If the device accepts the request, Device Management sends your uploaded payload file to the device and monitors the update process.

**Tip:** Refer to the Pelion Device Management documentation for a [full review of the update process](https://www.pelion.com/docs/device-management/current/updating-firmware/index.html).
<!--Does it make sense to have/drive anyone else crazy having a tip and a note right next to each other?-->

<span class="notes">**Note:** MBL relies on Device Management to validate updates. For more information, [see the Security in firmware update section of the Device Management documentation](https://www.pelion.com/docs/device-management/latest/updating-firmware/security.html).</span>

### Application updates

After receiving a payload file containing application updates, for each application update, MBL:

1. Installs the version of the application contained in the update payload.
1. Stops the version of the application already running on the system.
1. Runs the newly installed version of the application.
1. Removes the previously installed version of the application.

The previously installed version of an application is removed <!--By what, MBL in general?--> only if the new version can be successfully installed and run <!--By MBL? By you, the user?-->. <!--Also, could we move the previous sentence to No. 3? Or add a number between 3 and 4? Something like, "1. Checks the new version installs and runs successfully. 1. Removes the previously..."? What do you think about making the next sentences a note?--> If the update payload contains multiple applications, the previously installed versions are removed only if *all* applications are successfully installed and run. If any application in the update payload fails to install or run, all newly installed application versions are removed, and the previously installed versions are restarted.<!--Holy passive voice, Batman.-->

### Root file system updates

To support `rootfs` updates, MBL devices have two root partitions:

* **Active**: a partition containing the root file system for the running system.
* **Inactive**: a partition available to receive a `rootfs` update payload.

After receiving a payload file containing a `rootfs` update, MBL:

1. Writes the contents of the root file system update payload to the inactive partition.
1. Flips a flag indicating which root file system partition is active, so that after a reboot the previously inactive partition becomes the active one, and vice versa.<!--I don't like "vice versa," but I'm struggling to come up with something better right now.-->
1. Reboots the device.
1. Mounts the root file system from the update payload.

The previously active (now inactive) root file system partition is now ready to receive the next `rootfs` update.

### Boot and kernel components updates

<span class="notes">**Note:** Currently these components are updated in place, so they are not robust to power failures or file corruption. This will be addressed in future releases.</span>

After receiving a payload file containing a boot component or kernel update binary, MBL:

1. Overwrites the contents of the corresponding partition <!-- link to the partions docs--> containing the boot component or kernel.
1. Reboots the device.
