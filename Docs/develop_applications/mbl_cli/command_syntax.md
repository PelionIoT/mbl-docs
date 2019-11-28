# Command syntax

To see the general structure of an MBL CLI command:

```
$ mbl-cli -h

The Mbed Linux OS CLI.

A toolbox for managing target devices running Mbed Linux OS.

optional arguments:
  -h, --help            show this help message and exit
  -a ADDRESS, --address ADDRESS
                        The ipv4/6 address or hostname of the device you want
                        to communicate with.

mbl-cli supports the following commands:
  discover and select a device to work with
    list                List Mbed Linux OS devices on the network.
    select              Select an Mbed Linux OS device to interact with.
    which               Show the currently selected device.
  
  transfer files to/from your device
    get                 Get a file from a device.
    put                 Put a file on a device.
  
  interact directly with your device's shell
    shell               Obtain an interactive shell, or run a single command, on a device.
  
  provision devices for cloud-based device management
    save-api-key        Save a Pelion Cloud developer API key to persistent storage.
    provision-pelion    Provision the selected device with developer and update authority certificates.
    get-pelion-status   Check if the device is correctly configured for Pelion Device Management.
```

Where `address` is the address of the debug interface on the device. For example, the address of the `usb0` interface on PICO-PI with IMX7D IoT devices or the address of the `eth1` interface on Raspberry Pi 3 devices.

If the device is [already selected](#device-discovery-and-selection), you can omit the `address` parameter.
