## Introduction

The "Hello world" application is a sample C application that prints "Hello, world" to the standard output (STDOUT). <!--- Standard output?--->

The application consists of the ```hello_world.c``` file <!---script?--->in the ```./mbl-core/tutorials/helloworld/src``` directory.  

The application is cross-compiled using Dockcross image (cross compiling toolchains in a Docker image), and runs on target inside OCI container. For more information on OCI containers, please refer to the [OCI containers](#oci-containers) section below.

The application packaging is done using ```opkg-utils``` helper scripts with opkg (Open PacKaGe management).  

`opkg-utils` builds an IPK package, which is installed on device. For more information about IPK, please refer [IPK format](#ipk-format) section below.

Cross compiling toolchains in the Docker image are built using an existing Dockcross Docker image for armv7 architecture.  

The opkg-utils helper scripts are included to the toolchain Docker container. The toolchain Docker file can be found at: `./mbl-core/tutorials/helloworld/cc-env/Dockerfile`.

For further reading about Dockcross, see [GitHub cross compiling toolchains in Docker images ](https://github.com/dockcross/dockcross).

Build commands are defined in a makefile, which consists of three sections:
 * Cross-compilation build
 * Create OCI bundle
 * Create IPK.

Each section is implemented for both release and debug variants.

### Setting up cross-compilation tools

1. On a Linux Ubuntu PC, install Docker CE as described here: [Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)      

1. ```cd mbl-core/tutorials/helloworld```              

1. Build Dockcross (cross-compiling toolchains in Docker images). The parent  
   image is defined in Dockerfile `dockcross/linux-armv7`. The Docker build command  
   prepares cross-compilation environment for building a user sample application  
   inside the container:
   ```        
   docker build -t linux-armv7:latest ./cc-env/
   ```    

1. The image created by a previous Docker build command does not need to be  
   run manually. Instead, there is a helper script to execute build commands  
   on source code existing on the local host filesystem. This script is bundled  
   with the image.  

   To install the helper script:  
    ```
    docker run --rm linux-armv7 > build-armv7  
    chmod +x build-armv7  
    sudo mv build-armv7 /usr/local/bin  
    ```

### Build the `helloworld` application with cross-compiler

Build produces IPK file:            
   * For release: ```./release/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk```          
   * For debug: ```./debug/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk```      

In order to build, invoke make toolchain command:        
   * To build both release and debug: ```build-armv7 make all```
   * To build release: ```build-armv7 make release```
   * To build debug: ```build-armv7 make debug```
   * To clean both release and debug artifacts: ```build-armv7 make clean```

### Install and run application on target

1. Create ```user-sample-app-package_1.0_armv7vet2hf-neon.tar```:  
    ```
    tar -cvf user-sample-app-package_1.0_armv7vet2hf-neon.tar user-sample-app-package_1.0_armv7vet2hf-neon.ipk  
    ```

    This command can be done for release/debug variants separately.
1. Copy `user-sample-app-package_1.0_armv7vet2hf-neon.tar` to the device under `/scratch`.  
1. Extract, install the <!--- what?--->and run IPK using ```app_update_manager.py``` script:    
    ```
    mbl-app-update-manager -i /scratch/user-sample-app-package_1.0_armv7vet2hf-neon.tar  
    ```
1. The sample application will be automatically set up to run on booting the device.

### OCI containers

OCI containers are defined by the Runtime Specification of OCI (Open Container Initiative).    

The OCI Runtime Specification defines the configuration, execution environment and lifecycle of a container. A container's configuration is specified in the `./mbl-core/tutorials/helloworld/src_bundle/config.json` for the supported platforms and details the fields that enable the creation of a container.

The execution environment is specified to ensure that application running inside a container have a consistent environment between runtimes along with common actions defined for the container's lifecycle.

The Runtime Specification defines format for encoding a container as a "filesystem bundle" - a set of files organized in a certain way, and containing all the necessary data and metadata for any compliant runtime to perform all standard operations against it. A Standard Container bundle contains all the information needed to load and run a container. This includes the following artifacts:
 * ```config.json```: contains configuration data.
 * container's root filesystem (rootfs): the directory referenced by root.path, if that property is set in ```config.json``` (path is nested JSON object of root).

The OCI Runtime Specification outlines how to run a file system bundle that is unpacked on disk.

For further reading about the **runc** container see [GitHub Open Container Initiative Runtime Specification](https://github.com/opencontainers/runtime-spec).

### IPK format

IPK is a format for software packages. An .ipk file is a specially constructed archive containing three parts:
 * ```data.tar.gz``` : Contains data.tar which contains the actual files belonging to this package, organized as they should appear after installation.
 * ```control.tar.gz``` : Contains control.tar which contains control file describing the package meta-data and any installation/removal scripts for the package.  
 * ```debian_binary``` : This file is for comparability with Debian's package system and currently is ignored by IPKg.  

For further reading about `opkg-utils` refer to [GitHub Yocto project opkg-utils](http://git.yoctoproject.org/cgit/cgit.cgi/opkg-utils).
