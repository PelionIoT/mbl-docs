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

If you are running an old version (1.x) of MBL CLI, please uninstall it and install version 2.0 instead:

```
npm uninstall mbl-cli -g --save
```

### Installation

Use pip to install MBL CLI. We recommend installing MBL CLI in a [Python virtual environment](https://www.python.org/dev/peps/pep-0405/), to avoid clashes with existing Python applications.

```bash
python3 -m venv mbl-cli-venv
source ./mbl-cli-venv/bin/activate
python -m pip install --upgrade pip
python -m pip install git+https://github.com/armmbed/mbl-cli.git@mbl-os-0.10
deactivate
```

You can now run it by entering the virtual environment (`source ./mbl-cli-env/bin/activate`) and then performing the `mbl-cli` command, or you can call it directly using `mbl-cli-env/bin/mbl-cli`.

If you must install MBL CLI using the 'system' Python, install it using the `--user` flag.

```bash
pip install --user git+https://github.com/armmbed/mbl-cli.git@mbl-os-0.10
```

## Setting up a developer connection over USB

You can use a USB developer connection to debug and test your applications. The connection is not interrupted by your development work, including work that disrupts network connectivity.

Before setting up, run `ifconfig` (or the equivalent command on your OS) and note the interface names; you need this information later to determine the name of your device's network interface.

### Connecting a device with a USB gadget network interface

If the device can use the kernel's USB gadget driver (for example, PICO-PI with IMX7D), connect the device to the PC using a USB cable. A new network interface is created on the development PC (for example `enp0s2222222a`).

```bash
$ ifconfig
enp0s2222222a Link encap:Ethernet  HWaddr xy:11:22:x3:44:xy  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:52489 errors:0 dropped:0 overruns:0 frame:0
          TX packets:52489 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:59176006 (59.1 MB)  TX bytes:59176006 (59.1 MB)

veth93e055c Link encap:Ethernet  HWaddr be:9e:84:59:53:ac  
          inet6 addr: fe80::bc3e:64ff:fe52:52ac/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:386 errors:0 dropped:0 overruns:0 frame:0
          TX packets:575 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:42239 (42.2 KB)  TX bytes:625618 (625.6 KB)
```

<span class="notes">**Note**: There is usually no link-local IP address assigned to the interface at this stage. The IPv6/4 addresses are assigned when you set up a managed connection in the [Setting up NetworkManager on Linux](#setting-up-networkmanager-on-linux) section. Even if there is an assigned link-local address, you still need to ensure the connection is managed; the link-local address will be periodically revoked and reassigned for an unmanaged connection.</span>

### Connecting a device with an Ethernet-to-USB adapter

<span class="notes">**Note**: If your device has an ethernet port and you don't want to use USB networking, you can connect your device directly over Ethernet to a router or network switch. The device will advertise its network presence on all available interfaces.</span>

If the device uses an Ethernet-to-USB adapter (for example, Raspberry Pi 3):

1. Connect the Ethernet-to-USB adapter's USB "male" connector to any of the four type-A USB ports of the Raspberry Pi 3 board.
2. Connect the Ethernet cable of the adapter to an available Ethernet port on the development PC. Alternatively, if you don't have a spare Ethernet port available, you can connect a second Ethernet-to-USB adapter to a USB port on the PC side, and connect the Ethernet cable from the device-side adapter to it.

   If another Ethernet-to-USB is used on the PC side, a new network interface is created on the PC (for example, `enx503eaa4e094c`).

```bash
$ ifconfig
enx503eaa4e094c Link encap:Ethernet  HWaddr xy:11:22:x3:44:xy  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:52489 errors:0 dropped:0 overruns:0 frame:0
          TX packets:52489 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:59176006 (59.1 MB)  TX bytes:59176006 (59.1 MB)

veth93e055c Link encap:Ethernet  HWaddr be:9e:84:59:53:ac  
          inet6 addr: fe80::bc3e:64ff:fe52:52ac/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:386 errors:0 dropped:0 overruns:0 frame:0
          TX packets:575 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:42239 (42.2 KB)  TX bytes:625618 (625.6 KB)
```

<span class="notes">**Note**: Usually there is no link-local IP address assigned to the interface at this stage. The IPv6/4 addresses will be assigned when you set up a managed connection in the [Setting up NetworkManager on Linux](#setting-up-networkmanager-on-linux) section. Even if there is an assigned link-local address, you still need to ensure the connection is managed; the link local address will be periodically revoked and reassigned for an unmanaged connection.</span>


### Setting up NetworkManager on Linux

<span class="notes">**Note**: These instructions assume you're using Ubuntu 16.4; the commands may be different or unnecessary in other operating systems.</span>

1. Create a named NetworkManager connection profile for the interface with the `link-local` IPv4 addressing method.

    This examples uses the name `mbl-ipv4ll`. Use the NetworkManager's command-line interface:

    ```
    $ sudo nmcli connection add ifname <interface-name-on-pc> con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
    ```  

    where `<interface-name-on-pc>` is the name of the network interface on the PC that connects to the device. You found this name in the connection setup.

    * For example, for PICO-PI with IMX7D and the interface `enp0s2222222a`:

      ```
      $ sudo nmcli connection add ifname enp0s2222222a con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
      Connection 'mbl-ipv4ll' (0076a29f-8338-8338-8338-0076a29fffff) successfully added.
      ```

    * For example, for a Raspberry Pi 3 connected to the PC's `eno0` interface (or `enx503eaa4e094c` if using our USB Ethernet adapter example):

      ```
      $ sudo nmcli connection add ifname eno0 con-name mbl-ipv4ll type ethernet -- ipv4.method link-local
      Connection 'mbl-ipv4ll' (475ebfb1-d67e-d67e-d67e-475ebfb1dddd) successfully added.
      ```

    <span class="notes">**Note**: If you are setting up managed connections for multiple boards, you must give them unique names; reusing a name overwrites the existing connection profile.</span>

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

    This creates a NetworkManager connection.

3. Inspect the NetworkManager connection using the `nmcli connection show` command.

    * For example, for a PICO-PI with IMX7D device using a USB-gadget interface:

      ```
      $ nmcli connection show
      NAME                UUID                                  TYPE            DEVICE        
      mbl-ipv4ll          0076a29f-8338-8338-8338-0076a29fffff  802-3-ethernet  enp0s2222222a
      Wired connection 1  99cf6de7-2297-2297-2297-99cf6de7aaaa  802-3-ethernet  --          
      ```     

    * For example, for a Raspberry Pi 3 device using an Ethernet-to-USB adapter:

      ```
      $ nmcli connection show
      NAME                UUID                                  TYPE            DEVICE     
      mbl-ipv4ll          475ebfb1-d67e-d67e-d67e-475ebfb1dddd  802-3-ethernet  enx503eaa4e094c
      eno1                a815455d-8f18-8f18-8f18-a815455ddddd  802-3-ethernet  eno1    
      ```     

4. The PC's network interface now has an allocated link-local address.  

    For example, for a PICO-PI with IMX7D device using a USB-gadget interface:

    ```
    $ ifconfig enp0s2222222a
    enp0s2222222a Link encap:Ethernet  HWaddr xy:11:22:x3:44:xy  
            inet addr:169.254.167.167  Bcast:169.254.255.255  Mask:255.255.0.0
            inet6 addr: xy80::xy18:x111:00f0:57c7/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:146 errors:0 dropped:0 overruns:0 frame:0
            TX packets:364 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:40589 (40.5 KB)  TX bytes:65923 (65.9 KB)
    ```

    For a Raspberry Pi 3 device using an Ethernet-to-USB adapter:

    ```
    $ ifconfig enx503eaa4e094c
    enx503eaa4e094c Link encap:Ethernet  HWaddr 1x:1y:11:22:33:z1  
        inet addr:169.254.4.179  Bcast:169.254.255.255  Mask:255.255.0.0
        inet6 addr: xy80::1111:x5yz:9xy9:x1y1/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:5529 errors:0 dropped:0 overruns:0 frame:0
        TX packets:2936 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:3176233 (3.1 MB)  TX bytes:915351 (915.3 KB)
    ```


***

Copyright © 2020 Arm Limited (or its affiliates)
