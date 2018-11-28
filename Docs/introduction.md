## Arm Mbed Linux OS

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/OS_v_MBL.png" width="50%" align="right">

Arm Mbed Linux OS (MBL) is a free, open-source IoT operating system for Cortex-A devices, based on the embedded Linux Yocto Project. Like Mbed OS, it provides the connectivity protocols and security mechanisms essential for IoT products, so that you can focus on developing your application. Integration with the Pelion IoT Platform offers remote management for large groups of devices.

-or-

Arm Mbed Linux OS (MBL) brings IoT essentials to powerful Cortex-A devices, a step above the constrained, normally single-purpose Cortex-M devices that Mbed OS supports. MBL is based on the embedded Linux Yocto Project, and provides a stable base for your application development, including connectivity protocols, security mechanisms and an integration with the Pelion IoT Platform for remote management of large groups of devices.

The [Arm Cortex-A](https://www.arm.com/products/silicon-ip-cpu) processor gives your IoT device the capability of running multiple complex applications and performing edge computing. MBL provides the set of services that IoT applications rely on, as well as the device management capabilities that field devices require. 


### Developer preview features

Mbed Linux OS is currently available as a developer preview to selected users. To request access, [please contact us](https://os.mbed.com/linux-os/).

The preview release provides:

* OCI-compliant containers for applications, helping to protect against compromised applications and facilitating a modern development workflow.
* Signed updates to protect against unauthorised changes.
* The integration with Pelion Device Management services offers:
    * Support for the Device Management Client for simple in-field provisioning and over-the-air device configuration.
    * Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
    * Large-scale management of device groups.
    * Device status monitoring, including notifications of connection status.
    * Device firmware update and application management.
    * Access control at the account level.

### Design

The following diagram illustrates the components and services provided by MBL. When used in conjunction with Device Management, MBL provides a secure platform for developing, operating and managing IoT applications.

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
