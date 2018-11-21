## Mbed Linux OS

Arm Mbed Linux OS (MBL) is a Linux distribution for IoT devices based on the Cortex-A processor family. Mbed Linux OS supports all of the essential layers needed to build amazing IoT products, so you can focus exclusively on adding your unique value. We want you to build products quickly, and minimise the ongoing support and maintenance burden for products you’ve already shipped.

MBL combines the benefits of Arm Mbed OS with the tools and recipes from the embedded-focused Yocto Project. It is designed to take advantage of the processing power of Crotex-A devices, which can handle complex workloads (reducing connectivity demands). It includes support for common IoT functions such as network management, secure storage and container management, as well as a standard Linux inter process communication (IPC) framework for both MBL and application processes. MBL also integrates with Pelion Device Management, including the firmware update services, by using the Device Management Client.

### Main features

MBL provides a uniform, open and verifiable approach to platform security. Each supported platform has:

* Secure, signed boot of Trusted Firmware, used for initializing trusted world firmware, including the trusted execution environment.
* Arm's Platform Security Architecture, which takes advantage of the security features of the newest hardware.
* Verified boot of normal world firmware.
* Signed updates to protect against unauthorised changes.
* Independent verification of application packages.
* Integrity checking of read-only file systems.
* Applications can be deployed in OCI-compliant containers, helping to protect against compromised applications and facilitating a modern development workflow.

MBL also offers:

* The option of full commercial support for customers who need a firm SLA and platform longevity.
* Tools to help you get prototyping and demoing quickly, including support for popular development boards and production ready modules.
* Support for the Device Management Client, to simplify in-field provisioning and eradicate the need for legacy serial connections for initial device configuration.
* Integration with the Pelion service for device provisioning, connectivity and updates. To allow different development teams to deliver updates more efficiency, the OS firmware and individual applications can be updated independently.

### Get started

There are two paths to working with MBL:

1. If you are a Linux developer, and interested in contributing to MBL or porting it to a new device:
    1. Please [build MBL locally so you can test](../getting-started/tutorial-connecting-an-mbl-device.html).
    1. Use our porting or contributing guides (coming soon).
1. If you are an application developer, and interested in building applications for devices that run MBL:
    1. Please [build MBL locally and flash it to your device](../getting-started/tutorial-connecting-an-mbl-device.html).
    1. Try our [example application](../getting-started/tutorial-user-application.html), or start writing your own application using the Linux APIs.
    1. You can also [use our update example](../getting-started/tutorial-updating-mbl-devices-and-applications.html) to send firmware updates over the air.

### Design

The following diagram illustrates the components and services provided by MBL. When used in conjunction with Mbed Cloud, MBL provides a secure platform for developing, operating and managing IoT applications.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/Application_containers.png)</span>

### Containers

To enhance application development and deployment, each device application process can run in its own container. The process isolation provided by containers also increases security and device reliability. It is based on OCI containers, which you can make using Docker on the host (creating a cross-compiled build of the application) and export.


### Pelion Device Management through MBL

For application developers, the Device Management layer provides the following services:

* Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
* Membership of a group of managed devices (Pelion Device Management) to simplify large-scale management
* Access control at the account level.
* Device status monitoring, including notifications of connection status.
* Device firmware update and application management

MBL uses the Pelion Device Management Update service to support updates of both the device's base firmware and application packages.

### Licensing

Both Mbed Linux OS and our test suites will be open source, helping you automate product testing in a modern continuous integration pipeline. For more information, [please see the Contributing section in the MBL source](https://github.com/ARMmbed/meta-mbl/blob/master/CONTRIBUTING.md).
