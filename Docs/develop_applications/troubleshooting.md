# Frequently asked questions

## How can I create a container?

You can create an OCI container from a Docker container. The [OCI documentation](https://github.com/opencontainers/runc#creating-an-oci-bundle) explains how to export a Docker container as a bare rootfs directory.

## Can I create containers without using Docker?

If your development environment can't run Docker, you can try another container engine such as [Podman](https://podman.io/).

<span class="notes">**Note** We have not tested these solutions ourselves.</span>.

## Where can I find more information about container configuration?

- [General information about container configuration](https://github.com/opencontainers/runtime-spec/blob/master/config.md)
- [Linux-specific information](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md)

## How can I give my container access to the network?

There are several ways to configure network access for an application. We'll review two:

* Use a network namespace described in the Linux configuration to configure a separate network stack for the container. See:

    * [Setting up a namespace](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md#namespaces)
    * [General information about namespaces](http://man7.org/linux/man-pages/man7/namespaces.7.html)

* Use a device network stack. Note that this requires [removing the network namespace from the `config.json`](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md#namespaces).

If you need to extend other capabilities, see:

* [Capability setting](https://github.com/opencontainers/runtime-spec/blob/master/config.md#linux-process)
* [A list of `CAP_NET_*` settings](http://man7.org/linux/man-pages/man7/capabilities.7.html)

Here is an example of the changes we made to the Hello World `config.json` to allow pinging through the device network:

```
--- config.json	2019-04-26 10:41:12.300363606 +0100
+++ config.json.device-network	2019-04-26 14:21:45.003513809 +0100
@@ -18,7 +18,9 @@
 			"bounding": [
 				"CAP_AUDIT_WRITE",
 				"CAP_KILL",
-				"CAP_NET_BIND_SERVICE"
+				"CAP_NET_BIND_SERVICE",
+				"CAP_NET_RAW",
+				"CAP_NET_BROADCAST"
 			],
 			"effective": [
 				"CAP_AUDIT_WRITE",
@@ -33,7 +35,9 @@
 			"permitted": [
 				"CAP_AUDIT_WRITE",
 				"CAP_KILL",
-				"CAP_NET_BIND_SERVICE"
+				"CAP_NET_BIND_SERVICE",
+				"CAP_NET_RAW",
+				"CAP_NET_BROADCAST"
 			],
 			"ambient": [
 				"CAP_AUDIT_WRITE",
@@ -135,9 +139,6 @@
 				"type": "pid"
 			},
 			{
-				"type": "network"
-			},
-			{
 				"type": "ipc"
 			},
 			{
```


## How can I package my container into an `ipkg`?

You must use the [opkg build tool](https://www.yoctoproject.org/software-item/opkg/) to produce a package suitable for installation on MBL.

You can either:

* Download and install a version [directly from Yocto](http://downloads.yoctoproject.org/releases/opkg/)
* Use the version that we use in our [Hello World tutorial](../develop-apps/hello-world-application.html). After you build Hello World, you can find a script called `build-armv7` or `build-arm64` in the `helloworld` directory. These scripts invoke a Docker container that contains the `opkg-build` tool.

To use opkg:

1. Create a `control` file in a directory called `CONTROL` that describes the package. For example:

    ```
    Package: my-package
    Version: 1.0
    Architecture: any
    Maintainer: myemail@company.com
    Section: base
    Priority: optional
    Description: This package includes my application.
    ```

    Another example of this control file is included in the Hello World application. [Debian documentation has more information about the control file fields](https://www.debian.org/doc/debian-policy/ch-controlfields.html).

1. To build the package: In the directory that contains your `root` rootfs directory and your `CONTROL` directory, run:

    ```
    opkg-build -Z "xz" -g root -o root
    ```

    <span class="notes">**Note:** Prepend the command with `build-armv7` or `build-arm64` when using the Hello World tool. For example `build-armv7 opkg-build -Z "xz" -g root -o root`</span>

## How can I debug container issues?

If you experience issues running your containerized application on your MBL device, you can:

- To investigate issues with the rootfs or `config.json`: edit them, then run the application using the device's application lifecycle script (which you can run with the [MBL CLI shell](../develop-apps/usage.html#remote-command-execution)):

    `mbl-app-lifecycle-manager -v run user-sample-app-package`

- Review the application logs in `/var/log/app/user-sample-app-package.log`, where `user-sample-app-package` is the name of your app.
