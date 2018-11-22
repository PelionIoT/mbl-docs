## Arm Mbed Linux OS

<!--managed operating system, built in security, ability to deploy and keep updating millions of devices, focus on just writing the app code rather than building their own os and maintanining and managing it-->

### Product vision

Arm Mbed Linux OS starts with a vision of a world with billions of internet-connected devices, and works backwards to understand how to bring these devices into being. Mbed assumes that IoT product designers and makers do not want to invest time, money and expertise in creating and maintaining operating systems, or their own builds of existing Linux projects; they want to focus on commercial applications that can rely on an IoT operating system delivered by a specialised company. Arm Mbed OS has been supporting productisation of constrained Cortex-M devices for years, and Mbed Linux OS now offers the same for the powerful Cortex-A devices.

The vision of billions of connected devices immediately led us to consider security. Devices must be secure from day one, and must receive security updates whenever those become available. So we drew on existing Arm products to provide device and connectivity security, and device management for millions of devices through the Pelion IoT Platform.



<!--Arm Mbed Linux OS (MBL) is a Linux distribution for IoT devices based on the Cortex-A processor family. It provides all IoT essentials, so you can focus exclusively on adding your unique value and on building quickly, with minimal support and maintenance.-->

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/OS_v_MBL.png" width="50%" align="right">

<!--
MBL combines the benefits of Arm Mbed OS with the tools and recipes from the embedded-focused Yocto Project. It takes advantage of the processing power of Cortex-A devices to handle complex workloads (reducing connectivity demands). MBL includes support for common IoT functions such as network management, secure storage and container management, as well as a standard Linux inter-process communication (IPC) framework for both MBL and application processes. MBL uses the Device Management Client to integrates with Pelion Device Management, including the firmware update services.

### Product vision

MBL will provide a uniform, open and verifiable approach to platform security. Each supported platform has:

* Secure, signed boot of Trusted Firmware, used for initializing trusted world firmware, including the trusted execution environment.
* Verified boot of normal world firmware.
* Independent verification of application packages.
* Integrity checking of read-only file systems.

It will also draw on other Arm resources to provide:

* Arm's Platform Security Architecture, which takes advantage of the security features of the newest hardware.
* Full commercial support for customers who need a firm SLA and platform longevity.
* Tools to help you get prototyping and demoing quickly, including support for popular development boards and production ready modules.
-->

### Developer preview features

The preview release provides:

* OCI-compliant containers for applications, helping to protect against compromised applications and facilitating a modern development workflow.
* Signed updates to protect against unauthorised changes.

And an integration with Pelion Device Management services:

* Support for the Device Management Client for simple in-field provisioning and over-the-air device configuration.
* Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
* Large-scale management of device groups.
* Device status monitoring, including notifications of connection status.
* Device firmware update and application management.
* Access control at the account level.

### Design

The following diagram illustrates the components and services provided by MBL. When used in conjunction with Mbed Cloud, MBL provides a secure platform for developing, operating and managing IoT applications.

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/Application_containers.png" width="75%" align="middle">

### Get started

There are two paths to working with MBL:

1. If you are a Linux developer, and interested in contributing to MBL or porting it to a new device:
    1. Please [build MBL locally so you can test](../getting-started/tutorial-connecting-an-mbl-device.html).
    1. Use our porting or contributing guides (coming soon).
1. If you are an application developer, and interested in building applications for devices that run MBL:
    1. Please [build MBL locally and flash it to your device](../getting-started/tutorial-connecting-an-mbl-device.html).
    1. Try our [example application](../getting-started/tutorial-user-application.html), or start writing your own application using the Linux APIs.
    1. You can also [use our update example](../getting-started/tutorial-updating-mbl-devices-and-applications.html) to send firmware updates over the air.

### Licensing

Both Mbed Linux OS and our test suites will be open source, helping you automate product testing in a modern continuous integration pipeline. For more information, [please see the Contributing section in the MBL source](https://github.com/ARMmbed/meta-mbl/blob/master/CONTRIBUTING.md).
