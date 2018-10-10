## Supported hardware

Supported boards:

* [NXP Warp7](https://www.nxp.com/support/developer-resources/nxp-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7). You will also need two micro USB cables.
* [Raspberry Pi 3]|(https://www.raspberrypi.org/products/) with a micro SD card and a C232HD-DDHSP-0 cable to connect it to a PC.

    <span class="warning">Raspberry Pi 3 is suitable for development only; do not use it for production.</span>


## Preparing a development environment

<!--Mbed Linux CLI isn't listed here. Is that weird?-->

### For MBL builds

To build and run Mbed Linux, you will need:

* A PC running Ubuntu 16.04.
<!--That's really old. Why are we forcing them into this? And if we are, should we say "if you need a VN, see below"?-->
* A Github account with access to private ARMmbed repositories.
* An SSH agent (required for cloning repositories non-interactively during the build process).
* To install:
    * Some software packages which are required to support Mbed Linux (see [Installing software dependencies](#installing-software-dependencies)).
    * Google's `repo` tool (see [Installing Google's repo tool](#install-google-repo)).
    * Mbed's `manifest-tool` with the Mbed Cloud SDK library (see [Installing the manifest tool](#install-manifest-tool)).


#### Installing software dependencies

The following packages are required for the quick start and subsequent tutorials:<!--are these all Python packages?-->

* bmap-tools
* chrpath.
* curl.
* diffstat.
* gawk.
* git.
* python-dev.
* python-pip.
* texinfo.
* wget.
* whiptail.

The command to install them will look something like this:

```
sudo apt-get install bmap-tools chrpath curl diffstat gawk git python-dev python-pip texinfo wget whiptail
```

#### Installing Google's repo tool

Download Google's [`repo` tool](https://gerrit.googlesource.com/git-repo):

```
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > /tmp/google-repo-tool
```

Then install it to your preferred location. For example, to install to `/usr/local/bin`:

```
sudo install -m 0755 /tmp/google-repo-tool /usr/local/bin/repo
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

#### Using virtual machines

<!--What do we want to say, other than "building will take you hours and hours?"-->

### For application development

<!--There's loads of stuff in the QR thing, but it might change before the release, so I'm not sure about adding it here-->
