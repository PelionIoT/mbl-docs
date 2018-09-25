## Requirements and development environment

Does it need text here?

### Hardware

Supported boards:

* [NXP Warp7](https://www.nxp.com/support/developer-resources/nxp-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7). You will also need two micro USB cables.
* [Raspberry Pi 3]|(https://www.raspberrypi.org/products/) with a micro SD card and a C232HD-DDHSP-0 cable to connect it to a PC.

    <span class="warning">Raspberry Pi 3 is suitable for development only; do not use it for production.</span>

## Device Management account

<!--Do they still need to request an account?-->
Request an account using the form at https://cloud.mbed.com/contact. You will need include the following information in the request:

* That you are using Mbed Linux OS.
* That you need an account with a high firmware storage limit, as Mbed Linux firmware packages can be 40-50MiB.

### Preparing a development environment

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

The following packages are required for the quick start and subsequent tutorials:

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

Download Google's [`repo` tool](https://gerrit.googlesource.com/git-repo) with the following command:
```
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > /tmp/google-repo-tool
```
Then install it to your preferred location. To install `repo` to `/usr/local/bin` for example, run the following command:
```
sudo install -m 0755 /tmp/google-repo-tool /usr/local/bin/repo
```

#### Installing the manifest tool

Make sure your GitHub SSH key is valid. See [the GitHub documentation for more information about connecting to GitHub with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/).

Install the Mbed Cloud manifest tool and Cloud SDK with the following commands:

```
pip install --user -U git+ssh://git@github.com/ARMmbed/manifest-tool-restricted.git#egg=manifest-tool
pip install --user mbed-cloud-sdk
```

See [the firmware manifest documentation](https://cloud.mbed.com/docs/current/updating-firmware/firmware-manifests.html) for more information about the manifest tool.

### Using virtual machines

<!--What do we want to say, other than "building will take you hours and hours?"-->
