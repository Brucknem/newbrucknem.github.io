---
layout: post
title:  "Nvidia @ Arch"
date:   2022-11-28 17:30:00 +0100
categories: Arch-Linux
tags: tutorials arch ricing nvidia
pin: false
toc: true
---

# All rendering via GPU

```shell
# Install packages
sudo pacman -Syu nvidia xorg-xrandr

# Load nvidia at boot, adjust problems with hybrid intel/nvidia gpus
# Add "nvidia_drm.modeset=1 ibt=off" to GRUB_CMDLINE_LINUX_DEFAULT=...
sudo vim /etc/default/grub

# Add nvidia to mkinitcpio
# Add "nvidia nvidia_modeset nvidia_uvm nvidia_drm" to MODULES=(...)
sudo vim /etc/mkinitcpio.conf

# Add pacman hook to autorun mkinitcpio when there is a nvidia update
sudo mkdir -p /etc/pacman.d/hooks
sudo vim /etc/pacman.d/hooks/nvidia.hook  # See below for content to paste

# Config xorg for nvidia
sudo vim /etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf  # See below for content to paste

# Adjust xorg init
echo "xrandr --setprovideroutputsource modesetting NVIDIA-0" >> ~/.xinitrc
echo "xrandr --auto" >> ~/.xinitrc

# Adjust SDDM
sudo vim /usr/share/sddm/scripts/Xsetup  # See below for content to paste

# Regenerate configs
sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Reboot to load the Nvidia drivers
sudo reboot now
```

## /etc/pacman.d/hooks/nvidia.hook

```shell
### /etc/pacman.d/hooks/nvidia.hook

[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux
# Change the linux part above and in the Exec line if a different kernel is used

[Action]
Description=Update NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

## /etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf
```shell
Section "OutputClass"
    Identifier "intel"
    MatchDriver "i915"
    Driver "modesetting"
EndSection

Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "PrimaryGPU" "yes"
    ModulePath "/usr/lib/nvidia/xorg"
    ModulePath "/usr/lib/xorg/modules"
EndSection
```

## /usr/share/sddm/scripts/Xsetup
```shell
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```