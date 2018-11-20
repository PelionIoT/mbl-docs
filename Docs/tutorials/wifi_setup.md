## Setting up Wi-Fi in Mbed Linux OS

Mbed Linux OS (MBL) uses **Connection Manager (ConnMan)** to manage Wi-Fi interfaces and connections. This page briefly reviews ConnMan, then explains how to use it to set up and manage Wi-Fi on MBL devices.

As an MBL developer, use ConnMan for all basic Wi-Fi operations, rather than interacting directly with the `wpa_supplicant` daemon or modifying the `wpa_supplicant.conf` file.

<span class="tips">This page reviews only some of the commonly used ConnMan options, within the context of MBL. You may be interested in [the full ConnMan documentation](https://01.org/connman/documentation).</span>

### connmanctl

**connmanctl** is a command-line interface (CLI). It can operate in two modes:

* A plain command input, when you want to run just one or two commands. Enter `connmanctl` followed by a command to initiate that command. For example, we can trigger an immediate run of the command `state`:

    ```
    # connmanctl state
      State = ready
      OfflineMode = False
      SessionMode = False
    ```

    Use `connmanctl --help` for more information.

* An interactive shell for more extensive use. To enter the interactive shell, enter `connmanctl`, without entering a specific command. Some operations, such as a first connection to a protected wireless access point, are only possible through the interactive shell mode.

    ```
    # connmanctl
    connmanctl> agent on
    Agent registered
    ```

    Interactive mode supports the same commands as the plain mode, but you don't need to prefix the commands with `connmanctl`. Enter `help` for more information.


<span class="tips">The official [connmanctl documentation is on the ConnMan site](https://01.org/connman/documentation).</span>

### Configuration and state

ConnMan automates many network operations and configurations by interacting with other daemons (DNS, DHCP and others) and by keeping a settings file, service files and other auto-generated data in the `/config/user/connman/` folder.

The folder contains:

* `/config/user/connman/settings` file: Current global and per-technology settings.
* `/config/user/connman/main.conf` file: The main ConnMan configuration file. Currently, most parameters are set to default (commented); the only set parameter is `PreferredTechnologies`, which prioritizes Ethernet interface default routes over the Wi-Fi interface when both are connected.
* An automatically generated **service profile**. For each Wi-Fi network that ConnMan connects to, it creates a directory in `/config/user/connman` to hold connection-specific configuration, such as passphrases, to support auto-connect on boot. In the initial state, there are no service profiles defined.

    To see all service profiles:

    ```
    # cat /config/user/connman/*/settings
    ```
* For advanced connections, you may manually generate service files and place them under `/config/users/connman` (see section [Connecting to a network using service configuration files](#connecting-to-a-network-using-service-conflagration-files)). These are also known as *provisioning files*, and their file extension is `.config`.

### Enabling Wi-Fi

ConnMan refers to network interface types as "technologies"; each technology supplies different services. For example, Wi-Fi is a technology (named `wifi`) that supplies a service called `wifi_dc85de828967_68756773616d_managed_psk`.

By default, all technologies are disabled at the very first startup to prevent unwanted wireless or wired communication. To enable Wi-Fi:

1. Enter `connmanctl state`.
1. Enter `ifconfig` to see the initial state. You will get an output similar to this, but the exact details depend on how you connect to your device:

  ```
  # connmanctl state
    State = idle
    OfflineMode = False
    SessionMode = False
  # ifconfig
  lo        Link encap:Local Loopback  
            inet addr:127.0.0.1  Mask:255.0.0.0
            inet6 addr: ::1/128 Scope:Host
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:20 errors:0 dropped:0 overruns:0 frame:0
            TX packets:20 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:2088 (2.0 KiB)  TX bytes:2088 (2.0 KiB)

  usb0      Link encap:Ethernet  HWaddr 32:0E:BF:08:80:24  
            inet6 addr: fe80::300e:bfff:fe08:8024/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:25590 errors:0 dropped:0 overruns:0 frame:0
            TX packets:43 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:4544764 (4.3 MiB)  TX bytes:4010 (3.9 KiB)

  ```

1. Enter `connmanctl enable wifi`.
1. The state changes to ready and the `wlan0` interface is active:

  ```
  # connmanctl enable wifi
  [83043.692440] IPv6: ADDRCONF(NETDEV_UP): wlan0: link is not ready
  Enabled wifi
  # [83044.504290] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready

  # ifconfig
  ...
  ...
  ...
  wlan0     Link encap:Ethernet  HWaddr A0:CC:2B:2C:CB:9B  
            inet addr:172.27.104.37  Bcast:172.27.105.255  Mask:255.255.254.0
            inet6 addr: fe80::a2cc:2bff:fe2c:cb9b/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:171 errors:0 dropped:0 overruns:0 frame:0
            TX packets:431 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:15360 (15.0 KiB)  TX bytes:57775 (56.4 KiB)
  # connmanctl state
    State = ready
    OfflineMode = False
    SessionMode = False
  ```

### Scanning for available Wi-Fi networks and inspecting results

To scan for nearby Wi-Fi networks:

```
# connmanctl scan wifi
Scan completed for wifi
```

To list the available networks found (among other services):

```
# connmanctl services
    AndroidAP5           wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none
    Edimax               wifi_a2b2a2b2a2b2_19999999_managed_wep
    mbed                 wifi_a1a1a1a1a1a1_91111111_managed_psk
    ourLocalNetwork      wifi_a0a0a0a0a0a0_41699999999e47_managed_ieee8021x
    Wired                gadget_000000000000_usb
```

The last command lists **all** available services. As you can see in the example output, there is also a `Wired` service (last line). The available Wi-Fi AP services are prefixed with `wifi_` and suffixed with the security protocol. For example:

* Open network (AndroidAP5): `_managed_none` suffix.
* WEP protected network (Edimax): `_managed_wep` suffix.
* WPA/WPA2 Enterprise 802.1X: (ourLocalNetwork) `managed_ieee8021x` suffix.
* WPA/WPA2 Personal (also known as WPA-PSK): `_managed_psk` suffix.

To more closely inspect a service after scanning, use `connmanctl services <service_id>`. For example, when we look at a local Wi-Fi network:

```
root@imx7s-warp-mbl:~# connmanctl services --properties wifi_a0a0a0a0a0a0_41699999999e47_managed_ieee8021x
/net/connman/service/wifi_a0a0a0a0a0a0_41699999999e47_managed_ieee8021x
  Type = wifi
  Security = [ ieee8021x ]
  State = idle
  Strength = 59
  Favorite = False
  Immutable = False
  AutoConnect = False
  Name = ourLocalWiFi
  Ethernet = [ Method=auto, Interface=wlan0, Address=A0:A0:A0:A0:A0:A0, MTU=1500 ]
  IPv4 = [  ]
  IPv4.Configuration = [ Method=dhcp ]
  IPv6 = [  ]
  IPv6.Configuration = [ Method=auto, Privacy=disabled ]
  Nameservers = [  ]
  Nameservers.Configuration = [  ]
  Timeservers = [  ]
  Timeservers.Configuration = [  ]
  Domains = [  ]
  Domains.Configuration = [  ]
  Proxy = [  ]
  Proxy.Configuration = [  ]
  Provider = [  ]
root@imx7s-warp-mbl:~#
```

### Connecting to an open (public) Wi-Fi network

Currently, ConnMan doesn't connect to public networks<!--clearly it does, but you have to give the service name, so what is it that it can't do?-->, because **Wireless Internet Service Provider roaming** (WISPr) is disabled.

To connect to a public open network (AndroidAP5 in the example), you have to use the service name:
```
# connmanctl connect wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none
[ 1321.787201] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
Connected wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none
# ifconfig
...
...
...
wlan0     Link encap:Ethernet  HWaddr A0:B1:A0:B1:A0:B1  
          inet addr:192.168.168.168  Bcast:192.168.168.168  Mask:255.255.255.0
          inet6 addr: fe80::fe80:fe80:fe80:fe80/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:40 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1922 (1.8 KiB)  TX bytes:7189 (7.0 KiB)
```

After connecting to AndroidAP5, `wlan0` is assigned an IP address (by the DHCP server).

If we check the state folder, we can see the service profile for this connection:

```
# ls -la /config/user/connman/
total 10
drwxr-xr-x    3 root     root          1024 Nov  6 14:12 .
drwxr-xr-x    7 root     root          1024 Oct 23 16:04 ..
-rw-r--r--    1 root     root          5731 Oct 22 16:03 main.conf
-rw-------    1 root     root           138 Nov  6 14:12 settings
drwx------    2 root     root          1024 Nov  6 14:12 wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none
```

ConnMan automatically created the folder `wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none`. Inside, there is a data (binary) file and a settings text file. The text file is used when inspecting a services using the `services <service_id>` option we introduced above, and includes all current settings.

```
# ls -la /config/user/connman/wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none/
total 7
drwx------    2 root     root          1024 Nov  6 14:12 .
drwxr-xr-x    3 root     root          1024 Nov  6 14:12 ..
-rw-------    1 root     root          4096 Nov  6 14:12 data
-rw-------    1 root     root           272 Nov  6 14:12 settings
# cat /config/user/connman/wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none/settings
[wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none]
Name=AndroidAP5
SSID=432432432432432
Frequency=2412
Favorite=true
AutoConnect=true
Modified=2018-11-06T14:12:25.981677Z
IPv4.method=dhcp
IPv4.DHCP.LastAddress=192.168.168.168
IPv6.method=auto
IPv6.privacy=disabled
```

Pay attention to the `AutoConnect=true` parameter. It means that ConnMnn will auto-connect to this network on reboot, if it is the only existing service profile, as well as in a few other cases (when the AP is nearby, and a few other configurations are applied).

You can change this file by using `connmanctl config <config data>`, or by editing it manually.

### Connecting to a protected network interactively

1. Use `connmanctl` in interactive mode.
1. Disconnect from the network AndroidAP5. and
1. Try connecting to the Edimax AP (which is WEP protected):

  ```
  ### connmanctl
  connmanctl> disconnect wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none
  [  131.769887] IPv6: ADDRCONF(NETDEV_UP): wlan0: link is not ready
  Disconnected wifi_a0b1a0b1a0b1a0b1_416416416416416_managed_none
  connmanctl> agent on
  Agent registered
  connmanctl> connect wifi_a2b2a2b2a2b2_19999999_managed_wep
  Agent RequestInput wifi_a2b2a2b2a2b2_19999999_managed_wep
    Passphrase = [ Type=wep, Requirement=mandatory ]
  Passphrase? XXXXXXXXXX
  connmanctl> [  146.007791] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
  Connected wifi_a2b2a2b2a2b2_19999999_managed_wep
  ```

<span class="notes">**Note:** ConnMan does not support password encryption.</span>

### Connecting to a network using service configuration (provisioning) files

For advanced configuration, use a provisioning file (with extension `.config`). Configurations include secured wireless access points that need complex authentication (such as WPA2 Enterprise), static IPs and so on. Each provisioning file can be used for multiple services at once.

For more information, refer to the [SysTutorials site](https://www.systutorials.com/docs/linux/man/5-connman.conf/).

### Connecting to a Wi-Fi WPA/WPA2 enterprise network

1. Disable Wi-Fi:

  ```
  # connmanctl disable wifi
  ```

1. Create a configuration provisioning file at `/config/user/connman/<name>.config`.

    <span class="tips">**Tip:** You can add the `-service` suffix to file names as a convention. For example, `name-service`.</span>

1. Add service information to the configuration file:

  ```
  [global]
  Name = my-ssid
  Description = Provide a short description

  [service_wifi_local_network]
  Type = wifi
  Name = <my-ssid>
  EAP = peap
  Phase2 = MSCHAPV2
  Identity = <my-username>
  Passphrase = <my-password>
  ```

    Replace `<my-ssid>`, `<my-username>`, and `<my-password>` with appropriate information. Amend the description.

1. Re-enable Wi-Fi:

  ```
  # connmanctl enable wifi
  ```

ConnMan is now connected to the specified network.

If you experience any issues, restart both ConnMan and `wpa_supplicant` daemons.
