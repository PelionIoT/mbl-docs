# Developing applications for Mbed Linux OS

## Application design

NOTE: a lot of this is Irit talking to herself; don't worry about typos, weird sentences and repetitions.

Mbed Linux OS (MBL) is designed for cloud-connected applications for IoT devices.

End user: my phone

Cloud: duh

Terminal IoT device: just an iot device

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/applications_map.png)</span>

What you're trying to achieve: the left-hand side of the diagram: device application to user application.

You are designing an Iot Device that manages its communication with the Device Management Client, as well as any processing and automation it needs to perform to be useful. You can design a single application to do all these things, or you can design smaller applications that provide a microservice. One can manage communication with Device Management Client, and will always be able to manage that, no matter what else the device requires. Another can be the application that processes sensor input. This, too, can be useful for more than one type of device. One application can be very specific - it performs the actions that device specialises in.

There's a tradeoff: the more you pack into the application container, the bigger it is. But the less it relies on the distro. Not relying on the distro means you can update your applications and your distro completely independently of each other. You can reduce the container size by using elements of the distro, like runtime library or Python versions, but you then have to be careful to always match any changes in the app and the distro. This makes your application less portable, and your updates more complicated.

At the moment, the communication between the device application and DM Client isn't possible - we're missing the d-bus to connect the two.



# Application development flow

## Preparing your device

<span class="tips">**Tip**: for a list of supported devices, [see our hardware requirements]().</span>

MBL applications are not compiled together with the MBL codebase or with any Pelion Device Management credentials (unlike Mbed OS, where the codebase, credentials and application form a single binary). This method allows you to deploy and manage multiple applications on a single device, but it requires some preparation work: To install applications on the device, you first need to install MBL and Device Management credentials on that device.

1. Review our list of [supported development boards](../first-image/hardware.html) and set up your [development environment](../first-image/development-environment.html).
1. Get an MBL image:
    * You can use [our evaluation image]().<!--when will we have this?-->
    * You can [build your own](). You will need to [set up a full development environment for this](../first-image/building-an-mbl-image.html).
1. [Flash the image to the device](../first-image/writing-and-booting-the-disk-image.html).
1. Provision the device with [Pelion Device Management credentials and an API key](../first-image/pelion-device-management-accounts-and-certificates.html) so that it can connect to your Device Management account.
1. [Set up your network connection](../first-image/connecting-to-a-network-and-pelion-device-management.html) and [test your Device Management connectivity](../first-image/verifying-that-the-device-is-connected-to-device-management.html).


## Application development requirements

1. [Install MBL CLI](../develop-apps/setting-up.html).
1. Set up [a USB connection to your device](), so you can work with MBL CLI.
1. Install Docker CE.

## Build examples

MBL applications run as containers from images prepared with Docker. A container has:

* Your application's executable files and scripts.
* A `config.json` file that lists the device resources, such as hardware or persistent memory, the application container can access. It also carries instructions for the device on how to run the application.

<span class="tips">More information about containers and packages is available in our [Reference section](../references/application-containers-and-packages.html).</span>

There are many ways to create images in the IPK format that the Linux package manager can run. For our own applications, we use Docker's cross-compiling tool, [dockcross](https://github.com/dockcross/dockcross), which has a standard image with everything we need to build applications for Linux on ARMv7. When we build MBL applications, the first stage of the build adds some tooling (opkg-utils and a helper script - build-armv7) to the standard dockcross image. The resulting mbl-dockcross image takes a standard MakeFile and builds according to its instructions:

1. Builds the application using mbl-dockcross to cross-compile.
1. Creates an OCI bundle, which combines our built application and our configuration file.
1. Creates an IPK file. This is the file format the MBL's package manager - which installs applications - can handle.

We then compress our IPK into a TAR, to match the requirements of the Device Management Update service.

The applications in the following tutorials all use MakeFile and dockcross to cross-compile. They introduce different levels of reliance on Docker and access from the dockerized application to the device:

1. The Hello World C application is entirely self contained. It has just one file to build - `hello_world.c` - and a `config.json` defining how to run the application on the device. The `config.json` doesn't give the application any special access to the device's software or hardware resources; the application doesn't even use the device's C runtime library.

    The application also has a very simple build - dockcross and a MakeFile - because as a simple C file it doesn't require any other tools to be containerised with it.

1. The QR scanner Python application (coming soon) uses its `config.json` to access the device's hardware resources and persistent memory.

    This application is Dockerised - because it's a Python application and needs the Python runtime environment with it, as well an the OpenCV library to capture camera frames, it cannot be built and converted to an OCI individually (the way we did with the Hello World C application). Instead, it's built and then bundled with all its Python dependencies.
