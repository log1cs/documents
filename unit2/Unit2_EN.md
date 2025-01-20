# Android Debug Bridge

Device used in this guide: Nokia 8 (NB1) - running LineageOS 22.1, Android 15 - build number `AP4A.241205.013`

This guide applied to these operating systems: Linux and Windows.

# What is ADB? What can it do?
The Android Debug Bridge (ADB) allows us to access a device’s CLI (or shell), letting us use native debugging tools like `logcat`.

`adb` is like a “Swiss-army knife” of Android development. It provides numerous functions that are described in detail by the command:

```
adb --help
```

**You can use ADB to:**
* Monitor kernel log, system log

* Making changes to your system (this requires root access)

# Install platform-tools
platform-tools provides necessary toolkit used for debugging on Android.

### Windows
1. Download the [Windows zip](https://dl.google.com/android/repository/platform-tools-latest-windows.zip) from Google

2. Extract it somewhere - for example, `%USERPROFILE%\adb-fastboot`

3. On Windows 7/8:

    - From the desktop, right-click My Computer and select Properties

    - In the System Properties window, click on the Advanced tab

    - In the Advanced section, click the Environment Variables button

    - In the Environment Variables window, highlight the `Path` variable in the Systems Variable section and click the Edit button

    - Append `;%USERPROFILE%\adb-fastboot\platform-tools` to the end of the existing Path definition (the semi-colon separates each path entry)

4. On Windows 10 or newer:

    - Open the Start menu, and type “advanced system settings”

    - Select “View advanced system settings”

    - Click on the Advanced tab

    - Open the “Environment Variables” window

    - Select the `Path` variable under “System Variables” and click the “Edit” button

    - Click the “New” button

    - Insert `%USERPROFILE%\adb-fastboot\platform-tools` in the text field.

### Linux
1. Download the [Linux zip](https://dl.google.com/android/repository/platform-tools-latest-linux.zip) from Google.

2. Extract it somewhere - for example, `~/adb-fastboot`.

3. Add the following to `~/.profile`:

```
if [ -d "$HOME/adb-fastboot/platform-tools" ] ; then
 export PATH="$HOME/adb-fastboot/platform-tools:$PATH"
fi
```

4. Log out and back in.

5. You may also need to set up udev rules: see [this repository](https://github.com/M0Rf30/android-udev-rules#installation) for more info.

# How to enable USB debugging

1. Open Settings.

2. Find "About devices" or "About tablet", usually this will be in the very last section of the Settings app.

3. Scroll to the bottom, tap 7 times on "Build number"

![buildnum](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_buildnum.png)

Until it returns "You are now a developer!"

![devoptions_enabled](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_devoptions_enabled.png)

4. Go back to System section, find Developer Options.

![devoptions](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_devoptions.png)

5. Toggle "USB debugging" on and you are good to go!

![usbdebugging](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_usbdebugging.png)

6. (This is optional) Toggle "Rooted debugging" on, in order to have rooted access to the system.

![rooteddbg](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_rooteddbg.png)


# How to enable Wireless Debugging
In case your board doesn't have a physical USB port, or whatever is the reason you can't use USB debugging, using Wireless Debugging is also an option.

Given that you already enabled Developer Options (see USB debugging guide) and connected to a WLAN network same with your host machine, this guide will cover you on how to use Wireless Debugging.

1. Open Developer Options, enable Wireless Debugging - then tap on it:

![wirelessdbg](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_wirelessdbg.png)

2. After you tap on Wireless Debugging, it will open Wireless Debugging Control Panel
There is 2 things you need to notice in the image below:

    1: `<IPADDR>:<PORT>` - used to connect to the target device.

    2: Paired devices - show a list of connected devices.

![wirelessdbgpanel](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_wirelessdbgpanel.png)

3. Back to your terminal, run this command:

```
adb connect <IP>:<PORT>
```

The expected output should be like this:
```
connected to <IP>:<PORT>
```

### Example:

Connecting to the target device with provided IP address `192.168.110.143` at port `38193`.

```
$ adb connect 192.168.110.143:38193
connected to 192.168.110.143:38193
```

# Popular ADB commands

### Opening ADB shell
First of all, run:

```
adb devices
```

ADB will start the server first and then query all available devices and then return:

```
List of devices attached
NB1GAD17A1001994        device
```

At the same time, your device will have this pop-up:

![adb](https://raw.githubusercontent.com/log1cs/documents/refs/heads/main/unit2/misc/unit2_adb.png)

In this example, `NB1GAD17A1001994` is the device serial number.

### Troubleshooting in case the device cannot be found

If you run `adb devices` and then got this one:

```
List of devices attached
NB1GAD17A1001994        unauthorized
```

Then you are likely forgot/or not pressing allow in the "Allow USB debugging" pop-up. Disconnect your device and then reconnect it again, then select "Always allow from this computer" and then select "Allow".

### Kill an ADB server
```
adb kill-server
```

### Install an app via ADB

```
adb install <file>
```

`<file>` is the given .apk you want to install on your device

### Push/pull

### To pull a file from your device:

```
adb pull <remote> <outputDir>
```

`<remote>` is the file you want to pull from your device, and `<outputDir>` is optional. If `<outputDir>` isn't specified, it will pull to the current directory.

Example:
```
adb pull /sdcard/install.log - pull /sdcard/install.log to your current directory
adb pull /sdcard/install.log C:\Users\USERNAME\Desktop\install.log - pull /sdcard/install.log to the given directory
```


### To push a file to your device:

```
adb push <input> <destinationDir>
```

`<input>` is the file you want to push and `<destinationDir>` is the directory you want to store the file.

Example:
```
adb push install.log /sdcard/install.log - push install.log to /sdcard/install.log.
```
