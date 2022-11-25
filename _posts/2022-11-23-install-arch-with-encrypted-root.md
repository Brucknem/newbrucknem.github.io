---
layout: post
title:  "Install Arch Linux with Luks encrypted root partition and Windows dual boot"
date:   2022-11-23 19:00:00 +0100
categories: Arch-Linux
tags: tutorials arch dualboot windows encryption
pin: false
toc: true
---

# Prepare Windows

1. Disable Bitlocker
2. Shrink the `C:` partition to the size you want. The remaining space will be used for the arch installation. 
3. Reenable Bitlocker

# Install the arch linux system

1. Download the ISO and burn to CD
2. Boot from the stick

## Set the keyboard layout and time/date

```zsh
loadkeys de
timedatectl set-ntp true
```

## Create partitions

1. Check the drive where you want to install arch to. For me this is `/dev/nvme0n1`. 
2. Open the tool to create the partitions `cfdisk /dev/nvme0n1`. Replace the drive with the one you want to install arch to.
   - Up and Down arrows: Move the curser.
   - Left and Right arrows: Select the action in the lower bar.
3. Check if there is a partition of *Type* `EFI System`.
   - Dualboot Windows: If you had Windows install previously this is already there. 
   - Only Arch: If you install Arch from scratch, create a new partition with the the *Type* `EFI System` and a size of `512M`. Use one of the *Free space* entries.
   - Remember the *Device* the partition of *Type* `EFI System` is stored on. This is you **EFI** partition. For me this is `/dev/nvme0n1p1`.
4. Create a **boot** partition of *type* `Linux filesystem` with size `512M`. Use one of the *Free space* entries. For me this is `/dev/nvme0n1p7`.
5. Create a **root** partition of *type* `Linux filesystem` with a size you can freely choose (at least `50G`). Use one of the *Free space* entries. For me this is `/dev/nvme0n1p8`.
6. Select `[ Write ]` to write the new partitions to the disk. Type `yes` to confirm.
7. Select `[ Quit ]` to close `cfdisk`.

### Tl;dr partitions layout

|Device|Type|Name|
|-|-|-|  
| `/dev/nvme0n1p1` | EFI System | EFI |
| `/dev/nvme0n1p7` | Linux Filesystem | boot |
| `/dev/nvme0n1p8` | Linux Filesystem | root |

## Create EFI and boot partition filesystems

### **Just Arch and NO Dualboot:** Run the following  
```zsh
# Create a FAT filesystem on the EFI System partition
mkfs.vfat -n "EFI System" /dev/nvme0n1p1 
```

### Always
```zsh
# Create a ext4 filesystem on the boot partition
mkfs.ext4 -L "boot" /dev/nvme0n1p7  # Confirm with 'y' if asked
``` 

## Create the encrypted root partition
```zsh
# Encrypt root partition
# Confirm with 'YES'; The passphrase is asked on every startup to decrypt the partition
cryptsetup luksFormat --type luks2 /dev/nvmn0n1p8 

# Decrypt the encrypted root partition
cryptsetup open /dev/nvme0n1p8 archlinux  # Enter the passphrase

# To check the success, the following should succed
ls /dev/mapper/archlinux

# Create a ext4 filesystem on the root partition
mkfs.ext4 -L "root" /dev/mapper/archlinux  # Confirm with 'y' if asked
```

## Install mount the partitions
```zsh
# Mount the partitions
mount /dev/mapper/archlinux /mnt            # root
mount --mkdir /dev/nvme0n1p7 /mnt/boot      # boot
mount --mkdir /dev/nvme0n1p1 /mnt/boot/efi  # EFI
```

## Install the arch system
```zsh
# 'base linux linux-firmware' are the bare minimum
# 'networkmanager' is for internet access in the new system
# 'vim' is my preferred editor
pacstrap -K /mnt base linux linux-firmware networkmanager vim

# Generate the fstab file that holds the partition definitions
genfstab -U /mnt > /mnt/etc/fstab

# Change into the new system
arch-chroot /mnt
```

## Configure the new arch system
```zsh
# Set timezone
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime  # Replace with your location

# Sync time
hwclock --systohc --utc

# Generate and select locales
vim /etc/locale.gen  # Uncomment the lines you need, e.g. de_DE.UTF-8
locale-gen           # Generate the locales
echo LANG=de_DE.UTF-8 > /etc/locale.conf    # Same as above
echo KEYMAP=de-latin1 > /etc/vconsole.conf  # Select the langugage you need

# Set the hostname
echo arch > /etc/hostname
echo 127.0.0.1  localhost arch >> /etc/hosts
echo ::1        localhost arch >> /etc/hosts

# Set a password for the root user; This will also be used for 'sudo' 
# This should be strong, as it gives full access to the system
passwd
```

## Create your user
```zsh
# Add your user
useradd -m -G wheel bruckner  # Replace with your username
passwd bruckner               # The user password for login etc.

# Give the 'wheel' group 'sudo' permission
EDITOR=vim visudo
# Uncomment the line # %wheel ALL=(ALL:ALL) ALL
```

## Install the GRUB bootloader
```zsh
# Install packages
pacman -Syu grub efibootmgr

# Adjust grub for encrypted root partition
vim /etc/default/grub
# Change: GRUB_CMDLINE_LINUX="" to GRUB_CMDLINE_LINUX="cryptdevice=/dev/nvme0n1p8:archlinux"

# Adjust the mkinitcpio.conf
vim /etc/mkinitcpio.conf
# Change: HOOKS=(... block ...) to HOOKS=(... block encrypt ...)

# Generate the mkinitcpio
mkinitcpio -p linux

# Install grub to the system
grub-install --boot-directory=/boot --efi-directory=/boot/efi /dev/nvme0n1p7

# To check success, you should see an 'arch' directory (The hostname)
ls /boot/efi/EFI

# Create the grub configs
grub-mkconfig -o /boot/grub/grub.cfg
grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg
```

# After the installation

## Reboot the PC

**!!! Congratulations !!!**

If you made it here, you have a minimal but fully functional arch installation.  

Its time to reboot the computer and start the system.

```zsh
# Exit out of the install environment
exit

# Reboot the computer
reboot
```

## Permanently enable networking
```zsh
# Autostart the networkmanager on boot
systemctl enable --now NetworkManager
```

## How to rice the bare system to a useful arch system

[Ricing Arch]({% post_url 2022-11-24-ricing-arch %})