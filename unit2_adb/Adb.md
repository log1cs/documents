# Enable ADB for debugging on Raspberry Pi 4

### Install platform-tools
```
sudo apt update
sudo apt install android-tools-adb
sudo apt install android-tools-fastboot
```

Or, if you want to download it instead of installing it:
```
https://developer.android.com/tools/releases/platform-tools
```

### Check ADB version
```
adb version
```

### Enable USB debugging/Bật gỡ lỗi USB
### Tiếng Việt
1. Sau khi boot được Android thì mở cài đặt 

2. Chọn "About Tablet"

3. Click vào "Build Number" 7 lần cho đến khi hiện "You are now a developer!"

4. Vào System, tìm Developer Options.

5. Tìm và bật "USB debugging".

### English
1. Once booted to Android, open Settings.

2. Find "About tablet", usually this will be in the very last section of the Settings app.

3. Tap 7 times on "Build number" until it returns "You are now a developer!"

4. Open System section, find Developer Options.

5. Toggle USB debugging on and you are good to go!

### Start ADB server
```
sudo adb start-server
```

### Restart ADB server in case something wrong happened
```
sudo adb kill-server
sudo adb start-server
```

### ADB shell
```
adb devices 
adb shell 
```

### Giải thích:
```
adb shell: Mở shell trên thiết bị đang kết nối tới
adb devices: xem các thiết bị đang kết nối 
```
