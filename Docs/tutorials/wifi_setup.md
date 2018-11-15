# Setting up Wi-fi in Mbed Linux OS

<!--Not edited yet-->

## ConnMan

Connection Manager (ConnMan) is a connection management daemon (`connmand`) for managing Internet connections within devices running the Linux operating system. It offers low memory consumption with a fast, coherent, synchronized reaction to changing network conditions.

ConnMan has various plugins and is a fully modular system that can support a variety of wired or wireless technologies. The [plug-in architecture](http://www.embedded-computing.com/networking/the-connman) allows for easy adaption and modification for various use cases. Configuration methods like [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) and domain name resolving are implemented using plug-ins. Additional information about `ConnMan` can be found in the following [link](https://01.org/connman/documentation).

Mbed Linux OS uses `ConnMan` to manage Wi-Fi interfaces and connections. In the common case, user should not interact directly with the `wpa_supplicant` daemon, nor try to modify the `wpa_supplicant.conf` file. All basic network operations should be done using ConnMan.

### `connmanctl`

`connmanctl` is a command-line interface (CLI). It can operates in 2 modes:
* A plain synchronous command input: You can use `connmanctl` straight from the shell. For example:
```
# connmanctl state
  State = ready
  OfflineMode = False
  SessionMode = False
```
* An asynchronous interactive shell: To enter the interactive shell, enter `connmanctl`. Some operations are permitted only in asynchronous shell mode:
```
# connmanctl
connmanctl> agent on
Agent registered
```

For help use `connmanctl --help` or just `help` in interactive mode.

The `connmanctl` man page can be found here:  
https://www.systutorials.com/docs/linux/man/8-connman/

### Configuration and state

ConnMan automates many of the network operations and configurations by interacting with other daemons (DNS, DHCP and others) and by keeping a Settings file, service files and other auto-generated data in the `/config/user/connman/` folder.

In this folder:
* `/config/user/connman/settings` file: Current global and per-technology settings.
* `/config/user/connman/main.conf` file: This is the main ConnMan's configuration file. Currently, all parameters are set to default (commented) and only one is set to prioritize Ethernet interface default routes over Wi-Fi interface when both are connected.
* Automatically generated `service profile` - each folder holds the name of a specific service profile often/recently used, with current configuration and state to support auto-connect on boot and other definitions. They contain fields for the passphrase, `essid` and other information. In the initial state, there are no service profiles defined.

To see all service profiles:
```
# cat /config/user/connman/*/settings
```
* For advanced connections, user may place service files under this folder (see section [Connecting to a network using service configuration files](#connecting-to-a-network-using-service-conflagration-files)). These are also known as provisioning files and end with `.config` prefix.

### Enabling Wi-Fi

Network interfaces are referred by ConnMan as `Technologies`. Wi-Fi is a type of technology. Each technology supply different types of `services`. For example: Wi-Fi is a technology (named `wifi`) and an Access Point (AP) called `MyNetwork` supplies a service called `wifi_dc85de828967_68756773616d_managed_psk`.
By default, all technologies are disabled at the very first startup in order to prevent unwanted wireless or wired communication from happening. Enter `connmanctl state`, then `ifconfig` to see the initial state (you should get something similar to the next printout, depends how you connect to your device):
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
After enabling Wi-Fi, the state moves to ready and the `wlan0` interface is up <!---too informal--->:
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

### Scanning for Wi-Fi available networks and inspecting results

To scan for nearby Wi-Fi networks:
```
# connmanctl scan wifi
Scan completed for wifi
```
To list the available networks found (among other services):
```
# connmanctl services
    AndroidAP5           wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none
    Edimax               wifi_a0cc2b2ccb9b_4564696d6178_managed_wep
    mbed                 wifi_a0cc2b2ccb9b_6d626564_managed_psk
    AccessNG             wifi_a0cc2b2ccb9b_4163636573734e47_managed_ieee8021x
    Guest-AccessNG       wifi_a0cc2b2ccb9b_47756573742d4163636573734e47_managed_none
    HOTBOX-2211          wifi_a0cc2b2ccb9b_484f54424f582d32323131_managed_psk
                         wifi_a0cc2b2ccb9b_hidden_managed_none
    Nicita-Box4          wifi_a0cc2b2ccb9b_4e69636974612d426f7834_managed_psk
    Carambola            wifi_a0cc2b2ccb9b_436172616d626f6c61_managed_psk
    NoBscell2            wifi_a0cc2b2ccb9b_4e6f427363656c6c32_managed_psk
    DIRECT-XJC48x Series wifi_a0cc2b2ccb9b_4449524543542d584a4334387820536572696573_managed_psk
    Metropoline WIFI1    wifi_a0cc2b2ccb9b_4d6574726f706f6c696e65205749464931_managed_none
    Wired                gadget_000000000000_usb
```
The last command lists **all** available services. You can see in the example output, there is also a `Wired` service in the last line. The avaiavle Wi-Fi AP services are prefixed with `wifi_` and suffixed with the security protocol. For example:
* Public (open network): `_managed_none` prefix.
* WEP protected network: `_managed_wep` prefix.
* WPA/WPA2 Enterprise 802.1X: `managed_ieee8021x` prefix.
* WPA/WPA2 Personal (also known as WPA-PSK): `_managed_psk` prefix.

To more closely inspect a service after scanning, use the ''`connmanctl services --properties <service_id>`' option. In the next example we inspect the AccessNG <!--- Don't think we want to use our actual SSID---> network:
```
root@imx7s-warp-mbl:~# connmanctl services --properties wifi_a0cc2b2ccb9b_4163636573734e47_managed_ieee8021x
/net/connman/service/wifi_a0cc2b2ccb9b_4163636573734e47_managed_ieee8021x
  Type = wifi
  Security = [ ieee8021x ]
  State = idle
  Strength = 59
  Favorite = False
  Immutable = False
  AutoConnect = False
  Name = AccessNG
  Ethernet = [ Method=auto, Interface=wlan0, Address=A0:CC:2B:2C:CB:9B, MTU=1500 ]
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

### Connecting to an open Wi-Fi Network (public Wi-Fi)

Currently, ConnMan doesn't support connection to public networks with captive portal<!--- eh?--->, since ***WISPr*** (Wireless Internet Service Provider roaming) is disabled.

To connect to a public open network (AndroidAP5 in the example) you have to use the service name:
```
# connmanctl connect wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none
[ 1321.787201] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
Connected wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none
# ifconfig
...
...
...
wlan0     Link encap:Ethernet  HWaddr A0:CC:2B:2C:CB:9B  
          inet addr:192.168.43.146  Bcast:192.168.43.255  Mask:255.255.255.0
          inet6 addr: fe80::a2cc:2bff:fe2c:cb9b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:40 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1922 (1.8 KiB)  TX bytes:7189 (7.0 KiB)
```
As you can see, `wlan0` is assigned an IP address (by an DHCP server) after connecting to AndroidAP5.

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

The folder `wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none` has been automatically created by ConnMan. Inside, there is a data (binary) file and a settings text file:
```
# ls -la /config/user/connman/wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none/
total 7
drwx------    2 root     root          1024 Nov  6 14:12 .
drwxr-xr-x    3 root     root          1024 Nov  6 14:12 ..
-rw-------    1 root     root          4096 Nov  6 14:12 data
-rw-------    1 root     root           272 Nov  6 14:12 settings
# cat /config/user/connman/wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none/settings
[wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none]
Name=AndroidAP5
SSID=416e64726f6964415035
Frequency=2412
Favorite=true
AutoConnect=true
Modified=2018-11-06T14:12:25.981677Z
IPv4.method=dhcp
IPv4.DHCP.LastAddress=192.168.43.146
IPv6.method=auto
IPv6.privacy=disabled
```
As you may have noticed, this file is used when inspecting a services using the `services --properties` option. Inside this file you can see all current settings. Pay attention to the `AutoConnect=true` parameter. On reboot, or in some other cases (when AP is nearby and other configurations applied), if this is the only existing service profile, ConnMan auto-connects to this network.

You may change this file using `connmanctl config <config data>` or by editing it manually.

### Connecting to a protected network interactively

Use `connmanctl` interactive asynchronous mode. Disconnect from AndroidAP5 and try connecting to Edimax AP (which is WEP protected):
```
### connmanctl
connmanctl> disconnect wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none
[  131.769887] IPv6: ADDRCONF(NETDEV_UP): wlan0: link is not ready
Disconnected wifi_a0cc2b2ccb9b_416e64726f6964415035_managed_none
connmanctl> agent on
Agent registered
connmanctl> connect wifi_a0cc2b2ccb9b_4564696d6178_managed_wep
Agent RequestInput wifi_a0cc2b2ccb9b_4564696d6178_managed_wep
  Passphrase = [ Type=wep, Requirement=mandatory ]
Passphrase? XXXXXXXXXX
connmanctl> [  146.007791] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
Connected wifi_a0cc2b2ccb9b_4564696d6178_managed_wep
```
As you may have noticed, when in `connmanctl` interactive mode, commands are typed exactly the same without the `connmanctl` prefix.

<span class="notes">**Note:** Password encryption is not supported by ConnMan.</span>

### Connecting to a network using service conflagration (provisioning) files

A provision file (must end with `.config`) is used for advanced configurations, such as secured wireless access points which need complex authentication (such as WPA2 Enterprise), static IPs and so on. Each provisioning file can be used for multiple services at once.

For more information, refer to: https://www.systutorials.com/docs/linux/man/5-connman.conf/

### Connecting to a Wi-Fi WPA/WPA2 enterprise network

1. Disable Wi-Fi:
```
# connmanctl disable wifi
```
1. Create a configuration provisioning file at `/config/user/connman/<name>-service.config`. The `-service` prefix on the file name is just a convention.<!---Then why are we including it? Is it necessary or not?--->
1. Add to the configuration file:
 ```
[global]
Name = my-ssid
Description = Provide a short description

[service_wifi_accessng]
Type = wifi
Name = <my-ssid>
EAP = peap
Phase2 = MSCHAPV2
Identity = <my-username>
Passphrase = <my-password>
```
1. Replace `<my-ssid>`, `<my-username>`, and `<my-password>` with appropriate information. Amend the description.
1. Now enable Wi-Fi:
```
# connmanctl enable wifi
```
ConnMan is now connected to the desired network.

In case of any issues, restart both ConnMan and `wpa_supplicant` daemons.
