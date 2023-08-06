# Ubuntu 23.04 dual boot with Encryption

Steps to install Windows 10/11 and Ubuntu 23.04 while using disk encryption for both systems.

> Back up your data before running these steps.

## Windows

### Installation

If you don't already have it installed, download and install Windows.

Windows will install on your entire disk.

### Preparation

To make room for Ubuntu, you need to resize the Windows partition leaving some Unallocated Space. This should be at least 50 GB, but if you have more space, >100 GB is much better.

You can use the Windows Disk Management to shrink the Windows partition (C:) or `diskpart`.

> If you already have BitLocker enabled on the Windows partition, make sure to backup your Recovery Key, it's very likely you will need it!

If the Windows partition is not encrypted with BitLocker yet, you can do that after setting up Ubuntu.

## Ubuntu 23.04

### Preparation

Download Ubuntu 23 and write the installation to an USB stick.

> IMPORTANT! Make sure to download the 23.04 [Legacy Desktop Installer](https://cdimage.ubuntu.com/releases/lunar/release/ubuntu-23.04-desktop-legacy-amd64.iso), not the normal installer.

Follow the instructions to write the ISO to the USB stick: https://ubuntu.com/tutorials/install-ubuntu-desktop#3-create-a-bootable-usb-stick

### Installation

Boot from USB
