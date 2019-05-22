# Provisioning your device

## Prerequisites

* MBL CLI and a developer connection. Please see [installation and developer connection instructions](../develop-apps/setting-up.html).

    <span class="notes">**Note:** If you are running an earlier version (1.x) of MBL CLI, please remove it (and install 2.0).</span>

* We have used [manifest tool](https://github.com/ARMmbed/manifest-tool) 1.4.8 in our testing. You should install the manifest tool in the same virtual environment you created for MBL CLI. 
    Install the manifest tool: 
    ```
    pip install manifest-tool==1.4.8
    ```
 
* See the Device Management documentation [for more information on installing the manifest-tool](https://www.pelion.com/docs/device-management/current/cloud-requirements/manifest-tutorial.html).

* <a href="https://portal.mbedcloud.com/login">A Pelion Device Management account</a>.

## Preliminary steps

### Mandatory: API key and update authenticity certificate

1. Create an API key for [Pelion Device Management](https://cloud.mbed.com/docs/latest/integrate-web-app/api-keys.html). Be sure to copy the key when prompted - you will need to store it on the device before you begin provisioning.

1. Create an `update_default_resources.c` file with your update authenticity certificate, created with the manifest tool:

    1. Create an update resources directory, such as `./update-resources`:

        ```
        mkdir ./update-resources
        ```

    2. Move into the new directory:

        ```
        cd ./update-resources
        ```

     1. Initialize the manifest tool, and generate an `update_default_resources.c` file:

        `manifest-tool init -q -d <domain> -m <device class>`

        Where:

        * `<domain>` is your company's domain, like `arm.com`.
        * `<device class>` is a unique identifier for the device class. If you're in development (using developer credentials), you can use `dev-device`.

### Optional: persistent storage locations

<span class="notes">**Note**: MBL CLI sets up defaults automatically; this manual step is optional, but if you chose to perform it, it should be done before any other provisioning steps.</span>

MBL CLI can fetch and store API keys, firmware update authority certificates and developer certificates from a persistent storage location you can configure. This simplifies the provisioning workflow and makes API authentication easier; MBL CLI will automatically find the API key when it needs to authenticate with the Pelion Device Management service APIs.

You can save your credentials to either a **user** store or a **team** store:

- User Store: Where API keys for each user are stored.

    Default location: `~/.mbl-store/user/` (permissions for this folder are set to `drwx------`). The folder is owned by the `*nix` user who created the store (the store is automatically created on first use).

    MBL CLI can save a single API key in the User Store, which it uses when it needs authentication with the Device Management APIs. If you save another API key, it will overwrite the previously stored key.

- Team Store: Where developer certificates and firmware update authority certificates for your devices are stored; any team member who has access to this folder can provision devices with these credentials (using MBL CLI). If you want to give your team access to this folder, you can set this folder to be shared over a cloud service, or make it discoverable on your network.

    Default location: `~/.mbl-store/team/` (permissions for this folder are set to `drwxrw----`). You should set this store's `user:group` according to the access permissions you have defined for your team.

To specify a storage location, create a config file `~/.mbl-stores.json` with the following contents:

```json
{
    "user": path/to/user/store,
    "team": path/to/team/store
}
```

If, on first use, you do not create your own `~/.mbl-stores.json`, it will be automatically created and populated with the default values.

## Provisioning instructions

To provision your device:

1. Connect the device to your development PC. Follow the instructions to [connect your device](../develop-apps/setting-up.html#setting-up-networking).
2. Run the `select` command. It returns a list of discovered devices; you can select your device by its list index:

    ```bash
    $ mbl-cli select
    Discovering devices. This will take up to 30 seconds.

    1: mbed-linux-os-9999: fe80::21b:63ff:feab:e6a6%enpos623

    $ 1

    ```

3. Store your API key on the device (replacing `<api-key>` with the real API key you saved from the portal).

    ```bash
    mbl-cli save-api-key <api-key>
    ```

    The command validates the key exists in Device Management, then saves the key in the User Store.

4. To provision your device with the developer and update certificates:

    ```bash
    mbl-cli provision-pelion  <cert-name> <update-cert-name> -c -p <update-cert-path>
    ```

    The arguments are:

    | Argument | Info |
    | --- | --- |
    |`<cert-name>`| A name for your developer certificate. |
    |`<update-cert-name>`| A name for your update authenticity certificate, created earlier using the manifest tool.|
    |`-c`| Tells MBL CLI to create a new developer certificate and save it in your Team Store.|
    |`-p`| Tells MBL CLI to parse an existing `update_default_resources.c` file and save it in the Team Store. For example, if your `update_default_resources.c` is located in `path/to/resources/update_default_resources.c` the form of the invocation would be: `-p path/to/resources/update_default_resources.c` |

    MBL CLI injects the certificates into your selected device's secure storage. MBL CLI also saves the certificates, under the given names, in your Team Store for use with other devices.
