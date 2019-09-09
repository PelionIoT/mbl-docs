# Production image configuration and build examples

The following examples assume:

* You have an output directory for your machine, for example:
    * `./artifacts-pico7`
    * `./artifacts-nxpimx8`
    * `./artifacts-pico6`
    * `./artifacts-warp7`
    * `./artifacts-rpi3`
* You have an [SSH agent](../first-image/development-environment.html). For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

Production Images are built by setting the Yocto DISTRO to mbl-production (`--distro mbl-production` parameter needs to be passed to **build.sh**), and include the following extra security features when compared to the Developer Images:

- Root user login with password: A file containing the plain text password (with a minimum of 12 characters) needs to be provided to the to **run-me.sh** using the parameter`--root-passwd-file PASSWD_FILE`.

- System log level messages output and configuration: The following variables are set in the mbl-production configuration file (`meta-mbl/meta-mbl-distro/conf/distro/include/mbl-distro-production.inc`): <br>
    - **ATF_PRODUCTION_CFG** and **OPTEE_PRODUCTION_CFG**: Affects ATF and OPTEE-OS log level messages, respectively, with the following possible value:<br>
        - **silent** (default): Only warning and error messages are printed out to the console.
    - **UBOOT_PRODUCTION_CFG**: Affects both U-boot and Linux Kernel log level messages, with the possible values:<br>
        - **silent** (default): disables both u-boot and kernel messages output<br>
        - **noconsole**: disables only u-boot messages output<br>
        - **minimal** (default): disables network booting, fastboot, usb mass storage and device firmware upgrade (DFU)<br>
    
    These variables can be customized by using the `--local-conf-data STRING` parameter passed to **build.sh**. For example: 
```
./mbl-tools/build/run-me.sh --builddir ./build-pico7 --outputdir ./artifacts-pico7 --root-passwd-file PASSWD_FILE -- --machine imx7d-pico-mbl --branch mbl-os-0.8 --distro mbl-production --local-conf-data "UBOOT_PRODUCTION_CFG=\"noconsole\""
```    
  

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
