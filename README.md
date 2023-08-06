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

Boot from USB and choose Try Ubuntu.

#### Partitioning

We will create the partitions for Ubuntu:

- Boot partition (about 1 GB)
- LUKS (encrypted) partition for Ubuntu (the rest of the unallocated disk space)
  - This partition will contain several other partitions - all encrypted

Since the disk already has an EFI partition created by Windows, we don't need to create another one.

The easier way to do this is to a graphical utility included with the Ubuntu Live disk. It's called "Disks". To find it, just press the Super key (the Win key on your keyboard) and type Disks in the search box.

##### Boot partition

Click the "Free space" and the + button to add a partition. Set the partition size to 1800 MB (that should be more than enough).

Click Next and make sure the Type is Ext4.

Click Create.

##### LUKS partition

Again, click the remaining "Free space" and the + button to add a partition. Set the partition size to the remaining free space on your disk (the default value).

Click "Next" and make sure the Type is also Ext4, but this time also check the "Password protect volume (LUKS)" underneath it.

Click "Next" and set a strong and memorable password for it. You will need to type it in at every boot, so make sure you won't forget it.

Click "Create".

What we achieve at this step, we created a LUKS partition encrypted with the password you set. Inside this special partition, there is an Ext4 partition that can be read only after the LUKS partition was unlocked using the password.

To verify this operation, you can open a Terminal and type `ls /dev/mapper`. The output should be something like:

```
control  luks-7a09c...
```

Replace `luks-7a09c...` with what label was generated for your device. Use that value in the following instructions.

##### Linux partitions within LUKS

Normally, what we have so far should be OK, but I also want a swap partition within LUKS. If you don't expect your computer to need much swap, you can skip this step.

Switching to the terminal again, we need to redo the LUKS volumes:

```
sudo su
pvcreate /dev/mapper/luks-7a09c...
```

This will prompt to delete the existing one and will create a new physical volume.

Next, we need to create a volume group with 2 volumes inside (one for swap and one for the root filesystem).

```
vgcreate ubuntu-vg /dev/mapper/luks-7a09c...
lvcreate -L 20G -n swap_1 ubuntu-vg
lvcreate -l 100%FREE -n root ubuntu-vg
```

### Ubuntu installation

Now we need to install Ubuntu on the newly created partitions.

Quit the terminal and Disks and start the "Install Ubuntu 23.04".

Select language, keyboard etc.

> Before getting to the disks part, the installer may hang for a few minutes. Be patient.

Chooese "Something else"

#### Selecting the partitions for Ubuntu

Select the boot partition (the 1800MB partition, type Ext4 we created previously), click "Change", set "Use as" to Ext4 and the mount point to `/boot`. You should also choose to format the partition.

Set the `/dev/mapper/ubuntu-vg-swap_1` as `swap area`.

Set the `/dev/mapper/ubuntu-vg-root` (the one spanning the rest of your disk) to Ext4, select to format it and set the mount point to `/`.

Below the table, set the Device for boot loader installation to you full device. It should be something like `/dev/sda` or `/dev/nvme0n1`.

Click Install now.

> Do not restart your computer after install!

### Post installation

After installation, choose to continue to use Ubuntu, don't restart.
