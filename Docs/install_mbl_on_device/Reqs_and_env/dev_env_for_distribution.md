# Preparing a development environment for the distribution

These requirements are for anyone building an MBL image that can connect to Pelion Device Management.

## Accounts

You need:

* A [Pelion Device Management](https://portal.mbedcloud.com/) account.
* A GitHub account.

## Software requirements

To build MBL, you need:

* A PC running Ubuntu.

    We tested on Ubuntu 16.04. You can work on any Linux-based OS that supports Docker, but you may need to install other packages.

* Full internet access (because the build process downloads packages from the
internet).

* The ability to connect to GitHub with SSH, so you can clone private repositories noninteractively during the build process. See the [GitHub documentation on connecting with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/) for more information.

And the following software:

* A few software packages that support building and developing on MBL:

    * `bmap-tools`.
    * `curl`.
    * `git`.
    * `minicom`.
    * `python-pip`.

    Install these packages with:

    ```
    sudo apt-get install bmap-tools curl git minicom python-pip
    ```

    Optionally, to be able to use Minicom without `sudo`, you can add your user to the `dialout` group by running:

    ```
    sudo usermod -a -G dialout $USER
    ```

    To make this group membership apply to your current shell session, see [the instructions below](#update-process-group-membership).

* The `build-mbl` tool.

    Check out the relevant branch from the repository (in this example, we use `mbl-os-0.8`):

    ```
    $ git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-os-0.8
    ```

    <span class="tips">**Tip:** The [mbl-tools repository](https://github.com/ARMmbed/mbl-tools) provides a collection of tools and recipes for building and testing MBL.</span>

* The Device Management manifest tool and SDK:

    ```
    pip install --user -U git+ssh://git@github.com/ARMmbed/manifest-tool.git#egg=manifest-tool
    pip install --user mbed-cloud-sdk
    ```
    Make sure your GitHub SSH key is valid.

    <span class="notes">**Note**: If you use a [Python virtual environment](https://www.python.org/dev/peps/pep-0405/) instead of performing a system install, you won't need to use the `--user` option.</span>

    See [the manifest tool documentation](https://cloud.mbed.com/docs/latest/updating-firmware/manifest-tool.html) for more information.

* MBL CLI. Please see [installation and network setup instructions](../develop-apps/setting-up.html).

    <span class="notes">**Note:** If you are running an earlier version (1.x) of MBL CLI, please remove it (and install 2.0).</span>

* Docker CE, to use `build-mbl` script from the `mbl-tools` repository to build MBL. [Download and install from the Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

* You must add yourself to the `docker` group to be able to run the Docker build environment. See [instructions in the Docker Linux documentation](https://docs.docker.com/install/linux/linux-postinstall/). To make this group membership apply to your current shell session, see [the instructions below](#update-process-group-membership).

## Using virtual machines

Building open embedded distributions requires the compilation of hundreds of different packages.  Using virtual machines on a laptop to build Mbed Linux OS will take a very long time - it could be 8 hours or more for the first build; you need a powerful machine to build in under an hour.

For our own builds, we use:

* Intel Xeon W2145 Processor 8 Core (11 MB Cache, 3.70 GHz).
* 32 GB RAM.
* 2 TB hard drive, 7200RPM, 3.5", SATA.
* 2.5" 256 GB SATA Solid State Drive.
* 256 GB SSD PCIe.

### USB mass storage

There can be USB pass-through issues from host to the virtual machine.

For example, on **VirtualBox**, the WaRP7 did not come up as a mass storage device after running the u-boot command to sync the USB bridge (detailed below) - it needed an additional reset. The full process to see the mass storage device is:

1. On the WaRP7's console, perform the USB mass storage u-boot command: `ums 0 mmc 0`.
1. In the virtual machine's settings, under **Ports** > **USB**, click the **Add new USB filter** button and select the **FSL USB download gadget**.
1. Reset the WaRP7.
1. Perform the `ums` command again.

<h3 id="update-process-group-membership">Updating your shell process's group memberships</h3>

When you add your user to a new group with the `usermod` command, your current shell process's group membership is not automatically updated. To see the effect of the `usermod` command in your current shell session, you need to log in again. You can do this by running:

```
exec sudo login
```


***

Copyright © 2020 Arm Limited (or its affiliates)
