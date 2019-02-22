# Building an MBL image

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

Please note that each release has its own branch. Throughout this guide, the release branch is assumed to be `mbl-os-0.5`.

## Building scripts

<span class="notes">**Note**: You need to use an SSH agent to build. For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).</span>


If you haven't already done so, check out the relevant branch from the `build-mbl` repository (in this example, we use `mbl-os-0.5`):

```
$ git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-os-0.5
```

The repository includes the `run-me.sh` script, which:

1. Creates and launches a Docker container that encapsulates the MBL build environment.
1. Launches a build script, `build.sh`, inside the container.

The `run-me.sh` and `build.sh` scripts are called in a single command, with options that control what to build and how. The general form of a `run-me.sh` invocation is:

```
./mbl-tools/build-mbl/run-me.sh [RUN-ME.SH OPTIONS]... -- [BUILD.SH OPTIONS]...
```

<span class="tips">Note the use of `--`. It separates options for `run-me.sh` from options for `build.sh`.</span>

To invoke the `run-me.sh` help menu, use:

```
./mbl-tools/build-mbl/run-me.sh -h
```

To invoke the `build.sh` help menu, use:

```
./mbl-tools/build-mbl/run-me.sh -- -h
```

The following build options are mandatory:

| Name | Information |
| --- | --- |
| `--branch` | Select the MBL branch to build. For example, to build the branch `mbl-os-0.5`: `./mbl-tools/build-mbl/run-me.sh -- --branch mbl-os-0.5 --machine raspberrypi3-mbl` |
| `--machine` | Select the target device. The options are [ PICO-PI with IMX7D, `imx7d-pico-mbl`], [ NXP 8M Mini EVK, `imx8mmevk-mbl`], [ Warp7, `imx7s-warp-mbl`] and [ Raspberry Pi 3, `raspberrypi3-mbl`]. Example: `./mbl-tools/build-mbl/run-me.sh -- --machine <MACHINE>` |
| `--builddir` | Create a build directory. This option is for `run-me.sh`. You must use a different build directory for every device (machine), and we recommend including the device's name in the directory's name. Note that this directory includes all other artifacts, such as build and error logs. For example, if you've created `mkdir /path/to/my-build-dir`, the builddir will be `./mbl-tools/build-mbl/run-me.sh --builddir /path/to/my-build-dir` |
| `--outputdir` | Specify the output directory for all build artifacts (pinned manifest, target specific images etc). For example, if you're created `mkdir /path/to/artifacts`, the outpudir will be `./mbl-tools/build-mbl/run-me.sh --outputdir /path/to/artifacts` |
| `--inject-mcc` | At the moment, you need to build your Device Management resources (that you obtained above) into the image. `./mbl-tools/build-mbl/run-me.sh --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c` |

An example using all mandatory options:

```
./mbl-tools/build-mbl/run-me.sh --builddir /path/to/builddir --inject-mcc /path/to/mbed_cloud_dev_credentials.c --inject-mcc /path/to/update_default_resources.c --outputdir /path/to/artifacts -- --branch mbl-os-0.5 --machine <MACHINE>
```

The following build options are not mandatory, but you may find that they improve the development process:

| Name | Information |
| --- | --- |
| `--downloaddir` | Cache downloaded artifacts between successive builds (do not use cacheing for parallel builds). For example, if you've created `mkdir /path/to/downloads`, the downloaddir will be `./mbl-tools/build-mbl/run-me.sh --downloaddir /path/to/downloads` |
| `--external-manifest` | You can build using a pinned manifest, which is an encapsulation created by a build and containing enough information to allow an exact rebuild. The manifest is created in your output directory (`outputdir`). To use it to rebuild, run `./mbl-tools/build-mbl/run-me.sh --external-manifest /path/to/pinned-manifest.xml` |

## Build examples

The following examples assume:

* You have your [Device Management developer credentials](../getting-started/preparing-device-management-sources.html#downloading-device-management-developer-credentials).
* You have your [update resources file](../getting-started/preparing-device-management-sources.html#creating-an-update-resources-file).
* You have an output directory for your machine, for example `./artifacts-warp7` or `./artifacts-rpi3`. <!-- JIJ: Could expand these with examples for all platforms -->
* You have an [SSH agent](../getting-started/development-environment.html). For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).


<!-- JIJ: Could expand below with examples for all platforms -->

### Warp7 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --builddir ./build-warp7 --outputdir ./artifacts-warp7 -- --machine imx7s-warp-mbl --branch mbl-os-0.5
```

### Raspberry Pi 3 device

```
./mbl-tools/build-mbl/run-me.sh --inject-mcc ./cloud-credentials/mbed_cloud_dev_credentials.c --inject-mcc ./update-resources/update_default_resources.c --builddir ./build-rpi3 --outputdir ./artifacts-rpi3 -- --machine raspberrypi3-mbl --branch mbl-os-0.5
```

## Building outputs

The build process creates the following files:

| File | Path | Information |
| --- | --- | --- |
| Full disk image  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.wic.gz` | This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware. |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.wic.bmap` | This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the device. Some blocks of the image represent unused storage space, which does not actually need to be written. |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.tar.xz` | This is a compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the Updating MBL tutorial](../update/index.html)). |

The process also creates release images, which don't contain packages for testing and debugging (it removes packages such as OP-TEE test suite, Dropbear to support SSH during development and test, and the strace utility).

| File | Path |
| --- | --- |
| Full test disk image | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-production/images/mbl-image-production-<MACHINE>.wic.gz` |
| Full test disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-production/images/mbl-image-production-<MACHINE>.wic.bmap` |
| Root file system archive | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-production/images/mbl-image-production-<MACHINE>.tar.xz` |
