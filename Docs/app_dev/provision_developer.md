# Developer credentials for Pelion Device Management

<!--https://confluence.arm.com/display/mbedlinux/Pelion+Provisioning+During+Development-->
<!--https://confluence.arm.com/display/mbedlinux/Development+Provisioning+Use-Cases-->

<!--Assumption: a team of developers are working with one or more devices that are associated with a single pelion account. They all have their own account, with one or more API keys.-->

## Process goal and overview

Pelion Device Management offers a **developer certificate** to connect a development device to a Device Management account. The developer certificate doesn't require setting up a certificate authority (unlike the full credentials for production devices). Instead,  developers can simply download the certificate from the Device Management Portal. They then need to get the certificate on the device in a process called **provisioning**.

Preparing an MBL device for Pelion Device Management Connectivity requires:

1. Create a Pelion Device Management account.
1. Create an API key (using the Device Management Portal). The provisioning process, explained below, will use the API key to generate a developer certificate and to access all service APIs.
1. Create a firmware update authority (to be able to send applications to the device). This step generates both a public key certificate and its corresponding private key.
1. Get an MBL image (either download a pre-built image or build a new one).
1. Flash the image to the board.
1. Check the device's provisioning status.
1. Provision a new device or reprovision an existing device.

<span class="tips">You also need to know your device's vendor and class ID.</span>

## Provisioning process breakdown

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
    * Update certificate.
    * Vendor and class IDs.

<span class="tips">**Tip**: MBL can store credentials for either individual developers or entire development teams.</span>

<!--do we want to discuss KCM? Is this page actually for app developers, in which case they won't care what's happening on the device so long as it works, or is it for people who may need to properly understand both sides?-->

## Tools

## Content from Rob W - unedited

### Device Provisioning Support

To connect your devices to Pelion Device Management, you must provision the device with security credentials that establish trust with the cloud services.

During development, you don't need to go through the full factory provisioning process every time you want to test a device or a software version.

Pelion Device Management offers a developer mode to speed up the development process. This mode relies on a developer certificate. The developer certificate allows your test device to connect to your Device Management account. Developer certificates are used to establish trust between the device and a Pelion Device Management account.

You can use the same certificate on up to 100 devices. You can generate another certificate if you need to provision more devices.

Previously the developer certificate had to be added to your Mbed Linux OS build configuration.
Now you can provision your devices dynamically at runtime using the MBL-CLI.

To update the firmware on a device you must upload a signed [firmware manifest](https://cloud.mbed.com/docs/current/updating-firmware/firmware-manifests.html) to the Pelion cloud.

An update authority certificate is used to sign the firmware update manifest. The manifest is then uploaded to Pelion Device Management and can be used to initiate a firmware update campaign. MBL-CLI will accept an update certificate created with the `manifest-tool` and use this to provision your device. MBL-CLI also provides a command to create an update authority certificate. [More information on the update authority certificate](https://cloud.mbed.com/docs/current/updating-firmware/update-auth-cert.html).

Pelion provides several REST APIs relating to device management. The API keys used to authenticate with the Pelion APIs are created in the Pelion Device Management portal. MBL-CLI requires use of an API key to perform provisioning activities.

MBL-CLI defines persistent storage locations on your developer machine for Pelion API keys, firmware update authority certificates and developer certificates. This storage feature helps to simplify the provisioning workflow and allow easier authentication with Pelion APIs.

#### Persistent Storage

It is possible to save Pelion Cloud credentials to either a 'User Store' or 'Team Store' depending on the context of the object being saved.

These stores are located on your developer machine at locations you can optionally specify.

- User Store: This is where per user Pelion API keys are stored.
- Team Store: This is where developer certificates and firmware update authority certificates for your devices are stored.

The Team Store can be set to a location accessible by your team (for example a cloud share). If your team then specify this location for their Tema Store config (see below) they can use the CLI tool to access the device certificates to provision other devices.

To specify a storage location you must provide a config file `~/.mbl-stores.json` with the following contents:

```json
{
    "user": path/to/user/store,
    "team": path/to/team/store
}
```

The MBL-CLI will use the following defaults if no config is given:

- Default location for the User Store is `~/.mbl-store/user/` (Permissions for this folder are set to drwx------). Owning user is the `*nix` user who created the store (the store is automatically created on first use).
- Default location for shared storage in the Team Store is `~/.mbl-store/team/` (Permissions for this folder are set to drwxrw----). You should set this store's user:group according to access permissions you have defined for your team.

The defaults will be saved to `~/.mbl-stores.json` after first use, where they can be edited if required.

#### Developer Provisioning Workflow

The developer provisioning workflow consists of the following steps:

- Obtain a developer certificate from the Pelion Device Management service.
- Inject the certificate into secure storage on the target device.

To inject a developer certificate into your device using MBL-CLI, follow the steps below.

NOTE: This tutorial assumes you have already created a `update_default_resources.c` file using the `manifest-tool` as described [here](https://os.mbed.com/docs/mbed-linux-os/v0.5/getting-started/preparing-device-management-sources.html#creating-an-update-resources-file).

Connect your Mbed Linux device to your development PC. Run the `select` command.
The select command will return a list of discovered devices, you can select your device by its list index.

```bash
$ mbl-cli select
Discovering devices. This will take up to 30 seconds.

1: mbed-linux-os-9999: fe80::21b:63ff:feab:e6a6%enpos623

$ 1

```

Obtain an API key from the Pelion Device Management website. Copy the API key to your clipboard when prompted on the website.

Store your API key using the following command (replacing `<api-key>` with the real API key in your clipboard). This command validates the key exists in the Pelion Cloud. It then saves the key in the User Store.

MBL-CLI will automatically choose the first API key it finds in the User Store when it requires authentication with the Pelion APIs.

```bash
mbl-cli save-api-key <api-key>
```

Run the provisioning command.

```bash
mbl-cli provision-pelion  <cert-name> <update-cert-path> --create-dev-cert
```

This command will provision your selected device with the developer and update certificates. You must pass in a name for your developer certificate and the path to the `update_default_resources.c` file created using the manifest-tool. Because we passed in `--create-dev-cert` MBL-CLI will create a new developer certificate with the given name, then save it in the Team Store for use with other devices. If we had omitted this option, MBL-CLI would search the Team Store for a dev certificate with the given name. After obtaining the certificates, MBL-CLI will inject it into your selected device's secure storage.
