## Mbed Linux CLI

The Mbed Linux CLI is a toolbox for building your Mbed Linux applications and managing them on your target device.

<!--Do I have to use it? What other tools can I use?-->
<!--What does it use in the background?-->

You can [view the code on GitHub](https://github.com/ARMmbed/mbl-docs/), or [install it using npm](setting-up.html)
<!--Will anyone be able to install it? Right now the repo is private-->
## Setting up

### Prerequisites

* [Docker version 17.0.0 or higher](https://www.docker.com) - recommended for local application building.
* [Node.js version 6.0.0 or higher](https://nodejs.org), which includes `npm v3`.

    We recommend installing Node.js using the [Node version manager](https://github.com/creationix/nvm), because the installation below requires usage of your GitHub SSH credentials.

### Installation

MBL CLI is distributed using npm. To install the tool globally:

```bash
$ npm install -g ARMmbed/mbl-cli#build
```

<!--Let's say I didn't use the Node version manager; will I run into problems here? How would I solve them?-->
<!--Does that bring in any other dependencies?-->

## Usage

The general structure of an MBL CLI command is:

```bash
$ mbl-cli <command> [arguments]
```

### Options

* -v, --version: Show version number.
* -h, --help: Show help.

### Commands

#### Build

Build an Mbed Linux OS application from a source directory: <!--So the application, not the image? This is for people who already have the image on their device?-->

```bash
$ mbl-cli build [source] [path]
```

Where:

* `[source]` is the directory to the application source. Default: current working directory.
* `[path]` is an optional path to save the output image file to. Default: `mbl-image.tar` in the current working directory.

**Flags:**

```
--force, -f                 force a complete rebuild
--noemulation, -n           turn off emulation during build
--remote [host], -r [host]  build the application on a remote host, optionally
                            passing the host to use in the form `host:port`
```

#### Deploy

Deploy an application to a device, building if necessary:

```
$ mbl-cli deploy [source] [address]
```

Where:

* `[source]` is the path to a built application image or the directory of the application source to deploy.
* `[address]` is the address of the device to deploy to.<!--How do I know?-->

**Flags:**

```
--force, -f                 force a complete rebuild if building
--noemulation, -n           turn off emulation if building
--remote [host], -r [host]  build any application on a remote host, optionally
                            passing the host to use in the form `host:port`
--detached, -d              don't attach to application output
```

<!--Will all of this make sense to app developers, or is some of it too Linux-specific for assumed knowledge?-->

#### Device Management

<!--I thought you meant Pelion Device Management. Can I call this "Device operations" or some other horror designed purely to avoid confusion with our other product?-->

The structure of a device management command:<!--So what was the other group of commands? I just referred to them as generic MBL CLI commands, but is there a logical group name?-->

```
$ mbl-cli device <command> [address]
```

**Commands:**<!--Why is this a bold line, when it was a header for the other group of commands? Anyway, it was a bit of an eye-sore so I put it in a table; I hope it's better this way.-->
<!--except now it's not markdown... so how do I make the command font look good?-->
<table>
<tbody>
<tr>
<td>
Scan for devices connected to the system
</td>
<td>
<font face="console">
$ mbl-cli device scan
</font>
</td>
</tr>
<tr>
<td>
SSH onto a device, optionally specifying the device address to use
</td>
<td><font face="console">
$ mbl-cli device ssh [address]
</font>
</td>
</tr>
<tr>
<td>
Get output logs from a device, optionally attaching to the device output
</td>
<td><font face="console">
$ mbl-cli device logs [address] [--attach]
</font>
</td>
</tr>
<tr>
<td>
Start the application on a device
</td>
<td>
<font face="console">
$ mbl-cli device start [address]
</font>
</td>
</tr>
<tr>
<td>
Stop the application on a device
</td>
<td>
<font face="console">
$ mbl-cli device stop [address]
</font>
</td>
</tr>
<tr>
<td>
Restart the application on a device:
</td>
<td>
<font face="console">
$ mbl-cli device restart [address]
</font>
</td>
</tr>
</tbody>
</table>
