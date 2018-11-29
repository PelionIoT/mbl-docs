## Mbed Linux OS CLI

The Mbed Linux OS CLI (MBL CLI) is a command-line interface for developing with Mbed Linux OS (MBL). MBL CLI supports  
the following operations on MBL devices:

 * List all available MBL devices on the network.
 * Select a default MBL device for all MBL CLI commands.
 * Get a shell on a device.
 * Copy a file to or from a device.
 * Run a command on a device.

<span class="notes">**Note**: MBL CLI can only be used with devices running the MBL test image `mbl-console-image-test`, rather than the production image `mbl-console-image`.</span>

## Setting up

To use MBL CLI, you need to:

1. [Set up networking](#network-setup) on the target device.
1. [Set up MBL CLI](#setting-up-mbed-linux-os-cli) on your development PC.

### Network setup

This section explains how to connect devices to a development PC over USB.

<span class="notes">**Note**: We provide examples of network interface names such as `enp0s2222222a` and `eno0`, as well as UUIDs. These values may be different for your devices and PC - please do not use them without checking.</span>

#### Overview

The MBL kernel can support USB connections to IoT devices that have USB ports. The exact support mechanism depends on the USB port type on the device - peripheral or host:

* **Peripheral port(s)**: The kernel's USB Gadget driver mechanism creates
a `usb0` network interface on the device. The device then appears as a network interface on any other USB-connected device, such as the development PC.

    Example: WaRP7. For more details, see below.

* **Host port(s)**: For devices that cannot work with the kernel's USB Gadget driver, MBL offers two Ethernet interfaces: `eth0`, which belongs to the wired Ethernet port, and `eth1`, which is created by the CDC Ethernet driver when appropriate hardware is connected. The reliance on Ethernet means that you need an Ethernet-to-USB adapter to support USB networking to a device with USB host ports.

    Example: Raspberry Pi 3. For more details, see below.

By default, MBL attempts to obtain an IPv4 address for the `usb0` interface on WaRP7 or the `eth1` interface on Raspberry Pi 3 using DHCP. MBL falls back to assigning a link-local IPv4 address when DHCP timeout occurs.  

`usb0` on the WaRP7 and `eth1` on the Raspberry Pi3 are debug network intefaces. On both devices, the debug network interface is always given an IPv6 link-local address.

#### Connecting a WaRP7 device

To connect a PC to an a WaRP7 device over USB:

1. Connect the device to the PC using a USB cable.
1. Configure the PC to use a link-local IPv4 address (169.254.x.y) for the interface.
1. Make sure that link-local IPv4 address is assigned to the network interface.

For example, on an Ubuntu PC:

1. Connect your device to your computer.

    The computer instantiates the Ethernet `net_device` for the USB network interface, but does not assign it an IP address.

1. Use `ifconfig -a` to list available network interfaces.

    The interface instantiated in the previous step is listed without an IPv4 address:

    ```
    $ ifconfig -a

    < ... lines deleted to save space >

      enp0s2222222a Link encap:Ethernet  HWaddr ee:a9:74:68:fe:69  
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:136 errors:0 dropped:0 overruns:0 frame:0
            TX packets:322 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:37211 (37.2 KB)  TX bytes:57393 (57.3 KB)

    < ... lines deleted to save space >
    ```  

1. Assign an IPv4 address [as explained below](#assigning-an-ipv4-address-to-the-network-interface).

#### Connecting a Raspberry Pi 3 device

To connect a PC to an a Raspberry Pi 3 device over an Ethernet-to-USB adapter:

1. Connect an Ethernet-to-USB adapter's USB "male" connector into any of
   the four type-A USB ports of the Raspberry Pi 3 board.
1. Connect the Ethernet cable of the adapter to an available Ethernet port on the development PC.

   If an extra USB Ethernet adapter is used on the PC side, a new network interface is created on the PC (for example, `enx503eaa4e094c`).

1. Check which network interface belongs to the port that is connected to the
   Raspberry Pi 3 device.
1. Configure the PC to use a link-local IPv4 address (169.254.x.y) for the interface.
1. Determine the address assigned to the interface.

For example, on an Ubuntu PC and a Raspberry Pi 3 device connected to an RTL8153 Gigabit Ethernet-to-USB adapter:

1. Connect the Raspberry Pi 3 device to the PC

1. Use `ifconfig -a` to identify the network interface:

    ```
    $ ifconfig -a
    < ... lines deleted to save space >

    eno0 Link encap:Ethernet  HWaddr 6c:0b:84:67:18:f5  
        UP BROADCAST MULTICAST  MTU:1500  Metric:1
        RX packets:5463 errors:0 dropped:0 overruns:0 frame:0
        TX packets:2839 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:3149461 (3.1 MB)  TX bytes:900468 (900.4 KB)
        Memory:fb100000-fb17ffff

    < ... lines deleted to save space >
    ```  

1. Assign an IPv4 address [as explained below](#assigning-an-ipv4-address-to-the-network-interface).

#### Assigning an IPv4 address to the network interface

<span class="tips">If you don't want to use the NetworkManager command line interface, you can use the `nm-connection-editor` GUI. See the [`nmcli` man page](https://linux.die.net/man/1/nmcli) for more information.</span>

To assign an IPv4 address to the network interface on an Ubuntu PC using NetworkManager:

1. Create a NetworkManager connection profile called `mbl-ipv4ll` for the
interface with the `link-local` IPv4 addressing method. Use the NetworkManager's command line interface:

    ```
    $ sudo nmcli connection add ifname <interace-name-on-pc> con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
    ```  

    where `<interface-name-on-pc>` is the name of the network interface on the PC that connects to the device. You found this name in the connection setup.

    * For example, for WaRP7 and the interface `enp0s2222222a`:

      ```
      $ sudo nmcli connection add ifname enp0s2222222a con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
      Connection 'mbl-ipv4ll' (0076a29f-8338-8338-8338-0076a29fffff) successfully added.
      ```

    * For example, for a Raspberry Pi 3 connected to the PC's `eno0` interface (or `enx503eaa4e094c` if using our USB Ethernet adapter example):

      ```
      $ sudo nmcli connection add ifname eno0 con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
      Connection 'mbl-ipv4ll' (475ebfb1-d67e-d67e-d67e-475ebfb1dddd) successfully added.
      ```

1. Activate the `mbl-ipv4ll` connection profile:

    ```
    $ sudo nmcli connection up mbl-ipv4ll
    ```

    * If this command finishes with the error
      `Error: Connection activation failed: No suitable device found for this connection.`, please verify that the device is managed by NetworkManager by checking that the field `managed` is set to `true` in the
      `/etc/NetworkManager/NetworkManager.conf` configuration file on the PC.   

    The NetworkManager connection is created.

1. Inspect the NetworkManager connection using the `nmcli connection show` command.

    * For example, for a WaRP7 device:

      ```
      $ nmcli connection show
      NAME                UUID                                  TYPE            DEVICE        
      mbl-ipv4ll          0076a29f-8338-8338-8338-0076a29fffff  802-3-ethernet  enp0s2222222a
      Wired connection 1  99cf6de7-2297-2297-2297-99cf6de7aaaa  802-3-ethernet  --          
      ```     

    * For example, for a Raspberry Pi 3 device:

      ```
      $ nmcli connection show
      NAME                UUID                                  TYPE            DEVICE     
      eno1                a815455d-8f18-8f18-8f18-a815455ddddd  802-3-ethernet  eno1    
      mbl-ipv4ll          475ebfb1-d67e-d67e-d67e-475ebfb1dddd  802-3-ethernet  eno0
      ```     

1. The PC's network interface now has an allocated IPv4 link-local address.  

    * For example, for a WaRP7 device:
      ```
      $ ifconfig enp0s2222222a
      enp0s2222222a Link encap:Ethernet  HWaddr ba:77:68:c0:73:df  
              inet addr:169.254.167.167  Bcast:169.254.255.255  Mask:255.255.0.0
              inet6 addr: fe80::b418:c138:20f0:57c7/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:146 errors:0 dropped:0 overruns:0 frame:0
              TX packets:364 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:40589 (40.5 KB)  TX bytes:65923 (65.9 KB)
      ```

    * For example, for a Raspberry Pi 3 device:

      ```
      $ ifconfig eno0
      eno0 Link encap:Ethernet  HWaddr 6c:0b:84:67:18:f5  
          inet addr:169.254.4.179  Bcast:169.254.255.255  Mask:255.255.0.0
          inet6 addr: fe80::3714:e5ad:7eb2:c3a5/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5529 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2936 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:3176233 (3.1 MB)  TX bytes:915351 (915.3 KB)
          Memory:fb100000-fb17ffff
      ```


### Setting up MBL CLI

#### Prerequisites

The only dependency of MBL CLI that you need to install is Node.js (version v8.10.0 or higher).

We recommend installing Node.js from NodeSource's binary distribution using [these instructions](https://github.com/nodesource/distributions#installation-instructions).

#### Installation

MBL CLI is distributed using npm. To install MBL CLI, run:

```
$ npm install -g mbl-cli
```

## MBL CLI command syntax

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

## MBL CLI usage

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

#### Remote command execution

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

### Device update

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

#### Rootfs update

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


#### Update an application

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
