# Production image build examples

The following examples assume:

* You have an output directory for your machine, for example:
    * `./artifacts-pico7`
    * `./artifacts-nxpimx`
    * `./artifacts-pico6`
    * `./artifacts-warp7`
    * `./artifacts-rpi3`
* You have an [SSH agent](../first-image/development-environment.html). For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

To build the Production variant images the `--root-passwd-file PASSWD_FILE` needs to be passed to **run-me.sh** and `--distro mbl-production` to **build.sh**.

The `PASSWD_FILE` passed with `--root-passwd-file PASSWD_FILE` parameter constains the root user password in plain text with the minimum of 12 characters.

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
