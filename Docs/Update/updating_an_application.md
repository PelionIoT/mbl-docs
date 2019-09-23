# Updating an application

There are two ways to update an MBL application:

* [Using MBL CLI](#using-mbl-cli).
* [Using the manifest tool](#using-the-manifest-tool).

MBL provides a wrapper around the Device Management firmware update service and manifest tool. This means you can use the same file type (`tar`, containing the `ipk`) for both processes, allowing you to pick the update method independently of your packaging process. It also means that any file that MBL CLI manages to update, the manifest tool should also be able to update, so you can use MBL CLI as a proxy for testing on Device Management.

First you will need to prepare the update payload for the application.

## Prepare the update payload

You will need a build environment of Mbed Linux OS as this contains the scripts needed to create the update payload for applications. You can either re-use a [full build of Mbed Linux OS](../first-image/building-a-developer-image.html) or create a new environment just for application update.

1. If you are going to create a new build environment - [set up the build mbl tools](../install_mbl_on_device/Reqs_and_env/dev_env_for_distribution.html) and create an outputdir directory:

    ```
    git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-os-0.8
    mkdir /path/to/artifacts
    ```

1. Copy your application `ipk` file into your build outputdir directory:

    ```
    cp /path/to/application/package.ipk /path/to/artifacts
    ```

1. Enter the [mbl-tools interactive mode](../develop-mbl/mbed-linux-os-distribution-development-with-mbl-tools.html#running-build-mbl-in-interactive-mode) for your build. For example for PICO-PI with IMX7D:

    ```
    ./mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/artifacts -- --machine imx7d-pico-mbl --branch mbl-os-0.8  interactive
    ```

    Please see the other [build examples](../install_mbl_on_device/building_an_image/build_examples.html).

1. Use the `create-update-payload` tool to create a tar file containing the application package.

    These are the arguments that the tool accepts:

    | Name | Value | Information |
    | --- | --- | --- |
    | `--app` | APPLICATION_PATH | Specifies the application package name and path to add to the payload. |
    | `--output-tar-path` | FILE_PATH | File name and path for the payload tar file to be created. |

    <span class="notes">**Note:** The create-update-payload tool needs the bitbake environment to work. When using the interactive mode, only the builddir (`/path/to/build`) or outputdir (`/path/to/artifacts`) directories are available in the build environment, so they must be used as the paths for the APPLICATION_PATH and FILE_PATH arguments.</span>

    For example, to create an update payload file `/path/to/artifacts/payload-app.tar` containing `package.ipk`, run:

    ```
    user01@dev-machine:~$ create-update-payload --app /path/to/artifacts/package.ipk --output-tar-path /path/to/artifacts/payload-boot2.tar
    ```

## Using MBL CLI to update

<span class="tips">You can find installation and general usage instructions for MBL CLI [in the application development section](../develop-apps/the-mbl-command-line-interface.html).</span>

To install or update an application:

1. Transfer an application update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli [-a address] put <application update payload> <destination on device under the /scratch partition>
   ```

   For example, if `payload.tar` is the payload name for an application update, and 169.254.111.222 is a link-local IPv4 address on the device:

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

1. Run `mbl-firmware-update-manager`.

    ```
    $ mbl-firmware-update-manager /absolute/path/to/payload.tar
    ```

    The application is installed.

    <span class="notes">**Note:** To keep the update package after a successful update, use the optional argument `--keep`.</span>

1. The application starts automatically, without a device reboot.

## Using the manifest tool to update

1. Find the device ID in the `mbl-cloud-client` log file at `/var/log/mbl-cloud-client.log`, using the following command on the device's console:

    ```
    root@mbed-linux-os-1234:~# grep -i 'device id' /var/log/mbl-cloud-client.log
    ```

    If you only have one registered device, or if each devices has a been assigned a descriptive name in Portal, you can go to [Device Management Portal](https://portal.mbedcloud.com) > **Device Directory** to find the device ID.

1. Change the current working directory to the directory where the manifest tool was initialized. You initialized the manifest tool [when you created the `update_default_resources.c` file](../first-image/provisioning-for-pelion-device-management.html#creating-an-update-resources-file).

1. Run the following command:

    ```
    user01@dev-machine:manifests$ manifest-tool update device --device-id <device-id> --payload <payload-file> --api-key <api-key>
    ```

    Where:

    * `<device-id>` is the device ID.
    * `<payload-file>` is the update payload (`.tar` file).
    * `<api-key>` is the Pelion API key you generated as a prerequisite.

The system does not reboot after an application update.

## Identifying the running applications

You can use `runc list` to list all the active applications and their status.

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
