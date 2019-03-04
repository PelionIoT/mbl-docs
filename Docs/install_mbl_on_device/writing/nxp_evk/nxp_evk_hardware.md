# NXP 8M Mini EVK

<span class="notes">This process currently uses a micro-SD card, but will be updated in future to explain usage of the on board EMMC</span>

<!--
1. Check the current block storage devices on your PC:-->

<!-- JIJ: lsblk method part 1 -->

<!--1. Connect a micro-SD card to your PC. You can now see new storage devices:-->

<!-- JIJ: lsblk method part 2 and I have left in some of the stuff from RPi3 as it doesn't hurt, but could be part of the generic docs -->


<!--
    <span class="notes">In the commands below, replace `/dev/sdX` with the device file name for the SD card _without_ a number at the end. </span>-->
<!--
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

1. Connect the NXP 8M Mini EVK USB-C power socket <!--JIJ: This is a different USB-C socket to the one I mentioned above!--> <!--to the kits power brick and plug this into a wall socket.<!--JIJ: Urk! That probably needs re-phrasing -->

<!--The device now boots into MBL.
-->
