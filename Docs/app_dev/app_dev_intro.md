# Developing applications for Mbed Linux OS


<!--**This page will be a review of the steps - each of these has its own full page of instructions**-->

<!--what are we assuming they know? just how to write apps, or do we assume they're already familiar with docker etc?-->

## Preparing your environment

1. [Install MBL CLI]().
1. Set up [a USB connection to your device](), so you can work with MBL CLI.
<!--so they have to install Docker CE and dockcross and what else?-->

## Preparing your device

MBL applications are not compiled together with the MBL codebase or with any Pelion Device Management credentials (unlike Mbed OS, where the codebase, credentials and application form a single binary). This method allows you to deploy and manage multiple applications on a single device, but it requires some preparation work: To install applications on the device, you first need to install MBL and Device Management credentials on that device.

1. Get an MBL image:
    * You can use [our evaluation image]().
    * You can [build your own]().
1. [Flash the image to the device]().
1. Provision the device with [Pelion Device Management credentials and API key]() so that it can connect to your Device Management account.
1. [Set up your network connection]().

## Preparing your application

MBL applications run as containers from images prepared with Docker.<!--is there a way to prepare a container without Docker? They're just following the OCI spec, right?--> A container has:

* Your application executable file.
* A `config.json` file that lists the device resources, such as hardware or persistent memory, the application container can access. It also carries running instructions for the device. <!--does it also tell the device how to run the application? the "process" bit?-->

<span class="tips">More information about containers and packages is available in our [Reference section]().</span>


### Building your application

1. Build your application using [dockcross](https://github.com/dockcross/dockcross) to cross-compile.<!--do they have to use dockcross, or do we just happen to always use it?-->
2. Create an OCI bundle, which combines your built application and your configuration file.<!--that's still with dockcross, right? That's why they have to use it?-->
2. Create an IPK file. This is the file format the MBL's package manager - which installs applications - can handle.<!--we use opkg-utils for that, and I think those scripts are *in* the image... are they?-->

<!--I think the bit I'm not clear about is why we take the OCI into an IPK and then a tar rather than put the OCI in a tar, skipping the IPK; is it because the OCI must be in an IPK to be able to run on the device, and the update service requires a TAR? Does MBL CLI also use the update service? If not, why a tar?-->
Docker's cross-compiling tool, dockcross, has a standard image to build applications for Linux on ARMv7. When we build MBL applications, the first stage of the build adds some tooling to the standard dockcross image. The resulting dockcross image takes a standard MakeFile and builds according to its instructions. The MakeFile in our Hello World application instructs dockcross to compile the code, create an OCI container that includes the compiled code and the `config.json`, and pack the OCI into an IPK .<!--why don't we create our own dockcross image, rather than start our dockerfile by modifying the dockcross?-->

<!--hello world dockerfile takes the standard dockcross image for linux on ARMv7 and adds some extra tooling. Then it takes their makefile and follows its instructions to compile, then create an OCI and then an IPK.-->

<!--a lot of this info is in the containers reference-->

The first decision to make is how to build your application: you need to compile your files into a bundle that you can the package along with the `config.json` file.

There are two main paths:

* Using a compiler and a linker file such as MakeFile. <!--dockcross to cross-compile, adding the `opkg-utils` utility.dockcross is a Docker-based cross-compiling toolchain. We build a new Docker image with dockcross as our starting point, adding the opkg utilities. The image is defined in cc-env/Dockerfile. You can build it with docker build: -->
* Using Docker. <!--Because we don't want Docker to run on the device itself, we run it on the development environment and put its output on the device. But that's obvious, if you know Docker. Should we assume they know it? No, since even we don't always use Docker.-->
<!--I don't think we mention that Docker is optional in the Prerequisites-->

Applications are packaged as IPK files, compressed in a TAR, then installed on an MBL device. You can run the application on the device by starting a container (or multiple containers) from your installed package.<!--how much can we assume application developers know about Linux?-->

<!--"To create the OCI container, we use dockcross to cross-compile the application, then the opkg-utils helper scripts to package the compiled application as an IPK."-->
<!--but all of this is triggered by the MakeFile, so the MakeFile orchestrates the work, it's not part of what's compiled. The part that's compiled is the config.json. -->
<!--To install the application, the device unpacks an IPK from an application update payload. To build the IPK, use a host machine and cross-compiling toolchains from a dockcross Docker image we provide for ARMv7 architecture. We provide the image in the application's repository.
<!--dockcross is used to build binaries for many different platforms. dockcross performs a cross compilation where the host build system is a Linux x86_64 / amd64 Docker image (so that it can be used for building binaries on any system which can run Docker images) and the target runtime system varies.-->
<!--The script builds a Docker image with dockcross, adding the opkg-utils utility. It also installs a helper script (build-armv7) to execute build commands.-->
