## Developing applications for Mbed Linux OS

<!--**This page will be a review of the steps - each of these has its own full page of instructions**-->
<!--something about... what was I thinking about?-->


### Preparing your device

1. Get an MBL image:
    * You can get [our evaluation image]().
    * You can [build your own]().
1. Provision the image with [Pelion Device Management credentials]().
1. [Flash the image to the device]().
1. [Set up your network connection](), and verify that the device [can connect to your Device Management account]().

### Preparing your environment

1. [Install MBL CLI]().
1. Set up [a USB connection to your device](), so you can work with MBL CLI.

### Writing a configuration file

MBL applications use a `config.json` file to determine which device resources, such as hardware or the C runtime library, the application container can access. This impacts security - the more constrained the container, the more protected the device - so you should constrain your application as much as you can.

### Creating the application package

Applications are packaged as IPK files, compressed in a TAR, then installed on an MBL device. You can run the application on the device by starting a container (or multiple containers) from your installed package.

<!--can I install the IPK directly, or do I have to use an update with a TAR?-->

Cross-compilation build.
Create OCI bundle.
Create IPK.

### Building your application

The first decision to make is how to build your application: you need to compile your files into a bundle that you can the package with the `config.json` file.

There are two main paths:

* Using a compiler and a linker file such as MakeFile. <!--dockcross to cross-compile, adding the `opkg-utils` utility.dockcross is a Docker-based cross-compiling toolchain. We build a new Docker image with dockcross as our starting point, adding the opkg utilities. The image is defined in cc-env/Dockerfile. You can build it with docker build: -->
* Using Docker. Because we don't want Docker to run on the device itself, we run it on the development environment and put its output on the device.
<!--I don't think we mention that Docker is optional in the Prerequisites-->

### Packing <!--or packaging--> your application

Whichever built method you use, you'll need to create an OCI bundle, which creates an IPK.

<!--then you need to create a TAR?-->
