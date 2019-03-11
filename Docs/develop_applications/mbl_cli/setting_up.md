# Setting up

To use MBL CLI, you need to:

1. [Set up MBL CLI](#setting-up-mbl) on your development PC.
1. [Set up a developer connection over USB](#set-up-a-developer-connection-over-USB) on the target device.

## Setting up MBL CLI

### Prerequisites

[Python v3.5 or higher](https://python.org) and `pip`.

Linux users require a few other dependencies. Install them using apt-get (this example is for Ubuntu 16.04):

`apt-get install --yes avahi-utils python3-cffi libssl-dev libffi-dev python3-dev`

### Uninstalling old versions

If you are running an old version (1.x) of MBL CLI, please remove it (and install version 2.0):

```
npm uninstall mbl-cli -g --save
```

### Installation

1. Use pip to install the MBL CLI. We recommend installing in a [Python virtual environment](https://www.python.org/dev/peps/pep-0405/).

    ```bash
    pip install git+ssh://git@github.com/armmbed/mbl-cli.git@mbl-0-6
    ```

## Set up a developer connection over USB

You can use a USB developer connection to debug and test your applications. The connection will not be interrupted by your development work, including work that disrupts network connectivity.

### Connecting a device with a USB gadget network interface

If the device can use the kernal USB gadget driver (for example, Warp 7), connect the device to the PC using a USB cable. A new network interface is created on the development PC.

### Connecting a device with an Ethernet-to-USB adapter

If the device uses an Ethernet-to-USB adapter (for example, Raspberry Pi 3):

1. Connect the Ethernet-to-USB adapter's USB "male" connector to any of the four type-A USB ports of the Raspberry Pi 3 board.
2. Connect the Ethernet cable of the adapter to an available Ethernet port on the development PC.

   If another USB Ethernet adapter is used on the PC side, a new network interface is created on the PC (for example, `enx503eaa4e094c`).

**Linux users must ensure the new interface is managed by NetworkManager by following the steps below.**

1. Create a NetworkManager connection profile called `mbl-ipv4ll` for the interface with the `link-local` IPv4 addressing method. Use the NetworkManager's command line interface:

    ```
    $ sudo nmcli connection add ifname <interface-name-on-pc> con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
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

2. Activate the `mbl-ipv4ll` connection profile:

    ```
    $ sudo nmcli connection up mbl-ipv4ll
    ```

    If this command finishes with `Error: Connection activation failed: No suitable device found for this connection.` or similar, please verify that the device is managed by NetworkManager. Check the `/etc/NetworkManager/NetworkManager.conf` configuration file on the Linux PC for an entry similar to:

    ```
    [device]
    match-device=interface-name:enp0s2222222a
    managed=1
    ```

    The NetworkManager connection is created.

3. Inspect the NetworkManager connection using the `nmcli connection show` command.

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

4. The PC's network interface now has an allocated link-local address.  

    For example, for a WaRP7 device:

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

    For a Raspberry Pi 3 device using an Ethernet-to-USB adapter:
    
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

Alternatively, you can connect the Raspberry Pi 3 to a network switch or router using its Ethernet port.
