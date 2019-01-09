## Mbed Linux OS Example - QR Scanner

<span class="notes">**Note**: Mbed Linux OS (MBL) is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span><!--is this going out with 0.5? If not, it won't need this note for 1.0-->

This tutorial creates a Python user application that runs on MBL devices and scans QR codes. By default, the application decodes the QR code to UTF-8 values and saves these values to a log. You can also use the application to upload the decoded QR codes to an Arm Treasure Data account.

### Requirements and hardware notes

* Your device must already be running an MBL image. Please [follow the tutorial](../getting-started/tutorial-building-an-image.html) if you don't have an MBL image yet.
* You must still have all [tool and accounts prerequisites](../getting-started/setting-up-and-supported-hardware.html).
* Cameras:
    * This application does not work with the Raspberry Pi 3 camera.
    * We recommend using the Logitech webcam series.

<!--do we expect this to work with any board, or just the Warp7?-->

### License

Please see the [License](https://github.com/ARMmbed/mbl-example-qr/blob/master/LICENSE.md) document on GitHub for more information.

### Contributing

If you want to contribute additional functionality or fixes to this application, please see the [Contribution guidelines](../references/contribution-guidelines.html).

## Application code

<span class="notes">**Note**: Mbed Linux OS (MBL) is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span><!--is this going out with 0.5? If not, it won't need this note for 1.0-->

### Code source

The application code and scripts are in the [https://github.com/ARMmbed/mbl-example-qr](https://github.com/ARMmbed/mbl-example-qr) repository on GitHub.

Please clone the repo to your development machine:

```
git clone git@github.com:ARMmbed/mbl-example-qr
```

### Code review

The application's core functionality is in the [`app.py`](https://github.com/ARMmbed/mbl-example-qr/blob/master/example-qr/example_qr/app.py) file<!--what do all the other files do? dockerize things and stuff like that?-->. The application is not MBL-specific; it can run on any operating system. To use it on an MBL device, we wrap the application in an OCI container.<!--what does https://github.com/ARMmbed/mbl-example-qr/blob/master/example-qr/example_qr/cli.py do? how does it interact with the main application?-->

<!--can we talk a little bit about the code? What are some guiding principles or best practices you used? Some interesting choices you had to make? How would users approach modifying it for their own things?-->

[![View code](https://www.mbed.com/embed/?url=https://github.com/ARMmbed/mbl-example-qr/blob/master/example-qr/example_qr/)](https://github.com/ARMmbed/mbl-example-qr/blob/master/example-qr/example_qr/app.py)

## Build and usage instructions

<span class="tips">**Tip**: Prefix the commands in these instructions with `sudo` if permission is denied.<!--not wild about this sentence--></span>

### Dockerizing the application

On the device, the application runs on a target inside an OCI container. To create the OCI container, we dockerize the `example-qr` Python package on a host machine.

To dockerize the application, run:

```
user01@dev-machine:mbl-example-qr$ sh scripts/dockerize-example-qr-app.sh
```

The script:

1. Builds a docker image that includes an installation of the `example-qr` application.
1. Creates the `release` directory.
1. Exports and extracts the filesystem contained in the image to `release/runtime-bundle-filesystem`.

### Setting up the cross-compilation tools

To install the application, the device unpacks an IPK from an application update payload. To build the IPK, use a host machine and cross-compiling toolchains from a dockcross Docker image we provide for ARMv7 architecture. We provide the image in the application's repository.

To set up the cross-compilation tools, run:

```
user01@dev-machine:mbl-example-qr$ sh scripts/setup-cross-compilation-tools.sh
```

The script builds a Docker image with dockcross, adding the `opkg-utils` utility. It also installs a helper script (`build-armv7`) to execute build commands.

### Creating the OCI bundle and generating the IPK

To build the OCI runtime bundle and generate the IPK, run:

```
user01@dev-machine:mbl-example-qr$ build-armv7 make release
```

The build produces an IPK file at `release/ipk/mbl-example-qr_1.0_armv7vet2hf-neon.ipk`.

### Installing and running the application on the device

To install the application on the device:

1. Generate the application update payload:

    ```
    user01@dev-machine:mbl-example-qr$ sh scripts/build-app-update-payload.sh
    ```
1. Choose an installation option and follow its tutorial:

    * [Send the application payload as an over-the-air firmware update with Device Management](../getting-started/tutorial-updating-mbl-devices-and-applications.html).
    * [Flash the application payload over USB with MBL CLI](../tools/device-update.html#update-an-application).

After installation, and after every reboot, the application turns on the video camera and waits for QR codes to be presented. It captures one or more QR codes, decodes them to UTF-8, and adds them to the log, in a file named `scanned_qrcodes.log`.

To retrieve previously scanned QR codes:

```
root@mbed-linux-os-1438:~# tail -f /home/app/mbl-example-qr/rootfs/home/scanned_qrcodes.log
```

<span class="tips">**Tip**: MBL redirects the logging information to the log file `/var/log/app/mbl-example-qr.log`.</span>
<!--this contradicts the previous line, which pointed to scanned_qrcodes.log (), and I also see only scanned_qr_codes.log in the app.py-->

### Optional: uploading QR codes to Treasure Data
<!--Really? Exciting!-->
You can modify the application to upload decoded QR codes to Arm Treasure Data.

1. Before [creating the OCI bundle](#creating-oci-bundle-and-ipk):

    1. Create or log into [your Arm Treasure Data account](https://console.treasuredata.com/).
    1. From the Treasure Data console, create a database for your uploaded QR Codes. [Please follow the Treasure Data guidelines](https://support.treasuredata.com/hc/en-us/articles/360001266348-Database-and-Table-Management).

    <span class="notes">**Note**: Your database must include at least one table; the table must include at least one column.</span>

    1. Retrieve your [Treasure Data **master** API Key](https://support.treasuredata.com/hc/en-us/articles/360000763288-Get-API-Keys).


1. In `src_bundle/config.json`, add an optional argument to pass the API key as `TD_API_KEY`. <!--See [`example-qr/example_qr/cli.py`](example-qr/example_qr/cli.py) for more information.--><!--please don't send them to read the code for a single command - just tell them how to do it-->

### Cleaning the build artefacts

<!--when and why?-->
To clean the content of the [`release`](release/) directory, run:

```
user01@dev-machine:mbl-example-qr$ make clean
```
