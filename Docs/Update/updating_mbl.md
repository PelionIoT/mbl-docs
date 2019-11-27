# Updating an MBL image

The steps to update the software on an MBL device are:
1. [Do a full build of the version of MBL that you wish to install on your device.](#building-mbl)
1. [Create the update payload file that contains the components that you wish to install.](#creating-the-payload)
1. [Apply the update to your device.](#applying-the-payload)
1. [Verify that the update has applied correctly.](#verifying-the-update)

These steps are detailed below.

## Building MBL

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
./mbl-tools/build/run-me.sh --builddir ./build-nxpimx --outputdir ./artifacts-nxpimx --boot-rot-key boot-keys/rot_key.pem --kernel-rot-key boot-keys/mbl-fit-rot-key.key --kernel-rot-crt boot-keys/mbl-fit-rot-key.crt -- --machine imx8mmevk-mbl --branch mbl-os-0.10
```

<span class="notes">**Warning:** Changes to the keys and certificates with an existing build may not be picked up. Please perform a clean first: `./mbl-tools/build-mbl/run-me.sh --builddir ./build-nxpimx -- clean`.</span>

## Creating the payload

To create the update payload:

1. Enter the [mbl-tools interactive mode](../develop-mbl/mbed-linux-os-distribution-development-with-mbl-tools.html#running-build-mbl-in-interactive-mode) for the build. For example, for PICO-PI with IMX7D:

    ```
    ./mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/artifacts -- --machine imx7d-pico-mbl --branch mbl-os-0.9  interactive
    ```

    Please see the other [build examples](../install_mbl_on_device/building_an_image/build_examples.html).

1. Use the `create-update-payload` tool to create an update payload file containing the required update components.

    These are the arguments the tool accepts:

    | Name                      | Value                      | Information |
    | ---                       | ---                        | --- |
    | `--bootloader-components` | `N [N ...]`                | Add bootloader components to the payload. `N` can be `1` or `2` and you can list one or both of these values. Bootloader component 1 usually contains the second stage boot of TF-A and bootloader component 2 usually contains TF-A bootloader stage three (BL3), OP-TEE and U-boot. |
    | `--kernel`                |                            | Add the kernel to the payload. Usually, this also includes the device tree blob, U-Boot boot script and initramfs. |
    | `--rootfs`                | `IMAGE_NAME`               | Add the root file system of the specified image to the payload. |
    | `--apps`                  | `IPK_PATH [IPK_PATH ... ]` | Specifies the paths of application packages (`.ipk` files) to add to the payload. |
    | `--output-path`           | `PATH`                     | File name and path for the payload file to be created. |
    | `--mbl-os-version`        | `MBL_OS_VERSION`           | Specify the version of MBL running on the device on which the payload will be installed. If not specified, `create-update-payload` will create a payload formatted for the MBL version that it came from itself. The earliest supported MBL version is 0.9.0. |

    <span class="notes">**Note:** The create-update-payload tool needs the bitbake environment to work. When using the interactive mode, only the builddir (`/path/to/build`) or outputdir (`/path/to/artifacts`) directories are available in the build environment, so you must use them as the paths for the `--apps` and `--output-path` arguments.</span>

    <span class="warnings">**Warning:** Different releases of MBL may use different payload
    formats. That means you may not be able to install payloads created for one
    version of MBL on a different version of MBL. See
    [Updating across releases of MBL](#updating-across-releases-of-mbl) for
    details about upgrading to a new release of MBL and using
    `create-update-payload`'s `--mbl-os-version` option.</span>


    For example:

    * To create an update payload file `/path/to/artifacts/payload.swu` containing bootloader components 1 and 2, the kernel component and the rootfs component for mbl-image-development, run:

      ```
      user01@dev-machine:~$ create-update-payload --bootloader-components 1 2 --kernel --rootfs mbl-image-development --output-path /path/to/artifacts/payload.swu
      ```

    * To create an update payload file `/path/to/artifacts/payload-kernel.swu` containing the kernel, run:

      ```
      user01@dev-machine:~$ create-update-payload --kernel --output-path /path/to/artifacts/payload-kernel.swu
      ```

    * To create an update payload file `/path/to/artifacts/payload-app.swu` containing an app `package.ipk`, run:

      ```
      user01@dev-machine:~$ create-update-payload --apps /path/to/artifacts/package.ipk --output-path /path/to/artifacts/payload-apps.swu
      ```

## Applying the payload

There are two ways to apply the update payload to your device:

* [Using MBL CLI](#using-mbl-cli-to-update).
* [Using the manifest tool](#using-the-manifest-tool-to-update).

### Using MBL CLI to update

<span class="tips">You can find installation and use instructions for MBL CLI [in the application development section](../develop-apps/the-mbl-command-line-interface.html).</span>

1. Transfer the update payload file to the `/scratch` partition on the device:

   ```
   $ mbl-cli [-a address] put <update payload file> <destination on device under the /scratch partition>
   ```

   For example, if `payload.swu` is the name of the update payload file, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli -a 169.254.111.222 put payload.swu /scratch
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
   $ mbl-firmware-update-manager /absolute/path/to/payload.swu
   ```

   For example:

   ```
   $ mbl-firmware-update-manager /scratch/payload.swu
   ```

   The update component is installed.

1. If prompted, press <kbd>Y</kbd>+<kbd>Enter</kbd> to reboot the device and use the newly installed component.

   <span class="notes">**Note:** To keep the update package after a successful update, use the optional argument `--keep`. Use the optional argument `--assume-yes` to automatically reboot after the update component has been installed.</span>

### Using the manifest tool to update

1. Find the device ID in the `mbl-cloud-client` log file at `/var/log/mbl-cloud-client.log`, using the following command on the device's console:

    ```
    root@mbed-linux-os-1234:~# grep -i 'device id' /var/log/mbl-cloud-client.log
    ```

    If you only have one registered device, or if each device has been assigned a descriptive name in Portal, you can go to [Device Management Portal](https://portal.mbedcloud.com) > **Device Directory** to find the device ID.

1. If you are doing a `rootfs` component update, identify which root file system partition is currently active to compare it to the active partition after the update. You can use the [`lsblk` command explained later](#identify-the-active-root-file-system-partition).

1. Change the current working directory to the directory where the manifest tool was initialized. You initialized the manifest tool [when you created the `update_default_resources.c` file](../first-image/provisioning-for-pelion-device-management.html#creating-an-update-resources-file).

1. Run the following command:

    ```
    user01@dev-machine:manifests$ manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
    ```

    Where:

    * `<device-id>` is the device ID.
    * `<payload-file>` is the update payload file.
    * `<api-key>` is the Pelion API key you generated as a prerequisite.

    The process can take a long time, depending on your file size and network bandwidth.

If you are updating one or more components that require a reboot to take effect (any component except applications), a reboot is automatically initiated.

## Verifying the update

You can check the status of the update by looking at the [update logs](monitor.html).

If you are updating the root file system, you can verify the update by [identifying the currently active root file system](#identify-the-active-root-file-system-partition) and checking it was different before the update.

For a kernel update, if you have changed the kernel version, you can use `uname -srm` on the device to check the version number now matches the new version.

A boot component update depends on the parts of the boot sequence you have changed. Review the serial output from the device to check version numbers.

If you have updated applications, you can check on their status by using the `runc list` command. See [Identifying the running applications](#identifying-the-running-applications) for more information.

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

### Identifying the running applications

To list all the active applications and their status, use `runc list` .

1. The first steps of an application update include stopping and terminating a running instance of the application (if one exists).

    If you perform `runc list` when the application is stopped, you will see:

    ```
    root@mbed-linux-os-1234:/home/app# runc list
    ID                        PID         STATUS      BUNDLE                                CREATED                          OWNER
    user-sample-app-package   0           stopped     /home/app/user-sample-app-package/0   2018-12-07T08:23:36.742467741Z   root
    ```

    If you perform `runc list` when the application is terminated, the application will not appear on the list.

1. When the application is installed or updated, it starts automatically.

    If you perform `runc list` for a running application, you will see:

    ```
    root@mbed-linux-os-1234:/home/app# runc list
    ID                        PID         STATUS      BUNDLE                                CREATED                          OWNER
    user-sample-app-package   3654        running     /home/app/user-sample-app-package/0   2018-12-07T08:23:36.742467741Z   root
    ```

    <span class="notes">**Note:** The [Hello World](../develop-apps/hello-world-application.html) application runs for about 20 seconds. When it finishes, it once again appears as stopped.</span>

## Updating across releases of MBL

Between releases of MBL, the format of the update payloads accepted
by the software may change. This means that an update payload created for one
release of MBL may not be suitable for installation on a different
release of MBL. Therefore, when creating update payloads to upgrade
from an earlier release of MBL, you should always use
`create-update-payload`'s `--mbl-os-version` option to specify the version of
MBL that will recieve the update.

Between releases of MBL, some components may change, while others remain the
same. The set of components that change across releases of MBL is documented in
the [Version specific upgrade notes](#version-specific-upgrade-notes) section
below, along with any other version specific notes or restrictions relating to
updates.

### Version specific upgrade notes

Note that upgrades to MBL 0.10 from versions earlier than 0.9 is not
supported.

#### Upgrading from MBL 0.9

Components changed since MBL 0.9:
* Bootloader component 2.
* The kernel component.
* The rootfs component.

MBL 0.9 does not support multi-component updates, so it is not
possible to upgrade all components to MBL 0.10 at once. The upgrade
must be done in two parts:
1. Update the rootfs component to MBL 0.10's version. To do
   this, follow the instructions above to create and install a payload
   containing only the rootfs component from your MBL 0.10 workarea.
   When invoking `create-update-payload`, make sure you specify
   `--mbl-os-version 0.9` to create a payload that can be installed on MBL 0.9.
   For example:
   ```
    user01@dev-machine:~$ create-update-payload --rootfs mbl-image-development --output-path /path/to/artifacts/rootfs_payload.swu --mbl-os-version 0.9
   ```
1. Once the rootfs component from MBL 0.10 has been installed on a
   device, it will be able to install multi-component update payloads in the
   format used by MBL 0.10, so you can update the rest of the
   components using a single update payload. To do this, follow the
   instructions above to create and install a payload containing bootloader
   component 2 and the kernel component from your MBL 0.10 workarea.
   When invoking `create-update-payload`, you may omit the `--mbl-os-version`
   argument, or explicitly specify the version `0.10`. For example:
   ```
   user01@dev-machine:~$ create-update-payload --bootloader-components 2 --kernel --output-path /path/to/artifacts/b2_and_kernel_payload.swu --mbl-os-version 0.10
   ```

<span class="warnings">**Warning:** Do not attempt to update the kernel component from version 0.9 to
version 0.10 before updating the rootfs. In MBL 0.10, the kernel
component enables a hardware watchdog that will reset the board if a service
from the rootfs component does not regularly stroke it. The 0.9 version of the
rootfs component does not contain the service that strokes the watchdog so
using a 0.10 kernel component with a 0.9 rootfs component will result in a boot
loop.</span>
