## Device update

You can update the device's root file system (rootfs) and applications. Update of boot loaders, the Linux kernel and other components will be supported in later versions.

Device update uses the `mbl-firmware-update-manager`, which is part of MBL:

```
$ mbl-firmware-update-manager -h

usage: mbl-firmware-update-manager [-h] -i UPDATE_FIRMWARE_TAR_FILE [-s] [-v]

optional arguments:

 -h, --help            show this help message and exit

 -i UPDATE_FIRMWARE_TAR_FILE, --install-firmware UPDATE_FIRMWARE_TAR_FILE

                       Install firmware from a firmware update tar file and

                       reboot (default: None)

 -s, --skip-reboot     Skip reboot after firmware update (default: False)

 -v, --verbose         Increase output verbosity (default: False)
```

For detailed instructions, as well as reference information about device update, please see the [Updating devices section](../update/index.html)
