# Guide on how to build kernel image & introduce to Linux kernel modules

You need some basic knowledge of Linux kernels in order to follow this guide.

Applied targets: Raspberry Pi 4, 4GB RAM version.

AOSP tag used in this guide: `android-15.0.0_r10`

## Getting started
In the previous guide, we already know how to build an AOSP image and flash it to the Pi, also setting up ADB. But have you wondered, how does system softwares interact (contacts) with hardware?

The answer is **kernel**.

Android runs on a Linux kernel. Usually Android handheld/devices are running on a downstream LTS branch of a Linux kernel (e.g. 4.4, 4.9, 4.19, 5.4, 5.10, 5.15,...), and Google maintains it for around 6 years until they're not supported anymore (end-of-life). 

We'll be digging down further on how to build a kernel image for Android, and Bazel build system.

This guide is for **standalone** build only.

40GB of storage space is required to clone the whole build system.

## Cloning the kernel source and the build system
To clone the kernel source, initialize an empty folder and do a `repo init` in that folder, in order to set up a cloning environment:

```
mkdir aosp-kernel
cd aosp-kernel
repo init -u https://android.googlesource.com/kernel/manifest -b common-android15-6.6-lts
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_kernel_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
```

Once you're done, run this command to sync the source code:
```
repo sync
```
Or if you want to save even more space, run this command (this is not recommended, as commit history are very important in this guide, but **FWIW**):

```
repo sync -c --force-sync --optimized-fetch --no-tags --no-clone-bundle --prune --retry-fetches=5 -j$(nproc --all)
```

To troubleshoot syncing issues, back to [Unit 1](https://github.com/log1cs/documents/blob/main/unit1/Unit1_EN.md).

After the cloning process has been done, head toward the next step.

## Build the kernel
Run the following command to build the kernel:
```
tools/bazel build --config=fast --config=stamp //common:rpi4
```

The process will take a long time, depends on your computer specifications. Grab a coffee when kernel is building.

The result will be like this, once it's done:
```
INFO: Elapsed time: x.xxxs, Critical Path: x.xxs
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
```

Actually I mildly wonder, how does Bazel do output streaming?

## Grab the output kernel image
Alright back to work, to grab the kernel image, head to:
```
aosp-kernel/bazel-bin/common/rpi4
```

Where `Image` file is the kernel image. You can replace existing files in `device/brcm/rpi4-kernel` directory or replace the existing files of the `boot` partition of the Pi.

# Linux kernel driver
In case you wondered "what is a kernel driver", refer to [this post](https://www.apriorit.com/dev-blog/195-simple-driver-for-linux-os). This one provides enough information for you to understand it better. 

**TL;DR:** This one is used to "communicate" with a specific hardware.

Do you remember your very first C/C++ course was? Yes, this time it's the ***Hello World*** stuff again, but as a kernel driver.

You can have your driver built-in to the kernel in Android, or as a loadable kernel module.

Enough with the talking, back to work.

## Create a kernel driver
To create a kernel driver, head towards your kernel source driver folder. If you're still following the guide, it should be in:

```
aosp-kernel/common/drivers
```

Create a new folder, name it whatever you wants, in this case, I'll name it as "HelloWorld"

```
mkdir HelloWorld
cd HelloWorld
```

## Implementing Makefile
To setup a rule for building in kernel, you need a `Makefile`.

Create a file named `Makefile` and open it, and then insert this line:

```
obj-$(CONFIG_HELLO) += HelloWorld.o
```

To understand the context better, this will build `HelloWorld.o` when CONFIG_HELLO is `=y` or `=m` in defconfig.

If not defined or `=n`, it won't build.

### Further explainations:
```
obj-y += <target>        # Build the driver into the kernel

obj-m += <target>        # Build the driver as a kernel module.
```

## Implementing Kconfig/Kbuild (or kernel config/kernel build)
Create a file named `Kconfig` and open it, and then insert this line:

### Kconfig
```
menuconfig HELLO
    tristate "Hello World string"
    default y
    help
        This kernel driver will print a Hello World string in dmesg.

        Say Y to compile this kernel driver.
        Say N to skip.
        Say M to build this driver as a kernel module.
```

This will add `CONFIG_HELLO=y` automatically when you regenerate the defconfig using `savedefconfig` because by default it's `=y`.

## The kernel driver itself
Create a file named `HelloWorld.c` and then open it, you can take the code below as a reference - or, just copy and paste the whole thing if you don't have time:

```
#include <linux/module.h>	 /* Needed by all modules */
#include <linux/kernel.h>	 /* Needed for KERN_INFO */
#include <linux/init.h>	 /* Needed for the macros */

// The license type -- this affects runtime behavior
MODULE_LICENSE("GPL");

// The author -- visible when you use modinfo
MODULE_AUTHOR("Bui Dinh Hien");

// The description -- see modinfo
MODULE_DESCRIPTION("Kernel driver that prints Hello World string");

static int __init hello_start(void)
{
	printk(KERN_INFO "Hello world!\n");
	return 0;
}

static void __exit hello_end(void)
{
	printk(KERN_INFO "Exiting kernel module.\n");
}

module_init(hello_start);
module_exit(hello_end);
```

## Add support for the driver.
In order to make your driver recognized by the build system, you will need to define it in `Makefile` and `Kconfig` at the top `drivers` folder.

In case you don't get what is "top `drivers` folder" is, this is the following path:

```
aosp-kernel/common/drivers
```

Open `Makefile`, and add this line at the bottom of the file:
```
obj-$(CONFIG_HELLO) += HelloWorld/
```

And then open `Kconfig`, and add this line before the `endmenu` line:
```
source "drivers/HelloWorld/Kconfig"
```

And you are pretty much done with the kernel driver part! Note that this is just a simple kernel driver, not suitable for daily use.

## Define the driver in defconfig
A kernel driver has zero chance to be working if it does not defined in defconfig or being set as `obj-y` in the Makefile. In order to turn it on, head towards:

```
arch/<arch>/configs/<defconfig>
```

If you are still following the guide, back to the `aosp-kernel` folder, and then navigate to:

```
aosp-kernel/common/arch/arm64/configs/
```

And then open a file named `android_rpi4_defconfig`. Under the first `CONFIG_` line, add the following configuration:

```
CONFIG_HELLO=y
```

Define it so when the build system regenerate `.config`, the following configuration will be added.

# Build and grab the kernel module.
There are two ways to compile the kernel module: One is to use `make` and another is to use Bazel. In this guide we're already using Bazel to compile the kernel, so let's just use it to compile the module.

```
TODO.
```

## Push the kernel module into your device
First of all, enable USB Debugging and Rooted debugging, before proceeding further.

Once you got it working, connect the board to your PC and run the following command:

```
adb root
adb devices
adb push HelloWorld.ko /sdcard  
```

This will push `HelloWorld.ko` into /sdcard folder, which is your internal storage.

### Load the kernel modules
Run the following command to load the kernel module in your device:

```
insmod /sdcard/HelloWorld.ko
```

This will load the kernel module.

To check if the module is actually loaded, just check in dmesg:
```
dmesg | grep "Hello"
```

If the module is successfully loaded, you will get something like this:

```
[   xx(xx).xxxxxx] Hello world!
```
That means the kernel module was successfully probed, and it's running.

In order to unload the kernel module, runs this command below:
```
rmmod hello.ko
```

This will unload the kernel module. At the same time, the module will print:
```
[   xx(xx).xxxxxx] Exiting.
```

