## Updating MBL images or applications

<!--something about how this covers images and apps separately, and each one has both manifest tool and MBL CLI explained-->

### Prerequisites

An MBL device can only be updated if the followings are available:

* The device is running an MBL image that can connect to a Pelion account. [Follow the first section in this series](../getting-started/pelion-accounts-and-certificates.html) to request an account.
* The build artefacts for the image to send as the update payload. See the [build tutorial](../getting-started/tutorial-building-an-image.html) for instructions.
* An internet connection on the device. [Follow the tutorial to set up an internet connection](../getting-started/tutorial-connecting-to-a-network-and-pelion-device-management.html).
* The directory in which the manifest tool was initialized, [as reviewed in the development environment setup](../getting-started/development-environment.html).

    <span class="notes">This *must* be the directory from which the `update_default_resources.c` file was obtained for building MBL.</span>

* A Pelion API key, to use the manifest tool from the command line. Follow the instructions in the [requirements section](..//getting-started/api-keys.html) to obtain an API key (when prompted to select a group to set the API key access level, select **Developers**). Make a note of the API key to use it later; for security reasons, the portal will not display it again.

### Notes

* You can monitor the update payload **download** progress by tailing the `mbl-cloud-client` log file:

    ```
    root@mbed-linux-os-1234:~# tail -f /var/log/mbl-cloud-client.log
    ```

* You can monitor the update payload **installation** progress by tailing:

    * `/var/log/arm_update_activate.log`: for messages about the overall progress of the installation, and messages specific to `rootfs` updates.
    * `/var/log/mbl-app-update-manager-daemon.log`: for messages about application updates.
