<h2 id="mbl-pelion-connect">Building an Mbed Linux OS image and connecting to Pelion Device Management</h2>

<span class="warnings">**Warning:** These instructions use the developer workflow and certificates as connectivity credentials. Do not use these credentials for production devices.</span>

### Overview

Mbed Linux OS (MBL) handles Pelion Device Management connectivity on behalf of the device, rather than relying on the user application to do it as Mbed OS does. This means that before you build the MBL image that runs on your device, you need to add your Pelion credentials to the working directory MBL builds from.

### Prerequisites

Please [make sure you have suitable hardware and all the required software]().

### Release branches

Each release has its own branch, such as `mbl-os-0.5`. Throughout this guide, the release branch is referred to as `mbl-XXX`. Replace it with the name of the branch you're working with. If you don't know which branch to work with, [use the latest branch available]().<!--is there a release list or a tag list I can send them to? I don't want them to have to go to the repo to count branches-->
<!--- Could we link to release notes? *Mel*--->

### Downloading Device Management developer credentials

To connect your device to your Pelion Device Management account, you need to add a credentials file to your application before you build it. For development environments, Pelion offers a *developer certificate* for quick connection:

1. Create cloud credentials directory, e.g. `./cloud-credentials`
1. To create a Pelion developer certificate (`mbed_cloud_dev_credentials.c`), follow the instructions for [creating and downloading a developer certificate](https://cloud.mbed.com/docs/v1.2/provisioning-process/provisioning-development.html#creating-and-downloading-a-developer-certificate).
<!--This is where being able to splice the content would be great; I would rather transclude it here than send them to another page. I'll see what the Web Team has.-->
1. Add the developer certificate to the credentials directory you've created.

### Creating an Update resources file

1. Create an update resources directory, such as `./update-resources`:
```
mkdir ./update-resources && cd ./update-resources
```
1. Initialize manifest tool settings and generate Update resources by running the following commands:
```
manifest-tool init -q -d arm.com -m dev-device
```
This generates a file `update_default_resources.c` that is required during the build process.

### mbl-tools
mbl-tools repository provides a collection of tools and recipes related to the build and test of Mbed Linux OS.
Checkout the `mbl-tools-XXX` branch from [mbl-tools git repository](https://github.com/ARMmbed/mbl-tools) using the following command
```
mkdir /path/to/working-dir/; cd /path/to/working-dir/
$ git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-tools-XXX
```

### Building scripts

The `run-me.sh` script creates and launches a docker container to encapsulate the MBL build environment then launch a build script, `build.sh`, inside the container to do the heavy lifting. <!---too idiomatic--->

There are a variety of options controlling what is built and how. The general form of a `run-me.sh` invocation is:
```
./mbl-tools/build-mbl/run-me.sh [RUN-ME.SH OPTIONS]... -- [BUILD.SH OPTIONS]...
```
Note the use of `--` to separate options to `run-me.sh` from options that are passed through to `build.sh`

To invoke the `run-me.sh` help menu use:
```
./mbl-tools/build-mbl/run-me.sh -h
```

To invoke the `build.sh` help menu use:
```
./mbl-tools/build-mbl/run-me.sh -- -h
```

#### Mandatory build script options

An example using all mandatory options:
```
./mbl-tools/build-mbl/run-me.sh --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c --outputdir /path/to/artifacts -- --branch mbl-XXX --machine <MACHINE>
```

* For more information about `--branch` option see [Select release branch](#Select-release-branch).
* For more information about `--machine` option see [Select target device](#Select-target-device).
* For more information about `--builddir` option see [Creating multiple build directories](#Creating-multiple-build-directories).
* For more information about `--outputdir` option see [Build Artifacts](#Build-Artifacts).
* For more information about `--inject-mcc` option see [Using Device Management Client Credentials](#Using-Device-Management-Client-Credentials)

##### Selecting a release branch

Different branches of MBL can be checkout and built by passing the `--branch` option through to `build.sh`.  The bleeding edge of mainline development takes place on the 'master' branch.

Checkout and build the release branch `mbl-XXX` for MBL:
```
./mbl-tools/build-mbl/run-me.sh -- --branch mbl-XXX --machine raspberrypi3-mbl
```

##### Selecting a target device

In order to select the target device use `--machine` option as follows:
```
./mbl-tools/build-mbl/run-me.sh -- --machine <MACHINE>
```
Select the <MACHINE> value for your MBL device from the table below:

| Device | MACHINE |
| ---  | --- |
| Warp7 | `imx7s-warp-mbl` |
| Raspberry Pi 3 | `raspberrypi3-mbl` |


##### Creating multiple build directories

The build scripts by default create and use a build directory under the current working directory.  An alternative build directory can be specified using the `--builddir` option to `run-me.sh`:
```
./mbl-tools/build-mbl/run-me.sh --builddir /path/to/my-build-dir
```

It is mandatory to use different build directories for each target device build.


##### Building Artifacts

Each build produces a variety of build artifacts including a pinned manifest, target specific images and license information.
To get build artifacts out of a build, pass the `--outputdir` option to specify which directory the build artifacts should be placed in:
```
mkdir /path/to/artifacts
./mbl-tools/build-mbl/run-me.sh --outputdir /path/to/artifacts
```

##### Building outputs

The build process creates the following files (which you need later):

* **A full disk image**: This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware.

<span class="tips">**Tip:** The image is created by the build process using the Wic tool from OpenEmbedded. For more information about Wic, see [the Yocto Mega Manual](https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#creating-partitioned-images-using-wic).</span>

* **A block map of the full disk image**: This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the IoT device. Some blocks of the image represent unused storage space that does not actually need to be written.
* **A root filesystem archive**: This is a compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the next tutorial]()).
<!--Looking at the update bit makes me wonder whether this is empty the first time you build an image.-->

**File locations**

The paths of these files are given in the table below, where `<MACHINE>` should be replaced with the MACHINE value for your device from the table in [Section 6](#set-up-build-env).

| File | Path |
| --- | --- |
| Full disk image           | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.wic.gz`   |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.wic.bmap` |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.tar.xz`   |

Test image is also being built, and contains more packages for testing and debuging (such as optee test suite, dropbear to support ssh during development and test, strace utility, and more).

| File | Path |
| --- | --- |
| Full test disk image           | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.wic.gz`   |
| Full test disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.wic.bmap` |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.tar.xz`   |

##### Using Device Management Client Credentials

The current Device Management Client requries key material to be statically built into the cloud client binary.
This is a temporary measure that is replaced with a dynamic key injection mechanism shortly, see [Download Mbed Cloud dev credentials file](#Download-Mbed-Cloud-dev-credentials-file) and [Create an Update resources file](#Create-an-Update-resources-file) sections.

In the meantime, the build scripts provide a work around using the `--inject-mcc` option:
```
./mbl-tools/build-mbl/run-me.sh --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c
```

#### Optional build script options

Optional build script options can be used to improve the development process.

##### Caching downloaded source artifacts

The build process involves the download of many source artifacts.  It is possible to cache downloaded source artifacts between successive builds. In practice, the cache mechanism is considered to be robust for successive builds.  It should not be used for parallel builds.

For example, to designate a directory to hold cached downloads between successive builds, pass the `--downloaddir` option to `run-me.sh`:
```
mkdir /path/to/downloads
./mbl-tools/build-mbl/run-me.sh --downloaddir /path/to/downloads
```

##### Pinned Manifests and Rebuilds

Each build produces a pinned manifest as a build artifact. A pinned manifest is a file that encapsulates sufficient version information to allow an exact rebuild. To get the pinned manifest for a build, use the `--outputdir` option to get the build artifacts:
```
mkdir /path/to/artifacts
./mbl-tools/build-mbl/run-me.sh --outputdir /path/to/artifacts
```
This produces the file `pinned-manifest.xml` in the directory specified with `--outputdir`.

To rebuild using a previously pinned manifest, use the `--external-manifest` option:
```
./mbl-tools/build-mbl/run-me.sh --external-manifest /path/to/pinned-manifest.xml
```

### Example quick start build lines

Following examples assumes that you already downloaded Device Management developer  credentials and created an update resources file, see [Download Mbed Cloud dev credentials file](#Download-Mbed-Cloud-dev-credentials-file) and [Create an Update resources file](#Create-an-Update-resources-file) sections.

Also, assuming that the user created an output directory for both Warp7 and RPi3 (e.g. `./artifacts-warp7` and `./artifacts-rpi3`).

#### Warp7 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --outputdir ./artifacts-warp7 -- --machine imx7s-warp-mbl --branch mbl-XXX
```

#### Raspberry Pi 3 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --outputdir ./artifacts-rpi3 -- --machine raspberrypi3-mbl --branch mbl-XXX
```

## Writing and booting the disk image 

This section contains instructions for writing the full disk image to a:
* Warp7 device
* Raspberry Pi 3 device

### User in dialout group

For both Warp7 and Raspberry Pi 3 devices, add the current user to the dialout group to allow access to ``/dev/ttyUSBn`` devices without using sudo:
<!--Is this mandatory, or just nicer?-->
```
sudo usermod -a -G dialout $USER
```
Verify group memberships by typing "groups":
```
groups
<your_user> dialout
```
The user might need to reboot PC before the group membership takes effect for their shell process.
<!--And this is okay because we've done the build, so we're not limited to the original shell instance anymore, right?-->

### Warp7 devices

To write your disk image to the Warp7's flash device, you must first access the Warp7's serial console. To do this:
1. Connect both the Warp7's I/O USB socket (on the I/O board) and the Warp7's mass storage USB socket (on the CPU board) to your PC.

You should now be able to see a USB TTY device, such as, `/dev/ttyUSB0`, on your PC.
1. Connect to the Warp7's console using a command such as:
    ```
    minicom -D /dev/ttyUSB0
    ```
    Use the following settings:
    * Baud rate 115200.
    * [8N1](https://en.wikipedia.org/wiki/8-N-1) encoding.
    * No hardware flow control.
1. Check the current storage devices on your PC:
    ```
    ls -l /dev/disk/by-id/
    ```
    A list of devices displays, similar to the following:
    ```
    total 0
    lrwxrwxrwx 1 root root  9 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39 -> ../../sda
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39-part1 -> ../../sda1
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39-part2 -> ../../sda2
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39-part3 -> ../../sda3
    lrwxrwxrwx 1 root root  9 Mar 19 10:38 ata-HL-DT-ST_DVD+_-RW_GHB0N_K8RD9II5408 -> ../../sr0
    lrwxrwxrwx 1 root root  9 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A -> ../../sdb
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part1 -> ../../sdb1
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part2 -> ../../sdb2
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part3 -> ../../sdb3
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part4 -> ../../sdb4
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part5 -> ../../sdb5
    ```
    You'll need to compare this output in the following steps, so save it for reference.
1. If you got a U-boot prompt <!--on the device or the terminal?-->, continue to the next step.
   If you got an operating system boot (for example, Android), reboot the device until you get a U-boot prompt, then press any key to prevent the operating system from booting again. Continue to the next step.
1. To expose the Warp7's flash device to Linux as USB mass storage, in the U-boot prompt type:
    ```
    ums 0 mmc 0
    ```
    On the Warp7 you should now see an ASCII-art "spinner", and on your PC you should see new storage devices:
    ```
    ls -l /dev/disk/by-id/
    ```
    In our example, the Warp7 appeared as `usb-Linux_UMS_disk_0` (the partitions on the device are also shown):
    ```
    total 0
    lrwxrwxrwx 1 root root  9 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39 -> ../../sda
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39-part1 -> ../../sda1
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39-part2 -> ../../sda2
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-Crucial_CT240M500SSD1_140709691C39-part3 -> ../../sda3
    lrwxrwxrwx 1 root root  9 Mar 19 10:38 ata-HL-DT-ST_DVD+_-RW_GHB0N_K8RD9II5408 -> ../../sr0
    lrwxrwxrwx 1 root root  9 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A -> ../../sdb
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part1 -> ../../sdb1
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part2 -> ../../sdb2
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part3 -> ../../sdb3
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part4 -> ../../sdb4
    lrwxrwxrwx 1 root root 10 Mar 19 10:38 ata-ST1000DM003-1CH162_W1D2QL7A-part5 -> ../../sdb5
    lrwxrwxrwx 1 root root  9 Mar 26 14:00 usb-Linux_UMS_disk_0-0:0 -> ../../sdc
    lrwxrwxrwx 1 root root 10 Mar 26 14:00 usb-Linux_UMS_disk_0-0:0-part1 -> ../../sdc1
    lrwxrwxrwx 1 root root 10 Mar 26 14:00 usb-Linux_UMS_disk_0-0:0-part2 -> ../../sdc2
    lrwxrwxrwx 1 root root 10 Mar 26 14:00 usb-Linux_UMS_disk_0-0:0-part3 -> ../../sdc3
    ```
    `mbl-console-image-imx7s-warp-mbl.wic.gz` is a full disk image so should be written to the whole flash device, not a partition.
    The device file for the whole flash device is the one without `-part` in the name (`/dev/disk/by-id/usb-Linux_UMS_disk_0-0:0` in this example).
1. Ensure that none of the Warp7's flash partitions are mounted by running the following command (you may need to adjust the path depending on the name of the storage device):
    ```
    sudo umount /dev/disk/by-id/usb-Linux_UMS_disk_0-0:0-part*
    ```
1. From a Linux prompt, write the disk image to the Warp7's flash device using the following command:
    ```
    sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image/images/mbl-console-image-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image/images/mbl-console-image-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>
    ```
    replacing `<device-file-name>` with the correct device file for the Warp7's flash device.

    This action may take some time.
1. When `bmaptool` has finished, eject the device:
    ```
    sudo eject /dev/disk/by-id/<device-file-name>
    ```
    Replace `<device-file-name>` with the correct device file for the Warp7's flash device.
1. On the Warp7's U-boot prompt, press Ctrl-C to exit USB mass storage mode.
1. Reboot the Warp7:
    ```
    reset
    ```
    The device should now boot into MBL.

### Raspberry Pi 3 devices

1. Connect a micro SD card to your PC. You should see:
    * The SD card device file in `/dev`, probably as `/dev/sdX` for some letter `X` (for example, `/dev/sdd`).
    * Device files for its partitions `/dev/sdXN` for the same letter `X` and some numbers `N` (for example, `/dev/sdd1` and `/dev/sdd2`).

    In the commands below, replace `/dev/sdX` with the device file name for the SD card _without_ a number at the end. You can use `lsblk` to identify the name of the SD card device.
1. Ensure that none of the micro SD card's partitions are mounted by running:
    ```
    sudo umount /dev/sdX*
    ```
    Replace `/dev/sdX` as mentioned above.
1. Write the disk image to the SD card device (not a partition on it):
    ```
    sudo bmaptool copy --bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-console-image/images/mbl-console-image-raspberrypi3-mbl.wic.bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-console-image/images/mbl-console-image-raspberrypi3-mbl.wic.gz /dev/sdX
    ```
    Replace `/dev/sdX` as mentioned above.

    This action may take some time.
1. When `bmaptool` has finished, eject the device:
    ```
    sudo eject /dev/sdX
    ```
1. Detach the micro SD card from your PC and plug it into the Raspberry Pi 3.
1. You need access to the device's console, so before powering it on the, either:
   * Connect it to a monitor and keyboard (using its HDMI and USB sockets).
   * Connect it to your PC. For example, if you're using a [C232HD-DDHSP-0](http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_C232HD_UART_CABLE.pdf) cable, use [this pin numbering reference](https://www.element14.com/community/servlet/JiveServlet/previewBody/73950-102-10-339300/pi3_gpio.png) and connect USB-UART colored wires as follows:
       * **Black** wire to pin **06**
       * **Yellow** wire to pin **08**
       * **Orange** wire to pin **10**

   ![](./pics/rpi3_uart-connection.JPG)

    The cable's TX and RX are used to communicate with the board.

    Connect the other end of the C232HD-DDHSP-0 cable to the USB port on your PC.
1. After connecting the Raspberry Pi 3, from your PC, run the command <!--why? what does it do?-->:
    ```
    minicom -D /dev/ttyUSB0
    ```
    Use the following settings:
    * Baud rate 115200.
    * [8N1](https://en.wikipedia.org/wiki/8-N-1) encoding.
    * No hardware flow control.
1. Connect the Raspberry Pi 3's micro USB socket to a USB power supply. It should now boot into MBL.

### Logging in to MBL

To log in to MBL, enter the username `root` with no password.
<!--- Does it prompt me automatically? Do I need to just wait? Where do I enter the credentials? *Mel*--->

### Setting up a network connection

If your device is connected to a network with a DHCP server using Ethernet, then it automatically connects to that network. Otherwise, follow [the instructions](setting-up-wifi-in-mbed-linux-os.html) for setting up Wi-Fi in MBL.

### Verifying that the device is connected to Device Management

While the device boots into MBL, `mbl-cloud-client` should automatically start and connect to Pelion. You can check whether it has connected by:
* Checking the device status on the [Device Managmenent Portal](https://portal.mbedcloud.com/).<!--I will need to add the images in our publishing format-->
* Reviewing the log file for `mbl-cloud-client` at `/var/log/mbl-cloud-client.log`.

If your device hasn't automatically connected to Pelion, it could be that:
* Networking wasn't configured before the device was rebooted. Check your configurations and reboot the device.<!--Easy enough to direct them to the WiFi bit, but is there anything that may have gone wrong with DHCP?-->
* There are issues with the network. The device retries periodically, but you may need to restart `mbl-cloud-client`:
    ```
    /etc/init.d/mbl-cloud-client restart
    ```
