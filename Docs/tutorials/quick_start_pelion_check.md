## Verifying that the device is connected to Device Management

When the device boots into MBL, `mbl-cloud-client` automatically starts and connects to Device Management. You can check whether it has connected by:

* Checking the device status on the [Device Management Portal](https://portal.mbedcloud.com/).
* Reviewing the log file for `mbl-cloud-client` at `/var/log/mbl-cloud-client.log`.

    Here is an example log output of a successful connection:

    ```
    2018-12-06T07:11:58+0000 [INFO][mClt]: MbedCloudClient::complete status (0)
    2018-12-06T07:11:58+0000 [INFO][mbl ]: Client registered
    2018-12-06T07:11:58+0000 [INFO][mbl ]: Endpoint Name: 0167827df31500000000000100100032
    2018-12-06T07:11:58+0000 [INFO][mbl ]: Device Id: 0167827df31500000000000100100032
    ```
If your device hasn't automatically connected to Device Management, it could be that:

* Networking wasn't configured before the device was rebooted. Check your configurations and reboot the device.
* There are issues with the network. The device retries periodically, but you may need to restart `mbl-cloud-client`:

     ```
     /etc/init.d/mbl-cloud-client restart
     ```
