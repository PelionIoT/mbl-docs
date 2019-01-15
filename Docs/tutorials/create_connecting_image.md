## Preparing Device Management sources

### Downloading Device Management developer credentials

MBL handles Device Management connectivity on behalf of the device, rather than relying on the user application to do it. This means you need to add your Device Management credentials to the working directory MBL builds from. For development environments, Pelion offers a *developer certificate* for quick connection:

1. Create cloud credentials directory, for example `./cloud-credentials`.
2. To create a Pelion developer certificate (`mbed_cloud_dev_credentials.c`), follow the instructions for [creating and downloading a developer certificate](../getting-started/provisioning-development.html).
3. Add the developer certificate to the credentials directory you've created.

<span class="notes">**Note:** These instructions use the developer workflow and certificates as connectivity credentials. Do not use these credentials for production devices.</span>

### Creating an update resources file

MBL and Device Management support over-the-air updates for devices, which you can do [in a later tutorial in this sequence](../getting-started/tutorial-updating-mbl-devices-and-applications.html). For a device to be updatable, it needs to include some update resources (such as credentials) in the original image.

1. Create an update resources directory, such as `./update-resources`:

    ```
    mkdir ./update-resources
    ```

1. Move into the new directory:

    ```
    cd ./update-resources
    ```

1. Initialize the manifest tool, and generate Update resources:

    ```
    manifest-tool init -q -d arm.com -m dev-device
    ```

    This generates the `update_default_resources.c` file. 

    You will use this file in the build process later.

## Building an MBL image

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

Please note that each release has its own branch. Throughout this guide, the release branch is assumed to be `mbl-os-0.5`.

### Building scripts

<span class="notes">**Note**: You need to use an SSH agent to build. For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).</span>

The `run-me.sh` script:

1. Creates and launches a Docker container that encapsulates the MBL build environment.
1. Launches a build script, `build.sh`, inside the container.

The `run-me.sh` and `build.sh` scripts are called in a single command, with options that control what to build and how. The general form of a `run-me.sh` invocation is:

```
./mbl-tools/build-mbl/run-me.sh [RUN-ME.SH OPTIONS]... -- [BUILD.SH OPTIONS]...
```

<span class="tips">Note the use of `--`. It separates options for `run-me.sh` from options for `build.sh`.</span>

To invoke the `run-me.sh` help menu, use:

```
./mbl-tools/build-mbl/run-me.sh -h
```

To invoke the `build.sh` help menu, use:

```
./mbl-tools/build-mbl/run-me.sh -- -h
```

The following build options are mandatory:

| Name | Information |
| --- | --- |
| `--branch` | Select the MBL branch to build. For example, to build the branch `mbl-os-0.5`: `./mbl-tools/build-mbl/run-me.sh -- --branch mbl-os-0.5 --machine raspberrypi3-mbl` |
| `--machine` | Select the target device. The options are [ Warp7, `imx7s-warp-mbl`] and [ Raspberry Pi 3, `raspberrypi3-mbl`]. Example: `./mbl-tools/build-mbl/run-me.sh -- --machine <MACHINE>` |
| `--builddir` | Create a build directory. This option is for `run-me.sh`. You must use a different build directory for every device (machine), and we recommend including the device's name in the directory's name. Note that this directory includes all other artifacts, such as build and error logs. For example, if you've created `mkdir /path/to/my-build-dir`, the builddir will be `./mbl-tools/build-mbl/run-me.sh --builddir /path/to/my-build-dir` |
| `--outputdir` | Specify the output directory for all build artifacts (pinned manifest, target specific images etc). For example, if you're created `mkdir /path/to/artifacts`, the outpudir will be `./mbl-tools/build-mbl/run-me.sh --outputdir /path/to/artifacts` |
| `--inject-mcc` | At the moment, you need to build your Device Management resources (that you obtained above) into the image. `./mbl-tools/build-mbl/run-me.sh --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c` |

An example using all mandatory options:

```
./mbl-tools/build-mbl/run-me.sh --builddir /path/to/builddir --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c --outputdir /path/to/artifacts -- --branch mbl-os-0.5 --machine <MACHINE>
```

The following build options are not mandatory, but you may find that they improve the development process:

| Name | Information |
| --- | --- |
| `--downloaddir` | Cache downloaded artifacts between successive builds (do not use cacheing for parallel builds). For example, if you've created `mkdir /path/to/downloads`, the downloaddir will be `./mbl-tools/build-mbl/run-me.sh --downloaddir /path/to/downloads` |
| `--external-manifest` | You can build using a pinned manifest, which is an encapsulation created by a build and containing enough information to allow an exact rebuild. The manifest is created in your output directory (`outputdir`). To use it to rebuild, run `./mbl-tools/build-mbl/run-me.sh --external-manifest /path/to/pinned-manifest.xml` |

### Build examples

The following examples assume:

* You have your [Device Management developer credentials](../getting-started/preparing-device-management-sources.html#downloading-device-management-developer-credentials).
* You have your [update resources file](../getting-started/preparing-device-management-sources.html#creating-an-update-resources-file).
* You have an output directory for your machine, for example `./artifacts-warp7` or `./artifacts-rpi3`.
* You have an [SSH agent](../getting-started/software-environment.html). For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

#### Warp7 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --builddir ./build-warp7 --outputdir ./artifacts-warp7 -- --machine imx7s-warp-mbl --branch mbl-os-0.5
```

#### Raspberry Pi 3 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --builddir ./build-rpi3 --outputdir ./artifacts-rpi3 -- --machine raspberrypi3-mbl --branch mbl-os-0.5
```

### Building outputs

The build process creates the following files:

| File | Path | Information |
| --- | --- | --- |
| Full disk image  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.wic.gz` | This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware. |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.wic.bmap` | This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the device. Some blocks of the image represent unused storage space, which does not actually need to be written. |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image-test/images/mbl-console-image-test-<MACHINE>.tar.xz` | This is a compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the Updating MBL tutorial](../getting-started/tutorial-updating-mbl-devices-and-applications.html)). |

The process also creates release images, which don't contain packages for testing and debugging (it removes packages such as OP-TEE test suite, Dropbear to support SSH during development and test, and the strace utility).

| File | Path |
| --- | --- |
| Full test disk image | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.wic.gz` |
| Full test disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.wic.bmap` |
| Root file system archive | `/path/to/artifacts/machine/<MACHINE>/images/mbl-console-image/images/mbl-console-image-<MACHINE>.tar.xz` |


## Writing and booting the disk image

This section contains instructions for writing the full disk image to a:

* Warp7 device.
* Raspberry Pi 3 device.

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
    * No hardware flow control (enabled by default).

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

    You'll need to refer to this output in the following steps, so save it for reference.

1. If you got a U-boot prompt on the device, continue to the next step.

   If you got an operating system boot (for example, Android), reboot the device until you get a U-boot prompt, and then press any key to prevent the operating system from booting again. Continue to the next step.

1. To expose the Warp7's flash device to Linux as USB mass storage, in the U-boot prompt, enter:

    ```
    ums 0 mmc 0
    ```

    On the Warp7, you now see an ASCII-art "spinner". On your PC, you now see new storage devices:

    ```
    ls -l /dev/disk/by-id/
    ```

    In this example, the Warp7 is listed as `usb-Linux_UMS_disk_0` (the partitions on the device are also shown):

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

    `mbl-console-image-test-imx7s-warp-mbl.wic.gz` is a full disk image, so it should be written to the whole flash device, not a partition.

    The device file for the whole flash device is the one without `-part` in the name (`/dev/disk/by-id/usb-Linux_UMS_disk_0-0:0` in this example).

1. Ensure that none of the Warp7's flash partitions are mounted (you may need to adjust the path depending on the name of the storage device):

    ```
    sudo umount /dev/disk/by-id/usb-Linux_UMS_disk_0-0:0-part*
    ```

1. From a Linux prompt, write the disk image to the Warp7's flash device (replace `<device-file-name>` with the correct device file for the Warp7's flash device; in this example, it would be `usb-Linux_UMS_disk_0-0:0`):

    ```
    sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image-test/images/mbl-console-image-test-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-console-image-test/images/mbl-console-image-test-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>
    ```

    This action may take some time.

1. When `bmaptool` has finished, eject the device (replace `<device-file-name>` with the correct device file for the Warp7's flash device; in this example, it would be `usb-Linux_UMS_disk_0-0:0`):

    ```
    sudo eject /dev/disk/by-id/<device-file-name>
    ```

1. On the Warp7's U-boot prompt, press <kbd>Ctrl</kbd>-<kbd>C</kbd> to exit USB mass storage mode.
1. Reboot the Warp7:

    ```
    reset
    ```

    The device now boots into MBL.

### Raspberry Pi 3 devices

1. Connect a micro-SD card to your PC. You now see:

    * The SD card device file in `/dev`, probably as `/dev/sdX` for some letter `X` (for example, `/dev/sdd`).
    * Device files for its partitions. `/dev/sdXN` for the same letter `X` and some numbers `N` (for example, `/dev/sdd1` and `/dev/sdd2`).

    <span class="notes">In the commands below, replace `/dev/sdX` with the device file name for the SD card _without_ a number at the end. You can use `lsblk` to identify the name of the SD card device.</span>

1. Ensure none of the micro-SD card's partitions are mounted (replace `/dev/sdX` as explained above):

    ```
    sudo umount /dev/sdX*
    ```

1. Write the disk image to the SD card device - not a partition on it (replace `/dev/sdX` as explained above):

    ```
    sudo bmaptool copy --bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-console-image-test/images/mbl-console-image-test-raspberrypi3-mbl.wic.bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-console-image-test/images/mbl-console-image-test-raspberrypi3-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. Detach the micro-SD card from your PC, and plug it into the Raspberry Pi 3.
1. You need access to the device's console, so before powering it on, either:

    * Connect it to a monitor and keyboard (using its HDMI and USB sockets).
    * Connect it to your PC. For example, if you're using a [C232HD-DDHSP-0](http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_C232HD_UART_CABLE.pdf) cable, use [this pin numbering reference](https://www.element14.com/community/servlet/JiveServlet/previewBody/73950-102-10-339300/pi3_gpio.png) and connect USB-UART colored wires:
       * **Grounds** wire (usually black) to pin **06**.
       * **Tx** wire (usually yellow) to pin **08**.
       * **Rx** wire (usually orange) to pin **10**.

    Connect the other end of the C232HD-DDHSP-0 cable to the USB port on your PC.

    The cable's TX and RX are used to communicate with the board.

1. After connecting the Raspberry Pi 3, from your PC, run the following command to connect to the device's serial console:

    ```
    minicom -D /dev/ttyUSB0
    ```

    Use the following settings:

    * Baud rate: 115200.
    * Encoding: [8N1](https://en.wikipedia.org/wiki/8-N-1).
    * No hardware flow control (enabled by default).

1. Connect the Raspberry Pi 3's micro-USB socket to a USB power supply.

    The device now boots into MBL.

### Logging in to MBL

To log in to MBL, wait for a login prompt, and then enter the username `root`. You will not be prompted for a password.
