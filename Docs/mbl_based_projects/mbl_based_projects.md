# Example project based on Mbed Linux OS

Mbed Linux OS (MBL) is Yocto-based, so you can use standard Yocto tools and
approaches to develop projects based on MBL. This document creates a minimal MBL-based project, illustrating an approach that adds packages to the distribution, with and without pre-existing BitBake recipes.

This document assumes familiarity with Yocto concepts and BitBake syntax. It is
not intended to replace the [Yocto documentation][yocto-docs].

## Solution overview

The steps to create the project are:
1. Create a new BitBake meta layer called `meta-mbl-project-example`. In step 4
   you will add an image recipe to this layer that specifies the packages to be
   included in the new project.
2. Create a config repository to hold the project's configuration.
3. Create a manifest repository to make it easy to download all of the
   repositories required for the project. This will also enable you to use MBL's
   build tools to build the new project.
4. Add an image recipe to the layer created in step 1.
5. Modify the distribution.

## 1. Create a new BitBake meta layer

You need a meta layer in which to add new BitBake recipes and
configuration.

1. Create a repository, for example a GitHub repo called `meta-mbl-project-example`.
1. Create a layer configuration file:
    1. Add a `conf` directory.
    1. In that directory, create a file called `layer.conf` with content like the following:

    ```
    BBPATH .= ":${LAYERDIR}"
    BBFILES += "${LAYERDIR}/recipes*/*/*.bb ${LAYERDIR}/recipes*/*/*.bbappend"
    BBFILE_COLLECTIONS += "meta-mbl-project-example"
    BBFILE_PATTERN_meta-mbl-project-example := "^${LAYERDIR}/"

    LAYERSERIES_COMPAT_meta-mbl-project-example = "warrior"
    LAYERDEPENDS_meta-mbl-project-example = "meta-mbl-distro"
    ```

Most of the lines here are standard and are explained in the [Creating Your Own
Layer section of the Yocto Mega Manual][yocto-create-layer], but please note:

* The [`LAYERSERIES_COMPAT`][yocto-layerseries-compat] line specifies the
  versions of OpenEmbedded-Core that your layer is compatible with. The value
  should probably match the value in the `LAYERSERIES_COMPAT` line in
  `meta-mbl-distro`'s layer.conf, which can be found in [`meta-mbl`][meta-mbl]
  at `meta-mbl-distro/conf/layer.conf`. Make sure you check the branch or tag
  of `meta-mbl` on which you are basing your project.
* The [`BBFILE_PRIORITY`][yocto-bbfile-priority] for the new layer should be
  higher than the `BBFILE_PRIORITY` for the `meta-mbl-distro` layer, so that
  the `meta-mbl-project-example` layer can override recipes and settings from
  `meta-mbl-distro`.
* The [`LAYERDEPENDS`][yocto-layer-depends] line declares a dependency on the
  `meta-mbl-distro` layer. This enables BitBake to automatically determine that
  `meta-mbl-project-example`'s `BBFILE_PRIORITY` should be higher than
  `meta-mbl-distro`'s without you having to hard-code a value.

## 2. Create a config repository

Currently, the easiest way to create a config repository for an MBL-based
project is to fork the [mbl-config][mbl-config] repository, which contains a `bblayers.conf` file. In this example,
we'll assume your fork is called `mbl-project-config-example`.

To add your new layer to the project, edit the `BBLAYERS` variable in the `bblayers.conf` file. The value
of the `BBLAYERS` variable is the list of layers to include in the
project. Add your layer to that list. Make sure you edit the file from the
branch or tag of mbl-config on which you are basing your project.

For example, after adding `meta-mbl-project-example`, the `BBLAYERS` variable definition might look like:

```
BBLAYERS = " \
  ${OEROOT}/layers/meta-mbl/meta-mbl-distro \
  ${OEROOT}/layers/meta-mbl/meta-mbl-apps \
  ${BASELAYERS} \
  ${BSPLAYERS} \
  ${EXTRALAYERS} \
  ${OEROOT}/layers/openembedded-core/meta \
  ${OEROOT}/layers/meta-mbl/openembedded-core-mbl/meta \
  ${OEROOT}/layers/meta-mbl-project-example/ \
"
```

## 3. Create a manifest repository

The new project will use code from many repositories. To make it easy to create
a work area for the project with all of the repositories cloned to the right
places within the work area<!--why places? won't it be cloned to a single place? or should "work areas" be plural, like the last sentence?-->, you can <!--can or must?-->create a repo manifest file containing information about the
project's repos, and then use [Google's git-repo tool][google-git-repo] to
initialize work areas using that manifest.

MBL's build tools use [Google's git-repo tool][google-git-repo] to initialize work areas, so you must create a manifest repository in order to use them.

1. Create a repository to contain the manifest file, for example a GitHub repo called `mbl-project-manifest-example`.
1. The new project will be very similar to the MBL project, so you can copy the `default.xml` manifest file from
[mbl-manifest][mbl-manifest] repository into the new repo. Use the `mbl-manifest` branch or tag on which you want to base your project. <!--why not fork, like they did for the config repo?-->
1. Modify the `default.xml` file as needed.  

    1. The manifest will contain a line for the config repo that looks something like:

       ```
       <project name="armmbed/mbl-config" path="conf" remote="github" revision="mbl-os-0.8">
       ```

       Change this line to refer to a branch of your new config repo instead of `armmbed/mbl-config`. For example:

       ```
       <project name="your-github-user/mbl-project-config-example" path="conf" remote="github" revision="master">
       ```

       Replace:
       * `your-github-user` with the GitHub user or organization that has your `mbl-project-config-example` repository.
       * `master` with the branch or tag of `mbl-project-config-example`
         containing the `bblayers.conf` you created in step 2.

    1. Add a line within the

       ```
       <manifest>
       ...
       </manifest>
       ```

       block for your new meta layer. Something like:

       ```
       <project name="your-github-user/meta-mbl-project-example" path="layers/meta-mbl-project-example" remote="github" revision="master" />
       ```

       Replace:
       * `your-github-user` as above.
       * `master` with the branch or tag of `meta-mbl-project-example` that
         contains the `layer.conf` you created in step 1.

1. Use [MBL's build tools][mbl-tools] to build the project. Use the `--url` option in `build.sh` to specify your
manifest repository (so that it doesn't use the default repo, [mbl-manifest][mbl-manifest]).

    For example:

    ```
    mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch master --machine imx7s-warp-mbl --url ssh://git@github.com/your-github-user/mbl-project-manifest-example
    ```

    Replace:

    * `/path/to/build` with a directory to use as a work area.
    * `/path/to/output` with a directory to contain build artifacts.
    * `imx7s-warp-mbl` with the name of your target machine.
    * `your-github-user` with your GitHub username or organization.
    * `master` with the branch or tag of `mbl-project-manifest-example`
      containing `default.xml`.

    If you want to initialize a work area for your project without doing a
    build you can use [Google's git-repo tool][google-git-repo] directly by
    running something like:
    ```
    repo init -u ssh://git@github.com/your-github-user/mbl-project-manifest-example -b master
    repo sync
    ```
    Replace:
    * `your-github-user` with your GitHub username or organization.
    * `master` with the branch or tag of `mbl-project-manifest-example`
      containing `default.xml`.

## 4. Add an image recipe

To modify the distribution, you need a new image recipe, based on an MBL
image. For example, to create a recipe for a new `mbl-image-example`
image:

1. In `meta-mbl-project-example`, under `recipes-core/images/`, create the file `mbl-image-example.bb`.
1. Add the following content to the `mbl-image-example.bb` file:

    ```
    inherit mbl-distro-image

    SUMMARY = "Example image based on Mbed Linux OS"
    DESCRIPTION = "A minimal image containing MBL and nothing else"
    HOMEPAGE = "https://example.com"
    ```

The `inherit mbl-distro-image` line means this recipe includes everything from a
standard MBL image into the new image.

To build this image using [MBL's build tools][mbl-tools], use the `--image` option in `build.sh`:

```
mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch master --machine imx7s-warp-mbl --url ssh://git@github.com/your-github-user/mbl-project-manifest-example --image mbl-image-example
```

Replace:

* `/path/to/build` with a directory to use as a work area.
* `/path/to/output` with a directory to contain build artifacts.
* `imx7s-warp-mbl` with the name of your target machine.
* `your-github-user` with your GitHub username or organization.
* `master` with the branch or tag of `mbl-project-manifest-example` containing
  `default.xml`.

## 5. Modify the distribution

### 5.1. Add packages to the distribution

To add a package to the distribution that already has a recipe available in
your project, add a line in your image recipe
(`recipes-core/images/mbl-image-example.bb` in `meta-mbl-project-example`) to
append the package name to the `IMAGE_INSTALL` variable.

For example, to add the `bash` package to the distribution, add the following line:

```
IMAGE_INSTALL_append = " bash"
```
<!--is the space before bash on purpose?-->

The `bash` package has a recipe in the `openembedded-core` meta layer that is
already included in the project.

### 5.2. Add layers to the distribution

To add a package from a layer that isn't already included in the project,
you first need to add the layer that contains the package <!--what layer? the layer that includes the package, I presume?-->to the project. You can follow the same procedure as the `meta-mbl-project-example` layer in steps 1 and 2:

1. Add the repository containing the layer to your manifest file (in
   `mbl-project-manifest-example`).
2. Add the layer to the `BBLAYERS` variable in `bblayers.conf` in
   `mbl-project-config-example`.

You can then add any of the layer's packages to your image by appending their names to `IMAGE_INSTALL` in your
image recipe.

<span class="tips">To find out if a layer providing the package you need already exists, try the [OpenEmbedded Layer Index][oe-layer-index].</span>

### 5.3. Add your own recipes

If you have software for which there aren't existing BitBake recipes, you can
add recipes to the `meta-mbl-project-example` layer, and then include the software
in the image by adding the package names to `IMAGE_INSTALL` (as above).

Details about writing recipes are beyond the scope of this document - please see
the [Yocto documentation][yocto-docs]. In particular, you may find the [Writing
a New Recipe section of the Yocto Project Development Manual][yocto-recipes]
useful.

When developing new software to run in your distribution, you may find the
Yocto SDK tools useful. See [Yocto Project Application Development and the
Extensible Software Development Kit (eSDK)][yocto-sdk] for more information.

[meta-mbl]: https://github.com/ARMmbed/meta-mbl
[mbl-config]: https://github.com/ARMmbed/mbl-config
[mbl-manifest]: https://github.com/ARMmbed/mbl-manifest
[mbl-tools]: https://github.com/ARMmbed/mbl-tools

[yocto-docs]: https://www.yoctoproject.org/docs/
[yocto-bbfile-priority]: https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#var-bb-BBFILE_PRIORITY
[yocto-layer-depends]: https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#var-bb-LAYERDEPENDS
[yocto-layerseries-compat]: https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#var-LAYERSERIES_COMPAT
[yocto-create-layer]: https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#creating-your-own-layer
[yocto-recipes]: https://www.yoctoproject.org/docs/1.6/dev-manual/dev-manual.html#new-recipe-writing-a-new-recipe
[yocto-sdk]: https://www.yoctoproject.org/docs/latest/sdk-manual/sdk-manual.html
[oe-layer-index]:https://layers.openembedded.org

[google-git-repo]: https://gerrit.googlesource.com/git-repo/


***

Copyright © 2020 Arm Limited (or its affiliates)
