# Preparing Device Management sources

## Downloading Device Management developer credentials

<!--do we still need this, or can we remove it now that provisioning does this using MBL CLI?-->
<!-- JH_TODO: we should remove this file. I'm not sure how to find other docs that link to this though... -->

MBL handles Device Management connectivity on behalf of the device, rather than relying on the user application to do it. This means you need to add your Device Management credentials to the working directory MBL builds from. For development environments, Pelion offers a *developer certificate* for quick connection:

1. Create cloud credentials directory, for example `./cloud-credentials`.
2. To create a Pelion developer certificate (`mbed_cloud_dev_credentials.c`), follow the instructions for [creating and downloading a developer certificate](../first-image/provisioning-development.html).
3. Add the developer certificate to the credentials directory you've created.

<span class="notes">**Note:** These instructions use the developer workflow and certificates as connectivity credentials. Do not use these credentials for production devices.</span>
