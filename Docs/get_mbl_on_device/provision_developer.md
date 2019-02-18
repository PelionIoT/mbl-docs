# Developer credentials for Pelion Device Management

<!--https://confluence.arm.com/display/mbedlinux/Pelion+Provisioning+During+Development-->
<!--https://confluence.arm.com/display/mbedlinux/Development+Provisioning+Use-Cases-->

<!--Assumption: a team of developers are working with one or more devices that are associated with a single pelion account. They all have their own account, with one or more API keys.-->

To connect your devices to Pelion Device Management, you must provision the device with security credentials that establish trust with the cloud services. Device Management offers a **developer certificate** to connect a development device to a Device Management account. The developer certificate doesn't require setting up a certificate authority (unlike the full credentials for production devices). Instead, you can simply download the certificate from the Device Management Portal. You then need to get the certificate on the device in a process called **provisioning**.

<span class="tips">You can use the same certificate on up to 100 devices. You can generate another certificate if you need to provision more devices.</span><!--not sure I want to state this here - DM might change their policy and we won't know. Better to link to the DM page that explains this, and that is more likely to be updated regularly-->


To prepare an MBL device for Pelion Device Management connectivity:
<!--must add links for all of these!!!-->
1. Create a Pelion Device Management account.
1. Create an API key (using the Device Management Portal). The provisioning process, explained below, will use the API key to generate a developer certificate. It will also save the API key itself to the device, because the device needs the API key to access all service APIs.
1. Create a firmware update authority (to be able to send application updates to the device). This step generates both a public key certificate and its corresponding private key.
1. Get an MBL image (either download a pre-built image or build a new one).
1. Flash the image to the board.
1. Check the device's provisioning status.
1. Provision the device by using MBL CLI to add your Device Management credentials to the device's persistent storage.

<span class="tips">You also need to know your device's vendor and class ID.</span>

## Provisioning process overview

The full provisioning process is:

1. Use the Account Management API to create a developer certificate. This step requires the API key obtained above.

    This step returns:

    * A developer certificate in PEM format.
    * A developer private key in PEM format.

1. Query the Account Management API for bootstrap server information. This step requires the API key obtained above.

    This step returns:

    * The bootstrap server's CA certificate.
    * The bootstrap server's URI.

1. Program the following to the device:

    * The developer certificate.
    * The private key.
    * The bootstrap server information.

1. Generate the fingerprint for the firmware update authority's public key certificate.

1. Program the following to the device:

    * Update certificate fingerprint.
    * Update certificate. <!--where did we get this?--><!--what the difference between this and an update authority certificate? "MBL-CLI also provides a command to create an update authority certificate."-->
    * Vendor and class IDs.

You can use MBL CLI to provision your device dynamically (at runtime).<!--move this - it's stuck in the middle-->

## Persistent storage for user and team credentials

MBL CLI can store and fetch API keys, firmware update authority certificates and developer certificates from a persistent storage location you configure. This simplifies the provisioning workflow and makes API authentication easier<!--why does it make authentication easier?-->

You can save your credentials to either a **user** store or a **team** store:

- User Store: Where API keys are stored for each user.

    Default location: `~/.mbl-store/user/` (permissions for this folder are set to `drwx------`). The folder is owned by the `*nix` user who created the store (the store is automatically created on first use).

    MBL CLI automatically chooses the first API key it finds in the User Store when it needs authentication with the Device Management APIs.
    <!--can it save multiples, or does it always overwrite with a new one?-->

- Team Store: Where developer certificates and firmware update authority certificates for your devices are stored; any team member can provision devices with these credentials (using MBL CLI). You can pick any location accessible to your team, including cloud locations <!--we just said it's to save on your developer machine-->.

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

Prerequisites:

* An `update_default_resources.c` file, created with the `manifest-tool` when you [built your device's first image](../first-image/preparing-device-management-sources.html#creating-an-update-resources-file).<!--what if they got an image from us?-->
* An API key from [Pelion Device Management](https://cloud.mbed.com/docs/latest/integrate-web-app/api-keys.html). Be sure to copy the key when prompted.

To provision your device:

1. Connect the device to your development PC.
1. Run the `select` command. It returns a list of discovered devices; you can select your device by its list index:

    ```bash
    $ mbl-cli select
    Discovering devices. This will take up to 30 seconds.

    1: mbed-linux-os-9999: fe80::21b:63ff:feab:e6a6%enpos623

    $ 1

    ```

1. Store your API key on the device (replacing `<api-key>` with the real API key you saved from the portal).

    ```bash
    mbl-cli save-api-key <api-key>
    ```

    The command validates the key exists in Device Management, then saves the key in the User Store.

1. To provision your device with the developed and update certificates:

    ```bash
    mbl-cli provision-pelion  <cert-name> <update-cert-path> --create-dev-cert
    ```

    You must pass in:

    * A name for your developer certificate. Because we passed in `--create-dev-cert`, MBL CLI will create a new developer certificate with the given name, then save it in the Team Store for use with other devices. If we had omitted this option, MBL CLI would have searched the Team Store for a developer certificate with the given name.

    * The path to the `update_default_resources.c` file created using the manifest tool.

    MBL CLI injects the certificates into your selected device's secure storage.