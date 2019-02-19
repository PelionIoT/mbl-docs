# Developing applications for Mbed Linux OS

<!--needs an intro paragraph-->
<!--JIJ: I suggest a few paragraphs from: https://confluence.arm.com/display/mbedlinux/Application+Containers+Overview -->
<!--JIJ: And maybe the first paragraph/diagram from: https://confluence.arm.com/display/mbedlinux/How+IoT+Applications+are+Structured -->

## Preparing your device

<span class="tips">**Tip**: for a list of supported devices, [see our hardware requirements]().</span>

MBL applications are not compiled together with the MBL codebase or with any Pelion Device Management credentials (unlike Mbed OS, where the codebase, credentials and application form a single binary). This method allows you to deploy and manage multiple applications on a single device, but it requires some preparation work: To install applications on the device, you first need to install MBL and Device Management credentials on that device.

1. Get an MBL image:
    * You can use [our evaluation image]().
    * You can [build your own](). You will need to [set up a full development environment for this]().
1. [Flash the image to the device]().
1. Provision the device with [Pelion Device Management credentials and an API key]() so that it can connect to your Device Management account.<!--I think this now happens after the flashing, but I could be wrong / JIJ: Yes, flash first, then provision the credentials-->
1. [Set up your network connection]() and [test your Device Management connectivity]().

## Application development requirements

1. [Install MBL CLI]().
1. Set up [a USB connection to your device](), so you can work with MBL CLI.
1. Install Docker CE<!--or any docker? is CE only important if you're building the image itself?--><!--does docker include runc?-->
<!--JIJ: we don't really have the full story here, at the moment the best advice is to follow the same process we have for the examples - which is docker CE-->

## Building your application

MBL applications run as containers from images prepared with Docker. A container has:

* Your application files <!--JIJ: These will be a combination of executables and/or scripts-->.
* A `config.json` file that lists the device resources, such as hardware or persistent memory, the application container can access. It also carries running instructions for the device. <!--does it also tell the device how to run the application? the "process" bit? / JIJ: Yes thats the last sentance - not sure what you mean by "process" -->

<span class="tips">More information about containers and packages is available in our [Reference section]().</span>

Docker's cross-compiling tool, dockcross, has a standard image with everything you need to build applications for Linux on ARMv7. When we build MBL applications, the first stage of the build adds some tooling (opkg-utils and a helper script - build-armv7) to the standard dockcross image. The resulting dockcross image takes a standard MakeFile and builds according to its instructions:

1. Builds your application using [dockcross](https://github.com/dockcross/dockcross) to cross-compile.<!--do they have to use dockcross, or do we just happen to always use it? / JIJ: They do not have to use dockcross - this whole section is more an example of how it can be done rather than a you must do it this way. -->
1. Creates an OCI bundle, which combines your built application and your configuration file.<!--that's still with dockcross, right? That's why they have to use it? / JIJ: The makefile states how dockcross can do this, but dockcross isn't needed for this bit, as its just creating a directory with your application files and your json file in it - thats all a container is-->
1. Creates an IPK file. This is the file format the MBL's package manager - which installs applications - can handle.<!--we use opkg-utils for that, and I think those scripts are *in* the image... are they? / JIJ: Currently we have added the opkg as another tool into a new version of dockcross, maybe refer to it as mbl-dockcross? -->
1. Adds your IPK into a TAR, to match the requirements of the Device Management Update service. <!-- JIJ Compression not supported-->

The applications in the following tutorials all use MakeFile and dockcross to cross-compile. They introduce different levels of reliance on Docker and access from the dockerized application to the device:

1. The Hello World C application is entirely self contained. It has just one file to build - `hello_world.c` - and a `config.json` defining how to run the application on the device. The `config.json` doesn't give the application any special access to the device's software or hardware resources; the application doesn't even use the device's C runtime library.

    The application also has a very simple build - dockcross and a MakeFile - because as a simple C file it doesn't require any other tools to be containerised with it.

1. The QR scanner Python application (coming soon) uses its `config.json` to access the device's hardware resources and persistent memory.

    This application is Dockerised - because it's a Python application and needs the Python runtime environment with it, as well an the OpenCV library to capture camera frames, it cannot be built and converted to an OCI individually (the way we did with the Hello World C application). Instead, it's built and then bundled with all its Python dependencies.
