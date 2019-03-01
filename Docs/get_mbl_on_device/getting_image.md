# Downloading an evaluation image

If you're only interested in developing applications (as opposed to contributing to MBL itself), you can download a prebuilt MBL evaluation image.

Each target hardware has its own release archive, with a hardware-specific image:

* [Raspberry Pi 3B or Raspberry Pi 3B+]()
* [Warp7]()
* [PICO-PI with IMX7D SoM]()
* [NXP 8M Mini EVK]()

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

## Where next

1. [Write the image to your board](../first-image/writing-an-image.html).
1. [Provision your board so it can connect to your Pelion Device Management account](../first-image/provisioning-for-pelion-device-management.html).
1. [Set up a network connection and make sure you can connect to your account](../first-image/connecting-to-a-network-and-pelion-device-management.html).
