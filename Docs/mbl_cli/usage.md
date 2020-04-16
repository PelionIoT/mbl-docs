# Usage

## Device discovery and selection

You can list available devices, and select one to use for all subsequent commands:

1. To list the devices:

    ```
    mbl-cli select
    ```

    For example:

    ```
    $ mbl-cli select
    Discovering devices...
    Select a device:
    1: mbed-linux-os-3006 (fe80::94f8:52ff:fe67:d5d8%enp0s20u1)
    2: No device
    mbed-linux-os-3006 (fe80::94f8:52ff:fe67:d5d8%enp0s20u1) selected
    ```

1. To select a device, enter its number from the list above.

    For example, to select the device `mbed-linux-os-3006`, type `1`.

    You can now omit the `address` argument from subsequent commands.

    The selected device's address is stored in a configuration file (`${HOME}/.mbl.cfg`), so the selection will apply to all subsequent runs of mbl-cli by the same user.

 1. To deselect a device: run `select` again and select a different device.

## Remote command execution

To execute commands on the device:

1. Select a device from the list of available devices using the `mbl-cli select` command.

1. Run a command on the device:

    ```
    $ mbl-cli run <command> [address]
   ```

    <span class="notes">The commands are run by the user `root`.</span>

For example, to show the statuses of the active interfaces on a device:

```
$ mbl-cli run "ifconfig -a" 169.254.11.94
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:144 errors:0 dropped:0 overruns:0 frame:0
          TX packets:144 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:9648 (9.4 KiB)  TX bytes:9648 (9.4 KiB)

usb0      Link encap:Ethernet  HWaddr 96:F8:52:67:D5:D8  
          inet6 addr: fe80::94f8:52ff:fe67:d5d8/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:273 errors:0 dropped:0 overruns:0 frame:0
          TX packets:357 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:30202 (29.4 KiB)  TX bytes:82367 (80.4 KiB)

usb0:avahi Link encap:Ethernet  HWaddr 96:F8:52:67:D5:D8  
          inet addr:169.254.11.94  Bcast:169.254.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
```

## Get shell access (SSH)

To get shell access over SSH to a device (as the user `root`), use the `shell` command:

```
$ mbl-cli shell [address]
```

For a previously selected device, omit the address:

```
$ mbl-cli shell
Connecting to mbed-linux-os-3006...
root@mbed-linux-os-3006:~#
```

After obtaining shell access, you can set up Wi-Fi on the device (see [Setting up a network connection](../getting-started/tutorial-connecting-to-a-network-and-pelion-device-management.html)).


***

Copyright © 2020 Arm Limited (or its affiliates)
