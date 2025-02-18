# Guide on how to build kernel image & introduce to Linux kernel modules

Bạn cần nắm vững kiến thức về Linux kernel cơ bản trước khi tiếp tục.

Board được sử dụng trong guide: Raspberry Pi 4, 4GB RAM version.

AOSP tag được sử dụng trong guide: `android-15.0.0_r10`

## Getting started
Trong các hướng dãn trước, chúng ta đã biết được cách build AOSP và nạp nó lên Raspberry Pi, đồng thời biết thêm về cách cài đặt ADB. Nhưng bạn có bao giờ nghĩ rằng, làm sao mà phần mềm lại có thể tương tác được với hệ thống không?

Câu trả lời là **kernel**.

Android chạy trên Linux kernel. Thông thường thì các thiết bị Android sẽ chạy trên một branch LTS downstream của Linux kernel (chẳng hạn như 3.18, 4.4, 4.9, 4.19, 5.4, 5.10, 5.15,...) và Google sẽ hỗ trợ upstream và cũng như vá lỗi cho những phiên bản đó trong vòng 6 năm cho đến khi phiên bản đó không còn được hỗ trợ nữa.

Chúng ta sẽ đào sâu hơn vào về việc build kernel image cho Android, và Bazel build system.

Dung lượng ổ cứng trống cần nhiều hơn hoặc bằng 40GB để tải cả source code của build system về.

## Tải kernel source và kernel build system
Để tải source code về, tạo 1 thư mục trống và chạy `repo init` ở trong đó để cài đặt môi trường clone:

```
mkdir aosp-kernel
cd aosp-kernel
repo init -u https://android.googlesource.com/kernel/manifest -b common-android15-6.6-lts
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_kernel_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
```

Khi mà bạn đã chạy xong command trên, chạy command dưới để clone source code:
```
repo sync
```
Nếu bạn muốn tiết kiệm dung lượng trống (không khuyến khích làm điều này, vì repo sẽ sync source không kèm với commit history - nhưng tuỳ vào lựa chọn của bạn):

```
repo sync -c --force-sync --optimized-fetch --no-tags --no-clone-bundle --prune --retry-fetches=5 -j$(nproc --all)
```

Để xem lại về các lỗi trong lúc sync, hãy xem lại trong [Unit 1](https://github.com/log1cs/documents/blob/main/unit1/Unit1_VI.md).

Khi mà source code đã tải về xong, chúng ta sẽ đến bước tiếp theo.

## Build the kernel
Chạy command dưới để build kernel:
```
tools/bazel build --config=fast --config=stamp //common:rpi4
```

Thời gian build sẽ tốn rất lâu, phụ thuộc vào cấu hình máy tính của bạn.

Kết quả sẽ trông như dưới, một khi tiến trình build đã xong:
```
INFO: Elapsed time: x.xxxs, Critical Path: x.xxs
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
```

## Lấy file output
Để lấy file kernel sau khi vừa build xong, chuyển sang directory dưới:
```
aosp-kernel/bazel-bin/common/rpi4
```

Trong đó `Image` là file của kernel image. Bạn có thể thay thế các file còn tồn tại ở trong thư mục `device/brcm/rpi4-kernel` hoặc thay thế các file ở trong thư mục `boot` của Raspberry Pi.

# Linux kernel driver
Trong trường hợp bạn tự hỏi "kernel driver là gì", hãy đọc [post này](https://www.apriorit.com/dev-blog/195-simple-driver-for-linux-os). 

Bạn có nhớ những chương trình đầu tiên mà bạn viết bằng C/C++ không? Đúng vậy, lần này nó sẽ là ***Hello world*** một lần nữa, nhưng lại là kernel driver.

Bạn có thể build driver của mình vào thẳng trong kernel, hoặc dưới 1 dạng module.

## Tạo ra 1 kernel driver
Để tạo một kernel driver, hãy vào thư mục driver của kernel source. Nếu bạn vẫn đang follow guide dưới, nó sẽ nằm trong thư mục này:

```
aosp-kernel/common/drivers
```

Tạo một thư mục mới, đặt tên là gì cũng được. Ở đây tôi sẽ đặt tên là `HelloWorld`.

```
mkdir HelloWorld
cd HelloWorld
```

## Cấu hình Makefile
Để cài đặt các rule để build ở trong kernel, bạn cần một `Makefile`

Tạo một file tên là `Makefile` và mở nó lên, sau đó thêm dòng này:

```
obj-$(CONFIG_HELLO) += HelloWorld.o
```

Để hiểu rõ hơn về đoạn code trên: `HelloWorld.o` sẽ được build khi CONFIG_HELLO được khai báo là `=y` hoặc `=m` trong defconfig.

Nếu không được khai báo hoặc khai báo là `=n`, thì driver sẽ không được build.

### Further explainations:
```
obj-y += <target>        # Tích hợp driver vào thẳng trong kernel.

obj-m += <target>        # Build driver dưới dạng module.
```

## Cấu hình Kconfig/Kbuild (hay còn gọi là kernel config/kernel build)
Tạo một file tên là `Kconfig` và mở nó lên, sau đó thêm dòng sau

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

`CONFIG_HELLO=y` sẽ được tự động thêm vào khi bạn regenerate config bằng `savedefconfig`.  

## The kernel driver itself
Tạo một file tên là `HelloWorld.c` và mở nó lên - bạn có thể tham khảo đoạn code dưới:

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

## Thêm driver vào trong driver global Makefile
Để `make` có thể thấy được driver của bạn, thì bạn phải khai báo driver trong Makefile hoặc `Kconfig` ở trong thư mục global `drivers`.

Nếu bạn vẫn không biết global `drivers` là gì, thì nó sẽ nằm ở thư mục dưới (với `common` là nơi chứa kernel source của bạn):
```
aosp-kernel/common/drivers
```

Mở `Makefile`, và thêm dòng dưới vào cuối file file:
```
obj-$(CONFIG_HELLO) += HelloWorld/
```

Sau đó thì mở `Kconfig`, thêm dòng dưới trước dòng `endmenu`:
```
source "drivers/HelloWorld/Kconfig"
```

Và thế là bạn đã xong với phần kernel driver! Lưu ý rằng đây chỉ là một driver đơn giản, không phải cho việc sử dụng hằng ngày.

## Khai báo trong defconfig
Một kernel driver có tỉ lệ chạy được là 0 nếu nó không được khai báo trong defconfig hoặc set là `obj-y` trong Makefile. Để bật nó lên, hãy truy cập thư mục dưới:

```
aosp-kernel/common/arch/<kiến trúc CPU>/configs/<defconfig của board>
```

Nếu bạn vẫn còn đang follow guide này, thì đường dẫn sẽ là:

```
aosp-kernel/common/arch/arm64/configs/
```

Trong đó sẽ có 1 file tên là `android_rpi4_defconfig`. Dưới dòng `CONFIG_` đầu tiên, thêm dòng dưới:

```
CONFIG_HELLO=y
```

Khai báo `CONFIG_HELLO=y` để khi build system regenerate `.config`, thì config trên sẽ được thêm vào.

# Build và cách lấy kernel module.
Có 2 cách để compile kernel module: Một là sử dụng `make` và hai là sử dụng Bazel. Ở đây chúng ta đang sử dụng Bazel để compile nguyên cả bộ kernel, cho nên là chúng ta sẽ sử dụng nó luôn để compile module.

```
TODO.
```

## Load kernel module ở trên thiết bị
Đầu tiên, hãy bật Gỡ lỗi USB và Gỡ lỗi với quyền Root ở trong Tuỳ chọn nhà phát triển.

Một khi bạn đã xong, kết nối board đó với PC và chạy lệnh dưới:

```
adb root
adb devices
adb push HelloWorld.ko /sdcard  
```

Lệnh trên sẽ đẩy `HelloWorld.ko` vào thư mục /sdcard, nghĩa là bộ nhớ trong của bạn.

### Load kernel modules
Chạy lệnh dưới để load kernel module vào trong thiết bị:

```
insmod /sdcard/HelloWorld.ko
```

Lệnh này sẽ load kernel module.

Để check xem module đã thực sự được load hay chưa, chúng ta chỉ cần check trong `dmesg`.

```
dmesg | grep "Hello"
```

Nếu module đã được load thành công, thì log dưới sẽ được print ra:

```
[   xx(xx).xxxxxx] Hello world!
```

Để unload kernel module, chạy lệnh dưới:
```
rmmod hello.ko
```

Lệnh trên sẽ unload kernel module, đồng thời sẽ print ra:
```
[   xx(xx).xxxxxx] Exiting.
```

