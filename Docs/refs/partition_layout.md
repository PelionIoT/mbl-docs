# Partition layout

## Design

The flash memory partition layout for MBL is driven by four high-level requirements:

* Use cases for device firmware update.
* Use cases for application management.
* Secure Boot and firmware verification.
* Use cases for device manufacturing.

In summary, requirements that influence the flash layout are:

| Requirement | Implication |
| --- | --- |
| It should be possible to update all device firmware except for early stage boot loaders. | Components that are loaded during boot need to start at a block number known by the loading boot loader. |
| A failure during the update process (such as a power loss) should not result in a bricked device.	| The previous known good version of a booted component should be maintained in flash until an updated version has been verified to be complete, authentic and viable. This means that all booted components should be banked to accommodate both the known good and the updated versions of a component. |
| A firmware update involving multiple components may extend across power failures. |	Non-volatile state must be held reflecting the presence of a component update. Only when all components are installed can the entire update be viewed as complete. |
| A multi-component update should be applied either in its entirety or not at all. A device should never be left in a state where only a partial update was performed. | A non-volatile state must be held, reflecting that a multi-component update is in progress but is not yet complete. |
| Data saved in flash memory that does not need to be modified in normal device operation should be write-protected. |	Some partitions should be created or modified to be read-only. Factory data written during the manufacturing process should be saved in a partition that is modified to read-only when a device boots after its life-cycle state has been modified to 'manufacture complete'. |
| A device should support the 'restore to factory' use case. | To allow the user configuration to be deleted without deleting the factory configuration, the two types of data should be kept separate. |

## General information

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/partition_layout.png)</span>

The partition layout for every board includes at least the following partitions:

| Label            | ro/rw | Type         | Mount point      | Size     | Contains |
|------------------|-------|--------------|------------------|----------|----------|
| boot             | ro    | vfat or ext4 | /boot            | 30MiB    | Kernel, device tree and U-Boot boot script |
| bootflags        | rw    | ext4         | /mnt/flags       | 20MiB    | Flags to determine which rootfs is active |
| rootfs1          | rw    | ext4         | /                | 500MiB   | Root filesystem for installation 1 |
| rootfs2          | rw    | ext4         | /                | 500MiB   | Root filesystem for installation 2 |
| factory_config   | rw    | ext4         | /config/factory  | 20MiB    | Factory configuration |
| nfactory_config1 | rw    | ext4         | /config/user     | 20MiB    | Non-factory config 1; user data configuration (listed in the diagram as "Config 1") |
| nfactory_config2 | rw    | ext4         | /config/user     | 20MiB    | Unused at this time |
| rootfs1_ver_hash | rw    | ext4         | /mnt/verity_hash | 20MiB    | dm_verity meta-data for rootfs1 |
| rootfs2_ver_hash | rw    | ext4         | /mnt/verity_hash | 20MiB    | dm_verity meta-data for rootfs2 |
| log              | rw    | ext4         | /var/log         | 20MiB    | Log files |
| scratch          | rw    | ext4         | /scratch         | 500MiB   | Temporary files (such as downloaded firmware files) |
| home             | rw    | ext4         | /home            | 450MiB   | User application storage |

## Board specific information

### Raspberry Pi 3

Full partition layout:

| Label            | Size   | U-Boot iface dev:part | Linux device file | Notes    |
|------------------|--------|-----------------------|-------------------|----------|
| boot             | 30MiB  | mmc 0:1               | mmcblk0p1         | Primary  |
| bootflags        | 20MiB  | mmc 0:2               | mmcblk0p2         | Primary  |
| rootfs1          | 500MiB | mmc 0:3               | mmcblk0p3         | Primary  |
| -                | -      | mmc 0:4               | mmcblk0p4         | Extended |
| rootfs2          | 500MiB | mmc 0:5               | mmcblk0p5         | Logical  |
| factory_config   | 20MiB  | mmc 0:6               | mmcblk0p6         | Logical  |
| nfactory_config1 | 20MiB  | mmc 0:7               | mmcblk0p7         | Logical  |
| nfactory_config2 | 20MiB  | mmc 0:8               | mmcblk0p8         | Logical  |
| log              | 20MiB  | mmc 0:9               | mmcblk0p9         | Logical  |
| scratch          | 500MiB | mmc 0:10              | mmcblk0p10        | Logical  |
| rootfs1_ver_hash | 20MiB  | mmc 0:11              | mmcblk0p11        | Logical  |
| rootfs2_ver_hash | 20MiB  | mmc 0:12              | mmcblk0p12        | Logical  |
| home             | 450MiB | mmc 0:13              | mmcblk0p13        | Logical  |

### WaRP7

WaRP7 uses an area of the disk for Trusted Firmware-A (TF-A) and Second Boot Loader (BL2) images. It is the first partition although it is not in the disk's partition table.

Full partition layout:

| Label            | Size   | U-Boot iface dev:part | Linux device file | Notes       |
|------------------|--------|-----------------------|-------------------|-------------|
| -                | 4MiB   | -                     | -                 | TF-A and BL2 |
| boot             | 32MiB  | mmc 0:1               | mmcblk0p1         | Primary     |
| bootflags        | 20MiB  | mmc 0:2               | mmcblk0p2         | Primary     |
| rootfs1          | 500MiB | mmc 0:3               | mmcblk0p3         | Primary     |
| -                | -      | mmc 0:4               | mmcblk0p4         | Extended    |
| rootfs2          | 500MiB | mmc 0:5               | mmcblk0p5         | Logical     |
| factory_config   | 20MiB  | mmc 0:6               | mmcblk0p6         | Logical     |
| nfactory_config1 | 20MiB  | mmc 0:7               | mmcblk0p7         | Logical     |
| nfactory_config2 | 20MiB  | mmc 0:8               | mmcblk0p8         | Logical     |
| log              | 20MiB  | mmc 0:9               | mmcblk0p9         | Logical     |
| scratch          | 500MiB | mmc 0:10              | mmcblk0p10        | Logical     |
| rootfs1_ver_hash | 20MiB  | mmc 0:11              | mmcblk0p11        | Logical     |
| rootfs2_ver_hash | 20MiB  | mmc 0:12              | mmcblk0p12        | Logical     |
| home             | 450MiB | mmc 0:13              | mmcblk0p13        | Logical     |


## Notes on firmware update

Each firmware installation has several partitions: a read-write root partition, a configuration partition, and a user application(s) partition. Using a separate partition for the user application eases the support of updates of just the user application.

To support firmware updates, MBL has storage for two separate firmware installations and a mechanism to select which installation to
boot into after reboots.

A script inside `initramfs` checks which root partition is currently active; by default this is `rootfs1`, but the script will change to `rootfs2` if a flag file called `rootfs2` is present in the `bootflags` partition.

For a configuration that should persist across firmware updates, MBL actually stores three versions: a configuration for each firmware installation (stored on the installation's `nfactory_config1` or `nfactory_config2` partitions) and the initial configuration installed at build time or in the factory (stored on the `factory_config` partition). During firmware updates, the configuration for the newly installed firmware will be created based on the configuration for the currently running installation, rules contained in a **configuration update** script attached to the new firmware and, possibly, the initial factory configuration.

Additionally, there is the `scratch` partition for temporary storage of firmware downloaded during the update process. This partition is shared between the firmware installations.

## Plans

* A new `BL3 FIP Image` partition should hold versions of the BL31 boot loader and associated components contained within a signed FIP image.
* The `rootfs_ver_hash` partitions should include meta-data for the dm-verity tool for the corresponding root filesystem.  
* The `rootfs_ver_hash`, `factory_config` and `rootfs` partitions should be switched to read-only mode (currently they are read-write).
* The new `BL3 FIP Image` and the existing `boot`, `rootfs1_ver_hash` and `nfactory_config1` partitions should be banked.
