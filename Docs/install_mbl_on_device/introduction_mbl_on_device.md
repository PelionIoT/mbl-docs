# Mbed Linux OS Images Types

MBL provides 3 different types of images:

 - Evaluation Image: pre-compiled image ready to be flashed on the supported targets. It is almost the same as a Development Image;

- Development Image: To be built using the build-mbl tool and contains packages, applications, and configurations suitable for the development of a product. To generate the mbl-image-development, the DISTRO is set to mbl-development;

- Production Image: To be built using the build-mbl tool, where different configuration options have been introduced to provide additional security protection to production images. To generate the mbl-image-production, the DISTRO is set to mbl-production. This work is still in progress;


# Installing MBL on a new device

To install Mbed Linux OS on a new device:

1. Review our list of [supported development boards](../first-image/hardware.html) and set up your [development environment](../first-image/development-environment.html).
1. Get an MBL image: You can get a [prebuilt evaluation image](../first-image/downloading-an-evaluation-image.html), or [build your own](../first-image/building-a-developer-image.html).
1. [Write the image to the device](../first-image/writing-an-image-to-supported-boards.html).
1. Provision the device with [Pelion Device Management credentials and an API key](../first-image/provisioning-for-pelion-device-management.html) so that it can connect to your Device Management account.
1. [Set up your network connection](../first-image/connecting-to-a-network-and-pelion-device-management.html) and [test your Device Management connectivity](../first-image/verifying-that-the-device-is-connected-to-device-management.html).

<span class="images">![](../Figures/install_process.png)</span>
