## Command syntax

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

## Usage

### Device discovery and selection

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

### Remote command execution

To execute commands on the device:

1. Select a device from the list of available devices using the `mbl-cli select` command.

1. Run a command on the device:

    ```
    $ mbl-cli run <command> [address]
   ```

    <span class="notes">The commands are run by the user `root`.</span>

For example, to show the statuses of the active interfaces on a device:

```
$ mbl-cli run ifconfig 169.254.11.94
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

### Get shell access (SSH)

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

After obtaining shell access, you can set up Wi-Fi on the device (see [Setting up a network connection](../getting-started/setting-up-a-network-connection.html)).

## Device update with MBL

You can update the device's root file system (rootfs) and applications. Update of boot loaders, the Linux kernel and other components will be supported in later versions.

Device update uses the `mbl-firmware-update-manager`:

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

### Rootfs update

To update the rootfs:

1. Prepare a tar file containing `rootfs.tar.xz`. For a detailed explanation, see the [root file system update workflow](../getting-started/tutorial-updating-mbl-devices-and-applications.html#workflow).

1. Transfer the rootfs update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli copy <rootfs update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the name of the payload file for the rootfs update, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli copy payload.tar /scratch 169.254.111.222
   ```

1. Use the MBL CLI `shell` command to get shell access on the device:

   ```
   $ mbl-cli shell [address]
   ```

   For example:

   ```
   $ mbl-cli shell 169.254.111.222
   ```

1. Inside the shell, run the `mbl-firmware-update-manager` script to install the rootfs:

   ```
   $ mbl-firmware-update-manager -i <full path to TAR file under /scratch>
   ```

   For example:

   ```
   $ mbl-firmware-update-manager -i /scratch/payload.tar
   ```

    The rootfs is updated.

1. The device automatically reboots.

<span class="notes">We recommend deleting the old tar files from the `scratch` partition after updates finish.</span>

### Update an application

You can update each application independently of any others.

To install or update an application:

1. Prepare a tar file containing an OPKG package (`.ipk` file) for an MBL application. For a detailed explanation, see the [workflow for an application update](../getting-started/tutorial-updating-mbl-devices-and-applications.html#workflow).

1. Transfer an application update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli put <application update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the payload name for an application update, and 169.254.111.222 is a link-local IPv4 address on the device:

   ```
   $ mbl-cli put payload.tar ./scratch 169.254.111.222
   28 Nov 09:42:04 - File transfer succeeded
   ```

1. Use the MBL CLI `shell` command to get shell access on the device:

  ```
  $ mbl-cli shell [address]
  ```

  For example:

  ```
  $ mbl-cli shell 169.254.111.222
  ```

1. Run `mbl-firmware-update-manager` with the `--skip-reboot` parameter.

    The application is installed.

1. The application you installed or updated starts automatically, without a device reboot.

<span class="notes">We recommend deleting the old tar files from the `scratch` partition after updates finish.</span>
