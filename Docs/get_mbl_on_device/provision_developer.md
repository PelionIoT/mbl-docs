# Provisioning for Pelion Device Management

<span class="notes">**Note**: This document assumes you already [have a device running MBL](../first-image/index.html).</span>

To connect your devices to Pelion Device Management, you must provision the device with security credentials that establish trust with the cloud services. Device Management offers a **developer certificate** to connect a development device to a Device Management account. The developer certificate doesn't require setting up a certificate authority (unlike the full credentials for production devices). Instead, you can simply download the certificate from the Device Management Portal. You then need to get the certificate on the device in a process called **provisioning**.

<span class="tips">**Tip**: See the Device Management documentation for [background on the developer certificate and its scope](https://cloud.mbed.com/docs/latest/connecting/provisioning-development-devices.html).</span>

## Prerequisites

* The [manifest tool](https://github.com/ARMmbed/manifest-tool). See the Device Management documentation [for an installation guide](https://cloud.mbed.com/docs/latest/cloud-requirements/manifest-tutorial.html).

* <a href="https://os.mbed.com/account/login/" target="blank">A Pelion Device Management Account</a>

* An API key from [Pelion Device Management](https://cloud.mbed.com/docs/latest/integrate-web-app/api-keys.html). Be sure to copy the key when prompted.

* An `update_default_resources.c` file, created with the manifest tool:

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

## Persistent storage locations

<span class="notes">**Note**: MBL CLI sets up defaults automatically; this manual step is optional, but if you chose to perform it, it should be done before any other provisioning steps.</span>

MBL CLI can fetch and store API keys, firmware update authority certificates and developer certificates from a persistent storage location you can configure. This simplifies the provisioning workflow and makes API authentication easier; MBL CLI will automatically find the API key when it needs to authenticate with the Pelion Device Management service APIs.

You can save your credentials to either a **user** store or a **team** store:

- User Store: Where API keys for each user are stored.<!--we don't use "per", as it's Latin and contradicts the Arm style guide. Despite half of English being Latin-->

    Default location: `~/.mbl-store/user/` (permissions for this folder are set to `drwx------`). The folder is owned by the `*nix` user who created the store (the store is automatically created on first use).

    MBL CLI can save a single API key in the User Store, which it uses when it needs authentication with the Device Management APIs. If you save another API key, it will overwrite the previously stored key.

- Team Store: Where developer certificates and firmware update authority certificates for your devices are stored; any team member who has access to this folder can provision devices with these credentials (using MBL CLI), because these aren't tied to a particular Device Management<!--Pelion isn't a single account right now to my knowledge - Data need their own users--> user account. If you want to give your team access to this folder, you can set this folder to be shared over a cloud service, or make it discoverable on your network.

    Default location: `~/.mbl-store/team/` (permissions for this folder are set to `drwxrw----`). You should set this store's `user:group` according to the access permissions you have defined for your team.

To specify a storage location, create a config file `~/.mbl-stores.json` with the following contents:

```json
{
    "user": path/to/user/store,
    "team": path/to/team/store
}
```

If, on first use, you do not create your own `~/.mbl-stores.json`, it will be automatically created and populated with the default values.

## Provisioning process overview

The full provisioning process is:

1. Use the Account Management API to create a developer certificate. This step requires the API key obtained above.

1. Program the following to the device:
<!--I don't think we explained these at any point-->
    * The bootstrap server CA certificate.
    * The private key.
    * The bootstrap server information.

    These are stored<!--we've used "store" to mean something else until now; is there another word we can use here? "included"?--> in the developer certificate referred to in the previous step.
    <!--can we say "program the developer certificate, which contains a b c "?-->

1. Use the manifest tool to generate the firmware update authenticity certificate.

1. Program the following to the device:

    * Update certificate fingerprint.<!--where did we get this?-->
    * Update certificate.
    * Vendor and class IDs.

    These are stored<!--same questions--> in the update authenticity certificate the manifest tool creates (`update_default_resources.c`).<!--again, might be clearer to say program `update_default_resources.c`, which contains....-->

You can use MBL CLI to provision your device dynamically (at runtime). MBL CLI will create the developer certificate for you, but requires an update authenticity certificate (which you can create using the manifest tool). MBL CLI will then program both certificate payloads to the device.

## Provisioning

To provision your device:

1. Connect the device to your development PC. Follow the tutorial to [connect and assign an IPv4 address](../develop-apps/setting-up.html#setting-up-networking)
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
    mbl-cli provision-pelion  <cert-name> <update-cert-path> --create-dev-cert
    ```

    You must pass in:

    * A name for your developer certificate. Because we passed in `--create-dev-cert`, MBL CLI will create a new developer certificate with the given name, then save it in the Team Store for use with other devices. If we had omitted this option, MBL CLI would have searched the Team Store for a developer certificate with the given name.

    * The path to the `update_default_resources.c` file created using the manifest tool.

    MBL CLI injects the certificates into your selected device's secure storage.
