## Application containers and packages

<span class="notes">The developer preview version of Mbed Linux OS (MBL) implements only some of the features reviewed in this document.</span>

Like mobile and desktop operating systems, MBL has a framework for user applications that provides independent development, debugging, testing and deployment. Crucially, each application is isolated in a container, so no matter how many applications are running, they cannot interfere with each other or with MBL itself. Applications can be individually managed and updated using Pelion Device Management, and signed by a signing authority independent of the signing needs of other applications.

The application framework promotes runtime and toolchain interoperability between MBL and other Linux distributions by relying on the Open Container Initiative (OCI) and the Open PacKaGe management (OPKG) formats. The OCI runtime format is based on the Docker runtime format, and the [specification](https://github.com/opencontainers/runtime-spec) details the contents of the runtime bundle. Using an open packaging standard also reassures adopters that they are not locked into a particular container technology.
 
### Application isolation

To protect an IoT device, application software should be isolated from other device software. This prevents a malicious or defective application from undermining device integrity. Isolation methods provide:

* **Memory isolation**: prevent access to memory associated with the rest of the system.
* **Limit file access**: allow application access only to a specified set of files.
* **Limit device access**: limit application access to peripheral devices and network interfaces.
* **Resource isolation**: limit resource (CPU, memory, file storage or network bandwidth) usage to prevent an application from using more than its fair share.
* **Resource ownership**: prevent an application from holding exclusive ownership of a device that the base platform depends on.

MBL uses the Linux kernel's isolation mechanisms as the basis of application isolation. The kernel supports:

* **Process isolation**: each process has access to a private virtual address space.
* **Namespace isolation**: partitions resources such as process IDs, file names and names associated with network access.
* **Cgroups**: isolates the resources (CPU, memory, file storage and network bandwidth) used by a group of processes.

### Application packaging

To deploy and run an application on a device, the application and associated components need to be packaged, via the OPKG package manager, to form an application image. This requires:

* A developer creates the application image at build time.
* An application can depend on platform provided services and devices, but any additional components that an application depends on, such as libraries, must be included in the application image package.
* A packaged application image must be deployable to a device using one of:

    * Incorporated in factory programmed flash image.
    * Updated via the Device Management Update service.
    * Updated during application development on a device running an evaluation build of MBL (which contains services that MBL CLI can use to perform local updates).

* The packaged application image must contain sufficient information to launch the application on the target device. This includes:

    * External platform dependencies.
    * Commands and arguments.
    * Environment variables.

### Application container runtime

MBL can unpack and run an application runtime bundle, implementing all necessary isolation methods. The OCI runtime component is compatible with the application runtime bundle, and comprises the following:

* Runtime spec - `config.json`.
* Directory containing the sub-tree that forms the root file system for the container.

The container runtime is autonomously managed, which provides:

* Application start after system initialization..
* Orderly application stop and restart during application update process.
* Orderly application stop on requested system shutdown.
* Application restart on abnormal exit.

Remote application management through Device Management requires presenting control endpoints through the Connect service. The endpoints expose application-specific maintenance operations such as:

* Start a stopped application instance.
* Stop a started application instance with an orderly shutdown.
* Force a started application instance to stop.
* Suspend and resume an application instance.

Applications may also need to be controlled by other clients, such as:

* Update manager, which coordinates running application instances to apply an update.
* Public container API to support container orchestration using third party tools
* Application developer for manual control of applications during development.
* Automated tester - controls running applications during test.

We recommend using an application runtime bundle that is as simple as possible, to minimize its resource footprint and security attack surface.

To meet all of these requirements, MBL uses an [OCI-compliant runtime called runC](https://github.com/opencontainers/runc). It was donated by Docker, and forms the low-level container runtime used by Docker Daemon.

### Application update

Over the lifetime of an IoT device, application images may need to be updated, either as part of a bigger package of updates or individually. MBL application updates are delivered using the Device Management Update service. The container runtime is controlled by an MBL specific component, which coordinates the application update process.

The application update process meets the following requirements:

* Support updates to individual applications.
* Support delta updates for large applications.
* Be resilient to failure (such as a power failure) during the application update process. If the update was not applied, the pre-update version should remain viable.

When MBL receives an application update:

1. MBL unpacks and verifies the integrity of an updated application image.
1. MBL stops the running application.
1. MBL starts the updated version of the application.
1. The application posts a confirmation message over the D-Bus update API to indicate that it is running successfully.
1. MBL deletes the old application image.
