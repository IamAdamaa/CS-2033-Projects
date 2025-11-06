---
title: "Arch Linux installation"
layout: default
---

# IamAdamaa.github.io
# Part 1: Installing Arch Linux

## Installing the Arch ISO file
- First I went to https://archlinux.org/download/  and went to the HTTP direct downloads
- selected one of the mirror sites from my location
- Found the HTTP download named "archlinux-2025.10.01-x86_64.iso" which is the ISO file and installed it
-------- 
## Set up the arch VM on VMware
- Created a new virtual machine on VMware
- Uploaded the arch ISO file I acquired earlier into the *Installer disc image file (ISO)* section

**Issue I ran into:** When uploading the ISO into VMware it said it "could not detect which OS is in the disc image". Then it made me manually select the Linux distro I am using, but Arch linux wa[...]  
- Gave the VM an adequate amount of resources.
	- 20 Gb of storage
	- 4 Gb of ram
	- 4 processors
- Pressed finish to create the VM
## Launching into the Arch Linux installation environment

- launched the VM and booted into the virtual console
- First thing I did was increase the font size using the `setfont ter-132b` command
- Then I verified the boot mode was UEFI 64-bit using the command `cat /sys/firmware/efi/fw_platform_size`  which should return `64`

**Issue I ran into:** The VM was originally booting in bios mode, so I had to change it to boot into UEFI mode in the advance settings of VM-ware


## Network configuration
- confirmed my network interface is listed and enabled using the command `ip link`
	- I had `ens33` which means I am connected via ethernet and "state **UP**" means its enabled

- To further confirm connection and that DNS is working I used the command `ping ping.archlinux`

## Partitioning the disk
- I first need to identify my storage device, so I used the command `lsblk`
- The device identifier for my storage device was named `sda`

- I used the partitioning tool `cfdisk`, since it was easiest to use, to create and modify my partitions using the command `cfdisk /dev/sda`
	- I followed the example layout on the installation guide *UEFI with GPT* since I'm using a system with UEFI
		**creating the partitions:**
		- First, created the EFI partition giving it 1Gb and giving it the file type EFI System
		- Then I made the swap partition giving it 4Gb, giving it the file type: Linux swap (This will be used as swap space if the OS runs out of RAM)
		- Lastly I created the root partition (/) where the operating system resides. I gave it the rest of my disk space and the file type: Linux root x86-64
## Formatting the partitions

- Now its time to format each partition with the appropriate file system
- First, I identified the names for each partition
	- EFI partition --> sda1
	- swap partition --> sda2
	- root partition --> sda3
- Then I formatted each partition
	- for EFI, I used the command `mkfs.fat -F32 /dev/sda1` to format it with the FAT32 file system
	- for swap, I first used command `mkswap /dev/sda2` to initialize the partition then `swapon /dev/sda2` to activate it
	- for root, use the command `mkfs.ext4 /dev/sda3` to create an Ext4 file system
## Mounting the file systems

- Now I need to tell the system where to write data to on the system by mounting the partitions
- I mounted the root partition to `/mnt` by using the command `mount /dev/sda3 /mnt`
- Then I mount the EFI partition to `/mnt/boot` using the command `mount --mkdir /dev/sda1 /mnt/boot`
## Installation
- Now, I need to install the base Arch Linux system into my mounted drive using the command `pacstrap -K /mnt base linux linux-firmware`
- then I created an fstab(file system table) file which is a config file written to `/mnt/etc/fstab` to tell linux which partitions to mount every time I boot 
	- To do this I used the command `genfstab -U /mnt >> /mnt/etc/fstab`
## Chroot
- use the command `arch-chroot /mnt` to interact with the new system's environment before actually booting into it
	- I did this to set the time zone, configure locales, set the hostname, and set the root password
- For time zone, use the command `ln -sf /usr/share/zoneinfo/America/Tulsa /etc/localtime`
- Then I use the command `hwclock --systohc` to synchronize the hardware clock with the system clock
- To set localization, use command `echo "LANG=en_US.UTF-8" > /etc/locale.conf` to create locale.conf file and put the LANG variable in it
- To set the host name, I used the command `echo archvm > /etc/hostname`
- Then I installed some essential packages like sudo, networkmanager, and nano using the command `pacman -S sudo networkmanager nano`
	- I enabled network manager using the command `systemctl enable NetworkManager`
- then I set a password for the root user using the command `passwd`
## Install bootloader
- I decided to use GRUB as my boot loader
- First, I installed grub and efi boot manager (neccessary for UEFI systems) using the command `pacman -S grub efibootmgr`
- Then to install the GRUB boot loader, I used the command `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
- Finally to create the GRUB configuration file, use command `grub-mkconfig -o /boot/grub/grub.cfg`
## Reboot
- exit chroot using `exit`
- unmount all partitions using `umount -R /mnt` so `fstab` can mount properly on boot
- Finally use `reboot` to reboot into the system


# Part 2: Setting up Arch Linux

## Setting up A desktop environment

- For this Arch Linux install, I decided to go with LXQt desktop environment 
- First, I had to download Xorg using the command `sudo pacman -S xorg`
	- Xorg is a X window system display server that makes graphical desktops possible by telling programs how to display things
- Then I installed LXQt itself and some themes for it by using the command `sudo pacman -S lxqt lxqt-themes`
- Since LXQt doesnt come with a built in window manager, I had to install one. I chose Openbox since its a popular choice. To install it I used the command `sudo pacman -S openbox`
- Next I installed a display manager so its easier to log in. To do this I used the commands `sudo pacman -S sddm` then `sudo systemctl enable sddm.service`
- Reboot

**Issue I ran into:** When I rebooted into LXQt, I got to the sddm display manager log in page, but I had not created a user at that point so there was no way to log in. I ended up having to switc[...]
