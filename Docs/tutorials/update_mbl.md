## Tutorial: updating Mbed Linux OS

Mbed Linux OS is composed of multiple components, including:
- bootloaders;
- a root file system;
- applications.

Mbed Linux OS currently supports over-the-air updates of the root file system and applications.

### How firmware gets updated

Firmware updates can be sent over the air to Mbed Linux OS devices using Pelion Device Management capabilities.

<span class="tips">A full review of Pelion Device Management Update is [available on the Pelion documentation site](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).</span>

Once an Mbed Linux OS device accepts a firmware update request from the Pelion cloud, the device downloads an update **payload** file that contains updates for one of the MBL components. The payload file is a tar file and can contain either:
- application updates - one or more OPKG packages (`.ipk` files); or
- a root file system update - a compressed tar file called `rootfs.tar.xz` that contains root file system contents.

#### Application Updates
After recieving a payload file containing application updates (`.ipk` files), for each application update, MBL will:
- Stop any existing version of the application from running.
- Remove any existing version of the application.
- Install the application update from the payload file.
- Start the updated application.

#### Root File System Updates
To support root file system updates, MBL devices have two root file system partitions called **banks**:

- An **active** bank: a partition containing the root file system for the running system.
- An **inactive** bank: a partition that is available to receive firmware updates.

Only one of the root filesystem partitions is active at any one time, and the other is available to receive a new version of the root file system during an update.

<a name="check-rootfs-bank"></a>You can find out which root file system bank is currently active on an MBL device by running the following command:
```
lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
```
This command prints the label of the block device currently mounted at `/`, which will be `rootfs1` if the first root file system bank is active or `rootfs2` if the second root file system bank is active. For example, when the first root file system bank is active you should see output like the following:
```
root@mbed-linux-os-1234:~# lsblk --noheadings --output "MOUNTPOINT,LABEL" | awk '$1=="/" {print $2}'
rootfs1
root@mbed-linux-os-1234:~#
```

After recieving a firmware update request and a payload file containing a root file system update (a `rootfs.tar.xz` file), MBL will:
- Write the contents of `rootfs.tar.xz` to the inactive root file system bank.
- Flip a flag indicating which root file system bank is active (so that after a reboot the previously inactive bank will become the active bank and the previously active bank will become the inactive bank).
- Reboot the device.

After the reboot MBL will mount the root file system from the update payload and the previously active (now inactive) root file system partition will become available to recieve the next root file system update.

<!--There's a question of how much of the theory should be explained here (and in the previous tutorial).-->

### Prerequisites

You can only update an MBL device if you have:

- A successfully completed build of Mbed Linux OS for your device. See the [build tutorial][build-tutorial] for instructions.
- A device running the MBL image that you built that can connect to your Pelion Device Management account. If you do not have one, please [follow the first tutorial in this series](connecting-an-mbl-device-and-using-an-applications.html).
- A directory in which the manifest tool was initialized, [as reviewed in our development environment setup](preparing-a-development-environment.html). This *must* be the directory from which the `update_default_resources.c` file used in the build of MBL was obtained. <!-- TODO: fix this link -->
- Additionally, in order to use the manifest tool to easily send updates to your device via the Pelion Device Management Cloud (rather than using the Pelion Device Management web interface), you will need a Pelion Device Management API key. If you don't already have an API Key, follow the instructions in the [Pelion Device Management docs](https://cloud.mbed.com/docs/current/integrate-web-app/api-keys.html#generating-an-api-key) to generate one. When prompted to select a group to set the API key access level, the default "Developers" group is suitable for performing firmware updates. You will need to make a note of the API key in order to use it later.

### Workflow

#### Create an update payload file

To create an application update payload file, make a tar file containing the `.ipk` files for the applications to install/update. Note that the `.ipk` files must not be in any subdirectories inside the tar file. For example, to create an update payload file at `/tmp/payload.tar` containing an `.ipk` file with path `/home/user01/my_app.ipk`, run the following command:
```
tar -cf /tmp/payload.tar -C /home/user01 my_app.ipk
```

To create a root file system update payload file, make a tar file containing the root file system archive from your MBL build. A symlink to the root file system archive can be found at `<your_workarea>/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz` where:
- `<your_workarea>` is the directory in which the `repo init` command was run to create the MBL workarea.
- `<MACHINE>` is the value that was given for the `--machine` option to the build script. See the [build tutorial][build-tutorial] to find out which value of `<MACHINE>` is suitable for your device.

Note that the file inside the update payload must be called `rootfs.tar.xz` and must not be in any subdirectory inside the tar file.

For example, if your workarea is `/home/user01/mbl/mbl-os-0.5` and you are using a WaRP7, run the following command to create a payload file at `/tmp/payload.tar`:
```
tar -cf /tmp/payload.tar -C /home/user01/mbl/mbl-os-0.5/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl '--transform=s/.*/rootfs.tar.xz/' --dereference mbl-console-image-imx7s-warp-mbl.tar.xz
```
The `--transform` option is used to rename all files added to the payload to `rootfs.tar.xz` and the `--dereference` option is used so that `tar` adds the actual root file system archive file rather than the symlink to it.

#### Update your device

To send a firmware update to your device, follow these steps:
1. Discover your device's device ID. To do this you can either:
   - Copy the device ID to the clipboard from the **Device Directory** tab on the Pelion Device Management web interface.
   - Search for the device ID in the `mbl-cloud-client`'s log file `/var/log/mbl-cloud-client.log`, using the following command on the device's console:
     ```
     grep -i 'device id' /var/log/mbl-cloud-client.log
     ```
1. At this point, if you are doing a root file system update, you may wish to check which root file system bank is currently active so that you can compare it to the active bank after the update. You can use the [`lsblk` command mentioned above](#check-rootfs-bank) to do this.
1. Change directory to the directory in which the manifest tool was initialized.
1. Run the following command:
   ```
   manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
   ```
   Where:
   - `<device-id>` is the device ID discovered in #1.
   - `<payload-file>` is the update payload (`.tar` file) that you created from the instructions in the previous section.
   - `<api-key>` is your Pelion Device Management API key. 
1. Once your device has recieved the update request, you can follow its download progress by looking in the `mbl-cloud-client`'s log file:
   ```
   tail -f /var/log/mbl-cloud-client.log
   ```
1. Once the download has completed and installation has begun, installation progress can be followed in the following log files:
   - `/var/log/arm_update_activate.log` - this log file contains messages about the overall progress of the installation and messages specific to root file system updates.
   - `/var/log/mbl-app-update-manager.log` - this log file contains messages specific to application updates.
1. Once installation of the update is complete, Mbed Linux OS will start running the new firmware. For root file system updates, a reboot will be automatically initiated to boot into the new firmware. For application updates, a reboot is not required and the new applications will start running automatically after the update.
1. If you are doing a root file system update, you may wish to check which root file system bank is currently active and compare it with the bank that was active before the update - it should have changed. You can use the [`lsblk` command mentioned above](#check-rootfs-bank) to do this.

[build-tutorial]: create_connecting_image.md
