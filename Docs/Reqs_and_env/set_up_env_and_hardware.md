## Supported hardware

Supported boards:

* [NXP Warp7](https://www.nxp.com/support/developer-resources/nxp-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7). You will also need two micro USB cables.
* [Raspberry Pi3 model B and B+](https://www.raspberrypi.org/products/) with a micro SD card and a C232HD-DDHSP-0 cable to connect it to a PC.

    <span class="warning">Raspberry Pi 3 is suitable for development only; do not use it for production.</span>

### Preparing a development environment

<!--Mbed Linux CLI isn't listed here. Is that weird?-->

To build and run Mbed Linux, you will need:

* A PC running Ubuntu 16.04.
<!--That's really old. Why are we forcing them into this? And if we are, should we say "if you need a VN, see below"?-->
* A [Pelion](https://portal.mbedcloud.com/) portal account
* A GitHub account with access to private ARMmbed repositories.<!--how do they get that?-->
* An SSH agent (for cloning repositories non-interactively during the build process). See the GitHub documentation for [information about adding an SSH key to the agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#adding-your-ssh-key-to-the-ssh-agent).
* Full internet access (for the build process, which downloads packages from the internet).

### Software requirements

You will also need to install:

* A few software packages that support Mbed Linux:

    * bmap-tools.
    * curl.
    * git.
    * python-pip.

    The command to install them will look something like this:

    ```
    sudo apt-get install bmap-tools curl git python-pip
    ````

* The Pelion manifest tool, with the Mbed Cloud SDK library (see [Installing the manifest tool](#install-manifest-tool) below):<!--what do you mean "with"?-->

    ```
    pip install --user -U git+ssh://git@github.com/ARMmbed/manifest-tool-restricted.git#egg=manifest-tool
    pip install --user mbed-cloud-sdk
    ```

    Make sure your GitHub SSH key is valid. See [the GitHub documentation for more information about connecting to GitHub with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/).

    See [the firmware manifest documentation](https://cloud.mbed.com/docs/latest/updating-firmware/firmware-manifests.html) for more information about the manifest tool.

* Docker CE, for the `mbl-tools` script `build-mbl`, and to be able to develop and builds apps on a PC. [Download and install from the Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

    You will need to add yourself to the docker group to run docker commands without sudo. See [instructions in the Docker Linux documentation](https://docs.docker.com/install/linux/linux-postinstall/).

<span class="tips">We recommend rebooting your PC when you've finished installing everything.</span>


#### Using virtual machines

<!--What do we want to say, other than "building will take you hours and hours?"-->
