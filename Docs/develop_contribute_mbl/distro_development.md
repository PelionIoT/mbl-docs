# Mbed Linux OS Distribution Development

<span class="notes">**Note**: Make sure you have followed [Preparing a development environment for the distribution - Software requirements](https://os.mbed.com/docs/mbed-linux-os/master/first-image/development-environment.html) instructions.</span>

The [mbl-tools](https://github.com/ARMmbed/mbl-tools/) is a set of tools to build, develop and test the Mbed Linux OS Distribution which is based on Openembedded/Yocto. All development workflows listed in the [Yocto Manuals](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html) are supported.

In order to be able to perform incremental builds, use [devtool](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#ref-devtool-reference), or just call `bitbake [options] <target>`, you need to run the **mbl-tool** build in *interactive mode*.The interactive mode consists of an interactive shell inside the Docker build environment with the bitbake environment setup, and to be launched you should have run a [complete build](https://os.mbed.com/docs/mbed-linux-os/master/first-image/building-a-developer-image.html) at least once. Unlike the build mode, the interactive mode only supports one `--machine` option at a time.

## Running mbl-tools build interactive mode

Issue the follwing command and change the appropriate options for `--builddir`, `--outputdir` and `--machine`:

`./mbl-tools/build-mbl/run-me.sh --builddir ./build-warp7 --outputdir ./artifacts-warp7 -- --branch mbl-os-0.6 --machine imx7s-warp-mbl interactive`

<span class="notes">**Note**: If you haven't run a complete build before you will be propmpted with the following message:

```
error: '<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl' path missing.
Please run a complete build for 'imx7s-warp-mbl' machine before using the interactive mode.
Would you like to run the complete build now? (Y/N):
```
</span>


The interactive shell should looks like the following:

`user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl$`

And now you are able to run bitbake commads and its associated tools (e.g: bitbake-layers, devtool, recipetool etc). To exit from it you can press `Ctrl+D` or enter `exit`.



### Examples

1. bitbake-layers (`$ bitbake-layers show-appends linux-fslc`):

```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl$ bitbake-layers show-appends linux-fslc
NOTE: Starting bitbake server...
Loading cache: 100% |########################################################################################################################################################################| Time: 0:00:00
Loaded 3438 entries from dependency cache.
=== Matched appended recipes ===
linux-fslc_4.20.bb:
  <builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl/conf/../../layers/meta-freescale-3rdparty/recipes-kernel/linux/linux-fslc_%.bbappend
  <builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl/conf/../../layers/meta-mbl/recipes-kernel/linux/linux-fslc_%.bbappend
  ```

2. Changing the kernel configuration (`$ bitbake virtual/kernel -c menuconfig`):


```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl$ bitbake virtual/kernel -c menuconfig
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

You should be able to visualize a screen like the following:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/dev_interactive_shell_kernel_menuconfig.png)</span>

After exiting from the menuconfig you can rebuild the linux kernel with `bitbake virtual/kernel` command:

`user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl$ bitbake virtual/kernel`


4. Rebuild an image

You can call `bitbake image-recipe-name` to generate the image:

```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl$ bitbake mbl-image-development
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

<span class="notes">**Note**: The images are placed in:
  
``` 
<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl/tmp-mbl-glibc/deploy/images/imx7s-warp-mbl/
```
 </span>


5.  Modify the source of an existing component built by Openembedded/Yocto

The [Use devtool modify to Modify the Source of an Existing Component](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#sdk-devtool-use-devtool-modify-to-modify-the-source-of-an-existing-component) section works inside the interactive mode.


6. Update the meta layers sources

Using `repo sync` you are able to update all the layers sources:

```
user@9c2c89bf20a6:<builddir>/machine-imx7s-warp-mbl/mbl-manifest/build-mbl$ repo sync
Fetching project openembedded/openembedded-core
Fetching project armmbed/meta-mbl
Fetching project Freescale/meta-freescale-3rdparty
Fetching project armmbed/mbl-config
Fetching project openembedded/bitbake
Fetching projects:  20% (2/10)
Fetching project git/meta-freescale
Fetching projects:  30% (3/10)
Fetching project git/meta-raspberrypi
Fetching projects:  40% (4/10)
Fetching project git/meta-virtualization
Fetching projects:  50% (5/10)
Fetching project openembedded/meta-openembedded
Fetching projects:  60% (6/10)
Fetching project openembedded/meta-linaro
remote: Enumerating objects: 123, done.        
remote: Counting objects: 100% (123/123), done.        
remote: Compressing objects: 100% (42/42), done.        
remote: Total 123 (delta 87), reused 104 (delta 81), pack-reused 0     
Receiving objects: 100% (123/123), 43.44 KiB | 0 bytes/s, done.
Resolving deltas: 100% (87/87), completed with 8 local objects.
Fetching projects:  70% (7/10)
From ssh://github.com/openembedded/openembedded-core
   28e631d..b32ec63  master     -> github/master
 + 76e3243...cdbdd11 master-next -> github/master-next  (forced update)
```
