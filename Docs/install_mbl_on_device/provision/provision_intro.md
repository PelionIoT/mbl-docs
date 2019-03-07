# Provisioning for Pelion Device Management

<span class="notes">**Note**: This document assumes you already [have a device running MBL](../first-image/index.html).</span>

To connect your devices to Pelion Device Management, you must provision the device with security credentials that establish trust with the cloud services. Device Management offers a **developer certificate** to connect a development device to a Device Management account. The developer certificate doesn't require setting up a certificate authority (unlike the full credentials for production devices). Instead, you can simply download the certificate from the Device Management Portal. You then need to get the certificate on the device in a process called **provisioning**.

To use Pelion Device Management for firmware updates of either your file system or your applications, you must also provision your device with an **update authenticity certificate**. This certificate is used to sign and verify [firmware update manifests](https://cloud.mbed.com/docs/latest/updating-firmware/firmware-manifests.html), and is created using the manifest tool.

Starting with Mbed Linux OS 0.6, your devices can be provisioned with developer and update authenticity certificates using MBL CLI 2.0.

<span class="tips">**Tip**: See the Device Management documentation for [background on the developer certificate and its scope](https://cloud.mbed.com/docs/latest/connecting/provisioning-development-devices.html).</span>

## Provisioning process review

The full provisioning process is:

1. Use the Account Management API to create a developer certificate. This step requires the API key.

1. Program the developer certificate to the device. It contains:

    * The bootstrap server CA certificate.
    * The private key.
    * The bootstrap server information.

1. Use the manifest tool to generate the firmware update authenticity certificate.

2. Program `update_default_resources.c` to the device. It contains:

    * Update certificate fingerprint.
    * Update certificate.
    * Vendor and class IDs.


You can use MBL CLI to provision your device dynamically (at runtime). MBL CLI will create the developer certificate for you, but requires an update authenticity certificate (which you can create using the manifest tool). MBL CLI will then program both certificate payloads to the device.
