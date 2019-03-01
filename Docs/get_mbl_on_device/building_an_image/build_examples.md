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

# Building outputs

The build process creates the following files:

| File | Path | Information |
| --- | --- | --- |
| Full disk image  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.wic.gz` | This is a compressed image of the entire flash. Once decompressed, this image can be directly written to storage media, and initializes the device's storage with a full set of disk partitions and an initial version of firmware. |
| Full disk image block map | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.wic.bmap` | This is a file containing information about which blocks of the uncompressed full disk image actually need to be written to the device. Some blocks of the image represent unused storage space, which does not actually need to be written. |
| Root file system archive  | `/path/to/artifacts/machine/<MACHINE>/images/mbl-image-development/images/mbl-image-development-<MACHINE>.tar.xz` | This is a compressed `.tar` archive, which you need when you update the device firmware (this topic is covered [in the Updating MBL tutorial](../update/index.html)). |
