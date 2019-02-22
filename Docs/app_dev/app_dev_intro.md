# Developing applications for Mbed Linux OS

When designing applications for IoT devices running MBL, we make the following assumptions:

* Our goal is to give end users control of the IoT device running our application.
* **End user applications** (on a phone, control panel and so on) and MBL IoT **device applications** both communicate with the same Pelion Device Management account. They do not communicate directly with each other.

<!--we're just being nice, right? you can use any cloud you want?-->
* For mass production, we may need to create an **install device** that can configure our IoT devices with information that cannot be assumed at the factory stage, such as access to secure WiFi networks.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/applications_map_highlight.png)<span>Application design focuses on connecting the IoT device and the end user's device over an application cloud</span></span>

## Designing modular applications

We're designing an Iot device that must handle an internet connection, as well as any processing and automation it needs to perform to be useful. We'll need some general functionality - like connectivity and I/O handling - for every single application we develop. Other functionality - like calculating and responding to commands from the end user's phone - will need to match the requirements of the device-specific application.
<!--is "device-specific" a clear concept?-->

We have three options:

* Create a single application that includes all the functionality we need. This is simple, but it makes our code less portable because it ties the generic code to device-specific code.
* Use one of two modular approaches:
    * Create smaller applications that provide one microservice each, and install these applications together with a very lean device-specific application.
    * Create base layers that provide one functionality each, and use them as base layers for many applications. Each application will add its own device-specific layer to these base layers.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/multi_apps.png)![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/application_from_layers.png)<span>An MBL IoT device can have multiple applications, and one or more of those applications can be made of base layers</span></span>

## Handling application dependencies

Applications will have dependencies, for example on a runtime library or Python. We can containerise an application with all of its dependencies, or rely on our Linux distribution to provide them. There's a tradeoff: the more we pack into the application container, the bigger it is. The advantage is that the application is independent of the distribution, so we can update the distribution without worrying about breaking the application with a mismatched dependency, and we can update the application without being forced to update the distribution at the same time.

## Connectivity

End user applications and MBL IoT device applications both communicate with the same Device Management account and rely on the Device Management services provided to that account.

MBL uses Device Management Client to connect to Device Management. Each device has only one instance of the client, which supports all applications running on that device.

## Installer devices

Installer devices configure deployed IoT devices that cannot be fully preconfigured. For example, so far we've been using our own Device Management account, but when we start producing devices for a market, we may not want to associate those devices with our own account. Instead, a company that bought a group of devices may want to associate them with its own account. To do that, they must be able to access the platform services on the device, which handle common functions on behalf of applications, including the network manager and secure storage (to store keys and credentials).

You don't need an installer device while developing your application, but you do need to think about what that application may later require from the installer device.
