# Downloading an evaluation image

If you're only interested in developing applications (as opposed to contributing to MBL itself), you can download a prebuilt MBL evaluation image.

## Prerequisites

Using MBL requires:

* A board from our [supported hardware list](../first-image/hardware.html).
* <a href="https://os.mbed.com/account/login/" target="_blank">A Pelion Device Management Account</a>.

## Evaluation images

[MBL evaluation images for the 0.9 release are on GitHub](https://github.com/ARMmbed/mbl-manifest/releases/tag/mbl-os-0.9.0)

## Archive content

All release archives contain:

* `.wic.gz`: a full disk image for the target.

* `.wic.bmap`: a file that helps bmap-tool copy only the necessary parts of the image onto the device.

* `source`: a directory containing source archives for each piece of software in the disk image.

* `license.manifest`: a file containing license information about each package installed on the root filesystem in the disk image.

* `image_license.manifest`: a file containing license information about each BitBake recipe that contributed files to the disk image, but which haven't been installed as packages on the root file system. This excludes the contents of Linux's initramfs.

* `initramfs-license.manifest`: a file containing license information about each package installed into Linux's intramfs.

* `initramfs-image_license.manifest`: a file containing license information about each BitBake recipe that contributed files to Linux's initramfs, but which haven't been installed as packages in the initramfs.

* `licenses.tar`: an archive containing license files for each piece of software in the disk image.

* `pinned-manifest.xml`: a file containing the versions of repositories used to build the disk image.

* `buildinfo.txt`: a file containing the version of the mbl-tools repository used to build the disk image and the command lines used to invoke the scripts in mbl-tools.

* `README`: a list of the files in the release archive.

## Device installation process

1. [Write the image to your board](../first-image/writing-an-image-to-supported-boards.html).
1. [Provision your board so it can connect to your Pelion Device Management account](../first-image/provisioning-for-pelion-device-management.html).
1. [Set up a network connection and make sure you can connect to your account](../first-image/connecting-to-a-network-and-pelion-device-management.html).


***

Copyright © 2020 Arm Limited (or its affiliates)
