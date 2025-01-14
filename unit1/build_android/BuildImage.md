# Building Android 15 for Raspberry Pi 4/5
### Installing dependencies

```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig  bc coreutils dosfstools e2fsprogs fdisk kpartx mtools ninja-build pkg-config python3-pip python-is-python3
```

### Python dependencies
```
sudo pip3 install meson mako jinja2 ply pyyaml dataclasses
```

### Installing repo
```
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo
echo 'PATH="${HOME}/.bin:${PATH}"' >> ~/.bashrc
source ~/.bashrc
```

### Configure email and username for Git
```
git config --global user.email "<your email here>"
git config --global user.name "<your username here>"
```

### Or, if you need Git via HTTP:
```
ls -a 
code .gitconfig OR vim .gitconfig
```

### .gitconfig
```
[http]
	cookiefile = %USERPROFILE%\\.gitcookies
	postBuffer = 1737418240
	version = HTTP/1.1
[user]
	name = ................
	email = ................
[color]
	ui = auto  
```

### Create a folder to clone AOSP source to
```
cd 
mkdir -p aosp/source/
cd aosp/source/
```

### Initialize AOSP source in the directory
```
repo init -u https://android.googlesource.com/platform/manifest -b android-15.0.0_r10
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
```

Or, if you want to save space, you could pass depth=1 into repo init command!

This will clone the source but, without the Git log (history) of all repositories. It's up to you.

```
repo init -u https://android.googlesource.com/platform/manifest -b android-15.0.0_r10 --depth=1
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
curl -o .repo/local_manifests/remove_projects.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/remove_projects.xml
```

### Download/sync the source code
```aosp_rpi4-trunk_staging-userdebug
repo sync -c --force-sync --optimized-fetch --no-tags --no-clone-bundle --prune --retry-fetches=5 -j$(nproc --all)
```

### Build the image
```
. build/envsetup.sh
lunch aosp_rpi4-trunk_staging-userdebug
make bootimage systemimage vendorimage -j$(nproc)
```

### Make Raspberry Pi flashable images
```
./rpi4-mkimg.sh
```
***

# Build the kernel

### Create a folder that is used to store kernel sources
```
cd ..
mkdir -p kernel/
cd kernel/
```

### Initialize kernel repositories
```
repo init -u https://android.googlesource.com/kernel/manifest -b common-android15-6.6-lts
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/raspberry-vanilla/android_kernel_manifest/android-15.0/manifest_brcm_rpi.xml --create-dirs
```

### Sync the kernel source
```
repo sync
```

### Build kernel image
```
BUILD_CONFIG=common/build.config.rpi4 build/build.sh
```
