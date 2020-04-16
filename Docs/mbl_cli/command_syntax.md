# Command syntax

To see the general structure of an MBL CLI command:

```
$ mbl-cli -h
mbl-cli <command> [arguments]

Commands:
  mbl-cli get <src> [dest] [address]  Copy a file from a device
  mbl-cli put <src> [dest] [address]  Copy a file to a device
  mbl-cli run <command> [address]     Run a command on a device
  mbl-cli select                      Select a device
  mbl-cli shell [address]             Get a shell on a device

Options:
  -v, --version  Show version number                                   [boolean]
  -h, --help     Show help                                             [boolean]

For more information about Mbed Linux OS, please visit http://mbed.com
```

Where `address` is the address of the debug interface on the device or the device's hostname. For example, the address of the `usb0` interface on WaRP7 IoT devices or the address of the `eth1` interface on Raspberry Pi 3 devices.

If the device is [already selected](#device-discovery-and-selection), you can omit the `address` parameter.


***

Copyright © 2020 Arm Limited (or its affiliates)
