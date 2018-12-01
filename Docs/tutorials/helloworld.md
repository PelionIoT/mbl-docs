## Hello world

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

This tutorial creates a user application that runs on your MBL device. It is a simple "Hello, World" in C, which prints to standard output (STDOUT). MBL redirects the output to the log file `/var/log/app/user-sample-app-package.log`.

<span class="notes">**Note:** Your device must already be running an MBL image. Please [follow the first tutorial in this series](../getting-started/tutorial-building-an-image.html) if you don't have an MBL image yet.</span>

### Source and build mechanisms overview

The application source file, `hello_world.c`, is in [https://github.com/ARMmbed/mbl-core/tree/master/tutorials/helloworld/src](https://github.com/ARMmbed/mbl-core/tree/master/tutorials/helloworld/src). On the device, the application runs on a target inside an OCI container. To create the OCI container, we use [dockcross](https://github.com/dockcross/dockcross) to cross-compile the application, then the `opkg-utils` helper scripts to package the compiled application as an IPK.

Cross-compiling toolchains in the Docker image are built using an existing dockcross Docker image for the ARMv7 architecture. The `opkg-utils` helper scripts are included in the toolchain Docker container, which you can find at [https://github.com/ARMmbed/mbl-core/blob/master/tutorials/helloworld/cc-env/Dockerfile](https://github.com/ARMmbed/mbl-core/blob/master/tutorials/helloworld/cc-env/Dockerfile).

The build commands are defined in a Makefile with three sections:

* Cross-compilation build.
* Create OCI bundle.
* Create IPK.

Each section is implemented for both release and debug variants.

<span class="notes">**Note:** For more information, please refer to the [reference about application containers and packages](../references/application-containers-and-packages.html) or the [dockcross documentation on GitHub](https://github.com/dockcross/dockcross).</span>

### Setting up the cross-compilation tools

1. On a Linux Ubuntu PC, install Docker CE as described in the [Docker CE for Ubuntu documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

1. Navigate to the `helloworld` folder in your clone:

    ```
    cd mbl-core/tutorials/helloworld
    ```

1. [dockcross](https://github.com/dockcross/dockcross) is a Docker-based cross-compiling toolchain. We build a new Docker image with dockcross as our starting point, adding the opkg utilities. The image is defined in `cc-env/Dockerfile`. You can build it with `docker build`:

    ```
    docker build -t linux-armv7:latest ./cc-env/
    ```    
1. The freshly built image can generate a script that makes it easy to work with. Run the image and capture the output to a file. Make the file an executable and place it in your path:

    ```
    docker run --rm linux-armv7 > build-armv7  
    chmod +x build-armv7  
    sudo mv build-armv7 /usr/local/bin  
    ```

### Building the application with the cross-compiler

To build, invoke the Make toolchain command: `build-armv7 make release`.

The build produces an IPK file at `./release/ipk/user-sample-app-package_1.0_armv7vet2hf-neon.ipk`.

To clean the build, run: `build-armv7 make clean`

### Installing and running the application on the device

To install the application on the device, follow one of our tutorials:

* [Send the application as an over-the-air firmware update](../getting-started/tutorial-updating-mbl-devices-and-applications.html).
* [Use MBL CLI to flash the application over USB](../tools/mbl-cli.html).
