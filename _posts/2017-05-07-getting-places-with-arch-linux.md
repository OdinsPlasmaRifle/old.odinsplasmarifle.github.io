---
layout: post
title: Getting Places With Arch Linux
comments: true
---

*LVM on LUKS Arch installation with systemd-boot*

![Arch Linux Banner](/public/images/posts/arch_linux_banner.png)

**NB**: This blog article it out of date, if you want to install Arch Linux using LVM on LUKS you should take a look [here]([https://wiki.archlinux.org/](https://gist.github.com/OdinsPlasmaRifle/e16700b83624ff44316f87d9cdbb5c94)).

## A New Dawn

Not so long ago, as a part of my on-going attempt to improve my understanding of Linux, I decided to install 
Arch Linux from scratch. This has been on my TODO list for ages but the purchase of a new laptop gave 
me the necessary push to actually get to it. I'm not a a complete newbee when it comes Linux as I have worked with 
Ubuntu for several years. In addition for the past 2 years my primary work environment has been Antergos, which is a 
great Arch Linux derivative. However, even though I felt pretty comfortable with basic Linux usage I still approached 
the task of installing Arch with a little trepidation. I'm happy to say that was a stupid mentality.

<!--break-->

One of my requirements for this install was to include drive encryption, which the default Arch installation guide 
does not address (without extensive digging). After a couple hours of browsing guides and reading the phenomenal 
[ArchWiki](https://wiki.archlinux.org/) I finally felt comfortable giving it a shot. This guide is explicitly tailored 
towards installing Arch with *LVM on LUKS* volume encryption and a *systemd-boot* boot partition.

## Getting Started (The Journey Begins)

Downloading Arch Linux is the first step. You can get an Arch ISO from the official Arch Linux 
[download](https://www.archlinux.org/download/) page.

In my case, I installed Arch from a USB drive. 

To do this, you need to mount the installation media on a USB drive:. I'll explain how to do this on a Linux machine but
if you are using Windows you may need to use something like `Rufus`.

Firstly, plug your drive in and find its name with the command `lsblk`. 

Before proceeding, make sure that the above drive is not mounted.

To mount the Arch ISO run the following command, replacing `/dev/sdx` with your drive, e.g. `/dev/sdb`. (do not append a partition number, so do not use something like `/dev/sdb1`):

```shell
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync
```

When the command is finished you will have a USB drive with a bootable Arch Linux installer.

Now shutdown the device you plan to install Arch on then plug the USB drive in and boot the device up again. Instruct your
device to boot from the USB if it does not do so automatically (You can do this in the BIOS).

## Preparation

Arch Linux will start up in and you should be greeted by a virtual console.

If your device has a small scree the font might be uncomfortably small. You can change the font using this command:

```shell
setfont sun12x22
```

Because this is a `systemd-boot` guide you should ensure the installer is running in UEFI mode:

```shell
ls /sys/firmware/efi
```

If there is any content in this folder then you are in UEFI mode and can continue. Otherwise change your boot mode in your BIOS.

Next check that there is an internet connection:

```shell
ping archlinux.org
```

For an easy installation you should plug an ethernet cable in. For wifi installations you will have to take a look at
the Arch wiki yourself.

Next, update the system clock:

```shell
timedatectl set-ntp true
```

Lastly to enable download mirrors, edit `/etc/pacman.d/mirrorlist` and locate your geographic region. Uncomment the 
mirrors you would like to use.

This should complete the basic install preparation.

### Partitioning

This step is a bit more advanced and deals with the partitioning of drives.

The first step is to get the name of the disk you want to format/partition:

```shell
lsblk
```

The name should be something like `/dev/sda`

Now because the drive is going to be encrypted and we want to ensure all old data is erased permanently you will want 
to shred the disk using the `shred` tool:


```shell
shred -v -n1 /dev/sdX
```

Now partition the disk using `gdisk`:

```shell
gdisk /dev/sda
```

Partition 1 should be an EFI boot partition (code: ef00) of 512MB. Partition 2 should be a Linux LVM partition (8e00). The 2nd partition can take up the full disk or only a part of it. Remember to write the partition table changes to the disk on configuration completion.

Once partitioned you can format the boot partition (the LVM partition needs to be encrypted before it gets formatted):

```shell
mkfs.fat -F32 /dev/sda1
```

### Encryption

Now that the partitioning is complete we can move to the really complex stuff. In this section we set up the 
Logical Volume Manager (LVM) and Linux Unified Key Setup (LUKS).

The first step is to `modprobe` for `dm-crypt`:

```shell
modprobe dm-crypt
```

Next, encrypt the disk (make sure you choose the correct disk to encrypt, i.e. don't encrypt the boot partition):

```shell
cryptsetup luksFormat /dev/sda2
```

Open the disk with the password set in the above command:

```shell
cryptsetup open --type luks /dev/sda2 lvm
```

Check that the lvm disk exists:

```shell
ls /dev/mapper/lvm
```

Create a physical volume:

```shell
pvcreate /dev/mapper/lvm
```

Create a volume group:

```shell
vgcreate volume /dev/mapper/lvm
```

Next, Create logical volumes. You should change the below commands to suit your system specs. In my case I have 
16GB RAM and a 512 GB SSD, hence the large sizes for root and swap volumes.

```shell
lvcreate -L20G volume -n swap
lvcreate -L40G volume -n root
lvcreate -l 100%FREE volume -n home
```

Format file system on the logical volumes:

```shell
mkfs.ext4 /dev/mapper/volume-root
mkfs.ext4 /dev/mapper/volume-root
mkswap /dev/mapper/volume-swap
```

Lastly, mount the volumes and file systems:

```shell
mount /dev/mapper/volume-root /mnt
mkdir /mnt/home
mount /mnt/boot
mount /dev/mapper/volume-home /mnt/home
mount /dev/sda1 /mnt/boot
swapon /dev/mapper/volume-swap
```

This completes the drive encryption but don't forget the password set in the above as
it is the only way to decrypt the disk (i.e. if you forget your password you will lose all your data).

## Installation

To start the actual Arch installation you will need to bootstrap the base system onto the disk using `pacstrap`:

```shell
pacstrap /mnt base base-devel vim
```

Generate `fstab`:

```shell
genfstab -p /mnt >> /mnt/etc/fstab
```

`chroot` into the system:

```shell
arch-chroot /mnt
```

Set time locale (Choose your own time zone):

```shell
ln -sf /usr/share/zoneinfo/Africa/Johannesburg /etc/localtime
```

Set clock:

```shell
hwclock --systohc
```

Uncomment `en_US.UTF-8 UTF-8` `en_US ISO-8859-1` and other needed localizations in `/etc/locale.gen`. Once done run
the following in order to generate the locale:

```shell
locale-gen
```

Create a config file for the chosen locale:

```shell
locale > /etc/locale.conf
```

Add an hostname:

```shell
vim /etc/hostname
```

The hostname can be anything and should just be placed at the top of the above file.

Update `/etc/hosts` to contain the following:

```text
127.0.1.1   myhostname.localdomain  myhostname
```

`myhostname` should be whatever you chose in the previous step.

Because we are using disk encryption we have to change the `initramfs`.

Edit the `/etc/mkinitcpio.conf`. Look for the HOOKS variable and move `keyboard` to before the `filesystems` and add `encrypt` and `lvm2` after `keyboard`. Here is an example:

```text
HOOKS="base udev autodetect modconf block keyboard encrypt lvm2 filesystems fsck"
```

Regenerate the `initramfs` based on the above changes:

```shell
mkinitcpio -p linux
```

Install a bootloader:

```shell
bootctl --path=/boot/ install
```

Create bootloader. Edit `/boot/loader/loader.conf`. Replace the file's contents with:

```text
default arch
timeout 3
editor 0
```

The `editor 0` ensures the configuration can't be changed on boot.

Next create a bootloader entry in `/boot/loader/entries/arch.conf`

```text
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options cryptdevice=UUID={UUID}:volume root=/dev/mapper/volume-root quiet rw
```

In order to get the UUID mentioned above, run the following command in vim:

```shell
:read ! blkid /dev/sda2
```

If you have reached this point, rejoice! You are now finished with the install configuration and the next 
few steps will simply entail finalizing the setup.

## Complete

Complete the installation by first exiting `chroot`:

```shell
exit
```

Now unmount everything:

```shell
umount -R /mnt
```

and finally reboot

```shell
reboot
```

## Conclusion (The End is Here...kinda)

You now have a working Arch system and can start turning it into something that is truly yours. Customizability in Arch
is an endless pursuit. As long as you have the time and dedication you can turn your system into literally anything you want.

For some general recommendations, software tips and additional system hardening take a look at:

[General Recommendations](https://wiki.archlinux.org/index.php/general_recommendations)  
[Applications](https://wiki.archlinux.org/index.php/list_of_applications)  
[Security and Hardening](https://wiki.archlinux.org/index.php/security)  

