# Updating MBL images or applications

This section explain how to update either your MBL image or any of your applications. For both updates, we provide two workflows: one using MBL CLI, the other using the manifest tool.

## Prerequisites

An MBL device can only be updated if the following are available:

* The device is running an MBL image that can connect to a Pelion account. [Follow the first section in this series](../first-image/provisioning-for-pelion-device-management.html) to request an account. <!-- JH_TODO: "the first section in this series" doesn't make sense any more -->
* The build artefacts for the image to send as the update payload. See the [build tutorial](../first-image/index.html) for instructions.
* An internet connection on the device. [Follow the tutorial to set up an internet connection](../first-image/connecting-to-a-network-and-pelion-device-management.html).
* The directory in which the manifest tool was initialized, [as reviewed in the provisioning prerequisites](../first-image/provisioning-for-pelion-device-management.html).

    <span class="notes">This *must* be the directory from which the `update_default_resources.c` file was obtained for provisioning MBL.</span>

* A Pelion API key, to use the manifest tool from the command line. Follow the instructions in the [requirements section](../first-image/provisioning-for-pelion-device-management.html) to obtain an API key (when prompted to select a group to set the API key access level, select **Developers**). Make a note of the API key to use it later; for security reasons, the portal will not display it again.

## Notes

<!-- JH_TODO: this section seems out-of-place... We haven't even explained how to perform an update, but we're explaining how to track update progress?! -->
* You can monitor the update payload **download** progress by tailing the `mbl-cloud-client` log file:

    ```
    root@mbed-linux-os-1234:~# tail -f /var/log/mbl-cloud-client.log
    ```

* You can monitor the update payload **installation** progress by tailing:

    * `/var/log/arm_update_activate.log`: for messages about the overall progress of the installation, and messages specific to `rootfs` updates.
    * `/var/log/mbl-app-update-manager-daemon.log`: for messages about application updates.
