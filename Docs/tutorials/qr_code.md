# Mbed Linux OS (mbl) mbl-app-qrcode Quick Response (QR) Code Reader Reference Application Repository

<!--This content should be replaced, says Jeremy.-->

## Purpose

This repository contains an Mbed Linux OS reference application for reading a Quick Response (QR) code.

For more information about Mbed Linux OS, please see [meta-mbl][meta-mbl].

The QR Code reader reference app scans basic QR codes using a video stream provided by a Video For Linux enabled camera device. This app has been developed on an x86 development machine, and then cross-compiled for use on the development board, in the case, a Warp7 board.  This demonstrates a typical development workflow that you might choose, to prototype your IoT applications.

This document provides you with instructions for how to build the QR app for:

* An Ubuntu 16.04 x86 development machine.
* A Warp7 development board.

**Assumptions**

These instructions assume that you have successfully installed and configured Mbed Linux onto your device and connected to the same WiFi network of your development machine.
For instructions on how to do this, see the [walkthrough][meta-mbl-walkthrough].


**Supported platforms**

This app has been tested on the following hardware:

- **Development machine**: x86 PC with Logitech C310 webcam
- **Development board**: Warp7 with integrated camera

## Dependencies

This QR app requires the following packages:

- [Quirc](https://github.com/dlbeer/quirc): a QR decoder library.
- [v4l2](https://www.linuxtv.org/): the Video4Linux library.
- [libjpeg](http://www.ijg.org/): a library for reading and writing jpeg images.
- [Docker](https://www.docker.com/): to build binaries and docker images that run on Arm target machines.

### Makefile commands

- `default`: builds the qr_code app and creates a Docker image.
- `run`: runs the image in a Docker container.
- `compile`: builds the qr_code app.
- `cross`: to be used inside cross-compilation docker image; see [Building for the IoT device](#building-for-arm) for instructions on how to use this.
- `cross-file`: special file read only version to work-around the IOTMBL-251 issue.
- `clean`: deletes Docker images and containers, and build artefacts.
- `status`: shows the status of Docker images and containers.

## Building the QR app for the development machine (x86 PC)

To build the app for your development machine, such as an x86 PC, you need to prepare your environment and install the software dependencies.

### Prepare your development environment

To be able to develop and build apps on your x86 PC, you need to make sure you have installed:

- **Docker**: https://docs.docker.com/install/ (or you could use the following command: `sudo apt-get install docker`).
- **`build-essential`**: a package which contains tools (such as, the gcc compiler, make tool, etc.) for compiling/building software from source (`sudo apt-get install build-essential`).

### Manage Docker as a non-root user

You will need to add yourself to the `docker` group to run `docker` commands without `sudo`. First, ensure the `docker` group is present:

```
sudo groupadd docker
```

This may return with a warning saying the group is already present. This warning is safe to ignore.

Now add yourself to the group:

```
sudo usermod -aG docker $USER
```

Finally, logout and login for this to take affect.

### Install the software dependencies

1. Install the jpeg library for reading and writing JPEG image files, using the following command:
    ```
    sudo apt-get install libjpeg-dev
    ```
1. Install the simple direct layer for graphics, using the following command:
   ```
   sudo apt-get install libsdl-gfx1.2-dev
   ```
1. Download, build and install Quirc using the following commands:
    ```
    git clone git@github.com:dlbeer/quirc.git quirc_x86
    cd quirc_x86
    sudo make install
    ```
1. Install the Video4Linux V4L2 libraries using the following command:
    ```
    sudo apt-get install libv4l-dev
    ```

### Build the QR app

To build the QR app for your x86 PC, use these commands:
 ```
 cd mbl-app-qrcode
 make compile
 ```

## Building the QR app for an IoT device (Warp7)

To build the app for your development board, such as a Warp7, you need to:

* Prepare your development environment.
* Prepare your cross-compilation environment and library location.
* Install any softare dependencies.

### Preparing your development environment

Before you can build the QR for the development platform, the Warp7 board, you need to make sure you have:

- Installed Docker: https://docs.docker.com/install/ (or you could use the following command: `sudo apt-get install docker`)
- Installed [Mbed Linux cli](https://github.com/ARMmbed/mbl-cli).
- Set the `WORKDIR` environment variable to the parent directory of the `mbl-app-qrcode` directory.

### Preparing the cross-compilation environment

Use the following commands to prepare your cross-compilation environment:

```
cd $WORKDIR/mbl-app-qrcode
docker build -t linux-armv6:latest ./cc-env/
docker run --rm linux-armv6 > build-armv6
chmod +x build-armv6
sudo mv build-armv6 /usr/local/bin
```

### Setting the cross-compilation library location

 To set the location for the cross-compilation library:

```
cd $WORKDIR/mbl-app-qrcode
mkdir lib
```

### Install Quirc

Use the following commands to install Quirc, the QR decoder library:

```
cd $WORKDIR
git clone git@github.com:dlbeer/quirc.git
cd quirc
build-armv6 make libquirc.a
cp libquirc.a ../mbl-app-qrcode/lib/
cp lib/quirc.h ../mbl-app-qrcode/lib/
```

### Install Video4Linux (libv4l2)

Install the Video4Linux libraries using the following commands:

```
cd $WORKDIR
wget -qO- https://linuxtv.org/downloads/v4l-utils/v4l-utils-1.12.6.tar.bz2 | tar -jx
cd v4l-utils-1.12.6
build-armv6 ./bootstrap.sh
build-armv6 ./configure --host=arm-linux-gnueabihf --disable-doxygen-doc --disable-doxygen-dot --disable-doxygen-html --disable-doxygen-ps --disable-doxygen-pdf --disable-qv4l2 --disable-nls --disable-rpath --disable-libdvbv5 --disable-v4l-utils --disable-v4l2-compliance-libv4l --disable-v4l2-ctl-libv4l
build-armv6 make
cp ./lib/libv4l2/.libs/libv4l2.so.0.0.0 ../mbl-app-qrcode/lib/libv4l2.so
cp ./lib/libv4lconvert/.libs/libv4lconvert.so.0.0.0 ../mbl-app-qrcode/lib/libv4lconvert.so
cp ./lib/include/libv4l2.h ../mbl-app-qrcode/lib
```

### Cross compile the QR app

**NOTE: Due to the known Video4Linux issue (IOTMBL-251) on WaRP7 the rest of these instructions
tell you how to build the version that does not use the camera and instead uses a QR code saved
into a file**

Use the following commands to compile the QR app for use on the Warp7 development board:

```
cd $WORKDIR/mbl-app-qrcode
build-armv6 make cross-file
```

### Create a docker image

Create an executable package that includes everything needed to run the QR application, using the following commands:

```
cd $WORKDIR/mbl-app-qrcode
cd file
mbl-cli build
```

### Transfer the docker image to Warp7

To copy mbl-image.tar to the device you can use one of the following methods, depending on the Mbed Linux image you have installed.

1. If you have used the **test image** (`mbl-console-image-test`), your Warp7 has an ssh client/server which can be used to copy the mbl-image.tar file to the device using the following commands:
    ```
    cd $WORKDIR/mbl-app-qrcode/file
    scp mbl-image.tar root@<device ip address>:/mbl-image.tar
    ```
1. If you have used the **production image** (`mbl-console-image`), you can use `wget` to copy the `mbl-image.tar` file to the device. Follow the steps below:
    1. **On your development machine**, start a temporary HTTP server:
        ```
        cd $WORKDIR/mbl-app-qrcode/file
        python -m SimpleHTTPServer
        ```
        If you have `python3`, the command to start the temporary HTTP server is: `python3 -m http.server`
    1. Connect via serial to Warp7 and login with `root` and no password:
        ```
        sudo minicom -D /dev/ttyUSB0
        ```
    1. **On your Warp7**, download the `mbl-image.tar` file with the following command:
        ```
        wget http://<development machine ip address>:8000/mbl-image.tar -O /mbl-image.tar
        ```

Once the mbl-image.tar file is copied to the device, create the docker image with following command:
```
docker image load -i /mbl-image.tar
```

### Running the QR app on warp7

SSH/serial console to the device and use the following command:

```
docker run --rm mbl-app
```

### Testing the QR app

**NOTE: Due to the known Video4Linux issue (IOTMBL-251) on WaRP7 the application will print the contents
of a pre-scanned QR code saved into the docker image**

### Running Arm executables on x86 linux

To run Arm executables on an x86 machine, install the following packages:

* binfmt-support.
* qemu-user-static.

## License

Please see the [License][mbl-license] document for more information.

## Contributing

Please see the [Contributing][mbl-contributing] document for more information.



[meta-mbl]: https://github.com/ARMmbed/meta-mbl/blob/master/README.md
[mbl-license]: LICENSE
[mbl-contributing]: CONTRIBUTING.md
[meta-mbl-walkthrough]: https://github.com/ARMmbed/meta-mbl/blob/master/docs/walkthrough.md
