## Preparing a development environment

### Accounts and credentials

You need:
* A [Pelion Device Management](https://portal.mbedcloud.com/) account.
* A GitHub account with access to private Arm Mbed repositories (supplied by Arm to users enrolled in the preview).

### Software requirements

To build MBL, you need:
* A PC running Ubuntu.
    We tested on Ubuntu 16.04. You can work on any Linux-based OS that supports Docker, but you may need to install other packages (such as minicom or equivalent for your OS).

* Full internet access (because the build process downloads packages from the
internet).

* The ability to connect to GitHub with SSH, so you can clone private repositories non-interactively during the build process. See [the GitHub documentation on connecting with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/) for more information.

* A few software packages that support building and developing on MBL:
    * `bmap-tools`.
    * `curl`.
    * `git`.
    * `python-pip`.

Install these packages with:
    ```
    sudo apt-get install bmap-tools curl git python-pip
    ```
* The `build-mbl` tool.

    Check out the relevant branch from the repository (replace `mbl-XXX` with the branch name):
    ```
    $ git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-XXX
    ```
    <span class="tips">**Tip:** The [mbl-tools repository](https://github.com/ARMmbed/mbl-tools) provides a collection of tools and recipes for building and testing MBL.</span>

* Install the Device Management manifest tool. See [Installing the manifest tool](#install-manifest-tool) below:
    ```
    pip install --user -U git+ssh://git@github.com/ARMmbed/manifest-tool.git#egg=manifest-tool
    pip install --user mbed-cloud-sdk
    ```
    Make sure your GitHub SSH key is valid.

See [the manifest tool documentation](https://cloud.mbed.com/docs/latest/updating-firmware/manifest-tool.html) for more information.

* Docker CE, to use `build-mbl` script from the `mbl-tools` repository to build MBL. [Download and install from the Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

Add yourself to the Linux group **Docker** to run commands without `sudo`. See [instructions in the Docker Linux documentation](https://docs.docker.com/install/linux/linux-postinstall/). We recommend using `exec sudo login` to reset the terminal session when you've finished installing everything.

### Using virtual machines

Building open embedded distributions requires the compilation of hundreds of different packages. You need a powerful machine to build in under an hour.

For our own builds, we use:
* Intel Xeon W2145 Processor 8 Core (11 MB Cache, 3.70 GHz).
* 32 GB RAM.
* 2 TB hard drive, 7200RPM, 3.5", SATA.
* 2.5" 256 GB SATA Solid State Drive.
* 256 GB SSD PCIe.
