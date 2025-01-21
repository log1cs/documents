# Getting started with AOSP 15 (Android 15) on Raspberry Pi 4

Applied target(s): Raspberry Pi 4, 4GB RAM version.

AOSP tag used in this guide: `android-15.0.0_r10`

# What is AOSP?
AOSP stands for Android Open Source Project. It is the open-source foundation of the Android operating system, developed and maintained by Google. AOSP provides the source code and development tools necessary for OEMs to create Android-based devices, applications, and custom ROMs.

# Hardware requirements to build AOSP
This one is kinda chonky. This is my recommendation of hardware specs, for whoever wanted to build AOSP:

| Components | Description                                                                                                        |
|:---------- |:------------------------------------------------------------------------------------------------------------------ |
| OS         | Any Linux distros can be used to build AOSP, but Ubuntu (newer or equal to 20.04) is recommended since it's less likely to run into issues while building. |
| CPU        | 8+ or more CPU cores. Lower CPU cores will result in a longer build time, and vice versa. |
| RAM        | A minimum of 64GB is required. Even though you can build with 16GB with a lot of Swap/zRAM, but it's not recommended. |
| Storage    | At least 500GB of free space to check out the whole source code and building the image. |

This guide will take Ubuntu 20.04 LTS as a reference to build AOSP. Depends on the distro you're using, the command might be different from this one in the guide.

# Dependencies
First of all, your buidling environment must have GLIBC installed. This is present on all Linux distros, but it must be newer than 2.17 to build AOSP.

To check the currently installed GLIBC version:
```
ldd --version | grep GLIBC
```

This is the expected output of the command above (depends on the Linux distro you are using, mine is Ubuntu), if you have glibc installed:
```
ldd (Ubuntu GLIBC 2.39-0ubuntu8.3) 2.39
```

Once that is done, proceed to the second step, which is installing the core dependencies to build Android:

### Installing the dependencies
Run the following command in your terminal:

```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig sudo apt-get install openjdk-11-jdk bc coreutils dosfstools e2fsprogs fdisk kpartx mtools ninja-build pkg-config python3-pip python-is-python3 && sudo pip3 install meson mako jinja2 ply pyyaml dataclasses --break-system-packages
```

The installation time might take longer, depends on your Internet plan.

### Installing Repo
Repo is a CLI binary that allows you to sync source code from a Git server. Because AOSP source code isn't light at all, a *repo sync* might take more than a few hours (maybe 10+) depends on your internet plan.

It's very useful, like you can just throw a manifest file on it and it'll done the sync jobs automatically for ya instead of doing a thousands of `git clone`.

It'd be better to install `repo` binary by hands instead of installing it via `apt` because the `apt` version is always outdated.

Run the following command to install `repo`:
```
export REPO=$(mktemp /tmp/repo.XXXXXXXXX)
curl -o ${REPO} https://storage.googleapis.com/git-repo-downloads/repo
gpg --recv-keys 8BB9AD793E8E6153AF0F9A4416530D5E920F5C65
curl -s https://storage.googleapis.com/git-repo-downloads/repo.asc | gpg --verify - ${REPO} && install -m 755 ${REPO} ~/bin/repo
```
This will download the latest available version of `repo`.

And that's it, you're done establishing your first AOSP building environment!

# Configuring Git username and password
To use any Git command, you must specify your username and email. In order to do that, use this command:

```
git config --global user.email "<your email here>"
git config --global user.name "<your username here>"
```

There is a dedicated guide to use HTTP for Git, in case you wanted to.

# Downloading the source
First of all, create a folder for your AOSP project:
```
cd
mkdir -p aosp/source/
cd aosp/source/
```

This will redirect you to ~/aosp/source.

### Initializing AOSP source in the directory
Run the following command to initialize the AOSP manifest, device tree/kernel for the Raspberry Pi 4, and repo configuration in the current directory:

```
repo init -u https://android.googlesource.com/platform/manifest -b android-15.0.0_r10
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
```

This one will set up `repo` to clone AOSP source, device tree/kernel sources/HALs for the Raspberry Pi 4.

To save space and remove the unneeded components to utilize maximum free space for building AOSP, run this one instead of the one above:

```
repo init -u https://android.googlesource.com/platform/manifest -b android-15.0.0_r10 --depth=1
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
curl -o .repo/local_manifests/remove_projects.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/remove_projects.xml
```

This one will set up `repo` to clone AOSP source and device tree/kernel sources/HALs for the Raspberry Pi 4, BUT without the commit history of every repositories (because the argument `--depth=1` is passed to the `repo init` command) and removing the unneeded projects in the end.

### Download/sync the source code
Run this command to sync the source code:
```
repo sync -c --optimized-fetch --no-tags --no-clone-bundle --prune --retry-fetches=5 -j$(nproc --all)
```

When it's done, `repo` will returns this, indicating that the sync operation is done successfully:
```
repo sync has finished successfully.
```

If you encountered an error while syncing, a few recommendations below could maybe solve your syncing problems:

### Some recommendations for fixing syncing issues

* Add `--force-sync` to force a full resync (this will force a full reclone source-wide, so your uncommited changes might be lost)

* Increasing `--retry-fetches=` so `repo` can try resync more times if a network failure (i.e. a network cut, unstable WLAN driver - this is a common issue with MediaTek WLAN on Linux, etc...) is detected during syncing.

**If nothing is wrong with the syncing stuff, let's proceed to the most exciting part - building AOSP!**

# Building AOSP image & generate a flashable image that can be used on the Raspberry Pi
Run the command below to build the AOSP image for the Raspberry Pi 4:

```
. build/envsetup.sh
lunch aosp_rpi4-ap4a-userdebug
make bootimage systemimage vendorimage -j$(nproc --all)
```

This will take quite a lot of time depending on your hardware specs, so grab a coffee or take a nap when it's building the image. 

For example, my hardware is equipped with the AMD Ryzen™ 9 5950X and 128GB of RAM with a 2TB Samsung 990 Pro NVMe SSD - took me around 2 hours to complete the build.

Once the building process is done, the build system will return this:

```
#### build completed successfully (xx:xx:xx (hh:mm:ss)) ####
```

But this is not done yet - usually on Android phones you can just grab the images and flash it on a unlocked bootloader device using fastboot - this is not the case with the Raspberry Pi.

### Why?
Because with App Imager, everytime you want to write an image on a SD card, it will format the whole thing and start writing from sector 0. And the output image we had has 3 images in total - boot.img contains kernel and cmdline configurations; system.img contains Android itself, and vendor.img stores HALs and blobs/services. And each time we wants to write an image, it has to write from sector 0 again. And Android needs 3 of them to work together, in order to boot it - so, you can't burn the images separately.

The solution for this is simple: `mkimg`. 

@KonstaT already made a shell script that can merge these 3 images into one image files, allow us to burn everything we just built into the SD card!

We'll get into it right now, just follow the guide below.

### Getting a flashable image

First of all you need `cd` to the device tree directory and execute a shell script: 
```
cd device/brcm/rpi4
./mkimg.sh
```

And it should return this, when it's done generating the image:
```
Done, created /out/target/product/rpi4/RaspberryVanillaAOSP15-<builddate>-rpi4.img
```

If you have made it this far, congratulations! The last step would be burning it to the SD card, in order to use Android on the Raspberry Pi 4.

# Burning the image to SD Card

App Imager is not installable by the regular `apt`, so you will need to install it via `snap` - another package management tools.

```
sudo apt update && sudo apt install snapd && snap install rpi-imager
```

When it's installed, open App Imager.

