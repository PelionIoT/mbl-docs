# Installing an MBL image

Please note that each release has its own branch. Throughout this guide, the release branch is assumed to be `mbl-os-0.9`.

## Building prerequisites

<span class="tips">**Tip**: This section is only relevant for development and production images. If you downloaded an evaluation image, you can skip the build stage [and go directly to writing](../first-image/writing-an-image-to-supported-boards.html).</span>

* All MBL repositories are public, but you will need an SSH agent to access your own private repositories during the build. For more information, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

* If you haven't already done so, check out the relevant branch from the `mbl-tools` repository (in this example, we use `mbl-os-0.9`):

    ```
    $ git clone https://github.com/ARMmbed/mbl-tools.git --branch mbl-os-0.9
    ```

    The repository includes the `run-me.sh` script, which:

    1. Creates and launches a Docker container that encapsulates the MBL build environment.
    1. Launches a build script (the **build-mbl** tool), `build.sh`, inside the container.

### Help menus

To invoke the `run-me.sh` help menu, use:

```
./mbl-tools/build/run-me.sh -h
```

To invoke the `build.sh` help menu, use:

```
./mbl-tools/build/run-me.sh --builddir /path/to/builddir -- -h
```
