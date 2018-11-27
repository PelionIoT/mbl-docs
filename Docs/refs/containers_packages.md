## Application containers and packages

In common with mobile and desktop device operating systems, MBL aims to provide an application framework that supports a delivery model where applications are independently developed, tested and deployed.  For IoT application developers, MBL intends to offer:

* A common device platform with a standard set of services and APIs accessible by applications.
* A guarantee of the integrity of the device platform.
* Application isolation to protect the device platform and other applications from bad behaviour by a running application.
* Support for multiple independent applications.
* Secure application update via Pelion Update service, independent of other device firmware.
* Application control via Pelion Connect service. 
* An application development flow that allows for efficient on-device debug and test of application code.
* Independent application signing where signing authority is delegated to support independent application delivery.
* A standard method for application packaging to promote runtime and toolchain interoperability.

### Application isolation

To protect an IoT device, application software should be isolated from other device software to prevent a malicious or defective application from undermining device integrity.  Isolation methods will be used that make the following guarantees:

* Memory isolation - it should not be possible for an application to access memory associated with the rest of the system.
* Limit file access - limit application access to a restricted set of files.
* Limit device access - limit application access to peripheral devices and network interfaces.
* Resource isolation - constraints on resource usage to prevent an application from using more than its fair share of CPU, memory, file storage or network bandwidth.
* Resource ownership - it should not be possible for an application to hold exclusive ownership of a device that the base platform depends on.

The Linux kernel supports the following features for application isolation:

* Process isolation - each process has access to a private virtual address space
* Namespace isolation - partitions resources such as process IDs, file names and names associated with network access
* Cgroups - isolates the resource used by a group of processes in terms of CPU, memory, file storage and network bandwidth

These kernel features will be used to meet requirements for application isolation.

### Application packaging

To deploy and run an application on a device, the application and associated components need to be packaged, via the OPKG package manager, to form an application image.   The application image will be created by an application developer at build time.  To allow the image to be authenticated, the package must support signing.  An application will depend on platform provided services and devices.  Any additional components that an application depends on, such as libraries, must be included in the application image package.  A packaged application image must be able to be deployed on a device via the following:

* Incorporated in factory programmed flash image.
* Updated via the Mbed Update service.
* Updated during application development on a device running MBL with special development facilities.

The packaged application image must contain sufficient information to launch the application on the target device.  This will include:

* External platform dependencies
* Commands and arguments
* Environment variables

Standardization work by the Open Container Initiative group (www.opencontainers.org) is currently underway to create open industry standards around container image formats and runtime.  The OCI was launched in 2015 by Docker and CoreOS.  The OCI image format is based on the Docker image format.  Both Docker and and CoreOS rkt are committed to supporting OCI images.  Open source tools are available for build support for OCI images (https://github.com/opencontainers/image-tools).  Using an open packaging standard for application containers has the following benefits for MBL:

* Choice of build toolchains
* Choice of runtimes
* Reassurance to adopters that they are not locked into a particular container technology.

### Application container runtime

An MBL platform must have the capability to unpack and run an application runtime bundle, implementing all necessary isolation methods.  The container runtime component must be compatible with the application runtime bundle.  An OCI runtime bundle comprises the following:

* Runtime spec - config.json
* Directory containing the sub-tree that forms the root filesystem for the conatiner

The container runtime should be autonomously managed to achieve the following:

* Application start after system initialization.
* Orderly application stop and restart during application update process.
* Orderly application stop on requested system shutdown.
* Application restart on abnormal exit.

Application control endpoints may be presented via the Pelion Connect service to allow the lifecycle of installed applications to be managed.  These will expose per application maintenance operations such as:

* Start a stopped application instance.
* Stop a started application instance with an orderly shutdown.
* Force a started application instance to stop.
* Suspend and resume an application instance.

Applications may also need to be controlled by other clients such as:

* Update manager - coordinates running application instances to apply an update.
* Public container API - to support container orchestration using third party tools
* Application developer - manual control of applications during development.
* Automated tester - controls running applications during test

It is highly desirable to use a container runtime that is as simple as possible to minimize its resource footprint and security attack surface.
An OCI compliant runtime called runC is used to meet these requirements (https://github.com/opencontainers/runc).  This was donated by Docker and forms the low-level container runtime used by Docker Daemon.

### Application update

Over the lifetime of an IoT device, application images may need to be updated, either as part of a bigger package of updates or individually.  Application updates will be delivered using the Pelion Update service.  The application update process should meet the following requirements:
 
* Support updates to individual application containers.
* Be resilient to a failure such as a power failure during the application update process.  If the update failed to be applied, the pre-update version should remain viable.
* Support delta updates for cases where the whole application is large.

Once an updated application image is unpacked and its interity is verified, the running application may be stopped.  After starting the updated version of the application, the application should post a confirmation message via the D-Bus update API to indicate that it is running successfully.  Only then can the old application image be deleted.

It can be assumed that the container runtime will be controlled by an MBL specific component that will coordinate the application update process.

