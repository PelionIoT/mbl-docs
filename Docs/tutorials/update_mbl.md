## Updating Mbed Linux OS

You can perform an over the air (OTA) update for the following components of Mbed Linux OS (MBL):
* Root file system
* Application

<!--currently - so we'll support updating the bootloader? I thought you don't update bootloaders because you can recover from an error in that update-->

### How software is updated

Send software updates to MBL devices using Pelion Device Management Portal. The update process begins when Device Management sends the device a request with a manifest. If the device accepts the request, Device Management sends a payload file containing an update for one or more MBL component.
<!---This doesn't actually really tell me anything about how to do an update. What do I need to do, and what does Portal do itself?--->
The payload <!---I don't like this word. We don't use this anywhere else.---> is a `.tar` file that can contain either:
* An application update: One or more OPKG packages (`.ipk` files).
* A `rootfs` update: A compressed `tar` file called `rootfs.tar.xz` <!--compressed `tar` within a tar?-->that contains root file system content.

Currently, MBL does **not** support updating both an application and a root file system in a single OTA update.

<span class="tips">**Tip:** Refer to the documentation for a [full review of the update process](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).</span>

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

* Writes the contents of `rootfs.tar.xz` to the inactive partition;
* Flips a flag indicating which root file system partition is active, so that after a reboot the previously inactive partition will become the active partition, and vice versa;
* Reboots the device; *and*
* Mounts the root file system from the update payload.

The previously active (now inactive) root file system partition is now available to receive the next `rootfs` update.

<!--There's a question of how much of the theory should be explained here (and in the previous tutorial).-->

### Prerequisites

An MBL device can only be updated if the followings are available:
<!--Leave the links empty; that will make them surface in our final checks and we'll know to fix them-->

* The build artifacts for the image to send as the update payload. See the [build tutorial]() for instructions.
* An internet connection on the device. [Follow the tutorial to setup an internet connection]().
* A device running an MBL image that can connect to a Pelion account. [Follow the first tutorial in this series]() to request an account.
* The directory in which the manifest tool was initialized, [as reviewed in the development environment setup]().
<span class="notes">This *must* be the directory from which the `update_default_resources.c` file was obtained for building MBL.</span>
* A Pelion API key, to use the manifest tool from the command line. Follow the instructions in the [Pelion docs](https://cloud.mbed.com/docs/current/integrate-web-app/api-keys.html#generating-an-api-key) to obtain an API key (when prompted to select a group to set the API key access level, select **Developers**). Make a note of the API key to use it later; for security reasons, the portal will not display it again.

### Workflow

1. Create an update payload file:

    * **For an application**: Make a `tar` file containing the `.ipk` files for the applications to update.

        Note that the `.ipk` files must be in the `.tar` 's root directory, not a subdirectory.

        For example, to create an update payload file at `/tmp/payload.tar` containing an `.ipk` file with the path `/home/user01/my_app.ipk`, run:

        ```
        user01@dev-machine:~$ tar -cf /tmp/payload.tar -C /home/user01 my_app.ipk
        ```

    * **For a root file system**: Make a `tar` file containing the root file system archive from the MBL build artefacts.

        A symlink to the root file system archive can be found in the build environment at `<mbl_workspace>/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz`

        Where:

        * `<mbl_workspace>` is the directory in which the `repo init` command created the MBL workspace.
        * `<MACHINE>` is the value that was given to the build script for the `--machine` option. See the [build tutorial]() to determine which value is suitable for the device in use.

        <span class="notes">**Note:** The file inside the update payload must be named `rootfs.tar.xz` and must be in the tar's root directory, not a subdirectory.</span>

        For example, to create an update payload file at `/tmp/payload.tar` containing a Warp7 root file system image `tar` file with the path `build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-console-image-imx7s-warp-mbl.tar.xz`, run:

        ```
        user01@dev-machine:~$ tar -cf /tmp/payload.tar -C /home/user01/mbl/mbl-os-0.5/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl '--transform=s/.*/rootfs.tar.xz/' --dereference mbl-console-image-imx7s-warp-mbl.tar.xz
        ```

        The `--transform` option renames all files added to the payload to `rootfs.tar.xz` and the `--dereference` option is used so that `tar` adds the actual root file system archive file rather than the symlink to it.

1. Find the device ID in the `mbl-cloud-client` log file at `/var/log/mbl-cloud-client.log`, using the following command on the device's console:
       ```
       root@mbed-linux-os-1234:~# grep -i 'device id' /var/log/mbl-cloud-client.log
       ```
  * Go to [Device Management Portal](https://portal.mbedcloud.com) > **Device Directory** to find the device ID. This is a valid option if only one device is registered or if each device has been assigned a descriptive name in Portal.
1. For a `rootfs` update, identify which root file system partition is currently active to compare it to the active partition after the update. The `lsblk` command explained [here](#identify-the-active-root-file-system-partition) can be used to that effect.
1. Change the current working directory to the directory where the manifest tool was initialized.
1. Run the following command:
    ```
    user01@dev-machine:manifests$ manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
    ```
    Where:
        * `<device-id>` is the device ID.
        * `<payload-file>` is the update payload (`.tar` file).
        * `<api-key>` is the Pelion API key.

#### Additional notes

The update payload download progress can be monitored by tailing the `mbl-cloud-client` log file as follows:
    ```
    root@mbed-linux-os-1234:~# tail -f /var/log/mbl-cloud-client.log
    ```
* The update payload installation progress can be monitored by tailing:
 * `/var/log/arm_update_activate.log`: for messages about the overall progress of the installation, and messages specific to `rootfs` updates.
 * `/var/log/mbl-app-update-manager.log`: for messages about application updates.

* For `rootfs` updates, a reboot will be automatically initiated to boot into the new firmware. Identify the currently active root file system and verify it was different pre-update. The `lsblk` command explained [here](#identify-the-active-root-file-system-partition) can be used to that  effect.

* The system does not reboot after an application update.

### Identify the active root file system partition

<!--I moved this; it didn't belong in the intro-->
<!--when and why would I want to do this?-->
To find out which root file system partition is currently active:
```
root@mbed-linux-os-1234:~# lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
```

This command prints the label of the block device currently mounted at `/`, which will be `rootfs1` if the first root file system partition is active or `rootfs2` if the second root file system partition is active.

For example, when the first root file system partition is active:
```
root@mbed-linux-os-1234:~# lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
rootfs1
root@mbed-linux-os-1234:~#
```
