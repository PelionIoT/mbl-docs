# Using the build scripts

The `run-me.sh` and `build.sh` scripts are called in a single command, with options that control what to build and how. The general form of a `run-me.sh` invocation is:

```
./mbl-tools/build-mbl/run-me.sh [RUN-ME.SH OPTIONS]... -- [BUILD.SH OPTIONS]...
```

<span class="tips">Note the use of `--`. It separates options for `run-me.sh` from options for `build.sh`.</span>


## Mandatory build flags

The following build options are mandatory:

| Name | Option Type | Information |
| --- | --- | --- |
| `--branch` | build.sh | Select the MBL branch to build. For example, to build the branch `mbl-os-0.6`: <br>`./mbl-tools/build-mbl/run-me.sh -- --branch mbl-os-0.6 --machine raspberrypi3-mbl` |
| `--machine` | build.sh | Select the target device. <br>The options are [**PICO-PI with IMX7D**, `imx7d-pico-mbl`], [**NXP 8M Mini EVK**, `imx8mmevk-mbl`], [**Warp7**, `imx7s-warp-mbl`] and [**Raspberry Pi 3**, `raspberrypi3-mbl`]. <br>Example: `./mbl-tools/build-mbl/run-me.sh -- --machine <MACHINE>` |
| `--builddir` | run-me.sh | Create a build directory. This option is for `run-me.sh`. <br>You must use a different build directory for every device (machine), and we recommend including the device's name in the directory's name. <br>Note that this directory includes all other artifacts, such as build and error logs. For example, if you've created `mkdir /path/to/my-build-dir`, the builddir will be `./mbl-tools/build-mbl/run-me.sh --builddir /path/to/my-build-dir` |
| `--outputdir` | run-me.sh | Specify the output directory for all build artifacts (pinned manifest, target specific images etc). <br>For example, if you're created `mkdir /path/to/artifacts`, the outpudir will be `./mbl-tools/build-mbl/run-me.sh --outputdir /path/to/artifacts` |

An example using all mandatory options:

```
./mbl-tools/build-mbl/run-me.sh --builddir /path/to/builddir --outputdir /path/to/artifacts -- --branch mbl-os-0.6 --machine <MACHINE>
```

## Optional build flags

The following build options are not mandatory, but you may find that they improve the development process:

| Name | Option Type | Information |
| --- | --- | --- |
| `--downloaddir` | run-me.sh | Cache downloaded artifacts between successive builds (do not use cacheing for parallel builds). <br>For example, if you create `mkdir /path/to/downloads`, the downloaddir will be `./mbl-tools/build-mbl/run-me.sh --downloaddir /path/to/downloads` |
| `--manifest` | build.sh | By default, building uses the `default.xml`, which uses branches of all the Arm maintained repositories. To use pinned versions for all repositories, specify `release.xml` as the manifest. You can combine this with `--branch refs/tags/mbl-os-0.6.0` to specify a particular release version. <br>Example: `./mbl-tools/build-mbl/run-me.sh -- --branch refs/tags/mbl-os-0.6.0 --manifest release.xml`|
| `--external-manifest` | run-me.sh | You can build using a pinned manifest, which is an encapsulation created by a build and containing enough information to allow an exact rebuild. The manifest is created in your output directory (`outputdir`). <br>To use it to rebuild, run `./mbl-tools/build-mbl/run-me.sh --external-manifest /path/to/pinned-manifest.xml` |

## Build outputs

The build process creates the following files:

| File | Path | File | Information |
| --- | --- | --- | --- |
| Full disk image  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/` | `mbl-image-development-<MACHINE>.wic.gz` | This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware. |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/` | `mbl-image-development-<MACHINE>.wic.bmap` | This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the device. Some blocks of the image represent unused storage space, which does not actually need to be written. |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/` | `mbl-image-development-<MACHINE>.tar.xz` | This is a compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the Updating MBL tutorial](../update/index.html)). |
