# Production image configuration and build examples

<span class="tips">**Tip**: If you downloaded an evaluation image, you can skip the build stage [and go directly to writing](../first-image/writing-an-image-to-supported-boards.html).</span>

To build production images, set the Yocto DISTRO to `mbl-production` (pass the `--distro mbl-production` parameter to **build.sh**). Production images include the following extra security features (compared to the development image):

- Root user login with password: A file containing the plain text password (minimum 12 characters) needs to be provided to **run-me.sh** using the parameter`--root-passwd-file PASSWD_FILE`.

- System log level messages output and configuration: The following variables are set in the mbl-production configuration file (`meta-mbl/meta-mbl-distro/conf/distro/include/mbl-distro-production.inc`):

    - **ATF_PRODUCTION_CFG: ** Affects ATF log level messages. The only possible value is **silent** (set by default) - only error messages are printed out to the console.
    - **OPTEE_PRODUCTION_CFG**: Affects OPTEE-OS log level messages. The only possible value is **silent** - only warning and error messages are printed out to the console.
    - **UBOOT_PRODUCTION_CFG**: Affects both U-Boot and Linux Kernel log level messages. The possible values are:
        - **silent** (default): Disables both U-Boot and kernel message output.
        - **noconsole**: Disables only U-Boot message output.
        - **minimal** (default): Disables network booting, fastboot, USB mass storage and device firmware upgrade (DFU) message output.<!--two defaults?--><!--is there a combination of values that gives me all error messages, or is that not something you'll ever do in production?-->

You can set these variables by:<!--does this refer just to log levels, or all the parameters above? I saw the password thing in the example, but the --local-conf-data bit comes long after it, so I'm not sure what this text refers to, and what the example script refers to - the script certainly seems to cover everything-->

- Manually editing the `local.conf` file inside the BitBake build directory. For example: `./build-pico7/machine-imx7d-pico-mbl/mbl-manifest/build-mbl-production/conf/local.conf`.
- Setting these variables in *one of* your repositories' configuration files:
  - In your `layer.conf` file in your meta layer repository.
  - In your `local.conf` file in your config repository.

  See the help on [creating an example project](../develop-mbl/example-project-based-on-mbed-linux-os.html) for more information on these repositories.
- Passing the `--local-conf-data STRING` parameter to **build.sh**.

    For example:

    ```
    ./mbl-tools/build/run-me.sh --builddir ./build-pico7 --outputdir ./artifacts-pico7 --root-passwd-file PASSWD_FILE -- --machine imx7d-pico-mbl --branch mbl-os-0.8 --distro mbl-production --local-conf-data "UBOOT_PRODUCTION_CFG=\"noconsole\"\nATF_PRODUCTION_CFG=\"\""
    ```

    <span class="tips">Note that `--local-conf-data` needs to be passed every run.</span>

The following are examples of build commands to produce development images for supported platforms.

The examples assume:

* You have an output directory for your machine, for example:
    * `./artifacts-pico7`
    * `./artifacts-nxpimx8`
    * `./artifacts-pico6`
    * `./artifacts-warp7`
    * `./artifacts-rpi3`
* You have an [SSH agent](../first-image/development-environment.html). For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

## PICO-PI with IMX7D

```
./mbl-tools/build/run-me.sh --builddir ./build-pico7 --outputdir ./artifacts-pico7 --root-passwd-file PASSWD_FILE -- --machine imx7d-pico-mbl --branch mbl-os-0.8 --distro mbl-production
```

## PICO-PI with IMX6UL

```
./mbl-tools/build/run-me.sh --builddir ./build-pico6 --outputdir ./artifacts-pico6 --root-passwd-file PASSWD_FILE -- --machine imx6ul-pico-mbl --branch mbl-os-0.8 --distro mbl-production
```

## NXP 8M Mini EVK

```
./mbl-tools/build/run-me.sh --builddir ./build-nxpimx --outputdir ./artifacts-nxpimx --root-passwd-file PASSWD_FILE -- --machine imx8mmevk-mbl --branch mbl-os-0.8 --distro mbl-production
```

## Warp7

```
./mbl-tools/build/run-me.sh --builddir ./build-warp7 --outputdir ./artifacts-warp7 --root-passwd-file PASSWD_FILE -- --machine imx7s-warp-mbl --branch mbl-os-0.8 --distro mbl-production
```

## Raspberry Pi 3

```
./mbl-tools/build/run-me.sh --builddir ./build-rpi3 --outputdir ./artifacts-rpi3 --root-passwd-file PASSWD_FILE -- --machine raspberrypi3-mbl --branch mbl-os-0.8 --distro mbl-production
```
