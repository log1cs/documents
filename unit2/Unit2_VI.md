# Android Debug Bridge

Thiết bị được sử dụng trong guide: Nokia 8 (NB1) - đang chạy trên LineageOS 21, Android 14 - số bản dựng `AP2A.240905.003` (tuy nhiên thì giữa Android 15 ở guide Tiếng Anh và Android 14 ở guide Tiếng Việt đều không khác gì nhau).

Hướng dẫn áp dụng cho các hệ điều hành sau: Linux and Windows.

# ADB là gì? Nó có thể làm được gì?
Android Debug Bridge (ADB) cho phép chúng ta truy cập CLI (hoặc shell) của một thiết bị, cho phép chúng ta sử dụng các công cụ debug native như `logcat`.

`adb` là một thành phần chủ chốt của lập trình Android. Các tính năng của adb được mô tả chi tiết bằng một câu lệnh:

```
adb --help
```

**Bạn có thể dùng ADB để:**
* Xem kernel log, system log

* Thay đổi các tệp hệ thống (yêu cầu quyền root)

# Cài đặt platform-tools
platform-tools cung cấp các công cụ cần thiết để debug trong Android..

### Windows
1. Tải về cho [Windows](https://dl.google.com/android/repository/platform-tools-latest-windows.zip) từ Google

2. Giải nén vào đâu đó - chẳng hạn, `%USERPROFILE%\adb-fastboot`

3. Trên Windows 7/8:

    - Từ desktop, chuột phải vào My Computer và chọn Properties

    - Ở cửa sổ System Properties, nhấn vào tab Advanced

    - Ở section Advanced, nhấn vào nút Environment Variables

    - Ở trong cửa sổ Environment Variables, bôi đen dòng `Path` ở trong Systems Variable section và nhấn nút Edit

    - Thêm `;%USERPROFILE%\adb-fastboot\platform-tools` vào cuối đường dẫn của Path hiện tại (mỗi entry cần một dấu móc phẩy)

4. Windows 10 hoặc mới hơn:

    - Mở Start menu, nhập “advanced system settings”

    - Chọn “View advanced system settings”

    - Chọn vào tab Advanced

    - Mở cửa sổ “Environment Variables”

    - Chọn biến `Path` ở trong “System Variables” và nhấn nút “Edit”

    - Nhấn vào nút “New”

    - Thêm `%USERPROFILE%\adb-fastboot\platform-tools` vào trong đó.

### Linux
1. Tải về cho [Linux](https://dl.google.com/android/repository/platform-tools-latest-linux.zip) from Google.

2. Giải nén vào đâu đó - chẳng hạn, `~/adb-fastboot`.

3. Thêm dòng sau vào `~/.profile`:

```
if [ -d "$HOME/adb-fastboot/platform-tools" ] ; then
 export PATH="$HOME/adb-fastboot/platform-tools:$PATH"
fi
```

4. Đăng xuất và đăng nhập lại.

5. Bạn có thể sẽ cần cấu hình quyền udev: xem [repo này](https://github.com/M0Rf30/android-udev-rules#installation) để biết thêm chi tiết.

# Cách bật USB Debugging

1. Mở "Cài đặt".

2. Tìm "Thông tin về điện thoại" hoặc "Thông tin về điện thoại", thường phần này sẽ nằm gần cuối ứng dụng Cài đặt.

3. Vuốt xuống cuối cùng, và bấm 7 lần vào "Số bản dựng"

![buildnum_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_buildnum_vi.png)

Cho đến khi có pop up hiện lên "Bạn đã là nhà phát triển!"

![devoptions_enabled_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_devoptions_enabled_vi.png)

4. Quay lại phần Hệ thống, tìm Tuỳ chọn nhà phát triển.

![devoptions_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_devoptions_vi.png)

5. Bật "Gỡ lỗi USB"

![usbdebugging_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_usbdebugging_vi.png)

6. (Không bắt buộc) Bật "Gỡ lôi với quyền root" lên, nếu bạn muốn có toàn quyền root vào hệ thống.

![rooteddbg_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_rooteddbg_vi.png)


# Cách để bật Debugging qua Wi-Fi
Trong trường hợp hệ thống của bạn không có cổng USB vật lý, hoặc lý do nào đó khiến bạn không thể dùng được USB debugging, thì bạn có thể sử dụng Wireless Debugging.

Cho rằng bạn đã bật sẵn Tuỳ chọn nhà phát triển (xem Cách bật USB debugging) và kết nối vào một mạng Wi-Fi chung với PC của bạn, thì hướng dẫn này sẽ chỉ bạn cách dùng Debugging qua Wi-Fi.

1. Mở Tuỳ chọn nhà phát triển, bật Gỡ lỗi qua Wi-Fi - rồi mở lên:

![wirelessdbg_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_wirelessdbg_vi.jpg)

2. After you tap on Wireless Debugging, it will open Wireless Debugging Control Panel
There is 2 things you need to notice in the image below:

2. Sau khi bạn mở Gỡ lỗi qua Wi-Fi, nó sẽ mở một bảng điều khiển ra.
Có 2 điều mà bạn cần lưu ý trong bức ảnh dưới:

    1: `<IPADDR>:<PORT>` - Sử dụng để kết nối vào thiết bị bạn muốn.

    2: Thiết bị đã ghép nối - Hiển thị danh sách các thiết bị đã kết nối.

![wirelessdbgpanel_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_wirelessdbgpanel_vi.png)

3. Quay lại terminal của bạn, chạy command sau:

```
adb connect <IP>:<PORT>
```

Output mong muốn sẽ như dưới:
```
connected to <IP>:<PORT>
```

### Ví dụ:

Kết nối vào thiết bị với địa chỉ IP `192.168.110.143` tại cổng `38193`.

```
$ adb connect 192.168.110.143:38193
connected to 192.168.110.143:38193
```

# Các câu lệnh ADB phổ biến

### Opening ADB shell
Đầu tiên, chạy command sau:

```
adb devices
```

ADB sẽ chạy server đầu tiên và sau đó sẽ quét hết tất cả các thiết bị khả dụng:

```
List of devices attached
NB1GAD17A1001994        device
```

Cùng lúc đó, thiết bị của bạn sẽ có 1 pop-up:

![adb_vi](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_adb_vi.png)

Trong ví dụ này, `NB1GAD17A1001994` là số Serial thiết bị của bạn.

### Troubleshooting in case the device cannot be found

Trong trường hợp bạn chạy `adb devices` và thấy output như thế này:

```
List of devices attached
NB1GAD17A1001994        unauthorized
```

Thì chắc bạn đã quên/hoặc chưa cho phép ở pop-up "Cho phép gỡ lỗi qua USB?". Ngắt kết nối thiết bị và kết nối lại với máy tính, sau đó chọn "Luôn cho phép từ máy tính này" và sau đó chọn "Cho phép".

### Kill tất cả các server ADB
```
adb kill-server
```

### Cài một ứng dụng qua ADB

```
adb install <file>
```

`<file>` là file .apk bạn muốn cài đặt trên thiết bị của bạn.

### Đẩy/Kéo file

### Để kéo một file từ thiết bị của bạn:

```
adb pull <remote> <outputDir>
```

`<remote>` là file bạn muốn kéo từ thiết bị của mình, và `<outputDir>` là một tuỳ chọn. Nếu `<outputDir>` không được khai báo, ADB sẽ lưu file vào thư mục bạn đang ở hiện tại.

Ví dụ:
```
adb pull /sdcard/install.log - Kéo /sdcard/install.log đến thư mục hiện tại
adb pull /sdcard/install.log C:\Users\USERNAME\Desktop\install.log - Kéo /sdcard/install.log đến một thư mục được cho khai báo.
```

### Để đẩy một file vào thiết bị của bạn:

```
adb push <input> <destinationDir>
```

`<input>` là file bạn muốn đẩy và `<destinationDir>` là nơi bạn muốn lưu file.

Example:
```
adb push install.log /sdcard/install.log - đẩy install.log vào /sdcard/install.log.
```