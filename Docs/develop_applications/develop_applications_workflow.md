# Application development flow

## Preparing your device

<span class="tips">**Tip**: for a list of supported devices, [see our hardware requirements](../first-image/hardware.html).</span>

MBL applications are not compiled together with the MBL codebase or with any Pelion Device Management credentials (unlike Mbed OS, where the codebase, credentials and application form a single binary). This method allows you to deploy and manage multiple applications on a single device, but it requires some preparation work: To install applications on the device, you first need to install MBL and provision Device Management credentials on that device.

* If you were given a device with MBL already installed and provisioned, please:

    1. [Install MBL CLI](../develop-apps/setting-up.html), which you will also use when developing applications.

    <span class="notes">**Note**: If you have MBL CLI 1.x installed, please [uninstall it, and install 2.0 instead](../develop-apps/setting-up.html#setting-up-mbl-cli). Use `--version` to check.</span>

    1. Set up [a developer (USB) connection to your device](../develop-apps/setting-up.html#setting-up-networking), so you can work with MBL CLI.

* If your device does not have MBL, please install an [image on it](../first-image/index.html).


## Build examples

MBL applications run as containers from images prepared with Docker. A container has:

* Your application's executable files and scripts.
* A `config.json` file that lists the device resources, such as hardware or persistent memory, the application container can access. It also carries instructions for the device on how to run the application.

<span class="tips">More information about containers and packages is available in our [Reference section](../references/application-containers-and-packages.html).</span>

There are many ways to create applications that can be packaged in the IPK format so that the MBL application framework can install and run them. For our example applications, we use a Docker cross-compiling tool, [dockcross](https://github.com/dockcross/dockcross), which has a standard image with everything we need to compile C and C++ applications for Linux on ARMv7. We add our own layer on top of the standard dockross image, and that layer includes the tools to create an IPK package (opkg-utils). The resulting cross-compilation docker image, referred to here as mbl-dockcross, takes a standard MakeFile and builds according to its instructions:

1. Builds the application using mbl-dockcross to cross-compile.
1. Creates an OCI bundle, the container, which combines our built application and our configuration file.
1. Creates an IPK file. This is the file format the MBL's package manager - which installs applications - can handle.

We then put our IPK into a TAR, to match the requirements of the Device Management Update service.

The **Hello World C** application is entirely self contained. It has just one file to build - `hello_world.c` - and a `config.json` defining how to run the application on the device. The `config.json` doesn't give the application any special access to the device's software or hardware resources; the application doesn't even use the device's C runtime library, as it is statically compiled. It uses a MakeFile and `mbl-dockcross` to cross-compile.

<span class="tips">More information about containers is available in the [application FAQ](../develop-apps/frequently-asked-questions.html).</span>

<!--1. The QR scanner Python application (coming soon) uses its `config.json` to access the device's hardware resources and persistent memory.

    This application is Dockerized (built in a Docker container) - because it's a Python application that (by choice) includes the Python runtime environment with it, as well as the OpenCV library to capture camera frames, it cannot be built and converted to an OCI individually (the way the Hello World C application is built). Instead, it's built and then bundled with all its Python dependencies.
-->


***

Copyright © 2020 Arm Limited (or its affiliates)
