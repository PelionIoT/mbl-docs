# Updating MBL

This section explain how to update the updatable components of MBL, including your applications. For updates, we provide two workflows: one using MBL CLI, the other using the manifest tool.

## Prerequisites

An MBL device can only be updated if the following are available:

* The device is running an MBL image that can connect to a Pelion account. Please [install an image and provision your devices before you continue](../first-image/index.html).
* The build artefacts for the image to send as the update payload. See the [build tutorial](../first-image/index.html) for instructions.
* An internet connection on the device. [Follow the instructions to set up an internet connection](../first-image/connecting-to-a-network-and-pelion-device-management.html).
* The directory in which the manifest tool was initialized, [as reviewed in the development environment setup](../first-image/provisioning-for-pelion-device-management.html).

    <span class="notes">This *must* be the directory from which the `update_default_resources.c` file was obtained when you provisioned your device.</span>

* A Pelion API key, to use the manifest tool from the command line. Follow the instructions in the [requirements section](../first-image/provisioning-for-pelion-device-management.html) to obtain an API key (when prompted to select a group to set the API key access level, select **Developers**). Make a note of the API key to use it later; for security reasons, the portal will not display it again.


***

Copyright © 2020 Arm Limited (or its affiliates)
