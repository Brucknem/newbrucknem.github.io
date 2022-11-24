---
layout: post
title:  "Install Ubuntu Linux with Luks encrypted root partition and Windows dual boot"
date:   2022-11-24 18:30:00 +0100
categories: Ubuntu
tags: tutorials ubuntu dualboot windows encryption
pin: false
toc: true
---
# Prepare Windows

1. Disable Bitlocker
2. Shrink the `C:` partition to the size you want. The remaining space will be used for the arch installation. 
3. Reenable Bitlocker

# Acknowledgements

I'm following the [great tutorial](https://www.mikekasberg.com/blog/2020/04/08/dual-boot-ubuntu-and-windows-with-encryption.html) of [Mike Kasberg](https://mikekasberg.com) on how to install Ubuntu in a dual boot setup with windows. 

Thank you Mike for the only really working tutorial out there. :raised_hands::raised_hands::raised_hands:

# Prepare the drives

## Boot from the USB stick

1. Download the ISO and burn to CD
2. Boot from the stick

## Create partitions

Same as for [arch linux]({%post_url 2022-11-23-install-arch-with-encrypted-root %})

## Create EFI and boot partition filesystems

Same as for [arch linux]({%post_url 2022-11-23-install-arch-with-encrypted-root %})

## Create the encrypted root partition

Same as for [arch linux]({%post_url 2022-11-23-install-arch-with-encrypted-root %})

I only changed the name of the mapper device to `ubuntu`.

```zsh
# Encrypt root partition
# Confirm with 'YES'; The passphrase is asked on every startup to decrypt the partition
cryptsetup luksFormat --type luks2 /dev/nvmn0n1p8 

# Decrypt the encrypted root partition
cryptsetup open /dev/nvme0n1p8 ubuntu  # Enter the passphrase

# To check the success, the following should succed
ls /dev/mapper/ubuntu

# Create a ext4 filesystem on the root partition
mkfs.ext4 -L "root" /dev/mapper/ubuntu  # Confirm with 'y' if asked
```

# Install the Ubuntu system

## Install via Ubiquity

Run the Ubuntu installer [Ubiquity](https://wiki.ubuntuusers.de/Ubiquity/) as you would normally do.

When asked for the **Installation type** choose **Something else**

- Choose `/dev/nvme0n1p7` with `ext4` as `/boot` 
- Choose `/dev/mapper/ubuntu` with `ext4` as `/` 
- Choose the bootloader device as `/dev/nvme0n1`

Finish the installation as usual.

Choose **Continue testing** when the installation is finished.

## Get the UUID of the root partition
```zsh
# Search for the UUID=... entry and note it down
sudo blkid /dev/nvme0n1p8
```

## Mount and change to the newly installed system

```zsh
# Mount the root partition
mount /dev/mapper/ubuntu /mnt

# Mount the boot partition
mount /dev/nvme0n1p7 /mnt/boot

# Mount the system devices
for n in proc sys dev etc/resolv.conf; do mount --rbind /$n /mnt/$n; done 

# Change into the new system
chroot /mnt

# Mount all filesystems mentioned in fstab
mount -a
```

## Setup the crypttab file
```zsh
# Install vim
apt install vim

# Open the file
vim /etc/crypttab
```

Paste the following into the file and replace the `UUID=...` with the UUID from above.

```zsh
# <target name> <source device> <key file> <options>
# options used:
#     luks    - specifies that this is a LUKS encrypted device
#     tries=0 - allows to re-enter password unlimited number of times
#     discard - allows SSD TRIM command, WARNING: potential security risk (more: "man crypttab")
#     loud    - display all warnings
ubuntu UUID=... none luks,discard
```

# After the installation

tbd!