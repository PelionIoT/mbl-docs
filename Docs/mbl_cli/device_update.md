# Device update

You can update the device's root file system (rootfs) and applications. Update of boot loaders, the Linux kernel and other components will be supported in later versions.

Device update uses the `mbl-firmware-update-manager`, which is part of MBL:

```
$ mbl-firmware-update-manager -h

usage: mbl-firmware-update-manager [-h] -i UPDATE_FIRMWARE_TAR_FILE [-s] [-v]

optional arguments:

 -h, --help            show this help message and exit

 -i UPDATE_FIRMWARE_TAR_FILE, --install-firmware UPDATE_FIRMWARE_TAR_FILE

                       Install firmware from a firmware update tar file and

                       reboot (default: None)

 -s, --skip-reboot     Skip reboot after firmware update (default: False)

 -v, --verbose         Increase output verbosity (default: False)
```

## Rootfs update

To update the rootfs:

1. Prepare a tar file containing `rootfs.tar.xz`. For a detailed explanation, see the [root file system update workflow](../getting-started/tutorial-updating-mbl-devices-and-applications.html#workflow).

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

## Update an application

You can update each application independently of any others.

To install or update an application:

1. Prepare a tar file containing an OPKG package (`.ipk` file) for an MBL application. For a detailed explanation, see the [workflow for an application update](../getting-started/tutorial-updating-mbl-devices-and-applications.html#workflow).

1. Transfer an application update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli put <application update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the payload name for an application update, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli put payload.tar /scratch 169.254.111.222
   28 Nov 09:42:04 - File transfer succeeded
   ```

1. Use the MBL CLI `shell` command to get shell access on the device:

  ```
  $ mbl-cli shell [address]
  ```

    For example:

    ```
    $ mbl-cli shell 169.254.111.222
    ```

1. Run `mbl-firmware-update-manager` with the `--skip-reboot` parameter.

    The application is installed.

1. The application you installed or updated starts automatically, without a device reboot.

<span class="notes">We recommend deleting the old tar files from the `scratch` partition after updates finish.</span>
