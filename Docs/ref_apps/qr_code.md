## Quick Response (QR) code reader application

<!--sourced from https://github.com/ARMmbed/mbl-app-qrcode/blob/master/README.md-->

### Purpose

The QR code reference application demonstrates a typical development workflow for prototyping IoT applications: it is developed on an x86 machine, then cross-compiled for a development board (in this case the NXP Warp7). It uses a video stream, provided by a Video for Linux-enabled camera device, to scan a basic QR code.

This document provides instructions for building the QR app for:

* An Ubuntu 16.04 x86 development machine. We tested with the Logitech C310 webcam.
* A Warp7 development board with integrated camera. Due to a known issue with the Video4Linux library on WaRP7, the application will print the contents of a pre-scanned QR code saved into the docker image.

<!--do I need to do both, or can I just skip to building for the device if I want to?-->

<span class="notes">**Note**: These instructions assume that you have already installed and configured Mbed Linux onto your device, and connected to the same WiFi network as your development machine. For instructions, see the [Getting started section](,,/getting-started/index.html).</span>

### Reference: Makefile commands

- `default`: builds the `qr_code` app and creates a Docker image.
- `run`: runs the image in a Docker container.
- `compile`: builds the `qr_code` app.
- `cross`: to be used inside the cross-compilation docker image; see [Building for the IoT device](#building-for-arm) for instructions on how to use this.<!--that's not in this doc - is it in the getting started set on the docs site?--><!--also, what does it do? I don't see you using it; there's just `make compile`-->
- `cross-file`: to compile the application for Warp7 devices. Due to a known issue with the Video4Linux library, we have included a version of the application that reads a QR code from a file in the Docker image rather than using the camera. <!--this didn't quite parse; is it a command that calls the file, or is it the name of the file Makefile needs to use?-->
- `clean`: deletes Docker images and containers, and build artefacts.
- `status`: shows the status of Docker images and containers.

### License

Please see the [License](https://github.com/ARMmbed/mbl-app-qrcode/blob/master/LICENSE) document on GitHub for more information.

### Contributing

If you want to contribute additional functionality or fixes to this application, please see the [Contribution guidelines](../references/contribution-guidelines.html).

## Building the application for an x86 development machine
<!--since I took it out of the context of the GitHub repo, I now need to link to that repo so people can source the code
https://github.com/ARMmbed/mbl-app-qrcode/blob/master/README.md-->

To build the app for your development machine, such as an x86 PC, you need to prepare your environment and install the software dependencies.

### Preparing your development environment

To develop and build this application on your x86 PC, make sure you have:

- All of the [requirements you installed](../getting-started/environment.html) to create the original image on your device.

    Make sure Docker is still installed, and that you are still part of the `docker` group:

    1. Verify that the group exists: `sudo groupadd docker` (This may return with a warning saying the group is already present; you can ignore this warning.)
    1. Add yourself using `sudo usermod -aG docker $USER`
    1. Log out and back in.

- **`build-essential`**: a package containing tools for compiling and building, such as the GCC compiler and Make tool: `sudo apt-get install build-essential`.
- A local clone of the application repository: `git clone git@github.com:ARMmbed/mbl-app-qrcode.git`

Install the following dependencies:

- [Quirc](https://github.com/dlbeer/quirc): a QR decoder library.

    ```
    git clone git@github.com:dlbeer/quirc.git quirc_x86
    cd quirc_x86
    sudo make install
    ```

- [v4l2](https://www.linuxtv.org/): the Video4Linux library: `sudo apt-get install libv4l-dev`
- [libjpeg](http://www.ijg.org/): a library for reading and writing JPEG images: `sudo apt-get install libjpeg-dev`
- [libsdl]: a simple direct layer for graphics: `sudo apt-get install libsdl-gfx1.2-dev`


### Building the application

To build the QR app for your x86 PC:

 ```
 cd mbl-app-qrcode
 make compile
 ```
 <!--now what? there's a bit down at the bottom about testing, but there's just a warning there about the Warp7 (which I'll place much earlier, because if I were a user I wouldn't bother installing this app if I knew it was faking a QR code). There is nothing here telling me how to use the application - how do I run it, what should I expect to see-->

## Building the QR app for an IoT device (Warp7)

To build the app for your development board, such as a Warp7, you need to:

* Prepare your development environment.
* Prepare your cross-compilation environment and library location.
* Install any software dependencies.

### Preparing your development environment

Before you can build the QR for the development platform, the Warp7 board, you need to make sure you have:

- All of the [requirements you installed](../getting-started/environment.html) to create the original image on your device.

    Make sure Docker is still installed, and that you are still part of the `docker` group:

    1. Verify that the group exists: `sudo groupadd docker` (This may return with a warning saying the group is already present; you can ignore this warning.)
    1. Add yourself using `sudo usermod -aG docker $USER`
    1. Log out and back in.

- Installed [Mbed Linux OS CLI](../tools/mbl-cli.html).
- A local clone of the application repository: `git clone git@github.com:ARMmbed/mbl-app-qrcode.git`
- Set the `WORKDIR` environment variable (cross-compilation library location) to the parent directory of the `mbl-app-qrcode` directory:

    ```
    cd $WORKDIR/mbl-app-qrcode
    mkdir lib
    ```

Install the following dependencies:
<!--why are these so much more complicated than the ones for x86 installation?-->

- [Quirc](https://github.com/dlbeer/quirc): a QR decoder library:

    ```
    cd $WORKDIR
    git clone git@github.com:dlbeer/quirc.git
    cd quirc
    build-armv6 make libquirc.a
    cp libquirc.a ../mbl-app-qrcode/lib/
    cp lib/quirc.h ../mbl-app-qrcode/lib/
    ```

- [v4l2](https://www.linuxtv.org/): the Video4Linux library.

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

- [libjpeg](http://www.ijg.org/): a library for reading and writing JPEG images.  <!--why didn't we give an installation command?-->

### Cross-compiling the application

1. Prepare your cross-compilation environment:

    ```
    cd $WORKDIR/mbl-app-qrcode
    docker build -t linux-armv6:latest ./cc-env/
    docker run --rm linux-armv6 > build-armv6
    chmod +x build-armv6
    sudo mv build-armv6 /usr/local/bin
    ```


1. Compile the application for the Warp7 development board:<!--is it worth also showing how to build for non-Warp7 boards? we didn't test, but someone might try, and they maybe won't need the pre-scanned version-->

    <span class="notes">**Note:** Due to a known issue with the Video4Linux library on WaRP7, the application must print the contents of a pre-scanned QR code saved into the docker image. These instructions are for building a version of the application that contains the pre-scanned code.</span>

    ```
    cd $WORKDIR/mbl-app-qrcode
    build-armv6 make cross-file
    ```

### Creating a docker image

Create an executable package, which includes everything needed to run the application:

```
cd $WORKDIR/mbl-app-qrcode
cd file
mbl-cli build
```

This generates an image file called `mbl-image.tar`.

### Transferring the docker image to the device

To copy `mbl-image.tar` to the device you can use one of the following methods, depending on the MBL image you have installed:

* If you have used the **test image** (`mbl-console-image-test`), your Warp7 has an SSH client/server. To copy `mbl-image.tar` to the device:

    ```
    cd $WORKDIR/mbl-app-qrcode/file
    scp mbl-image.tar root@<device ip address>:/mbl-image.tar
    ```
* If you have used the **production image** (`mbl-console-image`), use `wget` to copy `mbl-image.tar` to the device:

    1. **On your development machine**, start a temporary HTTP server:

        ```
        cd $WORKDIR/mbl-app-qrcode/file
        python -m SimpleHTTPServer
        ```

        If you have `python3`, the command to start the temporary HTTP server is: `python3 -m http.server`

    1. Connect to the device over a serial connection.
    1. Log in to the device with `root` and no password:

        ```
        sudo minicom -D /dev/ttyUSB0
        ```

    1. **On your Warp7**, run the following command to download `mbl-image.tar`:

        ```
        wget http://<development machine ip address>:8000/mbl-image.tar -O /mbl-image.tar
        ```

### Loading the docker image

When `mbl-image.tar` is on the device, load the docker image:

```
docker image load -i /mbl-image.tar
```

### Running the QR app on warp7

Connect to the device over SSH/serial console and run:

```
docker run --rm mbl-app
```

### Testing the QR app

<!--How do I test it? What do I do with the application once it's on the device? What will I see if I'm running the Warp7 pre-scanned version, and what can I expect to see if I decided to try the normal version on an untested board?-->


### Running Arm executables on x86 linux


<!--what does this relate to? this is very late in the document to tell people to install things-->

To run Arm executables on an x86 machine, install the following packages:

* `binfmt-support`
* `qemu-user-static`
