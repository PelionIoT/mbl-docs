## Tutorial: Building an MBL image that connects to Pelion Device Management

<span class="warnings">These instructions use the developer workflow and developer certificates. Do not use this workflow for production devices.</span><!--My first question as a reader: where can I find the development flow.-->

### Workflow for building Mbed Linux

MBL handles Pelion Device Management connectivity on behalf of the device, rather than relying on the application to do it.
<!--What are the advantages, by the way?-->

This means that before you build the MBL image that runs on your device, you need to add your Pelion credentials to the working directory MBL builds from.


<!--### Build versions

This document contains instructions for building the `master` branch of Mbed Linux OS. Instructions for building other branches are on those branches themselves. To build the latest stable branch of Mbed Linux OS, see the [the `alpha` branch walkthrough][mbl-alpha-walkthrough].-->

<!--Will we still need this comment in November?-->




To build Mbed Linux and do a firmware update over the air, follow these steps:

<!--I will recreate the list when everything's done-->

## Preparing resources for the build

### Create a working directory for the Mbed Linux build

The quick start uses the working directory `~/mbl` for the Mbed Linux OS build.

To create the working directory

```
mkdir ~/mbl
```

<!--Does it have to be called mbl? Will all the tutorials use the same working directory?-->

### Download Mbed Cloud dev credentials file

To connect your device to your Pelion Device Management account, you need to add a credentials file to your application before you build it. For development environments, Pelion offers a *developer certificate* for quick connections:

1. Create the directory `~/mbl/cloud-credentials`.
2. To create a Pelion developer certificate (`mbed_cloud_dev_credentials.c`), follow the instructions for [creating and downloading a developer certificate](provisioning-development.html).
<!--This is where being able to splice the content would be great; I would rather transclude it here than send them to another page. I'll see what the Web Team has.-->
1. Add the developer certificate to the `~/mbl/cloud-credentials` directory.

### Create an Update resources file

Initialize `manifest-tool` settings and generate Update resources:<!--What are they and what do they do? Why should I bother?-->

```
mkdir ~/mbl/manifests && cd ~/mbl/manifests
manifest-tool init -q -d arm.com -m dev-device
```

This generates a file `update_default_resources.c` that is required during the build process.
<!--Don't we need an update certificate as well? Or is this it?-->

## Preparing the build rules and environment

### Download the Yocto/OpenEmbedded manifest

The manifest contains the rules for downloading GitHub resources and building code for Mbed Linux. You need to add it to your working directory:

```
cd ~/mbl
mkdir mbl-master && cd mbl-master
repo init -u ssh://git@github.com/armmbed/mbl-manifest.git -b master -m default.xml
repo sync
```

### Set up the build environment

You need to configure your build environment for your device, including setting your working directory to the build directory<!--what does "setting your working directory to the build directory" mean? I feel almost like we're missing a word.--> (in this case `~/mbl/mbl-master/build-mbl`). To set up your build environment, use the following command:
<!--I think setup-environment relies on the manifest from the previous step and the machine-specific file from the config folder (https://github.com/ARMmbed/meta-mbl/tree/master/conf/machine). Is that correct? Worth stating that explicitly.-->

```
MACHINE=<machine> . setup-environment
```

Select the {MACHINE} value for your Mbed Linux device from the table below:

| Device | MACHINE |
| ---  | --- |
| Warp7 | `imx7s-warp-mbl` |
| Raspberry Pi 3 | `raspberrypi3-mbl` |

For example, to set up the build environment for a Warp7 board, run:

```
MACHINE=imx7s-warp-mbl . setup-environment
```

Once you run the setup-environment script, please:

* Remain in the same shell instance for the rest of the build process; only this shell instance is correctly configured to build MBL properly.
* Do not run setup-environment again. Invoking the script a second time can corrupt environment variables and cause `bitbake` commands to fail in unexpected places.

### Add sources to your build environment

Copy your Pelion developer certificate and Update resources file to the build directory:

```
cp ~/mbl/cloud-credentials/mbed_cloud_dev_credentials.c ~/mbl/mbl-master/build-mbl
cp ~/mbl/manifests/update_default_resources.c ~/mbl/mbl-master/build-mbl
```

## Building the image

### Running the build

To generate these files, run the following `bitbake` command from the build directory (`~/mbl/mbl-master/build-mbl`):

```
bitbake mbl-console-image
```

You will see several "WARNING" messages in the `bitbake` output - these are safe to ignore.

<span class="notes">**Note**: If you are using a virtual machine, the build may take several hours.</span>.

### Build outputs

The build process creates the following files (which you will need to use later):

* **A full disk image**: This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware.

    Tip: The image is created by the build process using the Wic tool from OpenEmbedded. For more information about Wic, see [the Yocto Mega Manual](https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#creating-partitioned-images-using-wic).

* **A block map of the full disk image**: This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the IoT device. Some blocks of the image represent unused storage space that does not actually need to be written.

* **A root filesystem archive**: This is a compressed `.tar` archive, which you will need when you update the device firmware (this topic is covered [in the next tutorial]()). 
<!--Looking at the update bit makes me wonder whether this is empty the first time you build an image.-->

**File locations**

The paths of these files are given in the table below, where `<MACHINE>` should be replaced with the MACHINE value for your device from the table in [Section 6](#set-up-build-env).

| File | Path |
| --- | --- |
| Full disk image           | `~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.wic.gz`   |
| Full disk image block map | `~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.wic.bmap` |
| Root file system archive  | `~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz`   |

## Writing the disk image to your device and booting it

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

The user might need to log out and log in again before the group membership will take effect for their shell process.
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

    You'll see a list of devices similar to the following:

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

1. Ensure that none of the Warp7's flash partitions are mounted by running the following command (you may have to adjust the path depending on the name of the storage device):

    ```
    sudo umount /dev/disk/by-id/usb-Linux_UMS_disk_0-0:0-part*
    ```

1. From a Linux prompt, write the disk image to the Warp7's flash device using the following command:

    ```
    sudo bmaptool copy --bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-console-image-imx7s-warp-mbl.wic.bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-console-image-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>
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

    The device should now boot into Mbed Linux.

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
    bmaptool copy --bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/raspberrypi3-mbl/mbl-console-image-raspberrypi3-mbl.wic.bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/raspberrypi3-mbl/mbl-console-image-raspberrypi3-mbl.wic.gz /dev/sdX
    ```

    Replace `/dev/sdX` as mentioned above.

    This action may take some time.
1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```
1. Detach the micro SD card from your PC and plug it into the Raspberry Pi 3.

1. You need access to the device's console, so before powering it on the, you'll need to either:

    * Connect it to a monitor and keyboard (using its HDMI and USB sockets).
    * Connect it to your PC. For example, if you're using a [C232HD-DDHSP-0](http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_C232HD_UART_CABLE.pdf) cable, use [this pin numbering reference](https://www.element14.com/community/servlet/JiveServlet/previewBody/73950-102-10-339300/pi3_gpio.png) and connect USB-UART colored wires as follows:

        * **Black** wire to pin **06**
        * **Yellow** wire to pin **08**
        * **Orange** wire to pin **10**

        ![](./pics/rpi3_uart-connection.JPG)

    The cable's TX and RX are used to communicate with the board.

    Connect the other end of the C232HD-DDHSP-0 cable to the USB port on your PC.

1. After connecting the Raspberry Pi 3, from your PC run a command <!--why? what does it do?-->:

    ```
    minicom -D /dev/ttyUSB0
    ```

    Use the following settings:
    * Baud rate 115200.
    * [8N1](https://en.wikipedia.org/wiki/8-N-1) encoding.
    * No hardware flow control.
1. Connect the Raspberry Pi 3's micro USB socket to a USB power supply. It should now boot into Mbed Linux.

### Logging in to Mbed Linux

To log in to Mbed Linux, enter the username `root` with no password.

### Setting up a network connection

If your device is connected to a network with a DHCP server using Ethernet, then it automatically connects to that network. Otherwise, follow [the instructions](setting-up-wifi-in-mbed-linux-os.html) for setting up WiFi in Mbed Linux.

### Check whether the device has connected to Pelion Device Management

While the device boots into Mbed Linux, `mbl-cloud-client` should automatically start and connect to Pelion. You can check whether it has connected by:

* Checking the device status on the [Device Managmenent Portal](https://portal.mbedcloud.com/).<!--I will need to add the images in our publishing format-->
* Reviewing the log file for `mbl-cloud-client` at `/var/log/mbl-cloud-client.log`.

If your device hasn't automatically connected to Pelion, it could be that:

* Networking wasn't configured before the device was rebooted. Check your configurations and reboot the device.<!--Easy enough to direct them to the WiFi bit, but is there anything that may have gone wrong with DHCP?-->
* There are issues with the network. The device retries periodically, but you may need to restart `mbl-cloud-client`:

    ```
    /etc/init.d/mbl-cloud-client restart
    ```
