## Tutorial: updating an Mbed Linux OS device over the air

There are two main components to an MBL device: Mbed Linux OS, and the application working on top of it. MBL supports over-the-air updates for both components independently of each other. This tutorial focuses on updating Mbed Linux OS; we also have a tutorial [for updating the application]()<!--not yet we don't-->.

**How firmware gets updated**

<!--There's a question of how much of the theory should be explained here (and in the previous tutorial).-->


To enable firmware update, the full disk image contains a partition table and all the partitions required by Mbed Linux, including two root partitions:

- The running bank: a device partition storing the rootfs for the running system.
- The non-running bank: a device partition that will receive the firmware update.<!--Called this partition in the previous tutorial-->

Only one of the root filesystem partitions is **active** at any one time, and the other is available to receive a new version of firmware during an update. At the end of the update process, the root partition with the new firmware becomes **active** and the other root partition becomes available to receive the next firmware update.  

During a firmware update, the update software:<!--You mean update client? What, by the way, manages the updates on an MBL device?-->

- Writes the new software rootfs to the non-running bank.
- Sets the non-running bank to be the running bank next time the device boots. <!--Can I say "this automatically frees the other running bank, which can accept the next update"?-->
- Reboots the device.

<span class="tips">Updates rely on Pelion Device Management capabilities. A full review of Pelion Device Management Update is [available on the Pelion documentation site](https://cloud.mbed.com/docs/latest/updating-firmware/index.html).</span>

### Workflow

Here be workflow.


- 12.1. [Update Step 1](#update2-1): Prerequisites.
- 12.2. [Update Step 2](#update2-2): Upload a firmware image to the cloud.
- 12.3. [Update Step 3](#update2-3): Create a manifest.
- 12.4. [Update Step 4](#update2-4): Upload the manifest to the cloud.
- 12.5. [Update Step 5](#update2-5): Create a filter for your device.
- 12.6. [Update Step 6](#update2-6): Run a campaign to update the device firmware.


### Assumptions

You can only update an MBL device if you have:

* A device running an MBL image that can connect to your Pelion Device Management account, and contains all the needed certificates. If you do not have one, please [follow the first tutorial in this series](connecting-an-mbl-device-and-using-an-applications.html).
* An instance of the manifest tool, [as reviewed in our development environment setup](preparing-a-development-environment.html), and initialized (if you have not initialized it yet, please [review the previous tutorial])().<!--This isn't great. Is there a way to add the initalization instructions to the manifest tool installations, or is it too context specific?-->
* A root file system archive that contains the firmware upgrade.<!--How? This is about building a new image, right? And then they need just a bit of it? But how do they get it?-->

<!--Mbed CLI now allows uploading the image, generating the manifest and then uploading the manifest in one giant go. It even starts the campaign. The problem is that it also tries to build the image, which it probably can't do for MBL. Is there a way to use half of it? https://os.mbed.com/docs/v5.10/tools/cli-update.html-->

### Identify the active partition

Before you begin updating your device, you should check which partition is active (the running bank)<!--Why do we keep using two names?-->. This will allow you to make sure your update succeeded, because a successful update changes the active partition.

Run `lsblk` on the device to check which partition is mounted at `/`. That is the active partition. <!--What does the output look like?-->

<!--The rest of the content has not been edited yet-->

#### 12.2. Update Step 2: Upload a firmware image to Mbed Cloud

To upload your firmware update image to the Cloud:

- Log into the [Mbed Cloud Portal](https://portal.mbedcloud.com/login).
- On the **Firmware Update** tab, select **Images>Upload new images**.
- Select the update image on your local hard disk that contains the root file system image for upgrading. For a Warp7 device, for example, the update image will have a filename like this: `mbl-console-image-imx7s-warp-mbl.tar.xz`.
- Provide a name for the firmware image, such as, "test\_image\_20180125\_1", and if required, a description.
- Press the **Upload firmware image** button.
- Copy the firmware image URL. You will need this URL to create the manifest (described in the next section).

#### 12.3. Update Step 3: Create a manifest

To use the manifest-tool to create a manifest for the firmware image:

- Make sure the current working directory is where `manifest-tool init` was performed (`~/mbl/manifests`).<!---->
- Create a symbolic link to the firmware image that was uploaded in [Update Step 2](#update2-2):
  ```
  ln -s ~/mbl/mbl-master/build-mbl/tmp-mbl-glibc/deploy/images/<MACHINE>/mbl-console-image-<MACHINE>.tar.xz test-image
  ```
  where `<MACHINE>` should be replaced with the MACHINE value for your device from the table in [Section 6](#set-up-build-env). This step is required because sometimes `manifest-tool` doesn't cope well with long file names.
- Create a manifest called "test-manifest" by using the following command:
    ```
    manifest-tool create -p test-image -u URL -o test-manifest
    ```
    Where:
    - The **test-image** is the symlink to the firmware image uploaded in [Update Step 2](#update2-2).
    - The **URL** is the firmware image URL copied to the clipboard in the previous section.
    - The **test-manifest** is the name of the output manifest file.



#### 12.4. Update Step 4: Upload the manifest to the Mbed Cloud

To upload the test-manifest to the Mbed Cloud:

- On the Mbed Cloud Portal, from the **Firmware Update** tab, select **Manifests>Upload new manifest**.
- Give the manifest a name, a description (if required) and select the manifest to use.

#### 12.5. Update Step 5: Create a filter for your device

<span class= "notes"> **Note:** the device ID changes each time you flash the device with a new full disk image. The normal firmware update mechanism does not change the device ID.</span>

Before you can configure an update campaign, you need to create a device filter, as follows:

- To get the device ID, you can either:
    - Copy the device ID to the clipboard from the **Device Directory** tab, on Mbed Cloud Portal.
	  - Search for the device ID in the `mbl-cloud-client` log file `/var/log/mbl-cloud-client.log`, using the following command:
    ```
    grep -i 'device id' /var/log/mbl-cloud-client.log
    ```
- On Mbed Cloud Portal, from the **Device Directory** tab, select **Saved filters** and click **Create New Filter**.
- Click **Add attribute** and select **Device ID**.
- Paste the Device ID into the Device ID edit box and click **Save Filter**.

#### 12.6. Update Step 6: Run an update campaign

To create a campaign to update the firmware on a device:

- On the Mbed Cloud Portal, from the **Firmware Update** tab, select **Update campaigns>Create campaign**.
- Provide:
    * A name for the campaign.
    * A description (optional).
- Select the:
    * Manifest file (in this example, `test-manifest`).
    * Device filter created in update step 5.
- Click **Save** to create the new update campaign.

To run your update campaign:

- On the **Firmware update** tab, select **Update campaigns** and find your update campaign from the list.
- Press **Start** to run your test campaign.
    - The firmware update process may take a while to complete.
    - On the device, you can monitor the device console to see the update occurring using `tail -f /var/log/mbl-cloud-client.log` .
    - On the Mbed Cloud Portal, as the campaign progresses, it reports the **Publishing** state.
    - When the firmware update has completed, the Mbed Cloud Portal will report that the campaign is in the **Deployed** state.
- Once the firmware update is complete, the device reboots.
- When the device comes up, login and verify that the running bank (partition) has changed from that noted in update step 1.


#### 12.2.7. Figures

<a name="fig1"></a>
![fig1](pics/01.png "Figure 1")
Figure 1: The mbed cloud portal login.

<a name="fig2"></a>
![fig2](pics/02.png "Figure 2")
Figure 2: Select the team you want to use e.g. arm-mbed-linux.

<a name="fig3"></a>
![fig3](pics/03.png "Figure 3")
Figure 3: Dashboard after you login.

<a name="fig4"></a>
![fig4](pics/04.png "Figure 4")
Figure 4: Firmware update/Images screen.

<a name="fig5"></a>
![fig5](pics/05.png "Figure 5")
Figure 5: Firmware update/Images screen showing how to copy URL.

<a name="fig6"></a>
![fig6](pics/06.png "Figure 6")
Figure 6: Upload firmware image screen.

<a name="fig7"></a>
![fig7](pics/07.png "Figure 7")
Figure 7: Device details screen.

<a name="fig8"></a>
![fig8](pics/08.png "Figure 8")
Figure 8: Device directory screen for viewing device registration status and adding device filters.

<a name="fig9"></a>
![fig9](pics/09.png "Figure 9")
Figure 9: Device directory for adding a filter device id attribute.

<a name="fig10"></a>
![fig10](pics/10.png "Figure 10")
Figure 10: Device directory for adding a filter.

<a name="fig11"></a>
![fig11](pics/11.png "Figure 11")
Figure 11: Saved filters screen.

<a name="fig12"></a>
![fig12](pics/12.png "Figure 12")
Figure 12: Update campaigns screen.

<a name="fig13"></a>
![fig13](pics/13.png "Figure 13")
Figure 13: New update campaign screen.

<a name="fig14"></a>
![fig14](pics/15.png "Figure 14")
Figure 14: Firmware update/Campaign status screen.

<a name="fig15"></a>
![fig15](pics/16.png "Figure 15")
Figure 15: Test campaign details screen.

[mbl-alpha-walkthrough]: https://github.com/ARMmbed/meta-mbl/blob/alpha/docs/walkthrough.md
