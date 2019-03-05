# Setting up

To use MBL CLI, you need to:

1. [Set up MBL CLI](#setting-up-mbl) on your development PC.
1. [Set up networking](#setting-up-networking) on the target device.

## Setting up MBL CLI

### Prerequisites

[Python v3.5 or higher](https://python.org) and `pip`.

Linux users require a few other dependencies. Install them using apt-get (this example is for Ubuntu 16.04):

`apt-get install --yes python3-cffi libssl-dev libffi-dev python3-dev`

### Uninstalling old versions

If you are running an old version (1.x) of MBL CLI, please remove it (and install version 2.0):

```
npm uninstall mbl-cli -g --save
```

### Installation

To install MBL CLI:

1. Clone the repository.

    ```bash
    git clone git@github.com:armmbed/mbl-cli-python.git
    ```

1. Ensure your `pwd` is the `mbl-cli-python` directory (contains `setup.py`).

    ```bash
    cd mbl-cli-python
    ```

1. Use pip to install the MBL CLI. We recommend installing in a [Python virtual environment](https://www.python.org/dev/peps/pep-0405/).


    ```bash
    pip install .
    ```

## Setting up networking

Your device, running an MBL development image<!--this implies I should expect something else with other images-->, should broadcast its SSH services by default over mDNS. All you need to do is connect the device to your local network (or to your computer if using USB networking); an IPv6 address should be automatically assigned.

If for some reason you are unable to use IPv6 addresses, please see the section on [Assigning an IPv4 address to the device's network interface](#assigning-an-IPv4-address-to-the-network-interface).

<span class="tips">See the [appendix](#appendix) for details on the mechanism MBL uses to assign IP addresses to the various interfaces on the device.</span>


### Connecting a WaRP7 device

To connect a PC to a WaRP7 device over USB connect the device to the PC using a USB cable. A new network interface, with an assigned IPv6 address, is created on the development PC. 

Note that you may have to wait up to a minute for your device to fully boot before the mDNS broadcast begins.

### Connecting a Raspberry Pi 3 device

To connect a PC to a Raspberry Pi 3 device over an Ethernet-to-USB adapter:

1. Connect the Ethernet-to-USB adapter's USB "male" connector to any of the four type-A USB ports of the Raspberry Pi 3 board.
2. Connect the Ethernet cable of the adapter to an available Ethernet port on the development PC.

   If an extra<!--"extra" to what?--> USB Ethernet adapter is used on the PC side, a new network interface is created on the PC (for example, `enx503eaa4e094c`).

Alternatively, you can connect the Raspberry Pi 3 to a network switch or router using its Ethernet port. 

## Appendix

<span class="notes">**Note**: We provide examples of network interface names such as `enp0s2222222a` and `eno0`, as well as UUIDs. These values may be different for your devices and PC - please do not use them without checking.</span>

### Overview

The MBL kernel can support USB connections to IoT devices that have USB ports. The exact support mechanism depends on the USB port type on the device - peripheral or host:

* **Peripheral port(s)**: The kernel's USB Gadget driver mechanism creates a `usb0` network interface on the device. The device then appears as a network interface on any other USB-connected device, such as the development PC.

    Example: WaRP7. For more details, see below.<!--where is "below"? that word applies to the whole page, but I think you mean the very next paragraph-->

* **Host port(s)**: For devices that cannot work with the kernel's USB Gadget driver, MBL offers two Ethernet interfaces: `eth0`, which belongs to the wired Ethernet port, and `eth1`, which is created by the CDC Ethernet driver when appropriate hardware is connected. The reliance on Ethernet means that you need an Ethernet-to-USB adapter to support USB networking to a device with USB host ports.

    Example: Raspberry Pi 3. For more details, see below.<!--where is "below"? that word applies to the whole page, but I think you mean the very next paragraph-->

By default, MBL attempts to obtain an IPv4 address for the `usb0` interface on WaRP7 or the `eth1` interface on Raspberry Pi 3 using DHCP. MBL falls back to assigning a link-local IPv4 address when DHCP timeout occurs.  

`usb0` on the WaRP7 and `eth1` on the Raspberry Pi3 are debug network interfaces. On both devices, the debug network interface is always given an IPv6 link-local address.<!--what's the relationship between this paragraph and the previous one? It seems to say that these ports are by default IPv6, but in the previous paragraph they were by default IPv4-->


### Assigning an IPv4 address to the network interface

<span class="tips">If you don't want to use the NetworkManager command line interface, you can use the `nm-connection-editor` GUI. See the [`nmcli` man page](https://linux.die.net/man/1/nmcli) for more information.</span>

If you cannot use IPv6 on Linux, you can manually assign IPv4 addresses instead.

To assign an IPv4 address to the network interface on an Ubuntu PC using NetworkManager:

1. Create a NetworkManager connection profile called `mbl-ipv4ll` for the interface with the `link-local` IPv4 addressing method. Use the NetworkManager's command line interface:

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
