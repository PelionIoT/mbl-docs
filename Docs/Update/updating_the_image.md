# Updating an MBL image

There are two ways to update the updatable components in an MBL image:

* [Using MBL CLI](#using-mbl-cli).
* [Using the manifest tool](#using-the-manifest-tool).

First, you need to prepare the update payload for the updatable component.

## Prepare the update payload

You need a [full build of MBL](../first-image/building-a-developer-image.html) because this contains the files for the updatable components.

During the build, the build script signs the bootloader and kernel update components using either keys that were generated in the build, or by keys you supplied to the build, so you must use the same keys or keys derived from the same root of trust that you have used for the first image loaded on the device.

<span class="notes">**Warning:** If you update a device with update components from a build that used different keys, the device won't boot anymore. For example, if you are using an evaluation image and generate a new kernel or bootloader component, the keys will be different. You can recover the device by following the steps to flash a first image. In the future, there will be support for rollback to the last image using a bank switching mechanism for updates, apart from the bootloader component one.</span>

### Handling secure boot keys in the build

To support secure boot in MBL, the following signing keys and certificates are required:

* Trusted world private signing key used in preparing the Bootloader update component 1 (BL2) for the Trusted Firmware secure boot - `rot_key.pem`.
* Normal world private signing key and public certificate used in preparing the Bootloader update component 2 (BL3) for the Kernel update component (FIT) - `mbl-fit-rot-key.key` and `mbl-fit-rot-key.crt`.

<span class="notes">**Note:** The private keys should be kept securely.</span>

You can find more information about these keys and how to use them in the [porting guide about FIP and FIT images](../develop-mbl/bsp-sys-arch.html#partitioning-software-components-into-fip-and-fit-images).

If you do not supply these keys and certificates, they will be generated when you perform a build.

The default build action is that these keys will be placed in your output directory and used for subsequent builds in the same build directory unless you turn this off by setting `MBL_PERSIST_SIGN_KEYS` to `0` in your `local.conf` configuration.

If you do persist the keys, you can find the generated keys in your output directory: `/path/to/artifacts/machine/MACHINE/IMAGE/keys`, where `MACHINE` and `IMAGE` match the machine and image types you built - for example, `imx7s-warp-mbl` and `mbl-image-developement`.

You can supply these keys and certificates to a build by using the following build options:

* `--boot-rot-key ROTKEY` - where `ROTKEY` is the path and filename of `rot_key.pem`.
* `--kernel-rot-key FITROTKEY` - where `FITROTKEY` is the path and filename of `mbl-fit-rot-key.key`.
* `--kernel-rot-crt FITROTCRT` - where `FITROTCRT` is the path and filename of `mbl-fit-rot-key.crt`.

If you don't want the build to generate the keys, you can do so manually by using the following `openssl` commands:

```
mkdir boot-keys
openssl genrsa 2048 > boot-keys/rot_key.pem
openssl genrsa 2048 > boot-keys/mbl-fit-rot-key.key
openssl req -batch -new -x509 -key boot-keys/mbl-fit-rot-key.key > boot-keys/mbl-fit-rot-key.crt
```

<span class="notes">**Note:** You may need to install the OpenSSL tools using `sudo apt-get install openssl`.</span>

Then, for example, perform a build:

```
./mbl-tools/build/run-me.sh --builddir ./build-nxpimx --outputdir ./artifacts-nxpimx --boot-rot-key boot-keys/rot_key.pem --kernel-rot-key boot-keys/mbl-fit-rot-key.key --kernel-rot-crt boot-keys/mbl-fit-rot-key.crt -- --machine imx8mmevk-mbl --branch mbl-os-0.8
```

<span class="notes">**Warning:** Changes to the keys and certificates with an existing build may not be picked up. Please perform a clean first: `./mbl-tools/build-mbl/run-me.sh --builddir ./build-nxpimx -- clean`.</span>

### Creating the payload

1. Enter the [mbl-tools interactive mode](../develop-mbl/mbed-linux-os-distribution-development-with-mbl-tools.html#running-build-mbl-in-interactive-mode) for the build. For example, for PICO-PI with IMX7D:

    ```
    ./mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/artifacts -- --machine imx7d-pico-mbl --branch mbl-os-0.8  interactive
    ```

    Please see the other [build examples](../install_mbl_on_device/building_an_image/build_examples.html).

2. Use the `create-update-payload` tool to create a tar file containing the required update component.

    These are the arguments the tool accepts:

    | Name | Value | Information |
    | --- | --- | --- |
    | `--bootloader-component` | `1` | Add the first bootloader component to the payload. Usually this contains the second stage boot of TF-A. |
    | `--bootloader-component` | `2` | Add the second bootloader component to the payload. Usually this contains the third stage boot of TF-A. |
    | `--kernel` |  | Add the kernel to the payload. Usually this also includes the device tree blob and boot scripts. |
    | `--rootfs` | IMAGE_NAME | Add the root file system of the specified bitbake image to the payload. |
    | `--output-tar-path` | FILE_PATH | File name and path for the payload tar file to be created. |

    <span class="notes">**Note:** The create-update-payload tool needs the bitbake environment to work. When using the interactive mode, only the builddir (`/path/to/build`) or outputdir (`/path/to/artifacts`) directories are available in the build environment, so one of them must be used as the path in the FILE_PATH argument.</span>

    For example, to create an update payload file `/path/to/artifacts/payload-boot2.tar` containing the bootloader component 2, run:

    ```
    user01@dev-machine:~$ create-update-payload --bootloader-component 2 --output-tar-path /path/to/artifacts/payload-boot2.tar
    ```

    Or, for example, to create an update payload file `/path/to/artifacts/payload-kernel.tar` containing the kernel, run:

    ```
    user01@dev-machine:~$ create-update-payload --kernel --output-tar-path /path/to/artifacts/payload-kernel.tar
    ```

    Or, for example, to create an update payload file `/path/to/artifacts/payload-rootfs.tar` containing a root file system from the `mbl-image-development` image build, run:

    ```
    user01@dev-machine:~$ create-update-payload --rootfs mbl-image-development --output-tar-path /path/to/artifacts/payload-rootfs.tar
    ```

## Using MBL CLI to update

<span class="tips">You can find installation and use instructions for MBL CLI [in the application development section](../develop-apps/the-mbl-command-line-interface.html).</span>

1. Transfer the update payload file to the `/scratch` partition on the device:

   ```
   $ mbl-cli [-a address] put <update payload file> <destination on device under the /scratch partition>
   ```

   For example, if `payload.tar` is the name of the update payload file, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli -a 169.254.111.222 put payload.tar /scratch
   ```

   <span class="tips">You can use `mbl-cli select` to choose your device, instead of using the `-a address` option.</span>

1. Use the MBL CLI `shell` command to get shell access on the device:

   ```
   $ mbl-cli [-a address] shell
   ```

   For example:

   ```
   $ mbl-cli -a 169.254.111.222 shell
   ```

1. Inside the shell, run the `mbl-firmware-update-manager` script to install the update component:

   ```
   $ mbl-firmware-update-manager /absolute/path/to/payload.tar
   ```

   For example:

   ```
   $ mbl-firmware-update-manager /scratch/payload.tar
   ```

   The update component is installed.

1. When prompted, press <kbd>Y</kbd>+<kbd>Enter</kbd> key to reboot the device and use the newly installed component.

   <span class="notes">**Note:** To keep the update package after a successful update, use the optional argument `--keep`. Use the optional argument `--assume-yes` to automatically reboot after the update component has been installed.</span>

## Using the manifest tool to update

1. Find the device ID in the `mbl-cloud-client` log file at `/var/log/mbl-cloud-client.log`, using the following command on the device's console:

    ```
    root@mbed-linux-os-1234:~# grep -i 'device id' /var/log/mbl-cloud-client.log
    ```

    If you only have one registered device, or if each device has been assigned a descriptive name in Portal, you can go to [Device Management Portal](https://portal.mbedcloud.com) > **Device Directory** to find the device ID.

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

A reboot is automatically initiated to use the new update component.

## Verifying the update

You can check the status of the update by looking at the [update logs](monitor.html).

If you are updating the root file system, you can verify the update by [identifying the currently active root file system](#identify-the-active-root-file-system-partition) and checking it was different before the update.

For a kernel update, if you have changed the kernel version, you can use `uname -srm` on the device to check the version number now matches the new version.

A boot component update depends on the parts of the boot sequence you have changed. Review the serial output from the device to check version numbers.

### Identifying the active root file system partition

To verify that a root file system update succeeded, you can check which root file system partition is active before and after the update. If the update succeeded, the active partition changes.

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
