## Preparing a development environment

### Accounts and credentials

To build and run Mbed Linux OS (MBL), you need:

* A [Pelion Device Management](https://portal.mbedcloud.com/) account.
* A GitHub account with access to private ARMmbed repositories (supplied by Arm to users enrolled in the preview).

### Software requirements

To build MBL, you need:

* A PC running Ubuntu.

    We tested on Ubunutu 16.04. You can work on any linux-based OS that supports Docker, but you may have to install other packages (such as minicom or an equivalent for your OS).

* Full internet access (because the build process downloads packages from the internet).

* Make sure you can connect to GitHub with SSH, so you can clone private repositories non-interactively during the build process. See [the GitHub documentation about connecting with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/) for more information.

* A few software packages that support building and developing on MBL:

    * `bmap-tools`.
    * `curl`.
    * `git`.
    * `python-pip`.

    Install these packages with:
    ```
    sudo apt-get install bmap-tools curl git python-pip
    ````

* The [`build-mbl` tool, the installation instructions for which are in the mbl-tools repository](https://github.com/ARMmbed/mbl-tools).

    Check out the repository (replace `mbl-XXX` with the latest branch name):<!--can they check out master, or some other way of always knowing they're checking out the latest version rather than them having to go count branches?-->
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

* Docker CE, for the mbl-tools repo script `build-mbl`, to build MBL. [Download and install from the Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

    Add yourself to the Linux group "Docker" to run commands without `sudo`. See [instructions in the Docker Linux documentation](https://docs.docker.com/install/linux/linux-postinstall/).

<span class="tips">**Tip:** We recommend rebooting your PC when you've finished installing everything.</span>

### Using virtual machines

Building open embedded distributions requires the compilation of hundreds of different packages. You need a powerful machine to build in under an hour.

For our own builds, we use:

- Intel Xeon W2145 Processor 8 Core (11 MB Cache, 3.70 GHz)
- 32 GB RAM
- 2 TB hard drive, 7200RPM, 3.5", SATA
- 2.5" 256 GB SATA Solid State Drive
- 256 GB SSD PCIe
