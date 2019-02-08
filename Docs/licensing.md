## Licensing

### Source code

The source code for Mbed Linux OS that Arm provides is open source and follows the REUSE practices v2.0 for stating license information. Each Arm repository lists the licenses used in that repository following this practice.

### Binaries

The Mbed Linux OS distribution is made up of an aggregation of Open Source Software under many different licenses. The OpenEmbedded build framework that is used by Mbed Linux OS outputs a list of the licenses from the build recipes used to create the distribution.

The full list of licenses for a particular build of Mbed Linux OS can be found in either the OpenEmbedded builds deploy directory (for example: `build-mbl/tmp-mbl-glibc/deploy/licenses`) or if you have used the docker builds artifacts option, they are contained in the `licenses.tar.gz` archive in the `artifacts` directory.

There are three sets of licenses per machine:

* `mbl-image-development-<MACHINE>-<DATE>` - the development image licenses
* `mbl-image-initramfs-<MACHINE>-<DATE>` - the initramfs licenses (used in both development and production)
* `mbl-image-production-<MACHINE>-<DATE>` - the production image licenses

In each set there are two files that document the licenses:

* `license.manifest` - are all the system packages installed in the file system for that image
* `image_license.manifest` - are the binary files that have been put in the disk image

These license files will be bundled with any disk image binary supplied by Arm.

### GPLv3 statement

To enable efficient update of Mbed Linux OS the distribution is split into several logical binary sections referred to as updatable packages. Each updatable package can be updated individually and contains its own aggregation of software and licenses[3]. To verify that an updatable package is authentic a standard certificate signing procedure is used, where by the package that is run before it will validate that the next package has been signed by an accepted authority.

One updatable package of the Mbed Linux OS distribution - the root file system - contains GPLv3 code. The GPLv3 license section 6[X] indicates that for a “User Product”, “Installation Information” must be provided. For Mbed Linux OS this could be managed by providing a user key (which could be on request) that can only work with the user’s device. This key will allow the user to sign a new version of the root file system and then manually update only their device.
