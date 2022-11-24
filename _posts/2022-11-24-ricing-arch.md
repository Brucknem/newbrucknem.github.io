---
layout: post
title:  "Ricing arch: How I setup my arch system"
date:   2022-11-24 12:45:00 +0100
categories: Arch-Linux
tags: tutorials arch rice ricing 
pin: false
toc: true
---

# Essential

## KDE Plasma and SDDM
```zsh
# Install KDE Plasma
pacman -Syu plasma-meta       # Confirm everything with Enter

# Optional: Install the KDE apps that are well integrated into KDE
pacman -Syu kde-applications  # Confirm everything with Enter

# Install SDDM
pacman -Syu sddm              # Confirm everything with Enter

# Enable the display manager; This starts the graphical user interface of KDE Plasma
systemctl enable --now sddm
```

## A terminal emulator

If you installed the `kde-applications` the terminal `konsole` is included.

Nonetheless, I prefer [`alacritty`](https://github.com/alacritty/alacritty) as a terminal emulator:

```zsh
sudo pacman -Syu alacritty
```

## Yay

See [how to install yay]({% post_url 2022-11-24-install-yay %})

# Important

## Google Chrome

[Yay](#yay) has to be installed!

```zsh
# Install Google Chrome
yay -S google-chrome
```

# Nice to have
tdb!