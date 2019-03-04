
1. Ensure that none of the device's flash partitions are mounted (replace `/dev/sdX` as explained above):

    ```
    sudo umount /dev/sdX*
    ```

## Writing the disk image

1. From a Linux prompt, write the disk image to the device's flash device - not a partition on it (replace `/dev/sdX` as explained above):

    <span class="notes">**Note**: If you are not using the PICO-PI device, replace the device name (`imx7d-pico-mbl`) in the command with your device's: `imx8mmevk-mbl`, `imx7s-warp-mbl` or `raspberrypi3-mbl`.</span>

    ```
    sudo bmaptool copy --nobmap /path/to/artifacts/machine/imx7d-pico-mbl/images/mbl-image-development/images/mbl-image-development-imx7d-pico-mbl.wic.gz /dev/sdX
    ```

    This action may take some time.

    <span class="tips">**Tip**: Use `--nobmap` for the initial flashing, to ensure your flash is set up correctly. On all subsequent flashings, you can use the `--bmap` option with a bmap file to speed up the process: `sudo bmaptool copy --bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.bmap /path/to/artifacts/machine/imx7s-warp-mbl/images/mbl-image-development/images/mbl-image-development-imx7s-warp-mbl.wic.gz /dev/disk/by-id/<device-file-name>`</span>

1. When `bmaptool` has finished, eject the device:

    ```
    sudo eject /dev/sdX
    ```

1. For Warp7 (all image flashings) and for PICO-PI (except the first image flashing): in the U-boot prompt, press <kbd>Ctrl</kbd>-<kbd>C</kbd> to exit USB mass storage mode.

<!--and this is also for any pico that isn't the first time-->

1. For the PICO-PI: Unplug both USB cables from the device, and set the boot configuration jumper settings to boot from the on-board EMMC flash.<!--this, too, makes the file breakdown less than entirely useful-->

    <img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/pico7-flash-boot.jpg" width="25%" align="middle" />

1. Connect both your device's USB-C and micro-USB sockets back to your PC.

    The device now boots into MBL.
