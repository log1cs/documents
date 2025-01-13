# Flash image on Raspberry Pi 4

## Install App Imager to flash the image
```
cd ..
snap install rpi-imager
```

### Change file ownership
```
sudo chown user:root source/out/target/product/rpi4/RaspberryVanillaAOSP13.....
```

### Flash the image
### English
```
1. Click on "Choose OS"
2. Click on "Use custom"
3. Choose image in folder aosp/source/out/target/product/rpi4/RaspberryVanillaAOSP13-<builddate>-rpi4.img
4. Choose ramdisk storage 
5. Write flash image
6. Remove ramdisk from computer then connect ramdisk to computer
7. Copy replace Image from aosp/kernel/out/dist/ to /media/user/boot/ (ramdisk)
```

### Tiếng Việt
```
1. Sau khi cài xong thì mở App Imager lên 
2. Chọn choose OS
3. Kéo xuống chọn "Use custom" để mở .image trong folder aosp/source/out/target/product/rpi4/RaspberryVanillaAOSP13-<builddate>-rpi4.img
4. Chọn Choose storage để chọn thẻ nhớ
5. Chọn xong bấm Write để ghi vào thẻ nhớ 
6. Sau khi flash xong thì Android có thể chạy được rồi nhưng cần có kernel nữa để lập trình
7. Tháo usb ra cắm lại 1 lần để nhận, lúc này có 4 thư mục mình cần thay đổi file Image trong thư mục boot của ramdisk bằng Image trong aosp/kernel/out/dist/
```

## Copy kernel image
```
cp kernel/out/dist/Image /media/asus/boot/
sync
```
