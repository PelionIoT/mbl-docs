# Using the build scripts

The `run-me.sh` and `build.sh` scripts are called in a single command, with options that control what to build and how. The general form of a `run-me.sh` invocation is:

```
./mbl-tools/build/run-me.sh [RUN-ME.SH OPTIONS]... -- [BUILD.SH OPTIONS]... [BUILD.SH STAGE]
```

<span class="tips">Note the use of `--`. It separates options for `run-me.sh` from options for `build.sh`.</span>


## Mandatory build flags

The following build options are mandatory:

| Name | Option Type | Information |
| --- | --- | --- |
| `--branch` | build.sh | Select the MBL branch to build. For example, to build the branch `mbl-os-0.8`: <br>`./mbl-tools/build/run-me.sh -- --branch mbl-os-0.8 --machine raspberrypi3-mbl` |
| `--builddir` | run-me.sh | Create a build directory. This option is for `run-me.sh`. <br>You must use a different build directory for every device (machine), and we recommend including the device's name in the directory's name. <br>Note that this directory includes all other artifacts, such as build and error logs. For example, if you've created `mkdir /path/to/my-build-dir`, the builddir will be `./mbl-tools/build/run-me.sh --builddir /path/to/my-build-dir` |
| `--machine` | build.sh | Select the target device. <br>The options are [**PICO-PI with IMX7D**, `imx7d-pico-mbl`], [**NXP 8M Mini EVK**, `imx8mmevk-mbl`], [**PICO-PI with IMX6UL**, `imx6ul-pico-mbl`], [**Warp7**, `imx7s-warp-mbl`] and [**Raspberry Pi 3**, `raspberrypi3-mbl`]. <br>Example: `./mbl-tools/build/run-me.sh -- --machine <MACHINE>` |
| `--outputdir` | run-me.sh | Specify the output directory for all build artifacts (pinned manifest, target specific images etc). <br>For example, if you've created `mkdir /path/to/artifacts`, the outputdir will be `./mbl-tools/build/run-me.sh --outputdir /path/to/artifacts` |
| `--root-passwd-file` | run-me.sh | The file containing the root user password in plain text (**mandatory** when `--distro mbl-production` ). |

An example using all mandatory options:

```
./mbl-tools/build/run-me.sh --builddir /path/to/builddir --outputdir /path/to/artifacts -- --branch mbl-os-0.8 --machine <MACHINE>
```

## Optional build flags

The following build options are not mandatory, but you may find that they improve the development process:

| Name | Option Type | Information |
| --- | --- | --- |
| `--distro` | build.sh | Choose which distribution to build. The supported values are: **`mbl-development`** (default) or **`mbl-production`** |
| `--downloaddir` | run-me.sh | Cache downloaded artifacts between successive builds (do not use cacheing for parallel builds). <br>For example, if you create `mkdir /path/to/downloads`, the downloaddir will be `./mbl-tools/build/run-me.sh --downloaddir /path/to/downloads` |
| `--external-manifest` | run-me.sh | You can build using a pinned manifest, which is an encapsulation created by a build and containing enough information to allow an exact rebuild. The manifest is created in your output directory (`outputdir`). <br>To use it to rebuild, run `./mbl-tools/build/run-me.sh --external-manifest /path/to/pinned-manifest.xml` |
| `--image` | build.sh | Choose which image to build. The supported values are: **`mbl-image-development`** (default when no `--distro` is passed or for `--distro mbl-development` ) or **`mbl-image-production`** (default for `--distro mbl-production` ). |
| `--manifest` | build.sh | By default, building uses the `default.xml` manifest, which uses release branches of all the Arm maintained repositories. To use pinned versions for all repositories, specify `release.xml` as the manifest. You can combine this with `--branch` to specify a particular release version. <br>Example: `./mbl-tools/build/run-me.sh -- --branch refs/tags/mbl-os-0.8.0 --manifest release.xml`|
| `--root-passwd-file` | run-me.sh | The file containing the root user password in plain text (**optional** when `--distro mbl-development` ). |
| `--boot-rot-key` | run-me.sh | The private signing key used in preparing the bootloader update components. |
| `--kernel-rot-key` | run-me.sh | The private signing key used in preparing the bootloader and kernel update components. |
| `--kernel-rot-crt` | run-me.sh | The public certificate used in preparing the bootloader and kernel update components. |
| `--save-config` | run-me.sh | Saves the set of build options you have specified on the command line to a configuration file and exits. See [build configurations](#build-configurations) for more details. |
| `--load-config` | run-me.sh | Loads a saved set of build options from a configuration file. See [build configurations](#build-configurations) for more details. |

## Optional build stage

Optionally, specify the `[BUILD.SH STAGE]` after all other build options. The following table shows the main build stages that can be added to the build line. Stages `start`, `sync` and `build` will occur in that order, whereas `interactive` and `clean` are optional stages that are only performed when you invoke them.

<span class="tips">Don't specify a build stage when you perform the first build.</span>

| Name | Information |
| --- | --- |
| `start` | Start from the beginning of the build process. This is the first stage for a new build by default. |
| `sync` | [Update to the latest meta layer sources](../develop-mbl/mbed-linux-os-distribution-development-with-mbl-tools.html#examples-of-interactive-mode-commands) and then perform a build. |
| `build` | Perform a build of the existing meta layer sources. Use this to switch back from `interactive` mode. |
| `interactive` | Enter interactive mode, which allows you to [invoke BitBake commands](../develop-mbl/mbed-linux-os-distribution-development-with-mbl-tools.html#running-build-mbl-in-interactive-mode) and [create update payloads](../update/updating-an-mbl-image.html#prepare-the-update-payload). |
| `clean` | Delete the working build tree and start the building process again. |

## Build outputs

The build process creates the following files:

| File | Path | File | Information |
| --- | --- | --- | --- |
| Full disk image  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/` or <br> `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-production/images/`| `mbl-image-development-<MACHINE>.wic.gz` or <br> `mbl-image-production-<MACHINE>.wic.gz`| A compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware. |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/` or <br> `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-production/images/` | `mbl-image-development-<MACHINE>.wic.bmap` or <br> `mbl-image-production-<MACHINE>.wic.bmap`| A file containing information about which blocks of the uncompressed full disk image actually need to be written to the device. Some blocks of the image represent unused storage space, which does not actually need to be written. |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/` or <br> `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-production/images/` | `mbl-image-development-<MACHINE>.tar.xz` or <br> `mbl-image-production-<MACHINE>.tar.xz` | A compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the Updating MBL tutorial](../update/index.html)). |

## Build configurations

You can save the build options you supplied on the command line, so that it will be easier to run them again:

1. Perform a successful build invoking `run-me.sh` with your chosen options.
1. Invoke the same build again, but add `--save-config <CONFIG_FILE>` to the command line. This saves a list of all your build options into the `<CONFIG_FILE>`.
1. Use `run-me.sh --load-config <CONFIG_FILE>` to invoke a build with exactly the same options as the first build.

<span class="tips">**Tip:** When you need to perform builds for different machines or distro types, we recommend using different configuration files and specifying different build and output directories.</span>

### Saving a build configuration

Set up your build options and invoke a build using `run-me.sh`. For example, here's a production build on NXP 8M Mini EVK:

```
./mbl-tools/build/run-me.sh --builddir /path/to/builddir --outputdir /path/to/artifacts --root-passwd-file /path/to/root_passwd -- --branch mbl-os-0.9 --machine imx8mmevk-mbl --distro mbl-production
```

When you have a successful build, save the configuration to a file (`mbl-imx8-prod.cfg`), using the option `--save-config` added to the build options. For example:

```
./mbl-tools/build/run-me.sh --save-config mbl-imx8-prod.cfg --builddir /path/to/builddir --outputdir /path/to/artifacts --root-passwd-file /path/to/root_passwd -- --branch mbl-os-0.9 --machine imx8mmevk-mbl --distro mbl-production 
```

<span class="tips">**Tip:** We recommend not including a build stage (such as `interactive`) on your command line when you save it. Instead, manually specify it for each build.</span>

### Loading a build configuration

To perform a build with the saved configuration file, continuing the example above:

```
./mbl-tools/build/run-me.sh --load-config mbl-imx8-prod.cfg
```

You can include other options on the build line. For example, to invoke [interactive mode](../develop-mbl/mbed-linux-os-distribution-development-with-mbl-tools.html#running-build-mbl-in-interactive-mode):

```
./mbl-tools/build/run-me.sh --load-config mbl-imx8-prod.cfg -- interactive
```

<span class="tips">**Tip:** Note the use of `--`. It separates options for `run-me.sh` from options for `build.sh`.</span>

Extra options given on the command line will override those specified in the configuration file, except:

* Options that can be specified more than once (such as `--machine`) are concatenated.
* Build stages saved in the configuration file will be excuted before any stages specified on the command line (which is why we recommend not saving build stages in the configuration file).

We recommend using different configuration files to avoid these situations.

You can also load a configuration, add some additional arguments on the command line and then save it to a new configuration file. For example, to use a specific tagged release on the mbl-os-0.9 branch:

```
./mbl-tools/build/run-me.sh --load-config mbl-imx8-prod.cfg --save-config mbl-os-0.9.0-imx8-prod.cfg -- --manifest release.xml --branch refs/tags/mbl-os-0.9.0
```

This will create a `mbl-os-0.9.0-imx8-prod.cfg` file that contains all the build options from `mbl-imx8-prod.cfg`, as well as the `--manifest` option and a second `--branch` option. The branch option in this case will override the first branch option of `mbl-os-0.9`; for other options, the same exceptions stated above still apply.
