# Warp7 devices

<span class="warnings">UNFINISHED. DO NOT USE.</span>

<!--needs lsblk-->

1. Check the current storage devices on your PC:


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

1. Connect both the Warp7's USB socket (on the I/O board) and the Warp7's mass storage USB socket (on the CPU board) to your PC.

    You should now be able to see a USB TTY device, such as, `/dev/ttyUSB0`, on your PC.

1. Connect to the Warp7's serial console using a command such as:

    ```
    minicom -D /dev/ttyUSB0
    ```

    Use the following settings:

    * Baud rate: 115200.
    * Encdoing: [8N1](https://en.wikipedia.org/wiki/8-N-1).
    * No hardware flow control (enabled by default).

1. If you got a U-boot prompt on the device, continue to the next step.

   If you got an operating system boot (for example, Android), reboot the device<!--this is where it gets confusing - because we also call the flash bit "flash device"--> until you get a U-boot prompt, and then press any key to prevent the operating system from booting again. Continue to the next step.

1. To expose the Warp7's flash device to Linux as USB mass storage, in the U-boot prompt, enter:

    ```
    ums 0 mmc 0
    ```

    On the Warp7, you now see an ASCII-art "spinner". On your PC, you now see new storage devices:

    ```
    ls -l /dev/sdX/
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

1.  `mbl-image-development-imx7s-warp-mbl.wic.gz` is a full disk image, so it should be written to the whole flash device, not a partition; you need to unmount the partitions before you start writing.

    The device file for the whole flash device is the one without `-part` in the name (`/dev/sdX/usb-Linux_UMS_disk_0-0:0` in this example). So you can unmount all the partitions at once, by unmounting anything with `part` in its name (you may need to adjust the path to match the name of the storage device):

    ```
    sudo umount /dev/sdX/usb-Linux_UMS_disk_0-0:0-part*
    ```

1. From a Linux prompt on your PC, write the disk image to the Warp7's flash device (replace `<device-file-name>` with the correct device file for the Warp7's flash device; in this example, it would be `usb-Linux_UMS_disk_0-0:0`):

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/sdX/<device-file-name>
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/sdX/<device-file-name>`</span>

1. When `bmaptool` has finished, eject the device<!--is this the whole device or the flash device? I think flash device--> (replace `<device-file-name>` with the correct device file for the Warp7's flash device; in this example `usb-Linux_UMS_disk_0-0:0`):

    ```
    sudo eject /dev/sdX/<device-file-name>
    ```

1. On the Warp7's U-boot prompt, press <kbd>Ctrl</kbd>-<kbd>C</kbd> to exit USB mass storage mode.
1. Reboot the Warp7:

    ```
    reset
    ```

    The device now boots into MBL.

1. To log in to MBL, wait for a login prompt, and then enter the username `root`. You will not be prompted for a password.