## Monitoring progress

* You can monitor the update payload **download** progress by tailing the `mbl-cloud-client` log file:

    ```
    root@mbed-linux-os-1234:~# tail -f /var/log/mbl-cloud-client.log
    ```

* You can monitor the update payload **installation** progress by tailing:

    * `/var/log/arm_update_activate.log`: for messages about the overall progress of the installation, and messages specific to `rootfs`, kernel or boot component updates.
    * `/var/log/mbl-app-update-manager-daemon.log`: for messages about application updates.
