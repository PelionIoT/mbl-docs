## Mbed Linux CLI

The Mbed Linux CLI is a toolbox for building your Mbed Linux applications and managing them on your target device.

### Prerequisites

[Docker > v17.0.0](https://www.docker.com), recommended for local application building

[Node.js > v6.0.0](https://nodejs.org), which includes `npm v3`

- We recommend installing Node.js using the [Node version manager](https://github.com/creationix/nvm), as the installation below requires usage of your github SSH credentials.

### Installation

The CLI is distributed using npm. To install the tool globally:

```bash
$ npm install -g ARMmbed/mbl-cli#build
```

### Usage

```bash
$ mbl-cli <command> [arguments]
```

#### Options

- -v, --version - Show version number
- -h, --help - Show help

#### Commands

##### Build

Build an Mbed Linux application from a source directory.

```bash
$ mbl-cli build [source] [path]
```

Where `[source]` is the directory to the application source (defaults to the current working directory) and `[path]` is an optional path to save the output image file to (defaults to `mbl-image.tar` in the current working directory).

_Flags:_
```
--force, -f                 force a complete rebuild
--noemulation, -n           turn off emulation during build
--remote [host], -r [host]  build the application on a remote host, optionally
                            passing the host to use in the form `host:port`
```

##### Deploy

Deploy an application to a device, building as necessary.

```
$ mbl-cli deploy [source] [address]
```

Where `[source]` is the path to a built application image or the directory of the application source to deploy. `[address]` is the address of the device to deploy to.

_Flags:_
```
--force, -f                 force a complete rebuild if building
--noemulation, -n           turn off emulation if building
--remote [host], -r [host]  build any application on a remote host, optionally
                            passing the host to use in the form `host:port`
--detached, -d              don't attach to application output
```

##### Device Management

Device management commands.

```
$ mbl-cli device <command> [address]
```

_Commands:_

Scan for devices connected to the system.
```
$ mbl-cli device scan
```

SSH onto a device, optionally specifying the device address to use
```
$ mbl-cli device ssh [address]
```

Get output logs from a device, optionally attaching to the device output.
```
$ mbl-cli device logs [address] [--attach]
```

Start the application on a device.
```
$ mbl-cli device start [address]
```

Stop the application on a device.
```
$ mbl-cli device stop [address]
```

Restart the application on a device.
```
$ mbl-cli device restart [address]
```
