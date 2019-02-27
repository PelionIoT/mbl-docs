## Identifying the newly mounted devices


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

1. Ensure that none of the device's flash partitions are mounted (replace `/dev/sdX` as explained above):

    ```
    sudo umount /dev/sdX*
    ```

## Writing the disk image

1. From a Linux prompt, write the disk image to the device's flash device - not a partition on it (replace `/dev/sdX` as explained above):

    <span class="notes">**Note**: If you are not using the PICO device, replace the device name (`imx7d-pico-mbl`) in the command with your device's: `imx8mmevk-mbl`, `imx7s-warp-mbl` or `raspberrypi3-mbl`.</span>

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/imx7d-pico-mbl/images/mbl-image-development/images/mbl-image-development-imx7d-pico-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. Unplug both USB cables from the device, and set the boot configuration jumper settings to boot from the on-board EMMC flash.

    <img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/pico7-flash-boot.jpg" width="25%" align="middle" />

1. Connect both your device's USB-C and micro-USB sockets back to your PC.

    The device now boots into MBL.
