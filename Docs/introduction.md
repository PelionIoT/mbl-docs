## Arm Mbed Linux OS

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/OS_v_MBL.png" width="50%" align="right">

Arm Mbed Linux OS (MBL) is a free, open-source IoT operating system based on the embedded Linux Yocto Project. It is designed for [Cortex-A](https://www.arm.com/products/silicon-ip-cpu) devices, which can run multiple, complex applications and perform edge computing. MBL provides the common services these applications rely on, such as access to hardware peripherals, security and connectivity protocols and access to the [Pelion IoT Platform](https://cloud.mbed.com/docs).

Arm Mbed has been supporting the resource constrained Cortex-M devices for years through Mbed OS. By extending this support to Cortex-A devices, MBL brings the strong Mbed IoT base to use cases that require far more computing power, multipurpose applications and hardware interfaces.

The potential complexity of Cortex-A IoT devices increases their attack surface area, so MBL uses Arm's security offerings to protect communication with [Mbed TLS](https://tls.mbed.org/), and the device itself with the [Platform Security Architecture (PSA)](https://developer.arm.com/products/architecture/security-architectures/platform-security-architecture). It also isolates each application in its own OCI-compliant container, so a [compromised application cannot damage other applications or the device](../references/application-containers-and-packages.html).

You can use standard Linux development tools to build MBL applications, which rely on Linux Yocto APIs. This means you can easily adjust desktop Linux applications to work on your IoT devices.

### Developer preview features

MBL is currently available as a developer preview to selected users. To request access, [please contact us](https://os.mbed.com/linux-os/).

The preview release provides:

* OpenEmbedded based OS distribution, enabling extensibility and support for the latest updates and features.
* Secure boot methodology based on Trusted Firmware for both the ARMv7-A and ARMv8-A platforms.
* Trusted Execution Environment (OPTEE) integrated by default, to enable future secure world operations.
* Developer command-line tools to facilitate discovery, setup and local update of development devices.
* A lightweight, feature rich connection manager for Ethernet and Wi-Fi connections.
* OCI-compliant containers for applications, helping to protect against compromised applications and facilitating a modern development workflow.
* The integration with Pelion Device Management services offers:
    * Support for the Device Management Client for in-field provisioning and over-the-air device configuration.
    * Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
    * Large-scale management of device groups.
    * Device management and status monitoring, including notifications of connection status.
    * Over-the-air signed updates for applications and the root file system.
    * Access control at the account level.

### Design

The following diagram illustrates the components and services that MBL provides. When used in conjunction with Device Management, MBL provides a secure platform for developing, operating and managing IoT applications.

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/Application_containers.png" width="75%" align="middle">

### Get started

There are two paths to working with MBL:

1. If you are a Linux developer interested in contributing to MBL or porting it to a new device:
    1. Please [build MBL locally so you can test](../getting-started/tutorial-building-an-image.html).
    1. Use our porting or contributing guides (coming soon).
1. If you are an application developer interested in building applications for devices that run MBL:
    1. Please [build MBL locally and flash it to your device](../getting-started/tutorial-building-an-image.html).
    1. Try our [example application](../getting-started/tutorial-user-application.html), or start writing your own application using the Linux APIs.
    1. You can also [use our update example](../getting-started/tutorial-updating-mbl-devices-and-applications.html) to send firmware updates over the air.

### Licensing

Both MBL and our test suites will be open source, helping you automate product testing in a modern continuous integration pipeline. For more information, please see the [Contributing section in the MBL source](https://github.com/ARMmbed/meta-mbl/blob/master/CONTRIBUTING.md).
