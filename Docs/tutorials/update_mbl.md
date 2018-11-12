## Tutorial: updating Mbed Linux OS

Mbed Linux OS (MBL) includes three components:

- Bootloaders.
- A root file system.
- Applications.

MBL currently supports over-the-air updates of the root file system and applications.<!--currently - so we'll support updating the bootloader? I thought you don't update bootloaders because you can recover from an error in that update-->

### How firmware gets updated

We use Pelion Device Management to send firmware updates to MBL devices. The update process beings with Device Management sending the device a request with a manifest. If the device accepts the request, Device Management then sends a payload file containing an update for one or more MBL component.

The payload is a tar file that can contain either:

- Aa application update: one or more OPKG packages (`.ipk` files).
- A root file system update: a compressed tar file called `rootfs.tar.xz` <!--compressed tar within a tar?-->that contains root file system content.

Currently, MBL does **not** support updating both applications and the root file system in a single over-the-air update operation.

<span class="tips">A full review of Pelion Device Management Update is [available on the Pelion documentation site](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).</span>

#### Application updates

After receiving a payload file containing application updates, for each application update, MBL:

1. Stops any existing version of the application from running.
1. Removes any existing version of the application.
1. Installs the application update from the payload file.
1. Starts the updated application.

#### Root file system updates

To support root file system updates, MBL devices have two root file system partitions called **banks**:

- An **active** bank: a partition containing the root file system for the running system.
- An **inactive** bank: a partition available to receive firmware updates.

After receiving a payload file containing a root file system update, MBL:

1. Writes the contents of `rootfs.tar.xz` to the inactive root file system bank.
1. Flips a flag indicating which root file system bank is active, so that after a reboot the previously inactive bank will become the active bank, and the previously active bank will become the inactive bank.
1. Reboots the device.
1. Mounts the root file system from the update payload.

The previously active (now inactive) root file system partition is now available to receive the next root file system update.

<!--There's a question of how much of the theory should be explained here (and in the previous tutorial).-->

### Prerequisites

You can only update an MBL device if you have:
<!--Leave the links empty; that will make them surface in our final checks and we'll know to fix them-->

- A successful build of Mbed Linux OS on your device. See the [build tutorial]() for instructions.
- A device running an MBL image that can connect to your Pelion Device Management account. If you do not have one, please [follow the first tutorial in this series]().
- The directory in which you initialized the manifest tool, [as reviewed in our development environment setup]().

    <span class="notes>This *must* be the directory from which you obtained the `update_default_resources.c` file for your MBL build.</span>
- A Pelion Device Management API key, to use the manifest tool from the command line. If you don't already have an API Key, follow the instructions in the [Pelion Device Management docs](https://cloud.mbed.com/docs/current/integrate-web-app/api-keys.html#generating-an-api-key) (when prompted to select a group to set the API key access level, select **Developers**). You must make a note of the API key to use it later; the portal will not display it again, for security reasons.

### Workflow

1. Create an update payload file:

    - **For an application**: Make a tar file containing the `.ipk` files for the applications to update.

        Note that the `.ipk` files must be in the tar's root directory, not a subdirectory.

        For example, to create an update payload file at `/tmp/payload.tar` containing an `.ipk` file with the path `/home/user01/my_app.ipk`, run:

        ```
        tar -cf /tmp/payload.tar -C /home/user01 my_app.ipk
        ```

    - **For a root file system**: Make a tar file containing the root file system archive from your MBL build.

        A symlink to the root file system archive can be found at `<your_workarea>/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz`

        Where:

        - `<your_workarea>` is the directory in which the `repo init` command created the MBL work-area.
        - `<MACHINE>` is the value that was give to the build script for given the `--machine` option. See the [build tutorial]() to find out which value of `<MACHINE>` is suitable for your device.

        <span class="notes">The file inside the update payload must be called `rootfs.tar.xz` and must be in the tar's root directory, not a subdirectory..</span>

        For example, if your work-area is `/home/user01/mbl/mbl-os-0.5` and you are using a WaRP7, run the following command to create a payload file at `/tmp/payload.tar`:

        ```
        tar -cf /tmp/payload.tar -C /home/user01/mbl/mbl-os-0.5/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl '--transform=s/.*/rootfs.tar.xz/' --dereference mbl-console-image-imx7s-warp-mbl.tar.xz
        ```

        The `--transform` option is used to rename all files added to the payload to `rootfs.tar.xz` and the `--dereference` option is used so that `tar` adds the actual root file system archive file rather than the symlink to it.

1. Discover your device's device ID. To do this you can either:

   - Go to the [Device Management Portal](https://portal.mbedcloud.com), find your device in the **Device Directory** tab, and copy the device ID.
   - Search for the device ID in the `mbl-cloud-client` log file `/var/log/mbl-cloud-client.log`, using the following command on the device's console:

       ```
       grep -i 'device id' /var/log/mbl-cloud-client.log
       ```
1. If you are doing a root file system update, you may wish to check which root file system bank is currently active so that you can compare it to the active bank after the update. You can use the [`lsblk` command explained below](#checking-which-root-file-system-bank-is-active) to do this. This step is not mandatory.
1. Change your work context to the directory in which the manifest tool was initialized.
1. Run the following command:

    ```
    manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
    ```

    Where:

        - `<device-id>` is the device ID.
        - `<payload-file>` is the update payload (`.tar` file).
        - `<api-key>` is your Pelion Device Management API key.

        You should have all three from previous steps in this process.

1. When your device initiates the update download, you can follow its progress by looking in the `mbl-cloud-client` log file:

    ```
    tail -f /var/log/mbl-cloud-client.log
    ```

1. When the download completes and installation starts, you can follow its progress in the following log files:

    - `/var/log/arm_update_activate.log`: for messages about the overall progress of the installation, and messages specific to root file system updates.
   - `/var/log/mbl-app-update-manager.log`: for messages about application updates.

1. When the installation completes, the device starts running the new firmware.

    For root file system updates, a reboot will be automatically initiated to boot into the new firmware. For application updates, a reboot is not required.

1. If you are doing a root file system update, you may wish to check which root file system bank is currently active and compare it with the bank that was active before the update (if you checked it then) - it should have changed. You can use the `lsblk` command below to do this.

### Checking which root file system bank is active

<!--I moved this; it didn't belong in the intro-->
<!--when and why would I want to do this?-->

To find out which root file system bank is currently active:

```
lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
```

This command prints the label of the block device currently mounted at `/`, which will be `rootfs1` if the first root file system bank is active or `rootfs2` if the second root file system bank is active.

For example, when the first root file system bank is active:

```
root@mbed-linux-os-1234:~# lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
rootfs1
root@mbed-linux-os-1234:~#
```
