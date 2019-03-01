# Application code

<span class="notes">**Note**: Mbed Linux OS (MBL) is currently in limited preview. If you would like access to the code repositories, [please request to join the preview](https://os.mbed.com/linux-os/).</span>

## Source code

The application code and scripts are in the [https://github.com/ARMmbed/mbl-example-qr](https://github.com/ARMmbed/mbl-example-qr) repository on GitHub. Please download or clone it.

## Code review

The application's core functionality is in the [`app.py`](https://github.com/ARMmbed/mbl-example-qr/blob/master/example-qr/example_qr/app.py) file. We also include a standard `cli.py` file to handle CLI inputs when the application is started in a shell.

We use the `src_bundle/config.json` file to let the application interact with the device and peripherals across its container borders. The example below shows how the application accesses the video camera and the GPIO memory. You can modify the config file to adjust the reference application to your own device. For example, if your camera mounts on anything other than `/dev/video0`, or if the device's `major/minor` (81/0) are different, update the values in `src_bundle/config.json`:

```
"linux": {
     "devices": [{
         "path": "/dev/video0",
         "type": "c",
         "major": 81,
         "minor": 0,
         "uid": 0,
         "gid": 0
     }, {
         "path": "/dev/gpiomem",
         "type": "c",
         "major": 81,
         "minor": 0,
         "uid": 0,
         "gid": 0
     }],

```

The config file also maps the container's memory to the device's persistent memory, so that your QR codes remain in memory even after you stop the application (a containerized application does not save data between runs). You can adjust the application code to use the existing mapping:

```
{
    "destination": "/tmpC",
    "type": "ext4",
    "source": "/tmp",
    "options": [
        "rw",
        "bind"
    ]
},
```

You can also modify the build process. The Docker image requires an installation of OpenCV for capturing frames from the camera. OpenCV needs to be built from the source, because it is not available from the package manager of the base image OS. Building OpenCV requires adding multiple build tools to the Docker image, which increases its size. To reduce the size, the Dockerfile uses a multi-stage build, and copies only the built OpenCV and its dependencies to the final image. You can build on this functionality to reduce your image build time: create a Docker base image in Docker Hub, which you can then use to combine the the source code of the nested project (`example-qr`) with a built OpenCV to create a new image, and install only that image.
