# Updating an MBL image

There are two ways to update an MBL image:

* Using the manifest tool
* Using MBL CLI

## Using the manifest tool

1. Create an update payload file: Make a `tar` file containing the root file system archive from the MBL build artefacts.

    A symlink to the root file system archive can be found in the build environment at `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.tar.xz`

    Where:

    * `/path/to/artifacts` is the output directory specified for all build artefacts. See the [build tutorial](../first-image/building-a-developer-image.html) for more information.
    * `<MACHINE>` is the value that was given to the build script for the `--machine` option. See the [build tutorial](../first-image/building-a-developer-image.html) to determine which value is suitable for the device in use.

    <span class="notes">**Note:** The file inside the update payload must be named `rootfs.tar.xz` and must be in the tar's root directory, not a subdirectory.</span>

    For example, to create an update payload file at `/tmp/payload.tar` containing a Warp7 root file system image `tar` file with the path `build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-image-production-imx7s-warp-mbl.tar.xz`, run:

    ```
    user01@dev-machine:~$ tar -cf /tmp/payload.tar -C /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images '--transform=s/.*/rootfs.tar.xz/' --dereference mbl-image-development-imx7s-warp-mbl.tar.xz
    ```

    The `--transform` option renames all files added to the payload to `rootfs.tar.xz` and the `--dereference` option is used so that `tar` adds the actual root file system archive file rather than the symlink to it.


1. Find the device ID in the `mbl-cloud-client` log file at `/var/log/mbl-cloud-client.log`, using the following command on the device's console:

    ```
    root@mbed-linux-os-1234:~# grep -i 'device id' /var/log/mbl-cloud-client.  
    ```

    If you only have one registered device, or if each devices has a been assigned a descriptive name in Portal, you can go to [Device Management Portal](https://portal.mbedcloud.com) > **Device Directory** to find the device ID.

1. Identify which root file system partition is currently active to compare it to the active partition after the update. You can use the [`lsblk` command explained later](#identify-the-active-root-file-system-partition).

1. Change the current working directory to the directory where the manifest tool was initialized. You initialized the manifest tool [when you created the `update_default_resources.c` file](../first-image/provisioning-for-pelion-device-management.html#creating-an-update-resources-file).

1. Run the following command:

    ```
    user01@dev-machine:manifests$ manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
    ```

    Where:

        * `<device-id>` is the device ID.
        * `<payload-file>` is the update payload (`.tar` file).
        * `<api-key>` is the Pelion API key you generated as a prerequisite.

    The process can take a long time, depending on your file size and network bandwidth.

A reboot is automatically initiated to boot into the new firmware. Identify the currently active root file system and verify it was different pre-update. You can use the [`lsblk` command explained later](#identify-the-active-root-file-system-partition).

## Using MBL CLI

<span class="tips">You can find installation and general usage instructions for MBL CLI [in the application development section](../develop-apps/the-mbl-command-line-interface.html).</span>


1. Prepare a tar file containing `rootfs.tar.xz` as explained in the [manifest tool section above](#using-the-manifest-tool).

1. Transfer the rootfs update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli put <rootfs update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the name of the payload file for the rootfs update, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli put payload.tar /scratch 169.254.111.222
   ```

1. Use the MBL CLI `shell` command to get shell access on the device:

   ```
   $ mbl-cli shell [address]
   ```

   For example:

   ```
   $ mbl-cli shell 169.254.111.222
   ```

1. Inside the shell, run the `mbl-firmware-update-manager` script to install the rootfs:

   ```
   $ mbl-firmware-update-manager -i <full path to TAR file under /scratch>
   ```

   For example:

   ```
   $ mbl-firmware-update-manager -i /scratch/payload.tar
   ```

    The rootfs is updated.

1. The device automatically reboots.

<span class="notes">We recommend deleting the old tar files from the `scratch` partition after updates finish.</span>

## Identifying the active root file system partition

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
