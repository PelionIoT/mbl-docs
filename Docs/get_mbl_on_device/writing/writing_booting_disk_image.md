# Writing and booting the disk image

This section contains instructions for writing the full disk image to:

* PICO-PI with an IMX7D SoM.
* NXP 8M Mini EVK.
* Warp7 device.
* Raspberry Pi 3 device.

<!-- JIJ: I think we could refactor this (possibly) as the instructions to find out which device to flash to, could be the same as the new instructions I have written - they would work for SDcard, Warp7 or Pico or IMX8 - i.e. using lsblk, and the following instructions on unmount, bmap-tool and eject would then be all identical. Its just the plugging in, and the rebooting that is different -->
