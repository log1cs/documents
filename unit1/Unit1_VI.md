# Bắt đầu với AOSP

# AOSP là gì?
AOSP là một cụm từ viết tắt của "Android Open Source Project" (dự án Android mã nguồn mở). Đây là nền tảng mã nguồn mở của hệ điều hành Android, được phát triển và duy trì bởi Google. AOSP cung cấp các mã nguồn và các công cụ phát triển cần thiết cho các nhà sản xuất để tạo ra các thiết bị Android, ứng dụng hoặc những bản ROM tuỳ chỉnh dựa trên Android.

# Yêu cầu phần cứng để build AOSP
| Components   | Chú thích                                                                                                        |
|:------------ |:------------------------------------------------------------------------------------------------------------------ |
| Hệ điều hành | Mọi Linux distros đều có thể sử dụng để build AOSP, nhưng nên sử dụng Ubuntu (phiên bản 20.04 hoặc mới hơn) để build vì Ubuntu là một trong những distro có độ tương thích cao nhất với AOSP, nên bạn sẽ ít khi dính phải lỗi hơn. |
| CPU          | 8 nhân hoặc nhiều hơn. CPU càng yếu thì tốc độ build sẽ càng chậm, và ngược lại. |
| RAM          | Yêu cầu tối thiểu 64GB. Mặc dù có thể build với 16GB nhưng sẽ cần rất nhiều Swap (hoặc zRAM) - nhưng không khuyến khích. |
| Storage      | Dung lượng trống phải tối thiểu 500GB để tải source code và để build image. |

Guide này sẽ sử dụng Ubuntu 20.04 làm mẫu để build AOSP. Một số commands ở dưới đây có thể khác nếu bạn dùng các distro khác với hướng dẫn này.

# Dependencies
Đầu tiên thì bạn sẽ phải kiểm tra xem môi trường để build đã có GLIBC hay chưa. GLIBC luôn được cài sẵn ở mọi distro Linux, nhưng để build AOSP thì bạn cần phiên bản GLIBC mới hơn 2.17.

Để kiểm tra phiên bản GLIBC đã được cài đặt trong hệ thống:
```
ldd --version | grep GLIBC
```

Đây là đầu ra mong muốn của command trên (Output có thể khác nếu bạn sử dụng distro khác với hướng dẫn này), nếu như bạn đã cài đặt glibc:
```
ldd (Ubuntu GLIBC 2.39-0ubuntu8.3) 2.39
```

Một khi đã xong với việc check phiên bản GLIBC, thì chúng ta cùng nhau đến bước thứ 2 là cài đặt các core dependencies để build AOSP:

### Cài đặt các dependencies
Chạy command dưới để cài đặt các dependencies:

```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig sudo apt-get install openjdk-11-jdk bc coreutils dosfstools e2fsprogs fdisk kpartx mtools ninja-build pkg-config python3-pip python-is-python3 && sudo pip3 install meson mako jinja2 ply pyyaml dataclasses --break-system-packages
```

Việc cài đặt dependencies có thể tốn nhiều thời gian hơn phụ thuộc vào cấu hình máy hoặc tốc độ Internet của bạn.

### Cài đặt Repo
Repo là một chương trình CLI giúp bạn có thể tải các mã nguồn từ các máy chủ Git. Bởi vì mã nguồn của AOSP không hề nhẹ tí nào nên mỗi lần `repo sync` có thể tốn vài giờ đồng hồ để thành công (thậm chí là hơn 10 tiếng) phụ thuộc vào tốc độ Internet của bạn.

Repo khá hữu dụng trong AOSP, bạn chỉ cần bỏ đúng file manifest vào thôi và **repo** sẽ làm hết việc đồng bộ mã nguồn cho bạn một cách tự động thay vì nhập vài nghìn command `git clone` để tải mã nguồn AOSP.

Về việc cài `repo` thì nên cài bằng thủ công hơn là cài đặt nó qua `apt` bởi vì phiên bản ở trên `apt` luôn được update chậm hơn so với việc cài đặt repo thủ công.

Chạy command sau để cài đặt repo:
```
export REPO=$(mktemp /tmp/repo.XXXXXXXXX)
curl -o ${REPO} https://storage.googleapis.com/git-repo-downloads/repo
gpg --recv-keys 8BB9AD793E8E6153AF0F9A4416530D5E920F5C65
curl -s https://storage.googleapis.com/git-repo-downloads/repo.asc | gpg --verify - ${REPO} && install -m 755 ${REPO} ~/bin/repo
```

Command trên sẽ cài đặt phiên bản mới nhất của `repo`.

Và thế là xong, bạn đã setup xong môi trường để build AOSP đầu tiên của mình!

# Cấu hình username và password cho Git
Để sử dụng mọi lệnh trên Git, bạn phải khai báo email và username của mình. Để làm được điều đó, sử dụng lệnh sau:

```
git config --global user.email "<email của bạn ở đây>"
git config --global user.name "<username của bạn ở đây>"
```

Có một guide riêng nếu như bạn muốn [dùng HTTP cho Git](https://github.com/log1cs/documents/blob/main/unit1/misc/https_git.md), trong trường hợp bạn cần.

# Tải source code của AOSP về
Đầu tiên, tạo một thư mục để tải source code AOSP:

```
cd
mkdir -p aosp/source/
cd aosp/source/
```

Khi chạy xong command này thì đường dẫn hiện tại sẽ chuyển sang ~/aosp/source.

### Cấu hình repo để tải source code của AOSP
Chạy command dưới đây để cấu hình manifest của AOSP, device tree/kernel source/HALs cho Raspberry Pi 4:

```
repo init -u https://android.googlesource.com/platform/manifest -b android-15.0.0_r10
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
```

Command trên sẽ setup `repo` để tải source code của AOSP về, device tree/kernel source/HALs cho Raspberry Pi 4.

Để tiết kiệm dung lượng và xoá những thành phần không cần thiết để tối đa hoá dung lượng trống cho việc build, chạy command dưới thay vì command bên trên:

```
repo init -u https://android.googlesource.com/platform/manifest -b android-15.0.0_r10 --depth=1
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
curl -o .repo/local_manifests/remove_projects.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/remove_projects.xml
```

Command này sẽ setup `repo` để tải source code của AOSP về và device tree/kernel source/HALs cho Raspberry Pi 4, NHƯNG không kèm theo lịch sử commit cho tất cả các repository (vì `--depth=1` được thêm vào câu lệnh `repo init`) và cuối cùng `repo` sẽ xoá tất cả các repository không dùng đến.

### Tải/đồng bộ source code
Chạy command dưới để tải source code của AOSP về:

```
repo sync -c --optimized-fetch --no-tags --no-clone-bundle --prune --retry-fetches=5 -j$(nproc --all)
```

Một khi source code đã tải về xong, thì `repo` sẽ trả về như dưới - để bạn biết rằng quá trình tải về đã thành công:

```
repo sync has finished successfully.
```

Nếu như bạn gặp vấn đề trong lúc tải source code của AOSP về thì bạn có thể tham khảo một vài gợi ý dưới đây:

### Một vài gợi ý để sửa lỗi trong khi tải source code về máy

* Thêm `--force-sync` để bắt buộc đồng bộ lại source code từ đầu (sử dụng `--force-sync` sẽ bắt buộc clone lại tất cả mọi thứ trong source, cho nên các uncommited changes sẽ bị mất)

* Tăng số lần `--retry-fetches=` để `repo` thử tải về lại trong trường hợp có sự cố mạng diễn ra (chẳng hạn như bị ngắt mạng đột ngột, do driver WLAN không ổn định - điều này thường gặp ở những card WLAN của MediaTek,...)  

**Nếu như bạn không gặp vấn đề gì trong lúc tải/đồng bộ source code, thì chúng ta sẽ cùng đến bước tiếp theo - building AOSP!**

# Building AOSP image & tạo flashable image để nạp vào Raspberry Pi
Chạy command dưới để build AOSP image cho Raspberry Pi 4:
```
. build/envsetup.sh
lunch aosp_rpi4-trunk_staging-userdebug
make bootimage systemimage vendorimage -j$(nproc --all)
```

Quá trình build sẽ tốn rất nhiều thời gian phụ thuộc vào cấu hình phần cứng của bạn.

Chẳng hạn như cấu hình phần cứng được sử dụng trong guide bao gồm AMD Ryzen™ 9 5950X và 128GB RAM và SSD 2TB Samsung 990 Pro - sẽ build xong 1 image trong khoảng 2 tiếng.

Một khi quá trình build hoàn tất, thì hệ thống build sẽ trả về:

```
#### build completed successfully (xx:xx:xx (hh:mm:ss)) ####
```

Nhưng đó chưa phải tất cả - bình thường thì trên các thiết bị điện thoại Android thì bạn có thể lấy hết các images ở thư mục out/ rồi nạp vào các thiết bị đã được unlock bootloader bằng cách sử dụng `fastboot`. Nhưng cách này thì không áp dụng được với Raspberry Pi.

### Tại sao?
Bởi vì mỗi khi bạn muốn viết một image nào đó lên 1 chiếc thẻ nhớ bằng App Imager, nó sẽ xoá tất cả dữ liệu trong đó và bắt đầu viết từ sector 0. Và output của chúng ta bao gồm 3 images - boot.img chứa kernel và các cấu hình cmdline; system.img thì chứa Android, và vendor thì chứa blobs/HALs và các dịch vụ đi kèm. Và Android cần cả 3 để hoạt động, cho nên là bạn không thể burn các image riêng rẽ được.

Giải pháp cho vấn đề này đó là sử dụng `mkimg`

@KonstaT đã viết một shell script có thể gộp cả 3 images bạn vừa build được thành một image, cho phép bạn burn tất cả ra thẻ SD cùng 1 lúc.

Cách sử dụng được đề cập ở bên dưới, và chúng ta sẽ bắt đầu ngay bây giờ.

### Cách tạo flashable image

Đầu tiên thì chúng ta cần `cd` đến thư mục device tree và chạy script đó lên:

```
cd device/brcm/rpi4
./mkimg.sh
```

Sau khi gộp xong, thì script sẽ trả về:

```
Done, created /out/target/product/rpi4/RaspberryVanillaAOSP15-<builddate>-rpi4.img
```

Nếu như bạn đã theo hướng dẫn này đến đây, thì xin chúc mừng! Bước cuối cùng sẽ là burn image ra thẻ SD, để chạy Android trên Raspberry Pi 4

# Burn image ra thẻ SD

App Imager thì không thể cài đặt theo `apt` như thường, cho nên là bạn cần cài qua `snap` - cũng là một công cụ quản lý gói.

```
sudo apt update && sudo apt install snapd && snap install rpi-imager
```

Mở App Imager sau khi cài đặt xong.
