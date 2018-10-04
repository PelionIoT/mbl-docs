## Introduction to Mbed Linux OS


<!--Case for picking Mbed Linux and matching hardware over Mbed OS or any other hardware-->
<!---->

Mbed Linux OS (MBL) is a Linux distribution for IoT devices based on the Cortex-A processor family. In addition to the the rich set of services, libraries and drivers provided by Linux, MBL includes a version of the Pelion Device Management Client, allowing a device to connect to and be managed by Pelion Device Management. The combination of MBL and Pelion Device Management Client provides a secure(JJ:?) foundation layer for building IoT applications (JJ: as they can be easily updated??).

### Get started

There are two paths to working with MBL:

1. If you are a Linux developer, and interested in contributing to MBL or porting it to a new device:
    1. Please build MBL locally.<!--Why, actually? Can't they just grab the source code and start hacking? JJ: Well they could, but they still have to build it to test it - there is an intersting note about the build environment here which is to do with OpenEmbedded but it doesn't really change the get it and build it to start with--> 
    1. Use our porting or contributing guides. <!--Planned for March, sadly-->
1. If you are an application developer, and interested in building applications for devices that run MBL:
    1. Please build MBL locally and flash it to your device.
    1. Try our example application, <!--or start writing your own application using our API references - JJ: The API references I gave you are really internal only, and so APIs are really whatever they want to use from Linux - we will have some interfaces eventually...-->.

### blah why mbl blah

<!--Why MBL, instead of just Mbed OS?-->
<!--And instead of just Linux?-->
<!--What specific engineering problems does MBL solve?-->
<!--Basically, this is our chance to convince engineers to give MBL a go. It can't sound like marketing - they won't read that - but it needs to intrigue them, if not outright convince them.-->
<!--JJ: So we need a brief summary/bullet list of features - Trusted Firmware secure boot, integrity checking, etc... (which I note you already have below!) whatever we manage to get done or even future direction - depends on if Mark is happy giving away things on the docs website.-->

MBL includes services for common functions such as network management, secure storage and container management, in addition to the foundation layer provided by Pelion (see below). Platform service APIs are available to application processes using <!--who's using that - the app or MBL? JJ: Well both as MBL will use things like the bluetooth DBus API to provide functionality for Scout, and the application can also use it for talking to sensors--> a standard Linux IPC framework.


MBL provides a uniform, open and verifiable approach to platform security. Each supported platform has:

* Secure boot of Trusted Firmware, used for initializing trusted world firmware, including the trusted execution environment.
* Verified boot of normal world firmware.
* Independent verification of application packages.
* Integrity checking of read-only file systems.

### Containers

To simplify application development and deployment, device application processes run in a <!--JJ: best to limit this to one at the moment, though the main release may have more--> container. The process isolation provided by containers also enhances security and device reliability.

<!--What does the user have to do or know to use them? Not in detail, obviously, this is just the intro. But some overview. JJ: We haven't completely decided on this yet. We could mention we are supporting OCI containers which can be made by using docker on the host to create a cross-compiled build of the application and then export them as cimple OCI containers. But we may simplify/change this flow.  -->

### Pelion Device Management through MBL

For application developers, the Pelion layer provides the following services:

* Device discovery and secure identity in the Device Management device directory, to protect against impersonation or cloning.
* Membership of a group of managed devices (Pelion Device Management).<!--Why does the reader care? JJ: I don't know - presumably it provides device grouping for easier management?-->
* Access control at the account level.
* Device status monitoring, including notifications of connection status.
* Device firmware update and application management

MBL uses the Pelion Device Management Update service to support updates of both the device's base firmware and application packages.

### Design

The following diagram illustrates the components and services provided by MBL.  When used in conjunction with Mbed Cloud, MBL provides a secure platform for developing, operating and managing IoT applications.
