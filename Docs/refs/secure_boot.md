## Secure boot and firmware verification requirements
<!--who is this for? people porting MBL? people contributing to MBL? is it generic background info?-->
### Introduction

This document describes the requirements for the secure boot and firmware integrity verification of an Mbed Linux OS platform. To protect high value data, the secure boot procedure and subsequent integrity verification ensure a device will only run firmware from legitimate sources. The secure boot procedure also guarantees the integrity of the trusted execution environment by isolating it from possible vulnerabilities in other firmware components.

<!--To reflect the different levels of trust confidence in the sources of different firmware components, signed components are grouped as follows:-->
<!--The grouping by trust confidence is reflected in the chain-of-trust formed between boot components and their corresponding code signing authorities.-->
<!--those two lines seemed to say exactly the same thing-->

Boot components and their corresponding code signing authorities form a chain of trust, which reflects their grouping by levels of trust confidence.

The boot process needs to verify the following components:
<!--in this order, or does the order not matter?-->

* Trusted world firmware and boot loaders used beyond the SoC ROM boot loader.<!--I've found both "boot loader" and "bootloader" in this doc. In ISG we normally use "boot loader", so I changed it-->
* The trusted execution environment (TEE) and associated secure applications.
* Normal world boot loaders.
* The Linux kernel.
<!--there's an awful lot of assumed knowledge here. I want to make sure we're all okay with it-->

The secure boot process must accommodate field updates for all firmware components, including trusted firmware, boot loaders and TEE, as individuals. Device operators will then define their own policy for what can and cannot be updated.


The verification of application container bundles is beyond the scope of this document.<!--should this have come earlier, when we described the scope?-->

#### Terminology

This document uses the following terminology:

* **TF-A**: Trusted Firmware - secure world software for Arm Cortex-A devices (REF7).
* **DEK**: Data Encryption Key.
* **FIT**: Flattened Image Tree.
* **OTP**: One Time Programmable.
* **PSA**: Platform Security Architecture (REF8).
* **SoC**: System on Chip.
* **SPL**: Secondary Program Loader (U-Boot).
* **TEE**: Trusted Execution Environment.
* **TPM**: Trusted Platform Module.


#### References

* REF1: DM Verity project: [https://gitlab.com/cryptsetup/cryptsetup/wikis/DMVerity](https://gitlab.com/cryptsetup/cryptsetup/wikis/DMVerity)
* REF2: Trusted Base System Architecture CLIENT2 (TBSA-CLIENT2), Document number: ARM DEN 0021A-6, Copyright ARM Limited 2011-2013
* REF3: Trusted Board Boot Requirements CLIENT (TBBR-CLIENT), Document number: ARM DEN0006C-1, Copyright ARM Limited 2011-2015
* REF4: mkimage tool source code: [https://github.com/lentinj/u-boot/blob/master/tools/mkimage.c](https://github.com/lentinj/u-boot/blob/master/tools/mkimage.c)
* REF5: Verified Boot documents: [https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot)
* REF6: Integrity Measurement Architecture: [https://sourceforge.net/p/linux-ima/wiki/Home/](https://sourceforge.net/p/linux-ima/wiki/Home/)
* REF7: Trusted Firmware for Arm Cortex-A class devices: [https://www.trustedfirmware.org/index.html](https://www.trustedfirmware.org/index.html)
* REF8: Platform Security Architecture: [https://developer.arm.com/products/architecture/security-architectures/platform-security-architecture](https://developer.arm.com/products/architecture/security-architectures/platform-security-architecture)


### Summary of requirements and recommendations

This is a summary of the MBL secure boot's requirements and recommendations:<!--"highly desirable" is not a requirement, so I changed this to say "requirements and recommendations"-->

* All booted images must be signed and verified using a chain of trust. The root of trust for the chain must be an OTP <!--programmed--><!--"p" stands for programmable - is it correct to follow that with "programmed"?--> key.
* <!--For a Linux distribution, --><!--as opposed to what? this is an MBL doc-->We recommend using unmodified existing components (such as Trusted Firmware, OP-TEE and U-Boot) wherever possible, to exploit code maturity and familiarity by adopters. The boot process should use these components with build options that meet requirements for security and reuse.
* As much of the secure verification logic as possible should be common across supported platforms. This is explained in greater detail [later in this document](#platform-independent-boot-stage-verification).
* Trusted Firmware provides a generic solution for booting and verifying trusted world firmware and the first normal world boot stage (U-Boot).<!--and? this is phrased like background info, not a requirement or recommendation-->
* Use U-Boot to verify the Linux kernel, device tree and normal world boot script using Verified Boot, as used in Chrome OS.
* The TEE must be initialized early on in the boot sequence to reduce the probability of its integrity being compromised.
* Boot time verification can be limited to verifying all steps up to and including the Linux Kernel. Other software components must also be verified, but not necessarily during the boot process; to avoid significantly impacting boot time, software components held in the root file system or other mounted read-only file systems may be verified on-demand (as and when files are accessed).
* dm-verity (REF1) should be used to check the integrity of the read-only root FS.<!--is "should" an option, or a must? It's a dangerous word-->
* Provide a flow for device firmware development.<!--what does "flow" mean? And "provide"? and development... do you mean a way of flashing test images to devices?-->
* Development images should only be allowed to run on development devices<!--should meaning must, I presume, because of the next point-->; it must not be possible to run a development image on a production device.

### Trusted firmware

[Trusted Firmware](https://www.trustedfirmware.org/index.html) provides a reference implementation for the secure boot of trusted world components, up to loading and verifying the first normal world boot loader. Trusted Firmware meets the requirements specified in the Arm Trusted Boot Requirements specification (REF3) but with restrictions for ARMv7-A devices (because it was developed for the ARMv8-A architecture<!--I presume that was the connection-->).<!--where can I find those listed?-->

### PSA requirements
<!--I'm not always sure whether we're talking to users or to ourselves. Should this sentence be saying the users have to consider PSA, or should it say that we considered PSA?-->
All design choices that are made for Cortex-A class devices will be<!--you mean must be?--> aligned to the goals of PSA, based on:

* Trusted Base System Architecture CLIENT2 (TBSA-CLIENT2), Document number: ARM DEN 0021A-6, Copyright ARM Limited 2011-2013 (REF2)
* Trusted Board Boot Requirements CLIENT (TBBR-CLIENT), Document number: ARM DEN0006C-1, Copyright ARM Limited 2011-2015 (REF3)<!--can we link to those? are they publicly available somewhere?-->

These documents give guidance on the scope of the SoC Root of Trust key, and the boot sequence for trusted and normal world components.

#### Scope of SoC Root of Trust key

The Root of Trust (RoT) key, used by the SoC boot ROM, should only be used for verifying the firmware loaded and run by the boot ROM. By limiting the scope of the SoC RoT key to the verification performed by the ROM boot loader, no other component used during the boot flow is dependent on any SoC specific RoT key<!--so?-->. Other boot components may depend on SoC specific NV counters, stored in OTP, for rollback protection.

The type and size of the key are constrained by the SoC boot ROM capabilities. For example, NXP support RSA key pairs of sizes 1K, 2K and 4K. The signing authority is responsible for using an appropriate key size to meet longevity requirements. We also recommend encrypting the first loaded image if the SoC can support that.

Subsequent boot images should be verified using keys embedded either in the image of a previously loaded stage or in certificates held in secure storage. The scope of the SoC root of trust key should be restricted to verifying the first booted image.
<!--is this elaborating on the line "Other boot components may depend on SoC specific NV counters, stored in OTP, for rollback protection."?-->


#### Trusted world components should<!--should or must?--> be booted before normal world components

To avoid<!--avoid or reduce?--> the possibility of a normal world component compromising a trusted world<!--is it Trusted World or trusted world?--> component in some way, all trusted world components must be booted before normal world components. Also, to reduce the risk of unknown software running in secure mode, the size of any boot loader that runs in the secure state should<!--should or must?--> be minimized. A boot flow that uses Trusted Firmware, OP-TEE and U-Boot would<!--would or must? does it just naturally happen?--> follow the illustrated sequence:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/boot_sequence_for_components.png)</span>

### Platform-independent boot stage verification

After loading and verifying the initial image, loaded by the SoC boot ROM, the boot logic and verification of subsequent steps in the boot process should be common across supported platforms. This achieves:

* Minimal porting effort to new platforms.
* A single image format that includes signature and key blocks for all supported platforms.
* A common toolchain for image creation and signing (apart from the initial image loaded by the SoC boot ROM).
* Uniform security across platforms (within hardware constraints).

Ideally, the boot stage verification code should be free of SoC dependencies to allow a default software-only implementation, which can then be used on different platforms. However, you can consider using hardware features that offer significant benefits, such as support for cryptographic operations.<!--this again raises the question of who this document is for - Linux developers or people porting MBL, for example?-->

### A generic flow using trusted world firmware and U-Boot

The boot sequence chains the following boot mechanisms:

1. **SoC boot ROM** loads, verifies and runs the first trusted world image (TF-A BL2).
1. **TF-A** loads, verifies and runs trusted world components, including OP-TEE.
1. **TF-A** verifies and loads U-Boot.
1. <!--I made it two steps-->**U-Boot** loads, verifies and runs normal world components.

The following diagram illustrates the chaining of the secure boot steps:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/u_boot_flow.png)</span>

#### Trusted world boot

Booting of trusted world components is performed by Trusted Firmware<!--is that a proper name that should be capitalised? and what's the difference between Trusted Firmware and Trusted World Firmware?-->. After the ROM boot loader performs the initial boot, TF-A performs boot steps BL2, BL31, BL32 and BL33 (in the above diagram). The switch from secure to non-secure execution mode is done *before* entering BL33, when all trusted components (like OP-TEE) have already been initialised. For MBL, BL33 is U-Boot.<!--U-Boot is a step, not a thing?-->

#### Normal world boot

Booting of normal world components is performed by U-Boot, a boot loader for embedded devices most commonly used to boot the Linux kernel. Its many configuration options support different boot requirements, and MBL uses the **Verified Boot** option to provide a common boot solution for normal world firmware.

##### Verified Boot

U-Boot supports **Verified Boot**, a generic secure boot mechanism that can be used for embedded devices. The term is used in Chrome OS to refer to its secure boot and user-space integrity checking features. Verified Boot features have been mainlined into U-Boot, and you can enable them with a build configuration.

To create signing key pairs, use openSSL.<!--what's the context?-->

Verified Boot features:

* Uses the FIT format. FIT files can contain:
    * A signed image hash.
    * Public keys used for verifying subsequent boot stages.
* Images are signed using the generic mkimage tool (REF4).
* The Verified Boot build configuration depends on the Vboot library (REF5), which provides:
  * Hashing.
  * RSA signature checking.
  * Verified boot flow logic.<!--is this supposed to be "Verified Boot" the name or "verified (verb) boot flow logic (noun)"?-->
  * The Trusted Platform Module (TPM) library used for roll-back protection.

An IoT device is unlikely to include TPM hardware; it will instead rely on a trusted application running in the TEE for any TPM services used during secure boot. A trusted application can provide roll-back protection, equivalent to the TPM service, using one of the following platform-specific options:

* On-chip counters based on OTP fuses.
* If the storage is based on eMMC: use the Replay Protected Memory Block (RPMB), where the access key is only accessible to the secure side (for example, based on an SoC secret).
* Emulated within secure on-chip embedded NVM (securely managed by the Trusted OS).
* Secure flash (accessible to secure<!--secure what?--> only), protected by a derived key from the DEK (authenticated encryption).
* In future SoCs, ARM's CryptoCell/Island products will also provide a facility for roll-back protection.

For all of these, the trusted OS must have an application interface through which it can provide and increase counter values.
<!--don't think we've mentioned "trusted OS" before-->

### Integrity checking with dm-verity

The boot flow described above provides secure verification of all components in the boot chain, up to and including the Linux kernel. It does not verify the read-only root file system or any other mounted file system that may hold executables or configuration data. Checking the integrity of read-only files requires an additional solution: dm-verity.

Verified Boot, used in Chrome OS, relies on integrity checking of read-only block devices, performed before blocks are read. This in turn relies on the dm-verity kernel feature (REF1), which provides transparent checking of block devices. The dm-verity feature scans block devices on demand, and checks that block hashes match the expected hash (generated when the image file was created). If it detects a mismatch, it blocks access<!--whose access to what?-->. Use of block level integrity checking requires block-oriented firmware updates of a read-only device to ensure that block content hashes over the whole block device match the expected hashes.

### Image signing

The following diagram illustrates how image signing tools sign different boot components:

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-linux-os-docs-images/image_signing.png)</span>

<!--do we want some text to explain this diagram?-->

### Operation on open and closed devices

An SoC provides OTP hardware that determines the 'open' or 'closed' state of a device:

* Open: `boot_mode=open` means that if authentication of a boot chain component fails, an authentication failure event is logged but the boot process continues.
* Closed: `boot_mode=closed` means that if authentication of a boot chain component fails, the boot process is halted (or perhaps put into a recovery mode).<!--perhaps? what does it depend on?-->

At the end of the manufacturing process, a production device should be in the close state. It should also have have the following functionality disabled:

* Serial console.
* Debug access over network, for example using SSH.
* Debug and trace functions.
* JTAG.

Beyond the initial step performed by the Soc ROM boot loader, Mbed Linux OS only supports signed boot images, independent of whether the device is open or closed. Not supporting unsigned images means:

* Reduced implementation effort.
* Reduced test burden, while supporting easy developer project on-boarding.
* Reduced software attack surface, and therefore increased software security.

To support firmware development on an open device, the build process should automatically sign images using one or more developer keys. You can also choose to use the SoC signing tool to sign the image containing SPL (U-Boot, for MBL). When producing a release build, images must be signed by a trusted firmware supplier.
