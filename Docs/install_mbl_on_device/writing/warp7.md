# Warp7 devices

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

1. Connect both the Warp7's USB socket (on the I/O board) and the Warp7's power and mass storage USB socket (on the CPU board) to your PC.

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

   If you got an operating system boot (for example, Android), reboot the device until you get a U-boot prompt, and then press any key to prevent the operating system from booting again. Continue to the next step.

1. To expose the Warp7's flash device to Linux as USB mass storage, in the U-boot prompt, enter:

    ```
    ums 0 mmc 0
    ```

    On the Warp7, you now see an ASCII-art "spinner".

1. On your PC, you can now see new storage devices:

    ```
    lsblk
    ```

    In this example, the Warp7 is listed as `sdc` (the partitions on the device are also shown):

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

    <span class="notes">In the commands below, replace `/dev/sdX` with the device file name for the device's USB mass storage _without_ a number at the end.</span>

1.  Ensure that none of the Warp7's flash partitions are mounted (replace `/dev/sdX` as explained above)::

    ```
    sudo umount /dev/sdX*
    ```

1. From a Linux prompt, write the disk image to the board's USB mass storage - not a partition on it (replace `/dev/sdX` as explained above):

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/sdX`</span>

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. On the Warp7's U-boot prompt, press <kbd>Ctrl</kbd>-<kbd>C</kbd> to exit USB mass storage mode.
1. Reboot the Warp7:

    ```
    reset
    ```

    The device now boots into MBL.

1. To log in to MBL, wait for a login prompt, and then enter the username `root`. You will not be prompted for a password.


***

Copyright © 2020 Arm Limited (or its affiliates)
