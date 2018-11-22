## Verifying that the device is connected to Device Management

When the device boots into MBL, `mbl-cloud-client` should automatically start and connect to Device Management. You can check whether it has connected by:

* Checking the device status on the [Device Management Portal](https://portal.mbedcloud.com/).
* Reviewing the log file for `mbl-cloud-client` at `/var/log/mbl-cloud-client.log`.

If your device hasn't automatically connected to Device Management, it could be that:

* Networking wasn't configured before the device was rebooted. Check your configurations and reboot the device.
* There are issues with the network. The device retries periodically, but you may need to restart `mbl-cloud-client`:

     ```
     /etc/init.d/mbl-cloud-client restart
     ```
