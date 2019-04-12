# Updating an application

There are two ways to update an MBL application:

* [Using MBL CLI](#using-mbl-cli).
* [Using the manifest tool](#using-the-manifest-tool).

## Using MBL CLI

To install or update an application:

1. Prepare a tar file containing an OPKG package (`.ipk` file), as explained in the [manifest tool section above](#using-the-manifest-tool)

1. Transfer an application update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli put <application update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the payload name for an application update, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli -a 169.254.111.222 put payload.tar /scratch
   ```

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


## Using the manifest tool

1. Create an update payload file: Make a `tar` file containing the `.ipk` files for the applications to update.

    Note that the `.ipk` files must be in the `.tar`'s root directory, not a subdirectory (we use `-C` in the tar command to specify a directory where the package is).

    For example, to create an update payload file at `/tmp/payload.tar` containing an `.ipk` file with the path `/home/user01/my_app.ipk`, run:

    ```
    user01@dev-machine:~$ tar -cf /tmp/payload.tar -C /home/user01  my_app.ipk
    ```

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

    For a root file system update, the process can take a long time, depending on your file size and network bandwidth.

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
