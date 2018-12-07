## Updating Mbed Linux OS

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

### Prerequisites

An MBL device can only be updated if the followings are available:

* The device is running an MBL image that can connect to a Pelion account. [Follow the first section in this series](../getting-started/accounts-and-certificates.html) to request an account.
* The build artefacts for the image to send as the update payload. See the [build tutorial](../getting-started/tutorial-building-an-image.html) for instructions.
* An internet connection on the device. [Follow the tutorial to set up an internet connection](../getting-started/tutorial-connecting-to-a-network-and-pelion-device-management.html).
* The directory in which the manifest tool was initialized, [as reviewed in the development environment setup](../getting-started/environment.html).

    <span class="notes">This *must* be the directory from which the `update_default_resources.c` file was obtained for building MBL.</span>

* A Pelion API key, to use the manifest tool from the command line. Follow the instructions in the [requirements section](..//getting-started/api-keys.html) to obtain an API key (when prompted to select a group to set the API key access level, select **Developers**). Make a note of the API key to use it later; for security reasons, the portal will not display it again.

### Workflow

1. Create an update payload file:
    * **For an application**: Make a `tar` file containing the `.ipk` files for the applications to update.

        Note that the `.ipk` files must be in the `.tar`'s root directory, not a subdirectory.

        For example, to create an update payload file at `/tmp/payload.tar` containing an `.ipk` file with the path `/home/user01/my_app.ipk`, run:
        ```
        user01@dev-machine:~$ tar -cf /tmp/payload.tar -C /home/user01 my_app.ipk
        ```
    * **For a root file system**: Make a `tar` file containing the root file system archive from the MBL build artefacts.

        A symlink to the root file system archive can be found in the build environment at `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.tar.xz`

        Where:

        * `/path/to/artifacts` is the output directory specified for all build artefacts. See the [build tutorial](../getting-started/building-an-mbl-image.html) for more information.
        * `<MACHINE>` is the value that was given to the build script for the `--machine` option. See the [build tutorial](../getting-started/building-an-mbl-image.html) to determine which value is suitable for the device in use.

        <span class="notes">**Note:** The file inside the update payload must be named `rootfs.tar.xz` and must be in the tar's root directory, not a subdirectory.</span>

        For example, to create an update payload file at `/tmp/payload.tar` containing a Warp7 root file system image `tar` file with the path `build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-console-image-imx7s-warp-mbl.tar.xz`, run:
        ```
        user01@dev-machine:~$ tar -cf /tmp/payload.tar -C /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image-test/images '--transform=s/.*/rootfs.tar.xz/' --dereference mbl-console-image-test-imx7s-warp-mbl.tar.xz
        ```
        The `--transform` option renames all files added to the payload to `rootfs.tar.xz` and the `--dereference` option is used so that `tar` adds the actual root file system archive file rather than the symlink to it.
1. Find the device ID in the `mbl-cloud-client` log file at `/var/log/mbl-cloud-client.log`, using the following command on the device's console:

    ```
    root@mbed-linux-os-1234:~# grep -i 'device id' /var/log/mbl-cloud-client.log
    ```

    If you only have one registered device, or if each devices has a been assigned a descriptive name in Portal, you can go to [Device Management Portal](https://portal.mbedcloud.com) > **Device Directory** to find the device ID.

1. For a `rootfs` update, identify which root file system partition is currently active to compare it to the active partition after the update. You can use the [`lsblk` command explained later](#identify-the-active-root-file-system-partition).
1. Change the current working directory to the directory where the manifest tool was initialized. You initialized the manifest tool [when you created the `update_default_resources.c` file](../getting-started/preparing-device-management-sources.html#creating-an-update-resources-file).
1. Run the following command:

    ```
    user01@dev-machine:manifests$ manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
    ```

    Where:

        * `<device-id>` is the device ID.
        * `<payload-file>` is the update payload (`.tar` file).
        * `<api-key>` is the Pelion API key you generated as a prerequisite.

    For a root file system update, the process can take a long time, depending on your file size and network bandwidth.

#### Additional notes

* You can monitor the update payload **download** progress by tailing the `mbl-cloud-client` log file:

    ```
    root@mbed-linux-os-1234:~# tail -f /var/log/mbl-cloud-client.log
    ```

* You can monitor the update payload **installation** progress by tailing:

    * `/var/log/arm_update_activate.log`: for messages about the overall progress of the installation, and messages specific to `rootfs` updates.
    * `/var/log/mbl-app-update-manager-daemon.log`: for messages about application updates.

* For `rootfs` updates, a reboot is automatically initiated to boot into the new firmware. Identify the currently active root file system and verify it was different pre-update. You can use the [`lsblk` command explained later](#identify-the-active-root-file-system-partition).

* The system does not reboot after an application update.

### Identifying the active root file system partition

To verify that a root system update succeeded, you can check which root file system partition is active before and after the update. If the update succeeded, the active partition changes.

To identify the active root file system partition:

```
root@mbed-linux-os-1234:~# lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
```

This command prints the label of the block device currently mounted at `/`, which is `rootfs1` if the first root file system partition is active, or `rootfs2` if the second root file system partition is active.

For example, when the first root file system partition is active, you see:

```
root@mbed-linux-os-1234:~# lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
rootfs1
root@mbed-linux-os-1234:~#
```
