# Build examples

The following examples assume:

* You have an output directory for your machine, for example:
    * `./artifacts-warp7`
    * `./artifacts-rpi3`
    * `./artifacts-evk`
    * `./artifacts-nxpimx`
* You have an [SSH agent](../getting-started/development-environment.html). For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).


## PICO-PI with IMX7D

```
./mbl-tools/build-mbl/run-me.sh --builddir ./build-pico --outputdir ./artifacts-pico -- --machine imx7d-pico-mbl --branch mbl-os-0.6
```

## NXP 8M Mini EVK

```
./mbl-tools/build-mbl/run-me.sh --builddir ./build-nxpimx --outputdir ./artifacts-nxpimx -- --machine nxpimx-mbl --branch mbl-os-0.6
```

## Warp7

```
./mbl-tools/build-mbl/run-me.sh --builddir ./build-warp7 --outputdir ./artifacts-warp7 -- --machine imx7s-warp-mbl --branch mbl-os-0.6
```

## Raspberry Pi 3

```
./mbl-tools/build-mbl/run-me.sh --builddir ./build-rpi3 --outputdir ./artifacts-rpi3 -- --machine raspberrypi3-mbl --branch mbl-os-0.6
```
