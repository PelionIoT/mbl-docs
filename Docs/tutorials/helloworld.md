
Introduction
============

Hello world application is a sample C application printing "Hello, world" to the STDOUT.
The application consists of ```hello_world.c``` file located under the ```./mbl-core/tutorials/helloworld/src``` directory.  

The application is cross-compiled using dockcross image (cross compiling toolchains in Docker image) and  
runs on target inside OCI container. For more information about OCI containers please refer "OCI containers" section below.

Hello world application packaging is done using ```opkg-utils``` helper scripts for use with opkg (Open PacKaGe management).  
```opkg-utils``` build ipk package which will be installed on device. For more information about IPK please refer "IPK format" section below. 

For armv7 architecture compilation we use dockcross cross compiler which runs inside Docker container.
Cross compiling toolchains in Docker image is built using the parent image defined in ```./mbl-core/tutorials/helloworld/cc-env/Dockerfile```. 
In order to build IPK, the opkg-utils are installed into the Docker container. 
For further reading about dockcross see [GitHub cross compiling toolchains in Docker images ](https://github.com/dockcross/dockcross).

Build commands are defined in Makefile which consists of three sections:

 * Cross-compilation build
 * Create OCI bundle
 * Create IPK.
 
Each section is implemented for both release and debug variants.


Set up the cross-compilation tools            
==================================            
            
1. On Linux Ubuntu PC install Docker CE as described here: [Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)      
            
1. ```cd mbl-core/tutorials/helloworld```              
               
1. Build dockcross (Cross compiling toolchains in Docker images). The parent  
   image is defined in Dockerfile: ```dockcross/linux-armv7```. Docker build command  
   prepares cross-compilation environment for building  user sample application  
   inside the container.   
              
   ```        
   docker build -t linux-armv7:latest ./cc-env/
   ```    
     
1. The image created by a previous docker build command does not need to be  
   run manually. Instead, there is a helper script to execute build commands  
   on source code existing on the local host filesystem. This script is bundled  
   with the image.  
   To install the helper script perform the following:  
  
    ```
    docker run --rm linux-armv7 > build-armv7  
    chmod +x build-armv7  
    sudo mv build-armv7 /usr/local/bin  
    ```
              
Build "helloworld" application with cross compiler            
==================================================            
          
Build produces IPK file:            
          
   * For release: ```./release/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk```          
   * For debug: ```./debug/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk```      
     
In order to build, invoke make toolchain command:        
            
   * To build both release and debug: ```build-armv7 make all```
   * To build release: ```build-armv7 make release```
   * To build debug: ```build-armv7 make debug```
   * To clean both release and debug artifacts: ```build-armv7 make clean```
             
               
Install and run application on target            
=====================================            
            
1. Create ```user-sample-app-package_1.0_armv7vet2hf-neon.tar```:  
  
    ```
    tar -cvf user-sample-app-package_1.0_armv7vet2hf-neon.tar user-sample-app-package_1.0_armv7vet2hf-neon.ipk  
    ```
   
    This command can be done for release/debug variants separately.
            
1. Copy ```user-sample-app-package_1.0_armv7vet2hf-neon.tar``` to the device under ```/scratch```.  
            
1. Extract, install the and run ipk using ```app_update_manager.py``` script:    
  
    ```
    mbl-app-update-manager -i /scratch/user-sample-app-package_1.0_armv7vet2hf-neon.tar  
    ```
             
1. The sample application will be automatically set up to run on booting the device.


OCI containers
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
 * container's root filesystem (rootfs): the directory referenced by root.path, if that property is set in ```config.json```.

The OCI Runtime Specification outlines how to run a filesystem bundle that is unpacked on disk.
For further reading about runc container see [GitHub Open Container Initiative Runtime Specification](https://github.com/opencontainers/runtime-spec).


IPK format
==========

IPK is a format for software packages. An .ipk file is a specially constructed archive containing three parts:

 * ```data.tar.gz``` : contains data.tar which contains the actual files belonging to this package, organized as they should appear after installation.
 * ```control.tar.gz``` : contains control.tar which contains control file describing the package meta-data and any installation/removal scripts for the package.  
 * ```debian_binary``` : this file is for comparability with Debian's package system and currently is ignored by ipkg.  
 
For further reading about opkg-utils refer [GitHub Yocto project opkg-utils](http://git.yoctoproject.org/cgit/cgit.cgi/opkg-utils).
