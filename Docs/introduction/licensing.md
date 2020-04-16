# Licensing

## Source code

The source code for Mbed Linux OS that Arm provides is open source and follows the [**REUSE practices version 2.0**](https://reuse.software/practices/2.0/) for stating license information. In accordance with this practice, each Arm repository lists the licenses used in that repository.

## Binaries

The Mbed Linux OS distribution is made up of an aggregation of Open Source Software under many different licenses. The OpenEmbedded build framework that Mbed Linux OS uses outputs a list of the licenses from the build recipes used to create the distribution.

The full list of licenses for a particular build of Mbed Linux OS can be found in either the OpenEmbedded builds deploy directory (for example, `build-mbl-development/tmp/deploy/licenses`) or, if you have used the docker builds artifacts option, in the `licenses.tar.gz` archive in the `artifacts` directory.

<span class="notes">The full source code should be made available for a binary build, because some software licenses do not contain specific copyright declarations; instead, the copyrights can be found in the respective source code for that software.</span>

There are upt o three sets of license manifests (lists of the software and licenses) for every machine:

* `mbl-image-development-<MACHINE>-<DATE>`: the development image licenses.
* `mbl-image-initramfs-<MACHINE>-<DATE>`: the initramfs licenses.
* `mbl-image-development-<MACHINE>-<DATE>`: the development image licenses.

In each set, there are two files that document the licenses:

* `license.manifest`: all the system packages installed in the file system for that image.
* `image_license.manifest`: the binary files that have been put in the disk image.

These license files and full source code are bundled with any disk image binary supplied by Arm.

## GPLv3 statement

To enable efficient update of Mbed Linux OS, the distribution is split into several logical binary sections referred to as *updatable packages*. Each updatable package can be updated individually and contains its own aggregation of software and licenses. To verify that an updatable package is authentic, a standard certificate signing procedure is used, whereby each running package validates that the next package has been signed by an accepted authority.

One updatable package of the Mbed Linux OS distribution - the root file system - contains GPLv3 code. The GPLv3 license section 6 indicates that for a “User Product”, “Installation Information” must be provided. For Mbed Linux OS this could be managed by providing a user key (which could be on request) that can only work with the user’s device. This key will allow the user to sign a new version of the root file system and then manually update only their device.

## Acknowledgements

Mbed Linux OS includes OpenSSL and the following acknowledgements are applicable:

    This product includes software developed by the OpenSSL Project for use in the OpenSSL Toolkit.  (http://www.openssl.org/)
    This product includes cryptographic software written by Eric Young (eay@cryptsoft.com)


***

Copyright © 2020 Arm Limited (or its affiliates)
