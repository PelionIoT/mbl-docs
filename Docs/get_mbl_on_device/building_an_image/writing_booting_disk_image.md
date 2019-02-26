# Writing and booting the disk image

This section contains instructions for writing the full disk image to a:

* PICO-PI with an IMX7D SoM.
* NXP 8M Mini EVK.
* Warp7 device.
* Raspberry Pi 3 device.

<!-- JIJ: I think we could refactor this (possibly) as the instructions to find out which device to flash to, could be the same as the new instructions I have written - they would work for SDcard, Warp7 or Pico or IMX8 - i.e. using lsblk, and the following instructions on unmount, bmap-tool and eject would then be all identical. Its just the plugging in, and the rebooting that is different -->

## PICO-PI baseboard with the PICO-IMX7D SoM

### First-time

<span class="notes">This assumes you have already assembled your baseboard and SoM. These first-time instructions are distilled from [information from TechNexion](https://www.technexion.com/support/knowledgebase/loading-bootable-software-images-onto-the-emmc-of-picosom-on-pico-pi/). These instructions can also be used to recover the board if it cannot boot into u-boot.</span>

To write your first disk image, you need to:

1. Download and unzip the TechNexion image loader software: [ftp://download.technexion.net/development_resources/development_tools/installer/pico-imx6-imx6ul-imx7_otg-installer_20171101.zip](ftp://download.technexion.net/development_resources/development_tools/installer/pico-imx6-imx6ul-imx7_otg-installer_20171101.zip)

1. Set the PICO-PI into serial download mode by chainging the boot configuration jumper settings.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/pico7-serial-download.jpg)<span>

1. Connect both the PICO-PI USB-C socket and the micro-USB socket to your PC.

    You should now be able to see a USB TTY device, such as, `/dev/ttyUSB0`, on your PC.

1. Connect to the PICO-PI's console using a command such as:

    ```
    minicom -D /dev/ttyUSB0
    ```

    Use the following settings:

    * Baud rate: 115200.
    * Encdoing: [8N1](https://en.wikipedia.org/wiki/8-N-1).
    * No hardware flow control (enabled by default).

1. Check the current block storage devices on your PC:

    ```
    lsblk
    ```
    
    A list of devices displays. For example:
    
    ```
    NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb             8:16   0   1.8T  0 disk 
    └─sdb1          8:17   0   1.8T  0 part /mnt/2tb-disk
    sr0            11:0    1  1024M  0 rom  
    sda             8:0    0 238.5G  0 disk 
    ├─sda2          8:2    0   238G  0 part 
    │ ├─vg00-swap 253:1    0   7.5G  0 lvm  [SWAP]
    │ └─vg00-root 253:0    0 230.6G  0 lvm  /
    └─sda1          8:1    0   476M  0 part /boot
    ```

   You'll need to refer to this output in the following steps, so save it for reference.


1. Run the USB OTG (on the go) Loader, using the following commands on Linux:

    ```
    cd pico-imx6-imx6ul-imx7_otg-installer_20171101
    sudo linux/imx_usb pico-imx7d_bootbomb_20170112.imx 
    ```
    
    <span class="notes">Make sure you run the command as root using sudo otherwise the `imx_usb` loader will report `main:Could not open device vid=0x... pid=0x... err=-3`</span>
    
    You should see output on the PC that ends with:
    
    ```
    loading binary file(pico-imx7d_bootbomb_20170112.imx) to 877ff400, skip=0, fsize=259c00 type=aa

    <<<2464768, 2464768 bytes>>>
    succeeded (status 0x88888888)
    jumping to 0x877ff400
    ```
    
    On the console output from the device, it should end with:

    ```
    [    1.107086] Mass Storage Function, version: 2009/09/11                   
    [    1.112243] LUN: removable file: (no medium)                                 
    [    1.124192] Number of LUNs=1                                                 
    [    1.631131] configfs-gadget gadget: high-speed config #1: c                  
    ```

1. On your PC, you can now see new storage devices:

    ```
    lsblk
    ```
    
    In this example, the PICO-PI is listed as `sdc` (the partitions on the device are also shown):
    
    ```
    NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb             8:16   0   1.8T  0 disk 
    └─sdb1          8:17   0   1.8T  0 part /mnt/2tb-disk
    sr0            11:0    1  1024M  0 rom  
    sdc             8:32   1   7.3G  0 disk 
    └─sdc1          8:33   1    39M  0 part /media/user01/61DB-E6FE
    sda             8:0    0 238.5G  0 disk 
    ├─sda2          8:2    0   238G  0 part 
    │ ├─vg00-swap 253:1    0   7.5G  0 lvm  [SWAP]
    │ └─vg00-root 253:0    0 230.6G  0 lvm  /
    └─sda1          8:1    0   476M  0 part /boot
    ```
    <span class="notes">In the commands below, replace `/dev/sdX` with the device file name for the SD card _without_ a number at the end.</span>

1. Ensure that none of the PICO-PI's flash partitions are mounted (replace `/dev/sdX` as explained above):

    ```
    sudo umount /dev/sdX*
    ```

1. From a Linux prompt, write the disk image to the PICO-PI's flash device - not a partition on it (replace `/dev/sdX` as explained above)::

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/imx7d-pico-mbl/images/mbl-image-development/images/mbl-image-development-imx7d-pico-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. Unplug both USB cables from the PICO-PI, and set the boot configuration jumper settings to boot from the on board EMMC flash.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/pico7-flash-boot.jpg)<span>

1. Connect both the PICO-PI USB-C socket and the micro-USB socket back to your PC.

    The device now boots into MBL.

### Subsequent flashing

<!-- JIJ: This is the same as the Warp7 instructions, apart from the cabling set up, and I would suggest using the same lsblk method -->

## NXP 8M Mini EVK

<span class="notes">This process currently uses a micro-SD card, but will be updated in future to explain usage of the on board EMMC</span>

1. Check the current block storage devices on your PC:

<!-- JIJ: lsblk method part 1 -->

1. Connect a micro-SD card to your PC. You can now see new storage devices:

<!-- JIJ: lsblk method part 2 and I have left in some of the stuff from RPi3 as it doesn't hurt, but could be part of the generic docs -->

    * The SD card device file in `/dev`, probably as `/dev/sdX` for some letter `X` (for example, `/dev/sdd`).
    * Device files for its partitions. `/dev/sdXN` for the same letter `X` and some numbers `N` (for example, `/dev/sdd1` and `/dev/sdd2`).

    <span class="notes">In the commands below, replace `/dev/sdX` with the device file name for the SD card _without_ a number at the end. </span>

1. Ensure none of the micro-SD card's partitions are mounted (replace `/dev/sdX` as explained above):

    ```
    sudo umount /dev/sdX*
    ```

1. Write the disk image to the SD card device - not a partition on it (replace `/dev/sdX` as explained above):

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.gz /dev/sdX`</span>

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. Detach the micro-SD card from your PC, and plug it into the NXP 8M Mini EVK.
1. You need access to the device's console, so before powering it on, connect the micro-USB port to your PC.
    
1. After connecting the NXP 8M Mini EVK, from your PC, run the following command to connect to the device's serial console:

    ```
    minicom -D /dev/ttyUSB0
    ```

    <span class="notes">If you have also connected the USB-C cable to your PC from the NXP 8M Mini EVK, then this maybe `/dev/ttyUSB1`</span>

    Use the following settings:

    * Baud rate: 115200.
    * Encoding: [8N1](https://en.wikipedia.org/wiki/8-N-1).
    * No hardware flow control (enabled by default).

1. Connect the NXP 8M Mini EVK USB-C power socket <!--JIJ: This is a different USB-C socket to the one I mentioned above!--> to the kits power brick and plug this into a wall socket.<!--JIJ: Urk! That probably needs re-phrasing -->

    The device now boots into MBL.

## Warp7 devices

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

    `mbl-image-development-imx7s-warp-mbl.wic.gz` is a full disk image, so it should be written to the whole flash device, not a partition.

    The device file for the whole flash device is the one without `-part` in the name (`/dev/disk/by-id/usb-Linux_UMS_disk_0-0:0` in this example).

1. Ensure that none of the Warp7's flash partitions are mounted (you may need to adjust the path depending on the name of the storage device):

    ```
    sudo umount /dev/disk/by-id/usb-Linux_UMS_disk_0-0:0-part*
    ```

1. From a Linux prompt, write the disk image to the Warp7's flash device (replace `<device-file-name>` with the correct device file for the Warp7's flash device; in this example, it would be `usb-Linux_UMS_disk_0-0:0`):

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>`</span>

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

## Raspberry Pi 3 devices

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
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.bmap /path/to/artifacts/machine/raspberrypi3-mbl/images/mbl-image-development/images/mbl-image-development-raspberrypi3-mbl.wic.gz /dev/sdX`</span>

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

## Logging in to MBL

To log in to MBL, wait for a login prompt, and then enter the username `root`. You will not be prompted for a password.
