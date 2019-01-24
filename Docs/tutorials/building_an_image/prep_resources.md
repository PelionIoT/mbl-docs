## Preparing Device Management sources

### Downloading Device Management developer credentials

MBL handles Device Management connectivity on behalf of the device, rather than relying on the user application to do it. This means you need to add your Device Management credentials to the working directory MBL builds from. For development environments, Pelion offers a *developer certificate* for quick connection:

1. Create cloud credentials directory, for example `./cloud-credentials`.
2. To create a Pelion developer certificate (`mbed_cloud_dev_credentials.c`), follow the instructions for [creating and downloading a developer certificate](../getting-started/provisioning-development.html).
3. Add the developer certificate to the credentials directory you've created.

<span class="notes">**Note:** These instructions use the developer workflow and certificates as connectivity credentials. Do not use these credentials for production devices.</span>

### Creating an update resources file

MBL and Device Management support over-the-air updates for devices, which you can do [in a later tutorial in this sequence](../getting-started/tutorial-updating-mbl-devices-and-applications.html). For a device to be updatable, it needs to include some update resources (such as credentials) in the original image.

1. Create an update resources directory, such as `./update-resources`:

    ```
    mkdir ./update-resources
    ```

1. Move into the new directory:

    ```
    cd ./update-resources
    ```

1. Initialize the manifest tool, and generate Update resources:

    ```
    manifest-tool init -q -d arm.com -m dev-device
    ```

    This generates the `update_default_resources.c` file.

    You will use this file in the build process later.
