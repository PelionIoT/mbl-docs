# Arm Mbed Linux OS

<img src="https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/OS_v_MBL.png" width="25%" align="right" />

Arm Mbed Linux OS (MBL) is a free, open-source IoT operating system based on the embedded Linux Yocto Project. It is designed for [Cortex-A](https://www.arm.com/products/silicon-ip-cpu) devices, which can run multiple, complex applications and perform edge computing. MBL provides the common services these applications rely on, such as access to hardware peripherals, security and connectivity protocols and access to the [Pelion IoT Platform](https://cloud.mbed.com/docs).

Arm Mbed now supports all IoT device classes with a duo of operating systems: Mbed OS for Cortex-M microprocessors and Mbed Linux OS for Cortex-A microprocessors.

Because MBL is aimed specifically at IoT devices, it places a major emphasis on platform security. At the core of device security is Trusted Firmware (TF-A) and OP-TEE, an open source trusted execution environment that conforms to the Global Platform TEE specification. MBL also conforms to [Platform Security Architecture (PSA)](https://developer.arm.com/products/architecture/security-architectures/platform-security-architecture) for secure boot and other security measures, and relies on the Linux kernel's isolation mechanisms to protect device integrity and sensitive data: each application runs in its own OCI-compliant container, so a compromised application cannot damage other applications or the device.

You can develop and build applications in using a variety of standard tools such as C cross-compilation, Python or Node.js. Applications are then packaged and deployed to MBL in an application container.

<section class="row">
<div class="columns large-6 medium-6 small-12">
  <h3>On this page</h3>
  <ul class="guides-list">
          <ul data-tab-content>
                <li><a href="#preview">Public preview features</a></li>
                <li><a href="#design">Design diagram</a></li>
                <li><a href="#getting-started">Getting started</a></li>
                <li><a href="#doc-updates">Recently updated documentation</a></li>
            </ul>
    </ul>
</div>
</section>

<h2 id="preview">Public preview features</h2>

MBL is currently available as a public preview.

The preview release provides:

* An OpenEmbedded-based OS distribution, enabling extensibility and support for the latest updates and features.
* Support for [four development boards](../first-image/hardware.html) (two added for 0.6).
* Hardware and software-based isolation mechanisms for security:
    * Dedicated hardware within most Cortex-A devices enforces the most secure isolation boundary. This technology, called TrustZone, allows the most sensitive code and data to run within a so-called **Secure World**. MBL includes a trusted operating system for this Secure World called **Open Source Trusted Execution Environment** (OP-TEE) - an operating system within an operating system. It is loaded by the Trusted Firmware and typically used to protect cryptographic keys and other sensitive data assets.
    * Secure boot methodology based on Linaro's Trusted Firmware A for both the ARMv7-A and ARMv8-A platforms. Trusted Firmware is a minimal secure bootloader that runs when a Cortex-A microprocessor is executing in TrustZone's "secure world" mode.
    * Open Container Initiative (OCI)-compliant containers for applications, protecting against compromised applications and facilitating a modern development workflow. Based on the RUNC container runtime, this Docker-like system allows OEMs to package services or applications as independent container images that run entirely within a sandbox. This provides two benefits. First, if an attacker compromises a single application, it's far harder for the attack to spread beyond the infected container. This will help to reduce the impact of an attack and ensure a device can easily be restored to a secure state. Second, applications can be developed independently of the underlying IoT platform. For example, it's easier for a developer to build and test on their desktop workstation or laptop.
* Support for a developer connection to the IoT device (over USB), to develop without disrupting the production networking options.
* A developer command-line tool - MBL CLI - to facilitate:
    * Discovery, setup and local update of development devices.
    * First-time provisioning of device and update credentials to the IoT device (added in 0.6).
* A lightweight, feature rich connection manager for Ethernet and Wi-Fi connections.
* The integration with Pelion Device Management services offers:
    * Support for the Device Management Client for in-field provisioning and over-the-air device configuration.
    * Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
    * Large-scale management of device groups.
    * Device management and status monitoring, including notifications of connection status.
    * Access control at the account level.
    * Over-the-air, digitally signed and verified updates for applications and the root file system. MBL uses updates to send new features and security patches to deployed devices, fixing vulnerabilities before they're exploited.

<h2 id="design">Design diagram</h2>

The following diagram illustrates the components and services that MBL provides:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/Application_containers.png)<span>When used in conjunction with Device Management, MBL provides a secure platform for developing, operating and managing IoT applications</span></span>

The **application management framework** manages installing and running separate applications:

* **Core application configuration**: configures the set of core applications that are run automatically when a device boots.
* **Life-cycle management**: runs instances of an application.
* **Package management**: installs and updates application packages.

The **platform services** handle the common requirements of IoT applications:

* **Network manager**: uses ConnMan to manage network connections. Future releases will include **BLE** for local connections and a **modem manager** for cellular connections.
* **Logging framework**: tags and forwards logged events.
* **Secure services**: protect assets using secrets storage and cryptographic operations with access control.

The **Device Management Services** are integrated user-space agents that connect to the Pelion Device Management services. They provide secure remote ownership, update and monitoring for devices using the Device Management Client, and will provide a data client to manage application data. Together, the management agents extend the way in which a device is managed by providing:

* Additional device configuration access:

    * Over BLE (in a future release).
    * Cloud-based, using the Pelion Device Management Connect service.
    * Over USB (during manufacturing).

* Trusted Execution Environment and OP-TEE run security-sensitive applications within the isolated environment provided by Arm TrustZone.

<h2 id="getting-started">Getting started</h2>

There are two paths to working with MBL:

1. If you are a **Linux developer** interested in contributing to MBL or porting it to a new device:
    1. Please [build MBL locally so you can test](../first-image/index.html).
    1. Review our [contribution guide](../develop-mbl/index.html).
    1. Use [our update tutorial](../update/index.html) to send image firmware updates over the air.

1. If you are an **application developer** interested in building applications for devices that run MBL:
    1. We recommend [using an evaluation image](../first-image/downloading-an-evaluation-image.html).
    1. Review our [development guide](../develop-apps/index.html), our [command-line interface MBL CLI](../develop-apps/the-mbl-command-line-interface.html) and our [Hello World application](../develop-apps/hello-world-application.html).
    1. [Use our update tutorial](../update/index.html) to send application firmware updates over the air.

<h2 id="doc-updates">Recently updated documentation</h2>

* A standalone guide [for writing images to devices](../first-image/index.html), including evaluation images, new board support and cellular connections for version 0.6.
* A new [provisioning guide](../first-image/provisioning-for-pelion-device-management.html).
* Instructions for [removing the old version of MBL CLI](../develop-apps/setting-up.html#setting-up-mbl-cli).
* A new [guide for application developers](../develop-apps/index.html).
* New [information for MBL development](../develop-mbl/index.html), including tool reviews for incremental builds, [contribution guidelines and code style guides](../develop-mbl/contribution-guidelines.html).
* Revised [firmware update tutorials](../update/index.html).
* A new [reference about partition layout](../references/partition-layout.html).

The [full release note](https://github.com/ARMmbed/meta-mbl/blob/mbl-os-0.6/docs/release_note.md) is available on our public preview repository.
