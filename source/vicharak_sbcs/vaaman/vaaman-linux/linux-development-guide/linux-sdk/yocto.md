# Linux SDK

Working with Vicharak computer boards is very easy as we provide all the
necessary source, tools and scripts required to build your own Linux systems.

## Build Linux SDK from source

The Vicharak Linux SDK is based on SDK provided by Rockchip hence, it follows
the conditions and criteria of Rockchip.

Vicharak provides multiple different sources for building your own Linux images.
Take a look at the table below:

:::{list-table}
:header-rows: 1
:class: feature-table

-
  - Linux distribution
  - Version

-
  - Yocto
  - Mickledore (4.2)

-
  - Android
  - 12.1

:::

These sources are available at [Vicharak GitHub](https://github.com/vicharak-in)

### Installing the package dependencies for environment setup

#### Install source control tools

Building Vicharak linux systems requires both **Git** (For source code management)
and **Repo** (Google-built repository management tool).

Please refer to Android's source control tools setup (https://source.android.com/docs/setup/download) as we follow the same for building Vicharak's Linux systems.

#### Install package dependencies

:::{important}
Make sure you have properly installed `repo` tool before proceeding.
:::

**The following packages are required to build the rootfs image.**

```bash
sudo apt-get update -y

sudo apt install -y asciidoc autotools-dev bash bc binutils bison \
build-essential bzip2 chrpath cpio curl cvs dblatex default-jre \
device-tree-compiler diffstat expect-dev fakeroot file flex g++ gawk gcc \
gcc-aarch64-linux-gnu gcc-arm-linux-gnueabihf g+conf genext2fs git git-core \
git-gui gitk graphviz gzip intltool lib32stdc++6 libdrm-dev libglade2-dev \
libglib2.0-dev libgtk2.0-dev liblz4-tool libncurses5 libparse-yapp-perl \
libsigsegv2 libssl-dev libudev-dev libusb-1.0-0-dev m4 make mercurial mtools \
openssh-client parted patch patchutils perl python3 python-is-python3 \
qemu-user-static rsync sed subversion swig tar texinfo u-boot-tools unzip w3m \
wget
```

<!-- TODO: Fix when the issues section is available -->

:::{note}
The above packages might not be available to install on your system. Please
try to find the alternatives or a solution from the internet.
:::

### Getting the sources

We have structured our Linux development source code in the same way as the
official Android sources. i.e. we have various manifests with different
branches for different releases of our supported Linux distributions and products.
**repo** tool will take care of the source code syncing and updating.

Take a look at the following table to get the latest sources.

| Manifest         | Tag                 | URL                                                                                |
| ---------------- | ------------------- | ---------------------------------------------------------------------------------- |
| Yocto Mickledore | -                   | https://github.com/vicharak-in/rockchip-linux-manifests/tree/master       |
| Android 12       | android-12          | https://github.com/vicharak-in/rockchip-android-manifest                  |

#### Cloning the source

To clone the latest sources, use the following command.

::::{tab-set}

:::{tab-item} Yocto Mickledore (4.2)

```bash
repo init --no-tags --no-clone-bundle -u https://github.com/vicharak-in/rockchip-linux-manifests -b master
```

:::

:::{tab-item} Android 12

```bash
repo init --no-tags --no-clone-bundle -u https://github.com/vicharak-in/rockchip-android-manifest -b master -m rockchip-s-vicharak.xml
```

<!-- TODO: Fix when Android manifest is available -->

:::

::::

:::{tip}
You can also shallow clone entire SDK using `--depth=1` option.
Use the following command to sync the source with shallow history:

```bash
repo init --depth=1 --no-tags --no-clone-bundle -u <url> -b <branch> -m <manifest>
```

:::

#### Syncing the source

To download the code sources from the repositories to your local machine, use the following command.

```bash
repo sync -j$(nproc)
```

### Building Yocto Image

#### Mickledore (4.2)

After successfully syncing the Yocto sources, you are now ready to build a Yocto-based Linux image for Vicharak boards.

#### 1. Source the Yocto build environment

From the SDK root directory, initialize the Yocto build environment:

```bash
source oe-init-build-env
```

This will create (or switch to) a build/ directory containing the Yocto configuration files.

#### 2. Configure Yocto layers

Ensure that your conf/bblayers.conf file includes the meta-rockchip layer along with other required layers.

Example conf/bblayers.conf

```bash
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  ${TOPDIR}/../meta-openembedded/meta-oe \
  ${TOPDIR}/../meta-openembedded/meta-python \
  ${TOPDIR}/../meta-openembedded/meta-networking \
  ${TOPDIR}/../meta-openembedded/meta-filesystems \
  ${TOPDIR}/../meta-openembedded/meta-multimedia \
  ${TOPDIR}/../meta-qt5 \
  ${TOPDIR}/../meta-clang \
  ${TOPDIR}/../meta-rockchip \
  ${TOPDIR}/../poky/meta \
  ${TOPDIR}/../poky/meta-poky \
  ${TOPDIR}/../poky/meta-yocto-bsp \
"
```

::::{note}
Make sure all referenced layers exist relative to your build directory.
::::

#### 3. Select the target machine

To enable a specific Vicharak board, set the MACHINE variable in conf/local.conf.

You can find the list of supported machines at:
```bash
meta-rockchip/conf/machine/
```

Example conf/local.conf
```bash
MACHINE ?= "rk3399-vaaman"
```

#### 4. Enable systemd (recommended)

Vicharak Yocto images use systemd as the default init system. Enable it by adding the following to conf/local.conf:
```bash
INIT_MANAGER = "systemd"
```

To support Vicharak Specific Application, use below **local.conf** file :

```bash
# Copyright (c) 2023 Vicharak Computers LLP

include include/common.conf
include include/demo.conf

MACHINE_FEATURES += "ext4"
PACKAGE_CLASSES = "package_deb"
EXTRA_IMAGE_FEATURES:append = " package-management "
IMAGE_INSTALL:append = " rockchip-init rockchip-mount bitman bitman-dev rah rah-dev periplex x11-vicharak u-boot-update vicharak-config vicharak-pkg" 
KERNEL_DANGLING_FEATURES_WARN_ONLY = "1"
IMAGE_BOOT_FILES += "Image rk3399-vaaman-linux-rk3399-vaaman.dtb extlinux/extlinux.conf "
MACHINE = "rk3399-vaaman"
DISTRO_FEATURES:append = "systemd pam x11"
```

::::{note}
If another init system is already configured, ensure it is removed or overridden.
::::



#### 5. Building the Yocto image

Once the environment and configuration are complete, start building the image using bitbake.

Example build command:
```bash
bitbake core-image-full-cmdline
```

#### 6. Build output

After a successful build, the generated images can be found at:
```bash
build/tmp/deploy/images/<MACHINE>/
```

This directory will contain:

A **.wic image (SD Card)** (bootable disk image) and can direcly flashing using **dd** command or **Balena Etcher Tool**.

**Default Credential :**

```{note}
Username : root

Password : root
```
