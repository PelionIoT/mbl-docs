# Building an MBL developer image

<span class="tips">**Tip**: If you downloaded an evaluation image, you can skip the build stage [and go directly to writing](../first-image/writing-an-image-to-supported-boards.html).</span>

Please note that each release has its own branch. Throughout this guide, the release branch is assumed to be `mbl-os-0.6`.

## Prerequisite: setting up the build scripts

<span class="notes">**Note**: You need to use an SSH agent to build. For usage, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).</span>

If you haven't already done so, check out the relevant branch from the `mbl-tools` repository (in this example, we use `mbl-os-0.6`):

```
$ git clone git@github.com:ARMmbed/mbl-tools.git --branch mbl-os-0.6
```

The repository includes the `run-me.sh` script, which:

1. Creates and launches a Docker container that encapsulates the MBL build environment.
1. Launches a build script (the **build-mbl** tool), `build.sh`, inside the container.

To invoke the `run-me.sh` help menu, use:

```
./mbl-tools/build-mbl/run-me.sh -h
```

To invoke the `build.sh` help menu, use:

```
./mbl-tools/build-mbl/run-me.sh -- -h
```


***

Copyright © 2020 Arm Limited (or its affiliates)
