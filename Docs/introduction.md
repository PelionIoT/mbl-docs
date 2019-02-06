## Arm Mbed Linux OS

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/OS_v_MBL.png" width="25%" align="right" />

Arm Mbed Linux OS (MBL) is a free, open-source IoT operating system based on the embedded Linux Yocto Project. It is designed for [Cortex-A](https://www.arm.com/products/silicon-ip-cpu) devices, which can run multiple, complex applications and perform edge computing. MBL provides the common services these applications rely on, such as access to hardware peripherals, security and connectivity protocols and access to the [Pelion IoT Platform](https://cloud.mbed.com/docs).

Arm Mbed now supports all IoT device classes with a duo of operating systems: Mbed OS for Cortex-M microprocessors and Mbed Linux OS for Cortex-A microprocessors.

Because MBL is aimed specifically at IoT devices, it places a major emphasis on platform security. At the core of device security is Trusted Firmware (TF-A) and OP-TEE, an open source trusted execution environment that conforms to the Global Platform TEE specification. MBL also conforms to [Platform Security Architecture (PSA)](https://developer.arm.com/products/architecture/security-architectures/platform-security-architecture) for secure boot and other security measures, and relies on the Linux kernel's isolation mechanisms to protect device integrity and sensitive data: each application runs in its own OCI-compliant container, so a compromised application cannot damage other applications or the device.

You can develop and build applications in using a variety of standard tools such as C cross-compilation, Python or Node.js. Applications are then packaged and deployed to MBL in an application container.

### Developer preview features

MBL is currently available as a developer preview to selected users. To request access, [please contact us](https://os.mbed.com/linux-os/).

The preview release provides:

* An OpenEmbedded-based OS distribution, enabling extensibility and support for the latest updates and features.
* Hardware and software-based isolation mechanisms for security:
    * Dedicated hardware within most Cortex-A devices enforces the most secure isolation boundary. This technology, called TrustZone, allows the most sensitive code and data to run within a so-called **Secure World**. MBL includes a trusted operating system for this Secure World called **Open Source Trusted Execution Environment** (OP-TEE). OP-TEE is essentially an operating system within an operating system, which is loaded by the Trusted Firmware and would typically be used to protect cryptographic keys and other sensitive data assets.
    * Secure boot methodology based on Linaro's Trusted Firmware A for both the ARMv7-A and ARMv8-A platforms. Trusted Firmware is a minimal secure bootloader that runs when a Cortex-A microprocessor is executing in TrustZone's "secure world" mode.
    * Open Container Initiative (OCI)-compliant containers for applications, protecting against compromised applications and facilitating a modern development workflow. Based on the RUNC container runtime, this Docker-like system allows OEMs to package services or applications as independent container images that run entirely within a sandbox. This provides two benefits. First, if an attacker compromises a single application, it's far harder for the attack to spread beyond the infected container. This will help to reduce the impact of an attack and ensure a device can easily be restored to a secure state. Second, applications can be developed independently of the underlying IoT platform. For example, it's easier for a developer to build and test on their desktop workstation or laptop.
* Developer command-line tools to facilitate discovery, setup and local update of development devices.
* A lightweight, feature rich connection manager for Ethernet and Wi-Fi connections.
* The integration with Pelion Device Management services offers:
    * Support for the Device Management Client for in-field provisioning and over-the-air device configuration.
    * Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
    * Large-scale management of device groups.
    * Device management and status monitoring, including notifications of connection status.
    * Access control at the account level.
    * Over-the-air, digitally signed and verified updates for applications and the root file system. MBL uses updates to send new features and security patches to deployed devices, fixing vulnerabilities before they're exploited.

### Design diagram

The following diagram illustrates the components and services that MBL provides:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/Application_containers.png)<span>When used in conjunction with Device Management, MBL provides a secure platform for developing, operating and managing IoT applications</span></span>

The **application management** framework manages installing and running separate applications:

* Package management for installing and updating application packages.
* Life-cycle management for running instances of an application.
* Core application configuration for configuring the set of core applications that are run automatically when a device boots.

The **platform services** are general purpose services common across most IoT applications:

* The network manager uses ConnMan to manage network connections.
* Modem management for cellular connections (future).
* Local connectivity, for example BLE for connection to local devices.
* A logging framework to tag and forward logged events.
* Secure services to protect assets: secrets storage and cryptographic operations with access control.

MBL uses an integrated agent to connect to the **Pelion Device Management services**. The services will provide secure remote ownership, update and monitoring for devices using the Device Management Client.

**Management agents** extend the way in which a device is managed. They are user-space agents providing:

* Additional device configuration access:

    * Over BLE (in a future release).
    * Remote cloud-based (via the Pelion Connect service).
    * Over USB (during manufacturing).

* Trusted Execution Environment and OP-TEE provide an environment for running security-sensitive applications within the isolated environment provided by Arm TrustZone.

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
