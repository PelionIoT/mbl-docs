# Mbed Linux OS distribution development with mbl-tools

The Mbed Linux OS (MBL) distribution is based on OpenEmbedded and Yocto; it supports all of the development workflows listed in the [Yocto Manuals](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html). MBL distribution development relies on the [`mbl-tools` repository](https://github.com/ARMmbed/mbl-tools/), and in particular `build-mbl`, to build, develop and test the distribution.

You can use the Docker build environment's shell (interactive mode, [explained below](#running-build-mbl-in-interactive-mode)) to perform incremental builds, use [devtool](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#ref-devtool-reference), or just call `bitbake [options] <target>`.

## Requirements for mbl-tools

Install [all software requirements for a development environment](../first-image/development-environment.html).

## Building MBL using a custom repo manifest

You can use a custom repo manifest with `mbl-tools` to specify source revisions or use your own MBL forks. You can create a new manifest, or fork [`mbl-manifest`](https://github.com/ARMmbed/mbl-manifest).

To point to a custom manifest, you need to pass at least the options `--branch` and `--url`. If the manifest filename is not `default.xml`, use the `--manifest` option to pass the correct name.

For example (change parameters for `--builddir` and `--machine` as needed):

```
./mbl-tools/build/run-me.sh --builddir ./build-warp7 -- --machine imx7s-warp-mbl --branch mbl-os-0.8 --url git@github.com:your-github-account/mbl-manifest.git
```

<span class="notes">**Notes**: The `--url` must be in [the **Clone with SSH** format](https://git-scm.com/docs/git-clone#_git_urls_a_id_urls_a). We don't recommend using the `--outputdir` option with interactive mode, as any builds done during interactive mode will not update the output directory contents.</span>

## Repository access problems

If during a build you have a problem accessing a source code repository by SSH, you may see an output like:

```
Cloning into bare repository '/home/user/build/ssh.dev.azure.com.v3.myorg.myproject.myproject.git'...
ssh: Could not resolve hostname ssh.dev.azure.com:v3: Name or service not known
fatal: Could not read from remote repository.

Please make sure you have the correct access rights and the repository exists.

ERROR: myrecipe do_fetch: Fetcher failure for URL: 'git://git@ssh.dev.azure.com:v3/myorg/myproject/myproject.git;protocol=ssh;nobranch=1'. Unable to fetch URL from any source.
```

To fix this, use the build option `--repo-host HOSTNAME` to allow access to the source code repository host in the build environment. You can specify this option multiple times if you have multiple hosts.

For example:

```
mbl-tools/build/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch master --machine imx7s-warp-mbl --repo-host ssh.dev.azure.com --repo-host gitlab.com
```

## Running build-mbl in interactive mode

<span class="tips">This section only covers building a distribution; the Docker build environment was not designed for editing files (since it doesn't include a file editor such as `vim`). If you want to inspect or edit files, do so in your viewer/editor of choice before launching interactive mode. The BitBake layers are located at: `<builddir>/<mbl-machine-name>/mbl-manifest/layers` and the build directory at: `<builddir>/<mbl-machine-name>/mbl-manifest/build-mbl-development`.</span>

You need to run the **build-mbl tool** in interactive mode: an interactive shell inside the Docker build environment with the BitBake environment setup. This allows running commands for BitBake and its associated tools (such as bitbake-layers, devtool and receipetool).


1. To run build-mbl in interactive mode (change parameters for `--builddir` and `--machine` as needed):

    ```
    ./mbl-tools/build/run-me.sh --builddir ./build-warp7 -- --branch mbl-os-0.8 --machine imx7s-warp-mbl interactive
    ```

    <span class="tips">Unlike the build mode, the interactive mode only supports one `--machine` option at a time.</span>

1. The interactive shell prompt should be similar to:

    ```
    user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development$
    ```

To exit the interactive mode, press <kbd>Ctrl</kbd>+<kbd>D</kbd> or type `exit` and press <kbd>Enter</kbd>.

<span class="notes">**Note**: To perform a full build after exiting interactive mode, you must provide the `build` stage as follows (change parameters for `--builddir` and `--machine` as needed):</span>

```
./mbl-tools/build/run-me.sh --builddir ./build-warp7 -- --branch mbl-os-0.8 --machine imx7s-warp-mbl build
```

## Examples of interactive mode commands

### bitbake-layers

Using `$ bitbake-layers show-appends linux-fslc` to view the recipe appends for the linux-fslc metalayer:

```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development$ bitbake-layers show-appends linux-fslc
NOTE: Starting bitbake server...
Loading cache: 100% |########################################################################################################################################################################| Time: 0:00:00
Loaded 3438 entries from dependency cache.
=== Matched appended recipes ===
linux-fslc_4.20.bb:
  <builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development/conf/../../layers/meta-freescale-3rdparty/recipes-kernel/linux/linux-fslc_%.bbappend
  <builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development/conf/../../layers/meta-mbl/recipes-kernel/linux/linux-fslc_%.bbappend
  ```

### Changing the kernel configuration

Using `$ bitbake virtual/kernel -c menuconfig` to launch the Linux kernel's menuconfig command:

```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development$ bitbake virtual/kernel -c menuconfig
Loading cache: 100% |########################################################################################################################################################################| Time: 0:00:00
Loaded 3438 entries from dependency cache.
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.40.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "ubuntu-16.04"
TARGET_SYS           = "arm-oe-linux-gnueabi"
MACHINE              = "imx7s-warp-mbl"
DISTRO               = "mbl"
DISTRO_VERSION       = "0.0"
TUNE_FEATURES        = "arm armv7ve vfp thumb neon callconvention-hard"
TARGET_FPU           = "hard"
meta-mbl             = "HEAD:8fab5fe8f7053e4e928b61b104b20d1985a7b53e"
meta-filesystems     
meta-networking      
meta-oe              
meta-python          = "HEAD:f352612e772ec19927278b62da153b3164ba08e8"
meta-virtualization  = "HEAD:080f6b412da706e2c2a3c65fd06a05ac1c81880d"
meta-freescale       = "HEAD:4ee590743c3c897724c5add273ab04518fece770"
meta-freescale-3rdparty = "HEAD:3b60f3e31a9aba22bd635d7f0d34fd2304296bdc"
meta-raspberrypi     = "HEAD:7cfb0e892822f339e8a0c1f0207426cc388daa23"
meta-optee           = "HEAD:7e7f09b731ac8ec09cc87ed058908c8479b8c7ab"
meta                 = "HEAD:ad246f5ce0652bd917d85884176baa746e1379ff"

Initialising tasks: 100% |###################################################################################################################################################################| Time: 0:00:00
Sstate summary: Wanted 0 Found 0 Missed 0 Current 42 (0% match, 100% complete)
NOTE: Executing SetScene Tasks
NOTE: Executing RunQueue Tasks
Currently  1 running tasks (338 of 338)  99% |############################################################################################################################################################ |
0: linux-fslc-4.20+gitAUTOINC+73bb08073e-r0 do_menuconfig - 0s (pid 134)
NOTE: Tasks Summary: Attempted 338 tasks of which 337 didn't need to be rerun and all succeeded.
```

You should see something like:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/dev_interactive_shell_kernel_menuconfig.png)</span>

After exiting the kernel configuration screen, you can rebuild the Linux kernel with the `bitbake virtual/kernel` command:

`user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development$ bitbake virtual/kernel`

### Rebuilding an image

You can call `bitbake <image-recipe-name>` to generate the image:

```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development$ bitbake mbl-image-development
Loading cache: 100% |########################################################################################################################################################################| Time: 0:00:00
Loaded 3438 entries from dependency cache.
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.40.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "ubuntu-16.04"
TARGET_SYS           = "arm-oe-linux-gnueabi"
MACHINE              = "imx7s-warp-mbl"
DISTRO               = "mbl"
DISTRO_VERSION       = "0.0"
TUNE_FEATURES        = "arm armv7ve vfp thumb neon callconvention-hard"
TARGET_FPU           = "hard"
meta-mbl             = "HEAD:8fab5fe8f7053e4e928b61b104b20d1985a7b53e"
meta-filesystems     
meta-networking      
meta-oe              
meta-python          = "HEAD:f352612e772ec19927278b62da153b3164ba08e8"
meta-virtualization  = "HEAD:080f6b412da706e2c2a3c65fd06a05ac1c81880d"
meta-freescale       = "HEAD:4ee590743c3c897724c5add273ab04518fece770"
meta-freescale-3rdparty = "HEAD:3b60f3e31a9aba22bd635d7f0d34fd2304296bdc"
meta-raspberrypi     = "HEAD:7cfb0e892822f339e8a0c1f0207426cc388daa23"
meta-optee           = "HEAD:7e7f09b731ac8ec09cc87ed058908c8479b8c7ab"
meta                 = "HEAD:ad246f5ce0652bd917d85884176baa746e1379ff"

Initialising tasks: 100% |###################################################################################################################################################################| Time: 0:00:03
Sstate summary: Wanted 251 Found 248 Missed 3 Current 903 (98% match, 99% complete)
NOTE: Executing SetScene Tasks
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 4038 tasks of which 3672 didn't need to be rerun and all succeeded.
NOTE: Writing buildhistory
```

The images are placed in:

```
<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl-development/tmp/deploy/images/imx7s-warp-mbl/
```

### Modifying the source of an existing component

To modify the source of an existing component built by OpenEmbedded/Yocto, follow the [Use `devtool modify` to Modify the Source of an Existing Component](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#sdk-devtool-use-devtool-modify-to-modify-the-source-of-an-existing-component) section on the Yocto website. It works inside the interactive mode.

### Updating the meta layers sources

The **build-mbl tool** generates a pinned version of the specified manifest and uses this to perform a build of MBL. This means you cannot perform a `repo sync` command in the build directory to update all the layers sources. Instead, outside interactive mode, issue the **build-mbl tool** with the build stage `sync`. This syncs the repo, produces a new pinned manifest, then performs a full build with that manifest.

For example (change parameters for `--builddir` and `--machine` as needed):

```
./mbl-tools/build/run-me.sh --builddir ./build-warp7 -- --branch mbl-os-0.8 --machine imx7s-warp-mbl sync
```
