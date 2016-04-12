
# 96boards-manifest
Non-official repo manifest for 96boards OE builds. This guide assumes that you are already flashed a
bootloader and a partition table to the board as it is described here:

https://github.com/96boards/documentation/wiki/HiKeyGettingStarted#debian-linux-os

# Host machine setup

The build instructions are tested on Ubuntu 14.04. On a clean Ubuntu 14.04 install you will need some additional packages:

```
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
    build-essential chrpath libsdl1.2-dev xterm
```

The following issue was also reported with host machine setup. OP TEE build depneds on python crypto. If your have problem installing it, check the following link:

https://github.com/OP-TEE/optee_os/issues/663

You most likely noticed that the Hikey has no ethernet connector. The image was tested with an Apple USB Ethernet adapter.

# Build instructions

```
$ repo init -u https://github.com/kuscsik/96boards-manifest.git -b hikey_dev
$ repo sync
```

Get the graphics drivers from here

https://drive.google.com/a/linaro.org/file/d/0B8Uq4Q7WAxO4ZjJLdGJQR01DRkE/view?usp=sharing

and place the tar package in the following folder:

```
~/Public/oe-downloads/
```

Build the image:

```
$ source ./meta-los/script/envsetup.sh
$ bitbake los-chromium-image
```

Inside the OE image deploy folder (build/tmp-glibc/deploy/images/hikey) convert the ext4 image to a fastboot sparse image

```
$ ext2simg los-chromium-image-hikey.ext4 los-chromium-image-hikey.img
```

and flash it using fastboot

```
$ sudo fastboot flash fastboot tmp-glibc/deploy/images/hikey/fip.bin
$ sudo fastboot flash system los-chromium-image-hikey.img
```

Get the boot-fat.uefi.img.gz from here:

https://builds.96boards.org/snapshots/hikey/linaro/debian/latest/

And replace the default grub.cfg with the one from meta-los:

```
$ mkdir -p boot-fat
$ sudo mount -o loop,rw,sync boot-fat.uefi.img boot-fat
$ sudo cp conf/grub.cfg boot-fat/EFI/BOOT/
$ sync
$ sudo umount boot-fat
$ fastboot flash boot boot-fat.uefi.img
```

Important: Don't forget to unplug the OTG cable after flashing is done and connect a mouse/keyboard
to the board. Remove any SD card if present.

The grub.cfg is configured to 1920x1080@25 resolution, if that doesn't works for you for some reason,
try to edit the file and set it to a safe 800x600@60 resolution first.

Reboot.

# Running EME test

On the target, after logging in as root, run:

```
$ tee-supplicant &
$ portmap &
$ cdmiservice &

$ /usr/bin/chromium/chrome --no-sandbox  \
    --use-gl=egl --ozone-platform=wayland --no-sandbox --composite-to-mailbox --in-process-gpu --enable-low-end-device-mode \
    --enable-logging --v=0 \
    --start-maximized \
    --user-data-dir=data_dir \
    --blink-platform-log-channels=Media\
    --register-pepper-plugins="/usr/lib/chromium/libopencdmadapter.so#ClearKey CDM#ClearKey CDM0.1.0.0#0.1.0.0;application/x-ppapi-open-cdm" \
    http://people.linaro.org/~zoltan.kuscsik/chrome/eme_player.html
```

Select "External Clearkey" key system and hit Play.
