## ConnMan Tutorial

ConnMan (Connection Manager) is chosen as the MBed Linux OS network manager (other known options are NetworkManager, systemd-networkd, nectctl, wist and VMC).It is daemon that manage network connections Within embedded devices, and integrates a range of features usually split between many daemons such as DHCP, DNS and NTP.

#### ConnMan main attributes :
* Lightweight, small footprint with low memory consumption.
* Fast response time for changing network conditions.
* Automatically manages wired connections.
* Fully modular, and can be extended through plug-ins.Hence, it supports all kinds of wired and wireless technologies.
* Supports configuration methods such as DNS,DHCP using plug-ins.
* ConnMan can be used also throught a command line interface client - **connmanctl**, which can be run in two modes: a plain synchronous command input, and an asynchronous interactive shell.
* ConnMan manages wired connections automatically.

#### ConnMan on Mbed Linux OS

ConnMan comes as part of the Mbed Linux OS image, with few modifications:
* The official ConnMan depends on GNU General Public License version 3 (GPLv3) libraries. All these dependencies has been removed on build/linking stages by disabling some functionalities :
    * wispr (Wireless Internet Service Provider roaming) is not supported.
    * readline library has been replaced by libedit. For this reason, auto-complete and history are disabled for now for the connmanctl  CLI.
    * connman VPN daemin is not supported for now (disabled on compile time).
* In order to avoid conflicts with wpa_supplicant, AP scanning/selection is disabled in wpa_supplicant.conf (ap_scan=0)

#### ConnMan configuration and state
ConnMan connfiguration and state can be found under /config/user/connman/ :
* main.conf (also known as connman.conf) can be found under /config/user/connnman/main.config. by default, this file should be under /etc/connman/main.conf.
* ConnMan settings and state can also be found also under /config/user/connman. by default, these should be under /var/lib/connman.

ConnMan's main.conf is currently configured to give priority for Ethernet connections over WiFi connections. That means, if both connections exist at the same time, the default chosen route  will be the one of the Ethernet connection. The other WiFi connection stays connected in connman's 'ready'  state, and becomes 'Online' only if Ethernet cable is disconnected or connection is lost.

#### Supported (valiudated and tested) interfaces
For now, we have validated ConnMan operation under the following connections types :
* WiFi - non-protected APs and password protected APs (WEP, WPA, WPA2).
* Ethernet / Ethernet Over USB

#### Known issues
At this stage, connmanctl interactive mode has known display bugs while using advanced nevigation features such as end button, home button etc.

#### Additional references
ConnMan official documnetation :
https://01.org/connman/documentation

ConnMan can be downloaded from : https://mirrors.edge.kernel.org/pub/linux/network/connman/

ConnMan can be cloned from git :
git://git.kernel.org/pub/scm/network/connman/connman.git

ConnMan man pages (debian) :
https://manpages.debian.org/testing/connman/connman.8.en.html
https://manpages.debian.org/testing/connman/connman-service.config.5.en.html

connmanctl man pages (debian) : https://manpages.debian.org/testing/connman/connmanctl.1.en.html
https://manpages.debian.org/testing/connman/connman.conf.5.en.html
