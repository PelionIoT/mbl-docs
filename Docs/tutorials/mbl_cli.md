## Mbed Linux OS CLI

The Mbed Linux OS CLI is a command-line interface for developing with Mbed Linux OS. Mbed Linux OS CLI supports  
the following operations on Mbed Linux OS devices:

 * List all available MBL devices on the network and select an MBL device to be used by default by subsequent MBL CLI commands.
 * Get a shell on a device.
 * Copy a file to/from a device.
 * Run a command on a device.
 
In order to use Mbed Linux OS CLI, you need to [set up networking](#network-setup) on the target device and [set up Mbed Linux OS CLI](#setting-up-mbed-linux-os-cli) on your development PC.

Please note that Mbed Linux OS CLI can only be used with devices running the test image (`mbl-console-image-test`).

## Network setup
An Mbed Linux OS IoT device with suitable hardware supports networking over USB by using the  
Mbed Linux OS kernel's appropriate driver mechanism.

When a device with USB peripheral port(s) is used (such as a WaRP7), communication between 
the device and the development PC can be established by connecting the device 
directly to the development PC with a USB cable. The Linux kernel's USB Gadget 
driver is used on the device in this case.

When a device with USB host port(s) is used (such as a Raspberry Pi 3), communication between 
the device and a development PC can be established via a peripheral Ethernet-to-USB adapter 
plugged into one of the available USB host ports on the device. The Linux kernel's CDC 
Ethernet driver is used on the device for supporting this kind of communication.

* When running on WaRP7, the Mbed Linux OS kernel's USB Gadget driver mechanism creates 
  a `usb0` network interface on the IoT device and makes the IoT device itself appear 
  as a network interface to another device connected via USB (e.g. a development PC). 

* When Mbed Linux OS is installed on Raspberry Pi 3, the kernel's USB Gadget driver is not installed due to
  hardware limitations of the board, and the `usb0` interface does not exist. There are two Ethernet network 
  interfaces in Mbed Linux OS when installed on Raspberry Pi 3: `eth0`, which belongs to the wired 
  Ethernet port, and `eth1` which is created by the CDC Ethernet driver, once appropriate hardware 
  has been connected.  
  An Ethernet-to-USB hardware adapter is required in order to support USB networking on an IoT 
  device based on a Raspberry Pi 3 board. 
  Once an Ethernet-to-USB adapter's USB "male" connector is inserted into any of the four type-A 
  USB ports of the Raspberry Pi 3 board, the Mbed Linux OS kernel's CDC Ethernet driver mechanism creates an `eth1` 
  network interface. This makes it possible for another device to establish communication with the IoT device. 
  In order to establish communication with the development PC, the Ethernet cable of the 
  adapter should be connected to an available Ethernet port on the development PC. 

By default, Mbed Linux OS attempts to obtain an IPv4 address for the `usb0` interface on WaRP7 
or the `eth1` interface on Raspberry Pi 3 using DHCP and falls back to assigning a link-local IPv4 
address when DHCP timeout occurs.  

IPv6 is always present with a link local address on both WaRP7 and Raspberry Pi 3 devices.

#### Connecting a WaRP7 IoT device to a PC
To connect a PC to an Mbed Linux OS IoT WaRP7 device using the USB networking facility,
perform the following steps:

1. Connect the IoT device to the PC using USB cabel.
1. Configure the PC to use link-local IPv4 addressing for the interface and
   determine the address assigned to the interface.

For example, on an Ubuntu PC, do the following:

Use `ifconfig -a` to list available network interfaces. Once the IoT device 
has been connected to the development PC, the PC kernel will
instantiate the ethernet net_device for the USB network interface. This is shown
in the following listing, which shows the interface is present but has not yet 
been assigned an IP address:

  ```
  $ ifconfig -a
  < ... lines deleted to save space >

  enp0s20u5u4u4 Link encap:Ethernet  HWaddr ee:a9:74:68:fe:69  
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:136 errors:0 dropped:0 overruns:0 frame:0
            TX packets:322 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000 
            RX bytes:37211 (37.2 KB)  TX bytes:57393 (57.3 KB)

  < ... lines deleted to save space >
  ```  

#### Connecting a Raspberry Pi 3 IoT device to a PC
To connect a PC to an Mbed Linux OS IoT Raspberry Pi 3 device using an Ethernet-to-USB adapter,
perform the following steps:

1. Connect an Ethernet-to-USB adapter's USB "male" connector into any of 
   the four type-A USB ports of the Raspberry Pi 3 board, and the Ethernet 
   cable of the adapter into an available Ethernet port on the development PC.
1. Check which network interface belongs to the port that is connected to the 
   Raspberry Pi 3 device.
1. Configure the PC to use link-local IPv4 addressing for the interface and
   determine the address assigned to the interface.

For example, on an Ubuntu PC and a Raspberry Pi 3 device connected to an 
RTL8153 Gigabit Ethernet-to-USB adapter, do the following:

Connect the Raspberry Pi 3 device to the PC and use `ifconfig -a` to identify the network interface. 
  
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

  Note, that an IP address was not assigned yet to the `eno0` network interface. 

#### Assigning an IPv4 address to the network interface on a PC

On an Ubuntu PC using NetworkManager:

1. Create a NetworkManager connection profile called `mbl-ipv4ll` for the
interface with the `link-local` IPv4 addressing method using the NetworkManager's command line interface:

    ```
    $ sudo nmcli connection add ifname <interace-name-on-pc> con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
    ```  

    where `<interface-name-on-pc>` is the name of the network interface on the
    PC that connects to the IoT device.
    
    * For example, if using a WaRP7 and the name of the interface on the PC for the WaRP7 connection is `enp0s20u5u4u4`:
      ```
      $ sudo nmcli connection add ifname enp0s20u5u4u4 con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
      Connection 'mbl-ipv4ll' (0076a29f-6892-45bb-8338-2879b863efdf) successfully added.
      ```
      
    * If using a Raspberry Pi 3 connected to the PC's `eno0` interface:
      ```
      $ sudo nmcli connection add ifname eno0 con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
      Connection 'mbl-ipv4ll' (475ebfb1-d67e-47e9-afd2-8f2cf8a16cdd) successfully added.
      ```

1. Activate the `mbl-ipv4ll` connection profile:
    ```
    $ nmcli connection up mbl-ipv4ll
    ```
    * This step may not be required as NetworkManager may automatically enable the connection profile.
    * If this command finishes with the error
      `Error: Connection activation failed: No suitable device found for this connection.`
      verify that the device is managed by NetworkManager (that the field `managed` is set to `true` in the
      `/etc/NetworkManager/NetworkManager.conf` configuration file on the Linux development PC).   

1. The NetworkManager connection has now been created and can be inspected using the `nmcli connection show` command.
    * For example, when using a WaRP7 based IoT device:
      ```
      $ nmcli connection show
      NAME                UUID                                  TYPE            DEVICE        
      mbl-ipv4ll          0076a29f-6892-45bb-8338-2879b863efdf  802-3-ethernet  enp0s20u5u4u4
      Wired connection 1  99cf6de7-2297-3607-923a-4286fdbf357a  802-3-ethernet  --          
      ```     

    * When using a Raspberry Pi 3 based IoT device:
      ```
      $ nmcli connection show
      NAME                UUID                                  TYPE            DEVICE     
      eno1                a815455d-8f18-4f25-a8d1-39f0f89fc022  802-3-ethernet  eno1    
      mbl-ipv4ll          475ebfb1-d67e-47e9-afd2-8f2cf8a16cdd  802-3-ethernet  eno0
      ```     
     
    The PC's network interface (that communicates with an IoT device) should now have been 
    allocated an IPv4 link-local (169.254.x.y) address:
    * For example, when using a WaRP7 based IoT device:
      ```
      $ ifconfig enp0s20u5u4u4
      enp0s20u5u4u4 Link encap:Ethernet  HWaddr ba:77:68:c0:73:df  
              inet addr:169.254.131.167  Bcast:169.254.255.255  Mask:255.255.0.0
              inet6 addr: fe80::b418:c138:20f0:57c7/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:146 errors:0 dropped:0 overruns:0 frame:0
              TX packets:364 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:40589 (40.5 KB)  TX bytes:65923 (65.9 KB)
      ```

    * When using a Raspberry Pi 3 based IoT device:

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

Note, that network interface names (like `enp0s20u5u4u4`, `eno0` etc.) and connection's UUID values 
can be different on other development PCs.

An alternative to using the NetworkManager command line interface is to use the `nm-connection-editor` GUI to configure the interface. 

See the [`nmcli` man page](https://linux.die.net/man/1/nmcli) for more information.


## Setting up Mbed Linux OS CLI

### Prerequisites

The only dependency of MBL CLI that you need to install is Node.js (version v8.10.0 or higher). 
We recommend installing Node.js from NodeSource's binary distribution using [these instructions](https://github.com/nodesource/distributions#installation-instructions).

### Installation

MBL CLI is distributed using npm. To install the MBL CLI tool run:

```
$ npm install -g mbl-cli
```

## MBL CLI command

To see the general structure of an MBL CLI command, do the following:

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

## Mbed Linux OS CLI Usage

#### Device discovery and selection

Select the device from the list of available devices using the `mbl-cli select` command. For example, in order to select 
the device with hostname `mbed-linux-os-3006`, type `1` when presented with the output below.


```
$ mbl-cli select
Discovering devices...
Select a device:
1: mbed-linux-os-3006 (fe80::94f8:52ff:fe67:d5d8%enp0s20u1)
2: None
mbed-linux-os-3006 (fe80::94f8:52ff:fe67:d5d8%enp0s20u1) selected
```

After selecting the device, the optional `address` argument can be omitted from subsequent Mbed Linux OS CLI commands.

#### Get a shell access (SSH)

Use the Mbed Linux OS CLI `shell` command to get shell access (via SSH) on a device. The user gets the shell to the device with the user `root' permission:

```
$ mbl-cli shell [address]
```

Where `address` is the address of the debug interface on the device or the device's hostname. For example, the address of the `usb0` interface on WaRP7 IoT devices
or the address of the `eth1` interface on Raspberry Pi 3 devices. If the device is already selected, the `address` parameter can be omitted.
For example, to get a shell on a previously selected device, do the following:

```
$ mbl-cli shell
Connecting to mbed-linux-os-3006...
root@mbed-linux-os-3006:~# 
```

After obtaining shell access, you can set up Wi-Fi on the device (see [Setting up a network connection](https://os.mbed.com/docs/linux-os/current/getting-started/setting-up-a-network-connection.html)).

#### Device update

Currently device update can be done for the rootfs and applications. Update of boot loaders, the Linux Kernel and other components will be supported in later versions.

##### Rootfs update

In order to update the rootfs, prepare a tar file containing `rootfs.tar.xz`. For a detailed explanation of how to create a payload for rootfs updates see the [root file system update workflow](https://os.mbed.com/docs/linux-os/current/getting-started/tutorial-updating-mbl-devices-and-applications.html#workflow).

Once you have a rootfs update payload, follow these steps to update the device:

1. Transfer the rootfs update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli copy <rootfs update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the name of the payload file for the rootfs update and 169.254.6.215 is a link-local IPv4 address on the device, do the following:

   ```
   $ mbl-cli copy payload.tar /scratch 169.254.6.215
   ```

1. Use the Mbed Linux OS CLI `shell` command to get shell access (via SSH) on the device:

   ```
   $ mbl-cli shell [address]
   ```

   For example:

   ```
   $ mbl-cli shell 169.254.6.215
   ```
   
1. Inside the shell run the `mbl-firmware-update-manager` script to install the rootfs:

   ```
   $ mbl-firmware-update-manager -i <full path to TAR file under /scratch>
   ```

   For example:

   ```
   $ mbl-firmware-update-manager -i /scratch/payload.tar
   ```

   The device will automatically reboot after the rootfs update.
   It is recommended to delete old tar files from the `scratch` partition after updates finish.
   
   In order to see `mbl-firmware-update-manager`'s help menu run:

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

##### Update an application

In order to install/update an application, prepare a tar file containing an OPKG package (`.ipk` file) for an MBL application. 
For a detailed explanation of how to create a payload for an application install/update see the [workflow for an application apdate](https://os.mbed.com/docs/linux-os/current/getting-started/tutorial-updating-mbl-devices-and-applications.html#workflow).

To perform an application update, follow these steps:

1. Transfer an application update tar file to the `/scratch` partition on the device:

   ```
   $ mbl-cli put <application update payload> <destination on device under the /scratch partition> [address]
   ```

   For example, if `payload.tar` is the payload name for an application update and 169.254.6.215 is a link-local IPv4 address on the device, do the following:

   ```
   $ mbl-cli put payload.tar ./scratch 169.254.6.215
   28 Nov 09:42:04 - File transfer succeeded
   ```

1. Get a shell on the device and run `mbl-firmware-update-manager` with `--skip-reboot` parameter.
   The application will be installed on the device and started automatically. Each application runs on the device in a separate OCI container.
   It is recommended to delete old tar files from the `scratch` partition after application installations.
   
##### Remote command execution

Select a device from the list of available devices using the `mbl-cli select` command, then run a command on the device. The command will run on the device with the user `root` permission:

```
$ mbl-cli run <command> [address]
```

Where `address` is the address of the debug interface on the device or the device hostname. For example, the address of the `usb0` interface on WaRP7 IoT devices
or the address of the `eth1` interface on Raspberry Pi 3 devices.

For example, in order to show the statuses of the active interfaces on a device, run the following:

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

