## Hello world

The hello world application prints "Hello, world" to the STDOUT.<!--what is the context for this? is it supposed to teach me how to use tools? check that my MBL image works?-->
<!--what are the prerequisite? You already need a working MBL on the device, right?-->

### Source and build mechanisms overview

The application source file, `hello_world.c`, is in [https://github.com/ARMmbed/mbl-core/tree/master/tutorials/helloworld/src](https://github.com/ARMmbed/mbl-core/tree/master/tutorials/helloworld/src). On the device, the application runs on a target inside an OCI container. To create the OCI container, we use `opkg-utils` helper scripts to package the application as an IPK, then use [dockcross](https://github.com/dockcross/dockcross) to cross-compile it.

Cross compiling toolchains in the Docker image are built using an existing dockcross Docker image for the ARMv7 architecture. The `opkg-utils` helper scripts are included in the toolchain Docker container, which you can find at [https://github.com/ARMmbed/mbl-core/blob/master/tutorials/helloworld/cc-env/Dockerfile](https://github.com/ARMmbed/mbl-core/blob/master/tutorials/helloworld/cc-env/Dockerfile).<!--I have to say I got really confused about how many docker images we have-->

The build commands are defined in a Makefile with three sections:

1. Cross-compilation build.
1. Create OCI bundle.
1. Create IPK.

Each section is implemented for both release and debug variants.

<span class="tips">For more information, please refer to the [IPK format](#ipk-format) and [OCI containers](#oci-containers) sections below, or the [dockcross documentation on GitHub](https://github.com/dockcross/dockcross).</span>

### Setting up the cross-compilation tools

1. On a Linux Ubuntu PC, install Docker CE as described in the [Docker CE for Ubuntu documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/)      

1. Navigate to the `helloworld` folder in your clone:<!--do we know that they cloned it? shouldn't that be a step here?-->

    `cd mbl-core/tutorials/helloworld`           

1. [dockcross](https://github.com/dockcross/dockcross) is a Docker-based cross compiling toolchain. We build a new Docker image with dockcross as our starting point, adding the opkg utilities. The image is defined in `cc-env/Dockerfile` and can be built with `docker build`:

    ```
    docker build -t linux-armv7:latest ./cc-env/
    ```    

1. The freshly built image can generate a script that makes it easy to work with. Run the image and capture the output to a file. Make the file an executable and place it in your path:

    ```
    docker run --rm linux-armv7 > build-armv7  
    chmod +x build-armv7  
    sudo mv build-armv7 /usr/local/bin  
    ```

### Building the application with the cross-compiler            

The build produces an IPK file for either release or debug profiles:

* For release: `./release/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk`          
* For debug: `./debug/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk`    

To build, invoke the Make toolchain command for one of the following options:

* To build for both release and debug: `build-armv7 make all`
* To build just for release: `build-armv7 make release`
* To build just for debug: `build-armv7 make debug`

To clean both release and debug artefacts: `build-armv7 make clean`

### Installing and running the application on the device

1. Create the .tar file of the application:

    ```
    tar -cvf user-sample-app-package_1.0_armv7vet2hf-neon.tar user-sample-app-package_1.0_armv7vet2hf-neon.ipk  
    ```

    You can do this separately for either the release or the debug profile.

1. Copy `user-sample-app-package_1.0_armv7vet2hf-neon.tar` to the device under `/scratch`.  

1. Extract, install the and run the IPK using the `app_update_manager.py` script:    

    ```
    mbl-app-update-manager -i /scratch/user-sample-app-package_1.0_armv7vet2hf-neon.tar  
    ```

1. The sample application is automatically set up to run on the device's next boot.

### OCI containers and the IPK format

<!--this is **copied** without clearly admitting it and without linking to individual sources. the main question is why - does it offer any value here, rather than as a couple of references? -->
<!--if we need this content, we'll have to rewrite it.-->
<!--OCI containers
==============

OCI containers are defined by the Runtime Specification of OCI (Open Container Initiative).    

The OCI Runtime Specification defines the configuration, execution environment and lifecycle   
of a container. A container's configuration is specified in the ```./mbl-core/tutorials/helloworld/src_bundle/config.json```  
for the supported platforms and details the fields that enable the creation of a container.  
The execution environment is specified to ensure that application running inside a container
have a consistent environment between runtimes along with common actions defined for the container's lifecycle.  

The Runtime Specification defines format for encoding a container as a "filesystem bundle" - a set of files organized  
in a certain way, and containing all the necessary data and metadata for any compliant runtime to perform all  
standard operations against it. A Standard Container bundle contains all the information needed to load and  
run a container. This includes the following artifacts:

 * ```config.json```: contains configuration data.
 * container's root filesystem (rootfs): the directory referenced by root.path, if that property is set in ```config.json``` (path is nested JSON object of root).

The OCI Runtime Specification outlines how to run a filesystem bundle that is unpacked on disk.
For further reading about runc container see [GitHub Open Container Initiative Runtime Specification](https://github.com/opencontainers/runtime-spec).


IPK format
==========

IPK is a format for software packages. An .ipk file is a specially constructed archive containing three parts:

 * ```data.tar.gz``` : contains data.tar which contains the actual files belonging to this package, organized as they should appear after installation.
 * ```control.tar.gz``` : contains control.tar which contains control file describing the package meta-data and any installation/removal scripts for the package.  
 * ```debian_binary``` : this file is for comparability with Debian's package system and currently is ignored by ipkg.  

For further reading about opkg-utils refer [GitHub Yocto project opkg-utils](http://git.yoctoproject.org/cgit/cgit.cgi/opkg-utils).-->
