## Supported hardware

Supported boards:

* [NXP Warp7](https://www.nxp.com/support/developer-resources/nxp-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7). You will also need two micro USB cables.
* [Raspberry Pi3 B and B+](https://www.raspberrypi.org/products/) with a micro SD card and a C232HD-DDHSP-0 cable to connect it to a PC.

    <span class="warning">Raspberry Pi 3 is suitable for development only; do not use it for production.</span>


## Preparing a development environment

<!--Mbed Linux CLI isn't listed here. Is that weird?-->

### For MBL builds

To build and run Mbed Linux, you will need:

* A PC running Ubuntu 16.04.
<!--That's really old. Why are we forcing them into this? And if we are, should we say "if you need a VN, see below"?-->
* A [Pelion](https://portal.mbedcloud.com/) portal account
* A Github account with access to private ARMmbed repositories.
* An SSH agent (required for cloning repositories non-interactively during the build process). Adding SSH key to the ssh-agent is describe in the [following link](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#adding-your-ssh-key-to-the-ssh-agent).
* Full internet access is required as the build process downloads packages from the internet.
* To install:
    * Some software packages which are required to support Mbed Linux (see [Installing software dependencies](#installing-software-dependencies)).
    * Mbed's `manifest-tool` with the Mbed Cloud SDK library (see [Installing the manifest tool](#install-manifest-tool)).
    * Docker CE (see [Installing Docker CE](#install-docker-ce)).
* Reboot PC after you finish all installation is recommended.


#### Installing software dependencies

The following packages are required for the quick start and subsequent tutorials:<!--are these all Python packages?-->

* curl.
* git.
* python-pip.

The command to install them will look something like this:

```
sudo apt-get install bmap-tools chrpath curl git python-dev python-pip
```

#### Installing the manifest tool

Make sure your GitHub SSH key is valid. See [the GitHub documentation for more information about connecting to GitHub with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/).

Install the Mbed Cloud manifest tool and Pelion SDK:

```
pip install --user -U git+ssh://git@github.com/ARMmbed/manifest-tool-restricted.git#egg=manifest-tool
pip install --user mbed-cloud-sdk
```

See [the firmware manifest documentation](https://cloud.mbed.com/docs/latest/updating-firmware/firmware-manifests.html) for more information about the manifest tool.

<!--Can we add initalization instructions here, so that I don't have to send people to the first tutorial every time I remind them this needs initializing?-->

#### Installing Docker CE
Docker is needed in order to be able to use mbl-tools build-mbl script (and to be able to develop and build apps on your x86 PC).
Download and install latest version of [Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

##### Manage Docker as a non-root user
You will need to add yourself to the docker group to run docker commands without sudo. See the following [instructions](https://docs.docker.com/install/linux/linux-postinstall/).

#### Using virtual machines

<!--What do we want to say, other than "building will take you hours and hours?"-->

### For application development

<!--There's loads of stuff in the QR thing, but it might change before the release, so I'm not sure about adding it here-->
