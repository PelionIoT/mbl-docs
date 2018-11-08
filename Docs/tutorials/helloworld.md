
Introduction
============

Hello world application is a sample C application printing "Hello, word" to the STDOUT.
The application consists of hello_world.c C file located under the source ./src directory.  

The application will run on target inside runc container defined by the Runtime Specification of  
OCI (Open Container Initiative).  

The OCI Runtime Specification specifies the configuration, execution  
environment and lifecycle of a container. A container's configuration is specified in the ./src_bundle/config.json  
for the supported platforms and details the fields that enable the creation of a container.  

The execution environment is specified to ensure that applications running inside a container
have a consistent environment between runtimes along with common actions defined for the container's lifecycle.  
  
The Runtime Specification defines format for encoding a container as a "filesystem bundle" - a set of files organized  
in a certain way, and containing all the necessary data and metadata for any compliant runtime to perform all  
standard operations against it. A Standard Container bundle contains all the information needed to load and  
run a container. This includes the following artifacts:

 * config.json: contains configuration data.
 * container's root filesystem (rootfs): the directory referenced by root.path, if that property is set in config.json.

The OCI Runtime Specification outlines how to run a filesystem bundle that is unpacked on disk.
For further reading about runc container see [GitHub Open Container Initiative Runtime Specification](https://github.com/opencontainers/runtime-spec).

Application installation, update and removal is done using opkg-utils helper scripts utilizing opkg (Open PacKaGe management).  
opkg-utils build ipk package which will be installed on device.  
For further reading about opkg-utils refer [GitHub Yocto project opkg-utils](http://git.yoctoproject.org/cgit/cgit.cgi/opkg-utils).

Cross compiling toolchains in Docker image is built using the parent image defined in ./cc-env/Dockerfile.  
For further reading about dockcross see [GitHub cross compiling toolchains in Docker images ](https://github.com/dockcross/dockcross).

The application is cross-compiled using dockcross image (cross compiling toolchains in Docker image).

Then, build commands defined in Makefile could be run. Makefile consists of three sections:

 * Cross-compilation build
 * Create OCI bundle
 * Create IPK.
 
Each section is implemented for both release and debug variants.


Build an image from a Dockerfile            
================================            
            
1. On Linux Ubuntu PC install Docker CE as described here:                
            
    ```
    https://docs.docker.com/install/linux/docker-ce/ubuntu/   
    ```         
            
1. cd mbl-core/tutorials/helloworld            
               
1. Build dockcross (Cross compiling toolchains in Docker images). The parent  
   image is defined in Dockerfile: dockcross/linux-armv7. Docker build command  
   prepares cross-compilation environment for building  user sample application  
   inside the container.              
         
    ```      
    docker build -t linux-armv7:latest ./cc-env/    
    ```      
     
1. The image created by a previous docker build command does not need to be  
   run manually. Instead, there is a helper script to execute build commands  
   on source code existing on the local host filesystem. This script is bundled  
   with the image.  
   To install the helper script, run the image with no arguments, and redirect  
   the output to a file:  
  
    ```
    docker run --rm linux-armv7 > build-armv7  
    chmod +x build-armv7  
    sudo mv build-armv7 /usr/local/bin  
    ```
              
Build "helloworld" application with cross compiler            
==================================================            
          
Build produces IPK file:            
          
   * For release: ./release/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk          
   * For debug: ./debug/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk      
     
In order to build, invoke make toolchain command:        
            
   * build-armv7 make all       // build both release and debug              
   * build-armv7 make release   // build release              
   * build-armv7 make debug     // build debug              
   * build-armv7 make clean     // clean the build              
             
               
Run on target            
=============            
            
1. Create user-sample-app-package_1.0_armv7vet2hf-neon.tar:  
  
    ```
    tar -cvf user-sample-app-package_1.0_armv7vet2hf-neon.tar user-sample-app-package_1.0_armv7vet2hf-neon.ipk  
    ```
   
    This command can be done for release/debug variants separately.
            
1. Copy user-sample-app-package_1.0_armv7vet2hf-neon.tar to the device under /scratch.  
            
1. Extract, install the and run ipk using app_update_manager.py script:    
  
    ```
    mbl-app-update-manager -i /scratch/user-sample-app-package_1.0_armv7vet2hf-neon.tar  
    ```
             
1. Also, user sample application will run after rebooting the device.

