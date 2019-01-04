# Mbed Linux OS Example - QR Scanner

<span class="notes">**Note**: Mbed Linux OS (MBL) is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

This tutorial creates a user application that runs on MBL devices. It is an application written in Python, which scans QR codes and can upload them to a database.

<span class="notes">**Note:** Your device must already be running an MBL image. Please [follow the tutorial](https://os.mbed.com/docs/linux-os/v0.5/getting-started/tutorial-building-an-image.html) if you don't have an MBL image yet.</span>

## Dockerizing the application

On the device, the application runs on a target inside an OCI container. To create the OCI container, we dockerize the [`example-qr`](example-qr/) python package on a host machine.

Run the following command to dockerize the application:
```
user01@dev-machine:mbl-example-qr$ sh scripts/dockerize-example-qr-app.sh
```
The script builds a docker image which includes an installation of the example-qr application. The filesystem contained in the image is then exported then extracted to [`release/runtime-bundle-filesystem`](release/runtime-bundle-filesystem).


## Setting up the cross-compilation tools

The device expects to unpack an IPK from an application update payload in order to install the application. The IPK is built on a host machine using cross-compiling toolchains from a dockcross Docker image for ARMv7 architecture.

Run the following command to setup the cross-compilation tools:
```
user01@dev-machine:mbl-example-qr$ sh scripts/setup-cross-compilation-tools.sh
```
The script builds a Docker image with dockcross, adding the `opkg-utils` utility. It also installs a helper script (`build-armv7`) to execute build commands.


## <a name="creating-oci-bundle-and-ipk"></a> Creating the OCI bundle and generate the IPK

Run the following command to build the OCI runtime bundle and generate the IPK:
```
user01@dev-machine:mbl-example-qr$ build-armv7 make release
```
The build produces an IPK file at [`release/ipk/mbl-example-qr_1.0_armv7vet2hf-neon.ipk`](release/ipk/mbl-example-qr_1.0_armv7vet2hf-neon.ipk).

## Installing and running the application on the device

To install the application on the device:

* [Send the application as an over-the-air firmware update with Device Management](https://os.mbed.com/docs/linux-os/v0.5/getting-started/tutorial-updating-mbl-devices-and-applications.html).
* [Flash the application over USB with MBL CLI](https://os.mbed.com/docs/linux-os/v0.5/tools/device-update.html#update-an-application).

Run the following script to generate the application update payload to upload to Device Management or to the MBL CLI tool:
```
user01@dev-machine:mbl-example-qr$ sh scripts/build-app-update-payload.sh
```

After installation, and after every reboot, the application turns on the video camera and awaits for QR codes to be presented.

Saved QR codes can be observed with the following command:
```
root@mbed-linux-os-1438:~# tail -f /home/app/mbl-example-qr/rootfs/home/scanned_qrcodes.log
```

MBL redirects the logging information to the log file `/var/log/app/mbl-example-qr.log`.


## Uploading QR codes to Treasure Data

1. Before [creating the OCI bundle](#creating-oci-bundle-and-ipk), obtain a Treasure Data API Key (see how [here](example-qr/README.md#uploading-qrcodes)).
1. Modify [`src_bundle/config.json`](src_bundle/config.json) to add an optional argument to pass the API key. See [`example-qr/example_qr/cli.py`](example-qr/example_qr/cli.py) for more information.

## Cleaning the build artefacts

Run the following command to clean the content of the [`release`](release/) directory:
```
user01@dev-machine:mbl-example-qr$ make clean
``` 


## Notes

* The Raspberry Pi 3 camera does not work with the application.
* Logitech webcam series is the recommended web camera.
* Prefix the commands mentioned above with `sudo` if permission is denied.
* The `release` directory is created by the [`dockerize-example-qr-app.sh`](scripts/dockerize-example-qr-app.sh) script.


## License

Please see the [License][mbl-license] document for more information.


## Contributing

Please see the [Contributing][mbl-contributing] document for more information.


[mbl-license]: LICENSE.md
[mbl-contributing]: CONTRIBUTING.md
