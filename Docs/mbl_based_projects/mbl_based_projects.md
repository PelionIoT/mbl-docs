# Example project based on Mbed Linux OS

Mbed Linux OS is a project based on Yocto and standard Yocto tools and
approaches can be used to develop projects based on Mbed Linux OS. This
document will illustrate a simple approach to creating a project based on MBL,
but is not intended to replace the [Yocto documentation][yocto-docs]. This
document will assume familiarity with some core Yocto concepts. Please see the
[Yocto Project Concepts section of the Yocto Mega Manual][yocto-concepts] for
an introduction to these concepts.

## Goal

This document will walk through the steps necessary to create a minimal example
project, based on MBL, in which you can:
* Add packages with pre-existing BitBake recipes to the distribution.
* Add packages without pre-existing BitBake recipes to the distribution.

## Solution overview
The steps to create the project are:
1. Create a new BitBake meta layer called "meta-mbl-project-example". This
   layer will be used to define a new image recipe for building an image based on
   MBL.
2. Create a config repository to hold the project's configuration.
3. Create a manifest repository to make it easy to download all of the
   repositories required for the project. This will also enable you to use MBL's
   build tools to build the new project.
4. Add an image recipe to the new layer.
5. Modify the distribution.

## 1. Create a new BitBake meta layer
You will need a meta layer in which to add new BitBake recipes and
configuration. First, create a minimal layer which just consists of:
* A repository on e.g. GitHub called "meta-mbl-project-example".
* A layer configuration file.

Once the repository has been created, add a `conf` directory containing a file
called `layer.conf` with content like the following:
```
BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb ${LAYERDIR}/recipes*/*/*.bbappend"
BBFILE_COLLECTIONS += "meta-mbl-project-example"
BBFILE_PATTERN_meta-mbl-project-example := "^${LAYERDIR}/"

BBFILE_PRIORITY_meta-mbl-project-example = "20"

LAYERSERIES_COMPAT_meta-mbl-project-example = "warrior"
LAYERDEPENDS_meta-mbl-project-example = "meta-mbl-distro"
```

Most of the lines here are standard and are explained in the [Creating Your Own
Layer section of the Yocto Mega Manual][yocto-create-layer], although please note:
* The `BBFILE_PRIORITY` for the new layer should be higher than the `BBFILE_PRIORITY`
  for the meta-mbl-distro layer so that the meta-mbl-project-example layer can
  override recipes and settings from meta-mbl-distro.
* The `LAYERDEPENDS` line declares a dependency on the meta-mbl-distro layer.

## 2. Create a config repository
Currently, the easiest way to create a config repository for an MBL based
project is to fork the [mbl-config][mbl-config] repository. In this example,
we'll assume that you have a fork of the mbl-config repository called
"mbl-project-config-example".

Once you have a config repository, you can edit the `bblayers.conf` file in the
repository to add your new layer to the project. In `bblayers.conf` the value
of the `BBLAYERS` variable is the list of layers that are be included in the
project. Add your layer to that list. For example, after adding
meta-mbl-project-example, the BBLAYERS variable definition might look like:
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
places, you can create a repo manifest file containing information about the
project's repos and then use [Google's git-repo tool][google-git-repo] to
initialize work
areas using that manifest.

First, create a repository on e.g. GitHub called "mbl-project-manifest-example"
to contain the manifest file. The new project will be very similar to the MBL
project so you can start by copying the `default.xml` manifest file from
[mbl-manifest][mbl-manifest] into the new repo and making some small
modifications. Use the `default.xml` manifest from the branch or tag of
mbl-manifest that you want to use as the base for your project.

The manifest will contain a line for the config repo that looks something like:
```
  <project name="armmbed/mbl-config" path="conf" remote="github" revision="mbl-os-0.6">
```
You should change this line to refer to a branch of your new config repo
instead of `armmbed/mbl-config`. For example:
```
  <project name="your-github-user/mbl-project-config-example" path="conf" remote="github" revision="master">
```
Replacing `your-github-user` with the GitHub user or organization that has your
mbl-project-config-example repository.

You will also need to add a line within the
```
<manifest>
...
</manifest>
```
block for your new meta layer. Something like:
```
  <project name="your-github-user/meta-mbl-project-example" path="layers/meta-mbl-project-example" remote="github" revision="master" />
```
Replacing `your-github-user` as above.

At this point, you should be able to create a work area for the new project
using [Google's git-repo tool][google-git-repo] by running something like:
```
repo init -u ssh://git@github.com/your-github-user/mbl-project-manifest-example
repo sync
```
Replacing `your-github-user` with your GitHub username or organization.

You should also be able to use [MBL's build tools][mbl-tools] to build the
project. You'll have to use `build.sh`'s `--url` option to specify your
manifest repository so that it doesn't use the default repo,
[mbl-manifest][mbl-manifest]. E.g:
```
mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch master --machine imx7s-warp-mbl --url ssh://git@github.com/your-github-user/mbl-project-manifest-example
```
Replacing:
* `/path/to/build` with a directory to use as a work area.
* `/path/to/output` with a directory to contain build artifacts.
* `imx7s-warp-mbl` with the name of your target machine.
* `your-github-user` with your GitHub username or organization.

## 4. Add an image recipe
To modify the distribution, you'll need a new image recipe, based on an MBL
image. For example, you could create a recipe for a new "mbl-image-example"
image in meta-mbl-project-example at the path
`recipes-core/images/mbl-image-example.bb`:
```
inherit mbl-distro-image

SUMMARY = "Example image based on Mbed Linux OS"
DESCRIPTION = "A minimal image containing MBL and nothing else"
HOMEPAGE = "https://example.com"
```
The `inherit mbl-distro-image` line essentially includes everything from a
standard MBL image into the new image.

You can build this image using [MBL's build tools][mbl-tools] by
specifying `build.sh`'s `--image` option:
```
mbl-tools/build-mbl/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch master --machine imx7s-warp-mbl --url ssh://git@github.com/your-github-user/mbl-project-manifest-example --image mbl-image-example
```
Replacing:
* `/path/to/build` with a directory to use as a work area.
* `/path/to/output` with a directory to contain build artifacts.
* `imx7s-warp-mbl` with the name of your target machine.
* `your-github-user` with your GitHub username or organization.

## 5. Modify the distribution

### 5.1. Adding packages to the distribution

To add a package to the distribution that already has a recipe available in
your project, you simply need to add a line in your image recipe to append the
package name to the `IMAGE_INSTALL` variable. For example, to add the `bash`
package to the distribution, add the following line:
```
IMAGE_INSTALL_append = " bash"
```
The `bash` package has a recipe in the openembedded-core meta layer that is
already included in the project.

### 5.2. Adding layers to the distribution
To add a package from a layer that isn't already included in the project,
you'll first need to add the layer to the project. This can be done in the same
way as the meta-mbl-project-example layer was added to the project in steps 1
and 2:
1. Add the repository containing the layer to your manifest file (in
   mbl-project-manifest-example).
2. Add the layer to the `BBLAYERS` variable in `bblayers.conf` in
   mbl-project-config-example.

After performing these two steps you can add any packages that the layer
provides into your image by appending their names to `IMAGE_INSTALL` in your
image recipe.

To find out if a layer already exists that provides the package you need, the
[OpenEmbedded Layer Index][oe-layer-index] can be useful.

### 5.3. Adding your own recipes
If you have software for which there aren't existing BitBake recipes, you can
add recipes to the meta-mbl-project-example layer and then include the software
in the image by adding the package names to `IMAGE_INSTALL` as in the previous
subsections.

Details about writing recipes is beyond the scope of this document - please see
the [Yocto documentation][yocto-docs]. In particular, you may find the [Writing
a New Recipe section of the Yocto Project Development Manual][yocto-recipes]
useful.

When developing new software to run in your distribution, you may find the
Yocto SDK tools useful. See [Yocto Project Application Development and the
Extensible Software Development Kit (eSDK)][yocto-sdk] for more information.

[mbl-config]: https://github.com/ARMmbed/mbl-config
[mbl-manifest]: https://github.com/ARMmbed/mbl-manifest
[mbl-tools]: https://github.com/ARMmbed/mbl-tools

[yocto-docs]: https://www.yoctoproject.org/docs/
[yocto-concepts]: https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#overview-manual-concepts
[yocto-create-layer]: https://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#creating-your-own-layer
[yocto-recipes]: https://www.yoctoproject.org/docs/1.6/dev-manual/dev-manual.html#new-recipe-writing-a-new-recipe
[yocto-sdk]: https://www.yoctoproject.org/docs/latest/sdk-manual/sdk-manual.html
[oe-layer-index]:https://layers.openembedded.org

[google-git-repo]: https://gerrit.googlesource.com/git-repo/
