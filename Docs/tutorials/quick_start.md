## Quick start: connecting an MBL device to Pelion Device Management

Why is nothing building?

<!--I would like, as much as possible, to follow the pattern of the current Pelion quick start. This will mean splitting it into a quick start that just connects a device to Pelion so you can control it from the portal, then creating a second tutorial for update.-->

<span class="warnings">These instructions use the developer workflow and developer certificates. Do not use this workflow for production devices.</span>



### Introduction

<!--Well, yes. One needs an introduction.-->



### Build versions

This document contains instructions for building the `master` branch of Mbed Linux OS. Instructions for building other branches are on those branches themselves. To build the latest stable branch of Mbed Linux OS, see the [the `alpha` branch walkthrough][mbl-alpha-walkthrough].



### Workflow for building Mbed Linux and updating the firmware

To build Mbed Linux and do a firmware update over the air, follow these steps:

1. [Prepare your development environment](#prerequisites).
<!--Moved this to its own thing-->

1. [Create a working directory](#create-working-directory).
1. [Download an Mbed Cloud dev credentials file](#get-cloud-credentials) that contains the credentials that the device can use to access your Mbed Cloud account.
1. [Create an Update resources file](#create-update-resources) that contains a certificate used by the device to authenticate new firmware packages.
1. [Download the required Yocto/OpenEmbedded "meta" layers](#get-meta-layers). This data contains rules for downloading and building the code for Mbed Linux.
1. [Set up the build environment](#set-up-build-env). This configures the build for the correct device and injects any user-specific data into the build.
1. [Build Mbed Linux](#build-mbl). This builds Mbed Linux and generates (a) an image that can be written directly to the device and (b) an image that can be used as a firmware update payload.
1. [Write the disk image to your device and boot Mbed Linux](#write-image-and-boot).<!---->
1. [Log in to Mbed Linux](#log-in).
1. [Set up a network connection](#set-up-network)
1. [Check if the device has connected to Mbed Cloud](#check-mbl-cc)
1. [Perform a firmware update](#do-update).


### 1. Create a working directory for the Mbed Linux build

The quick start uses the working directory `~/mbl` for the Mbed Linux OS build.

To create the working directory

```
mkdir ~/mbl
```

<!--Does it have to be called mbl? Will all the tutorials use the same working directory?-->

### 2. Download Mbed Cloud dev credentials file

To connect your device to your Pelion Device Management account, you need to add a credentials file to your application before you build it.

1. Create the directory `~/mbl/cloud-credentials`.
2. To create a **credentials C file** (`mbed_cloud_dev_credentials.c`) and download it to your working directory `~/mbl/cloud-credentials`, follow the instructions for [creating and downloading a developer certificate](https://cloud.mbed.com/docs/v1.2/provisioning-process/provisioning-development.html#creating-and-downloading-a-developer-certificate).

### 4. Create an Update resources file
Initialize `manifest-tool` settings and generate Update resources by running the following commands:
```
mkdir ~/mbl/manifests && cd ~/mbl/manifests
manifest-tool init -q -d arm.com -m dev-device
```
This generates a file `update_default_resources.c` that is required during the build process.

### 5. Download the Yocto/OpenEmbedded "meta" layers
This data contains the rules for downloading and building code for Mbed Linux.  To download this data, use the following commands:

```
cd ~/mbl
mkdir mbl-master && cd mbl-master
repo init -u ssh://git@github.com/armmbed/mbl-manifest.git -b master -m default.xml
repo sync
```

### 6. Set up the build environment

You need to configure your build environment for your device, including setting your working directory to the build directory (in this case `~/mbl/mbl-master/build-mbl`). To set up your build environment, use the following command:

```
MACHINE=<machine> . setup-environment
```
Select the {MACHINE} value for your Mbed Linux device from the table below:

| Device         | MACHINE            |
| :---           | ---                |
| Warp7          | `imx7s-warp-mbl`   |
| Raspberry Pi 3 | `raspberrypi3-mbl` |

So, for example, to set up the build environment for a Warp7 board, your command would look like this:
```
MACHINE=imx7s-warp-mbl . setup-environment
```

<span class="notes">**Note:** During the build process, make sure you are using this shell instance when running `bitbake` commands, as the set-up script changes bitbake settings by setting environment variables of the shell process.</span>

**Warning:**: Do not source the setup-environment script more that once in a terminal session. Invoking the script a second time can corrupt environment variables and cause `bitbake` commands to fail in unexpected places.

Copy your Mbed Cloud dev credentials file and Update resources file to the build directory, as follows:
```
cp ~/mbl/cloud-credentials/mbed_cloud_dev_credentials.c ~/mbl/mbl-master/build-mbl
cp ~/mbl/manifests/update_default_resources.c ~/mbl/mbl-master/build-mbl
```

### 7. Build Mbed Linux

The build process will create the following files which you will need to use later:

* **A full disk image** This is a compressed image of the entire flash, created by the build process using the Wic tool from OpenEmbedded.  Once decompressed, this image can be directly written to storage media. See [this section](https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#creating-partitioned-images-using-wic) of the Yocto Mega Manual for more information about Wic. You will need this for [Step 8 Write the disk image to your device and boot Mbed Linux](#write-image-and-boot). You use the full disk image to initialize the device's storage with a full set of disk partitions and an initial version of firmware.
* **A block map of the full disk image** This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the IoT device. Some blocks of the image represent unused storage space that does not actually need to be written.
* **A root filesystem archive** This is a compressed tar archive, that you will need for a firmware update; see [Step 12 Performing a firmware update](#do-update). Once the device storage has been initialized, you can use the root file system archive to update the firmware, as this only requires a single root partition to be updated.

To generate these files, run the following `bitbake` command while still in the build directory (`~/mbl/mbl-master/build-mbl`):
```
bitbake mbl-console-image
```
You will see several "WARNING" messages in the `bitbake` output - these are safe to ignore.

**File locations**

The paths of these files are given in the table below, where `<MACHINE>` should be replaced with the MACHINE value for your device from the table in [Section 6](#set-up-build-env).

| Image type                | Path |
|---------------------------|------|
| Full disk image           | `~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.wic.gz`   |
| Full disk image block map | `~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.wic.bmap` |
| Root file system archive  | `~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz`   |

### 8. Write the disk image to your device and boot Mbed Linux

To enable firmware update, the full disk image contains a partition table and all the partitions required by Mbed Linux, including two root partitions. Only one of the root filesystem partitions is **active** at any one time, and the other is available to receive a new version of firmware during an update. At the end of the update process, the root partition with the new firmware becomes **active** and the other root partition becomes available to receive the next firmware update.  

This section contains instructions for writing the full disk image to a:

* Warp7 device
* Raspberry Pi 3 device

For both Warp7 and Raspberry Pi 3 devices add the current user to the dialout group to allow access to /dev/ttyUSBn devices without using sudo:
```
sudo usermod -a -G dialout $USER
```
Verify group memberships by typing "groups":
```
groups
<your_user> dialout
```
The users might need to log out and log in again before the group membership will take effect for their shell process.

#### 8.1. Write the full disk image to a Warp 7 device

To transfer your disk image to the Warp7's flash device, you must first access the Warp7's serial console. To do this:

1. Connect both the Warp7's I/O USB socket (on the I/O board) and the Warp7's mass storage USB socket (on the CPU board) to your PC. From your PC you should then be able to see a USB TTY device, such as, `/dev/ttyUSB0`.
1. Connect to the Warp7's console using a command such as:
    ```
    minicom -D /dev/ttyUSB0
    ```
    Use the following settings:
    * A baud rate of 115200.
    * [8N1](https://en.wikipedia.org/wiki/8-N-1) encoding.
    * No hardware flow control.
1. Depending on the previous contents of the device's storage, you may get a U-boot prompt or you may see an operating system (e.g. Android) boot. If an operating system boots, reboot the device and then press a key when U-Boot starts, to prevent it booting the operating system.
1. First take note of the current storage devices on your PC:
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

    We'll need to compare this output in the next step, so save it for reference.

1. To expose the Warp7's flash device to Linux as USB mass storage, when you get a U-Boot prompt, type:
    ```
    ums 0 mmc 0
    ```
    On the Warp7 you should now see an ASCII-art "spinner" and on your PC you should see some new storage devices appear:
    ```
    ls -l /dev/disk/by-id/
    ```

    In this case, the Warp7 appeared as "usb-Linux_UMS_disk_0" (the partitions on the device are also shown):

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

    `mbl-console-image-imx7s-warp-mbl.wic.gz` is a full disk image so should be written to the whole flash device, not a partition. The device file for the whole flash device is the one without `-part` in the name (`/dev/disk/by-id/usb-Linux_UMS_disk_0-0:0` in this example).
1. Ensure that none of the Warp7's flash partitions are mounted by running the following command (you may have to adjust the path depending on the name of the storage device):
    ```
    sudo umount /dev/disk/by-id/usb-Linux_UMS_disk_0-0:0-part*
    ```
1. From a Linux prompt, write the disk image to the Warp7's flash device using the following command:
    ```
    sudo bmaptool copy --bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-console-image-imx7s-warp-mbl.wic.bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/mbl-console-image-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>
    ```
    replacing `<device-file-name>` with the correct device file for the Warp7's flash device. This may take some time.
1. When `bmaptool` has finished eject the device:
    ```
    sudo eject /dev/disk/by-id/<device-file-name>
    ```
    replacing `<device-file-name>` with the correct device file for the Warp7's flash device.
1. On the Warp7's U-Boot prompt, press Ctrl-C to exit USB mass storage mode.
1. Reboot the Warp7 using the following command:
    ```
    reset
    ```
    The device should now boot into Mbed Linux.

#### 8.2. Write the full disk image to a Raspberry Pi 3 device

1. Connect a micro SD card to your PC. You should see the SD card device file in `/dev`, probably as `/dev/sdX` for some letter `X` (e.g. `/dev/sdd`) as well as device files for its partitions `/dev/sdXN` for the same letter `X` and some numbers `N` (e.g. `/dev/sdd1`, `/dev/sdd2`, etc.). In the commands below, `/dev/sdX` should be replaced with the device file name for the SD card _without_ a number at the end. The output of `lsblk` can be useful to identify the name of the SD card device.
1. Ensure that none of the micro SD card's partitions are mounted by running:
    ```
    sudo umount /dev/sdX*
    ```
    replacing `/dev/sdX` as mentioned above.
1. Write the disk image to the SD card device (not a partition on it) using the following command:
    ```
    bmaptool copy --bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/raspberrypi3-mbl/mbl-console-image-raspberrypi3-mbl.wic.bmap ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/raspberrypi3-mbl/mbl-console-image-raspberrypi3-mbl.wic.gz /dev/sdX
    ```
    replacing `/dev/sdX` as mentioned above. This may take some time.
1. When `bmaptool` has finished, eject the device:
    ```
    sudo eject /dev/sdX
    ```
1. Detach the micro SD card from your PC and plug it into the Raspberry Pi 3.

1. Before powering on the Raspberry Pi 3, you'll need to either connect it to a monitor and keyboard (using its HDMI and USB sockets) or connect it to your PC so that you can access its console. To access the console from your PC you can, for example, use a [C232HD-DDHSP-0](http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_C232HD_UART_CABLE.pdf) cable.
Use the following instructions to connect the C232HD-DDHSP-0 cable to your Raspberry Pi 3:

    * Using [this](https://www.element14.com/community/servlet/JiveServlet/previewBody/73950-102-10-339300/pi3_gpio.png) pin numbering as a reference connect USB-UART colored wires as follows:
        * **Black** wire to pin **06**
        * **Yellow** wire to pin **08**
        * **Orange** wire to pin **10**

        ![](./pics/rpi3_uart-connection.JPG)

    * The cable's TX and RX are used to communicate with the board.
    * Connect the other end of the C232HD-DDHSP-0 cable to the USB port on your PC.

    After connecting the Raspberry Pi 3, from your PC run a command like:
    ```
    minicom -D /dev/ttyUSB0
    ```
    Use the following settings:
    * A baud rate of 115200.
    * [8N1](https://en.wikipedia.org/wiki/8-N-1) encoding.
    * No hardware flow control.
1. Connect the Raspberry Pi 3's micro USB socket to a USB power supply. It should now boot into Mbed Linux.

### 9. Log in to Mbed Linux

To log in to Mbed Linux, enter the username `root` with no password.

### 10. Set up a network connection

If your device is connected to a network with a DHCP server using Ethernet, then it automatically connects to that network. Otherwise, follow [the instructions](https://github.com/ARMmbed/meta-mbl/blob/master/docs/wifi.md) for setting up wifi in Mbed Linux.

### 11. Check if the device has connected to Mbed Cloud

While the device boots into Mbed Linux, `mbl-cloud-client` should automatically start and connect to Mbed Cloud. You can check whether it has connected by:

* Checking the device status on the [Mbed Cloud Portal](https://portal.mbedcloud.com/) ([Checking device status](#fig8)).
* Reviewing the log file for `mbl-cloud-client` at `/var/log/mbl-cloud-client.log`.

If your device hasn't automatically connected to Mbed Cloud, this may occur if networking wasn't configured before the device was rebooted or if there are issues with the network.  The device retries periodically, but you may need to restart `mbl-cloud-client`, as follows:
```
/etc/init.d/mbl-cloud-client restart
```

## Tutorial: Firmware update for an MBL device
<!--I would rather have these as two, because they give the user quicker wins. I will not fight too hard for this if you feel it's wrong.-->
### 12. Perform a firmware update using the Mbed Cloud Portal

To perform a firmware update using the Mbed Cloud Portal, follow these steps:

- 12.1. [Update Step 1](#update2-1): Prerequisites.
- 12.2. [Update Step 2](#update2-2): Upload a firmware image to the cloud.
- 12.3. [Update Step 3](#update2-3): Create a manifest.
- 12.4. [Update Step 4](#update2-4): Upload the manifest to the cloud.
- 12.5. [Update Step 5](#update2-5): Create a filter for your device.
- 12.6. [Update Step 6](#update2-6): Run a campaign to update the device firmware.

**How firmware gets updated**

The device has two banks of software:

- The running bank. A device partition storing the rootfs for the running system.
- The non-running bank. A device partition that will receive the firmware update.

You can run `lsblk` on the device to check which partition is mounted at `/`.

During a firmware update, the update software:

- Writes the new software rootfs to the non-running bank.
- Sets the non-running bank to be the running bank next time the device boots.
- Reboots the device.

#### 12.1. Update Step 1: Prerequisites

Before you upgrade the firmware on the device, make sure you:

* Have a root file system archive that contains the firmware upgrade.
* Have an Mbed Cloud account.
* Know which device partition is active (the running bank),so that you can check the upgrade has been successful.
* Have installed and initialized the `manifest-tool`.

#### 12.2. Update Step 2: Upload a firmware image to Mbed Cloud

To upload your firmware update image to the Cloud:

- Log into the [Mbed Cloud Portal](https://portal.mbedcloud.com/login).
- On the **Firmware Update** tab, select **Images>Upload new images**.
- Select the update image on your local hard disk that contains the root file system image for upgrading. For a Warp7 device, for example, the update image will have a filename like this: `mbl-console-image-imx7s-warp-mbl.tar.xz`.
- Provide a name for the firmware image, such as, "test\_image\_20180125\_1", and if required, a description.
- Press the **Upload firmware image** button.
- Copy the firmware image URL. You will need this URL to create the manifest (described in the next section).

#### 12.3. Update Step 3: Create a manifest

To use the manifest-tool to create a manifest for the firmware image:

- Make sure the current working directory is where `manifest-tool init` was performed (`~/mbl/manifests`).
- Create a symbolic link to the firmware image that was uploaded in [Update Step 2](#update2-2):
  ```
  ln -s ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz test-image
  ```
  where `<MACHINE>` should be replaced with the MACHINE value for your device from the table in [Section 6](#set-up-build-env). This step is required because sometimes `manifest-tool` doesn't cope well with long file names.
- Create a manifest called "test-manifest" by using the following command:
    ```
    manifest-tool create -p test-image -u URL -o test-manifest
    ```
    Where:
    - The **test-image** is the symlink to the firmware image uploaded in [Update Step 2](#update2-2).
    - The **URL** is the firmware image URL copied to the clipboard in the previous section.
    - The **test-manifest** is the name of the output manifest file.

#### 12.4. Update Step 4: Upload the manifest to the Mbed Cloud

To upload the test-manifest to the Mbed Cloud:

- On the Mbed Cloud Portal, from the **Firmware Update** tab, select **Manifests>Upload new manifest**.
- Give the manifest a name, a description (if required) and select the manifest to use.

#### 12.5. Update Step 5: Create a filter for your device

<span class= "notes"> **Note:** the device ID changes each time you flash the device with a new full disk image. The normal firmware update mechanism does not change the device ID.</span>

Before you can configure an update campaign, you need to create a device filter, as follows:

- To get the device ID, you can either:
    - Copy the device ID to the clipboard from the **Device Directory** tab, on Mbed Cloud Portal.
	  - Search for the device ID in the `mbl-cloud-client` log file `/var/log/mbl-cloud-client.log`, using the following command:
    ```
    grep -i 'device id' /var/log/mbl-cloud-client.log
    ```
- On Mbed Cloud Portal, from the **Device Directory** tab, select **Saved filters** and click **Create New Filter**.
- Click **Add attribute** and select **Device ID**.
- Paste the Device ID into the Device ID edit box and click **Save Filter**.

#### 12.6. Update Step 6: Run an update campaign

To create a campaign to update the firmware on a device:

- On the Mbed Cloud Portal, from the **Firmware Update** tab, select **Update campaigns>Create campaign**.
- Provide:
    * A name for the campaign.
    * A description (optional).
- Select the:
    * Manifest file (in this example, `test-manifest`).
    * Device filter created in update step 5.
- Click **Save** to create the new update campaign.

To run your update campaign:

- On the **Firmware update** tab, select **Update campaigns** and find your update campaign from the list.
- Press **Start** to run your test campaign.
    - The firmware update process may take a while to complete.
    - On the device, you can monitor the device console to see the update occurring using `tail -f /var/log/mbl-cloud-client.log` .
    - On the Mbed Cloud Portal, as the campaign progresses, it reports the **Publishing** state.
    - When the firmware update has completed, the Mbed Cloud Portal will report that the campaign is in the **Deployed** state.
- Once the firmware update is complete, the device reboots.
- When the device comes up, login and verify that the running bank (partition) has changed from that noted in update step 1.


#### 12.2.7. Figures

<a name="fig1"></a>
![fig1](pics/01.png "Figure 1")
Figure 1: The mbed cloud portal login.

<a name="fig2"></a>
![fig2](pics/02.png "Figure 2")
Figure 2: Select the team you want to use e.g. arm-mbed-linux.

<a name="fig3"></a>
![fig3](pics/03.png "Figure 3")
Figure 3: Dashboard after you login.

<a name="fig4"></a>
![fig4](pics/04.png "Figure 4")
Figure 4: Firmware update/Images screen.

<a name="fig5"></a>
![fig5](pics/05.png "Figure 5")
Figure 5: Firmware update/Images screen showing how to copy URL.

<a name="fig6"></a>
![fig6](pics/06.png "Figure 6")
Figure 6: Upload firmware image screen.

<a name="fig7"></a>
![fig7](pics/07.png "Figure 7")
Figure 7: Device details screen.

<a name="fig8"></a>
![fig8](pics/08.png "Figure 8")
Figure 8: Device directory screen for viewing device registration status and adding device filters.

<a name="fig9"></a>
![fig9](pics/09.png "Figure 9")
Figure 9: Device directory for adding a filter device id attribute.

<a name="fig10"></a>
![fig10](pics/10.png "Figure 10")
Figure 10: Device directory for adding a filter.

<a name="fig11"></a>
![fig11](pics/11.png "Figure 11")
Figure 11: Saved filters screen.

<a name="fig12"></a>
![fig12](pics/12.png "Figure 12")
Figure 12: Update campaigns screen.

<a name="fig13"></a>
![fig13](pics/13.png "Figure 13")
Figure 13: New update campaign screen.

<a name="fig14"></a>
![fig14](pics/15.png "Figure 14")
Figure 14: Firmware update/Campaign status screen.

<a name="fig15"></a>
![fig15](pics/16.png "Figure 15")
Figure 15: Test campaign details screen.

[mbl-alpha-walkthrough]: https://github.com/ARMmbed/meta-mbl/blob/alpha/docs/walkthrough.md
