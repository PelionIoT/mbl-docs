<h2 id="mbl-pelion-connect">Building an Mbed Linux OS image and connecting to Pelion Device Management</h2>

This tutorial builds and installs an Mbed Linux OS (MBL) image on a device. This image includes credentials to connect to your Pelion Device Management account, but does not include a user application (that is covered [in the Hello World tutorial](../getting-started/tutorial-user-application.html)).

<span class="notes">Each release has its own branch, such as `mbl-os-0.5`. Throughout this guide, the release branch is referred to as `mbl-XXX`. Replace it with the name of the branch you're working with.</span>

<!--I need to review the steps, but I'll do that after I understand them-->

<span class="notes">**Note:** These instructions use the developer workflow and certificates as connectivity credentials. Do not use these credentials for production devices.</span>

## Prerequisites

Please [make sure you have suitable hardware and all the required software](../reqs-setup/index.html).

### Downloading Device Management developer credentials

MBL handles Device Management connectivity on behalf of the device, rather than relying on the user application to do it. This means you need to add your Device Management credentials to the working directory MBL builds from. For development environments, Pelion offers a *developer certificate* for quick connection:

1. Create cloud credentials directory, e.g. `./cloud-credentials`
2. To create a Pelion developer certificate (`mbed_cloud_dev_credentials.c`), follow the instructions for [creating and downloading a developer certificate](../reqs-setup/provisioning-development.html).
3. Add the developer certificate to the credentials directory you've created.

### Creating an update resources file

MBL and Device Management support over-the-air updates for devices, which you can do [in a later tutorial in this sequence](../getting-started/tutorial-updating-mbl-devices-and-applications.html). For a device to be updatable, it needs to include some update resources (such as credentials) in the original image.

1. Create an update resources directory, such as `./update-resources`:

    ```
    mkdir ./update-resources && cd ./update-resources
    ```

2. Initialize the manifest tool and generate Update resources:

    ```
    manifest-tool init -q -d arm.com -m dev-device
    ```

    This generates the `update_default_resources.c` file. Place it in the directory you created in the previous step.

    You will use this file in the build process later.

<!--this was covered in the software prereqs section
### mbl-tools

mbl-tools repository provides a collection of tools and recipes related to the build and test of Mbed Linux OS.
Checkout the `mbl-tools-XXX` branch from [mbl-tools git repository](https://github.com/ARMmbed/mbl-tools) using the following command
```
mkdir /path/to/working-dir/; cd /path/to/working-dir/
$ git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-tools-XXX

```
-->

## Building an MBL image

### Building scripts

The `run-me.sh` script:

1. Creates and launches a Docker container that encapsulates the MBL build environment.
1. Launches a build script, `build.sh`, inside the container.

The `run-me.sh` and `build.sh` scripts are called in a single command, with options that control what to build and how. The general form of a `run-me.sh` invocation is:

```
./mbl-tools/build-mbl/run-me.sh [RUN-ME.SH OPTIONS]... -- [BUILD.SH OPTIONS]...
```

<span class="tips">Note the use of `--`. It separates options for `run-me.sh` from options for `build.sh`.</span>

To invoke the `run-me.sh` help menu use:

```
./mbl-tools/build-mbl/run-me.sh -h
```

To invoke the `build.sh` help menu use:

```
./mbl-tools/build-mbl/run-me.sh -- -h
```

The following build options are mandatory:

| Name | Information |
| --- | --- |
| `--branch` | Select the MBL branch to build. For example, to build the branch `mbl-XXX`: `./mbl-tools/build-mbl/run-me.sh -- --branch mbl-XXX --machine raspberrypi3-mbl` |
| `--machine` | Select the target device. The options are [ Warp7, `imx7s-warp-mbl`] and [ Raspberry Pi 3, `raspberrypi3-mbl`]. Example: `./mbl-tools/build-mbl/run-me.sh -- --machine <MACHINE>` |
| `--builddir` | Create a build directory. This option is for `run-me.sh`. You must use a different build directory for every device (machine), and we recommend including the device's name in the directory's name. Note that this directory includes all other artefacts, such as build and error logs. For example, if you've created `mkdir /path/to/my-build-dir`, the builddir will be `./mbl-tools/build-mbl/run-me.sh --builddir /path/to/my-build-dir` |
| `--outputdir` | Specify the output directory for all build artefacts (pinned manifest, target specific images etc). For example, if you're created `mkdir /path/to/artifacts`, the outpudir will be `./mbl-tools/build-mbl/run-me.sh --outputdir /path/to/artifacts` |
| `--inject-mcc` | At the moment, you need to build your Device Management resources (that you obtained above) into the image. `./mbl-tools/build-mbl/run-me.sh --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c` |


An example using all mandatory options:
```
./mbl-tools/build-mbl/run-me.sh --builddir /path/to/builddir --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c --outputdir /path/to/artifacts -- --branch mbl-XXX --machine <MACHINE>
```

The following build options are not mandatory, but you may find that they improve the development process:

| Name | Information |
| --- | --- |
| `--downloaddir` | Cache downloaded artefacts between successive builds (do not use cacheing for parallel builds). For example, if you've created `mkdir /path/to/downloads`, the downloaddir will be `./mbl-tools/build-mbl/run-me.sh --downloaddir /path/to/downloads` |
| `--external-manifest` | You can build using a pinned manifest, which is an encapsulation created by a build and containing enough information to allow an exact rebuild. The manifest is created in your output directory (`outputdir`). To use it to rebuild, run `./mbl-tools/build-mbl/run-me.sh --external-manifest /path/to/pinned-manifest.xml` |


### Build examples

The following examples assume:

* You have your [Device Management developer credentials, as explained above](#downloading-device-management-developer-credentials).
* You have your [update resources file, as explained above](#creating-an-update-resources-file).
* You have an output directory for your machine, for example `./artifacts-warp7` or `./artifacts-rpi3`.


#### Warp7 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --outputdir ./artifacts-warp7 -- --machine imx7s-warp-mbl --branch mbl-XXX
```

#### Raspberry Pi 3 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --outputdir ./artifacts-rpi3 -- --machine raspberrypi3-mbl --branch mbl-XXX
```


### Building outputs

The build process creates the following test image files (which you need later):

| File | Path | Information |
| --- | --- | --- |
| Full disk image  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.wic.gz` | This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware. |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.wic.bmap` | This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the device. Some blocks of the image represent unused storage space, which does not actually need to be written. |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.tar.xz` | This is a compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the Updating MBL tutorial](../getting-started/tutorial-updating-mbl-devices-and-applications.html)). |

The process also creates a release images, which don't contain packages for testing and debugging (it removes packages such as OP-TEE test suite, Dropbear to support SSH during development and test, and the strace utility).

| File | Path |
| --- | --- |
| Full test disk image | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.wic.gz` |
| Full test disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.wic.bmap` |
| Root file system archive | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.tar.xz` |


## Writing and booting the disk image

This section contains instructions for writing the full disk image to a:

* Warp7 device
* Raspberry Pi 3 device

### Adding your user to the dialout group
<!--should we just put that in the prereqs bit?-->

To access ``/dev/ttyUSBn`` devices without using sudo, add the current user to the dialout group:

<!--Is this mandatory, or just nicer?-->
```
sudo usermod -a -G dialout $USER
```

Verify group memberships by entering `groups`:

```
groups
<your_user> dialout
```

You may need to reboot your computer before the group membership takes effect.

<!--And this is okay because we've done the build, so we're not limited to the original shell instance anymore, right?-->

### Warp7 devices

To write your disk image to the Warp7's flash device, you must first access the Warp7's serial console. To do this:

1. Connect both the Warp7's USB socket (on the I/O board) and the Warp7's mass storage USB socket (on the CPU board) to your PC.

    You should now be able to see a USB TTY device, such as, `/dev/ttyUSB0`, on your PC.

1. Connect to the Warp7's console using a command such as:

    ```
    minicom -D /dev/ttyUSB0
    ```
    Use the following settings:

    * Baud rate: 115200.
    * Encdoing: [8N1](https://en.wikipedia.org/wiki/8-N-1).
    * No hardware flow control.
1. Check the current storage devices on your PC:

    ```
    ls -l /dev/disk/by-id/
    ```

    A list of devices displays. For example:

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
    `mbl-console-image-test-imx7s-warp-mbl.wic.gz` is a full disk image so should be written to the whole flash device, not a partition.
    The device file for the whole flash device is the one without `-part` in the name (`/dev/disk/by-id/usb-Linux_UMS_disk_0-0:0` in this example).
1. Ensure that none of the Warp7's flash partitions are mounted by running the following command (you may need to adjust the path depending on the name of the storage device):
    ```
    sudo umount /dev/disk/by-id/usb-Linux_UMS_disk_0-0:0-part*
    ```
1. From a Linux prompt, write the disk image to the Warp7's flash device using the following command:
    ```
    sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image-test/images/mbl-console-image-test-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image-test/images/mbl-console-image-test-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>
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
    sudo bmaptool copy --bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-console-image-test/images/mbl-console-image-test-raspberrypi3-mbl.wic.bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-console-image-test/images/mbl-console-image-test-raspberrypi3-mbl.wic.gz /dev/sdX
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

## Setting up a network connection

If your device is connected to a network with a DHCP server using Ethernet, then it automatically connects to that network.

If your device uses Wi-Fi:

1. Install [ConnMan](https://01.org/connman/documentation).
1. Create a configuration provisioning file at `/config/user/connman/<name>.config`.

    <span class="tips">**Tip:** You can add the `-service` suffix to file names as a convention. For example, `name-service`.</span>

1. Add service information to the configuration file:

    ```
    [global]
    Name = my-ssid
    Description = Provide a short description

    [service_wifi_local_network]
    Type = wifi
    Name = <my-ssid>
    EAP = peap
    Phase2 = MSCHAPV2
    Identity = <my-username>
    Passphrase = <my-password>
    ```

     Replace `<my-ssid>`, `<my-username>`, and `<my-password>` with appropriate information. Amend the description.

1. Enable Wi-Fi:

  ```
  # connmanctl enable wifi
  ```

ConnMan is now connected to the specified network.

If you experience any issues, restart both ConnMan and `wpa_supplicant` daemons.

<span class="tips">For more information about ConnMan, please [see the Wi-Fi on MBL reference](../references/using-connman-for-mbl-wi-fi.html).</span>

## Verifying that the device is connected to Device Management

While the device boots into MBL, `mbl-cloud-client` should automatically start and connect to Pelion. You can check whether it has connected by:
* Checking the device status on the [Device Managmenent Portal](https://portal.mbedcloud.com/).<!--I will need to add the images in our publishing format-->
* Reviewing the log file for `mbl-cloud-client` at `/var/log/mbl-cloud-client.log`.

If your device hasn't automatically connected to Pelion, it could be that:
* Networking wasn't configured before the device was rebooted. Check your configurations and reboot the device.<!--Easy enough to direct them to the WiFi bit, but is there anything that may have gone wrong with DHCP?-->
* There are issues with the network. The device retries periodically, but you may need to restart `mbl-cloud-client`:
    ```
    /etc/init.d/mbl-cloud-client restart
    ```
