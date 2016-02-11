
# 96boards-manifest
Non-official repo manifest for 96boards OE builds. This guide assumes that you are already flashed a
bootloader and a partition table to the board as it is described here:

https://github.com/96boards/documentation/wiki/HiKeyGettingStarted#debian-linux-os

# Build instructions

```
# repo init -u https://github.com/kuscsik/96boards-manifest.git -b hikey
# repo sync
```

Get the graphics drivers from here 

https://drive.google.com/a/linaro.org/file/d/0B8Uq4Q7WAxO4ZjJLdGJQR01DRkE/view?usp=sharing

and place the tar package in the following folder:

```
~/Public/oe-downloads/
```

Build the image:

```
$ source ./meta-los/script/envsetup
$ bitbake los-weston-image
```

Inside the OE image deploy folder convert the ext4 image to a fastboot sparse image

```
$ ext2simg los-weston-image-hikey.ext4 los-weston-image-hikey.img
```

and flash it using fastboot

```
$ fastboot flash system los-weston-image-hikey.img
```

Get the boot-fat.uefi.img.gz from here:

https://builds.96boards.org/snapshots/hikey/linaro/debian/latest/https://builds.96boards.org/snapshots/hikey/linaro/debian/latest/ 

And replace the default grub.cfg with the one from meta-los:

```
$ mkdir -p boot-fat
$ sudo mount -o loop,rw,sync boot-fat.uefi.img boot-fat
$ cp conf/grub.cfg boot-fat/EFI/BOOT/
$ sync
$ sudo umount boot-fat.uefi.img
$ fastboot flash boot boot-fat.uefi.img
```

Important: Don't forget to unplug the OTG cable after flashing is done and connect a mouse/keyboard
to the board. Remove any SD card if present.

The grub.cfg is configured to 1920x1080@25 resolution, if that doesn't works for you for some reason,
try to edit the file and set it to a safe 800x600@60 resolution first.

Reboot.

