## Developer credentials for Pelion Device Management

<!--https://confluence.arm.com/display/mbedlinux/Pelion+Provisioning+During+Development-->
<!--https://confluence.arm.com/display/mbedlinux/Development+Provisioning+Use-Cases-->

<!--Assumption: a team of developers are working with one or more devices that are associated with a single pelion account. They all have their own account, with one or more API keys.-->

### Process goal and overview

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

### Provisioning process breakdown

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

### Tools
