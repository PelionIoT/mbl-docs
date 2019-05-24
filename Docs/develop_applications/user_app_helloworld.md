# Hello world

<span class="notes">**Note**: Mbed Linux OS is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

This tutorial creates a user application that runs on your MBL device. It is a "Hello, World" in C, which prints to standard output (STDOUT). MBL redirects the output to the log file `/var/log/app/user-sample-app-package.log`. This application demonstrates the use of dockcross and MakeFile to build a self-contained application: it requires no access to device resources (not even the runtime library) and no dockerization. This is not the only way to work with applications; we chose it because it's simple - using dockcross means we didn't need to install anything - and creates a small container, because there are no libraries or support files needed.

<span class="notes">**Note:** Your device must already be running an MBL image. Please [follow the instructions](../first-image/index.html) if you don't have an MBL image yet.</span>

## Source and build mechanisms overview

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/hello_world.png" width="50%" align="right" />

The application source file, `hello_world.c`, is in [https://github.com/ARMmbed/mbl-core/tree/mbl-os-0.6/tutorials/helloworld/src](https://github.com/ARMmbed/mbl-core/tree/mbl-os-0.6/tutorials/helloworld/src). On the device, the application runs on a target inside an OCI container.

The build commands are defined in a Makefile with three sections:

* Cross-compilation build.
* Create OCI bundle.
* Create IPK.

Each section is implemented for both release and debug variants.

We use the existing dockcross Docker image (for the ARMv7 or ARM64 architecture) to cross-compile the application and create an OCI container.

To the standard dockcross image we add `opkg-utils` helper scripts that can package the compiled application as an IPK. The Docker files to build the dockcross image for supported architectures are found at [https://github.com/ARMmbed/mbl-core/blob/mbl-os-0.6/tutorials/helloworld/cc-env](https://github.com/ARMmbed/mbl-core/blob/mbl-os-0.6/tutorials/helloworld/cc-env).

For more information, please refer to the [reference about application containers and packages](../references/application-containers-and-packages.html) or the [dockcross documentation on GitHub](https://github.com/dockcross/dockcross).

## Setting up the repository on the development machine

1. On a Linux Ubuntu PC, install Docker CE as described in the [Docker CE for Ubuntu documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

1. Clone the `mbl-core` repo:

    ```
    $ git clone git@github.com:ARMmbed/mbl-core.git --branch mbl-os-0.6
    ```

1. Navigate to the `mbl-core` project.

## Building the application with the cross-compiler

[dockcross](https://github.com/dockcross/dockcross) is a Docker-based cross-compiling toolchain. We build a new Docker image with dockcross as the starting point, adding the `opkg` utilities. The image is defined in `./tutorials/helloworld/cc-env/<arm-arch>/Dockerfile`, where `<arm-arch>` is the architecture type of the microprocessor on the target device:

| Target device | `<arm-arch>` value |
| --- | --- |
| NXP 8M Mini EVK | `arm64` |
| PICO-PI with IMX7D | `armv7` |
| Raspberry Pi 3 | `armv7` |
| Warp7 | `armv7` |

You can use the built image to generate a short-lived container to compile the application in. Do this by running the container and capturing the output as an executable file. The Make toolchain is then invoked along with the cross-compilation executable file to build a variant (release or debug) of the Hello World application.

**To build the application with all of the steps explained above:**

```
$ ./ci/lava/tests/lxc-build-helloworld.sh <arm-arch>
```

The build produces:

* An IPK file at `./tutorials/helloworld/release/ipk/user-sample-app-package_1.0_any.ipk`.
* A Pelion Device Management update payload package at `/tmp/user-sample-app-package_1.0_any.ipk.tar`.

<span class="tips">**Tip**: If you want to clean the build, run: `./tutorials/helloworld/build-<arm-arch> make clean`</span>

## Installing and running the application on the device

There are two ways to install the application on the device:

* [Send the application as an over-the-air firmware update with Pelion Device Management](../update/updating-an-application.html#using-the-manifest-tool).
* [Flash the application over USB with MBL CLI](../update/updating-an-application.html#using-mbl-cli).

## Using the application

After installation, and after every reboot, the application runs once and writes `hello world` to the STDOUT. MBL redirects the output to the log file `/var/log/app/user-sample-app-package.log`.
