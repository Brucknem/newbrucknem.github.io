---
layout: post
title:  "Ricing arch: How I setup my arch system"
date:   2022-11-24 12:45:00 +0100
categories: Arch-Linux
tags: tutorials arch ricing 
pin: false
toc: true
yay_needed: "[Yay](#yay) has to be installed!"

---

# Essential

## KDE Plasma and SDDM

Before you proceed: If you decide to not install the `kde-applications`, first install [a terminal emulator](#a-terminal-emulator).  
If you forgot to install it before starting KDE Plasma you won't be able to access a terminal easily.  

BUT: You can always hit `CTRL+ALT+F2` to start a login shell and bypass the display manager or desktop environment.

```zsh
# Install KDE Plasma
sudo pacman -Syu plasma-meta       # Confirm everything with Enter

# Optional: Install the KDE apps that are well integrated into KDE
sudo pacman -Syu kde-applications  # Confirm everything with Enter

# Install SDDM
sudo pacman -Syu sddm              # Confirm everything with Enter

# Enable the display manager; This starts the graphical user interface of KDE Plasma
sudo systemctl enable --now sddm
```

## A terminal emulator

If you installed the `kde-applications` the terminal `konsole` is included.

Nonetheless, I prefer [`alacritty`](https://github.com/alacritty/alacritty) as a terminal emulator:

```zsh
# Install the alacritty package
sudo pacman -Syu alacritty
```

## Yay

[Yay](https://github.com/Jguer/yay) is my preferred [AUR](https://aur.archlinux.org/) helper. 

See [how to install yay]({% post_url 2022-11-24-install-yay %})

### Further infos

- [Official documentation](https://github.com/alacritty/alacritty)

## os-prober

To dual boot with windows, `Grub` needs an extra package to detect the Windows bootloader.

```zsh
# Install package
sudo pacman -Syu os-prober

# Enable os-prober in GRUB
# Uncomment the line #GRUB_DISABLE_OS_PROBER=false
sudo vim /etc/default/grub

# Regenerate the grub config
sudo grub-mkconfig -o /boot/grub/grub.cfg
``` 

# Important

## Google Chrome

{{ page.yay_needed }}

```zsh
# Install Google Chrome
yay -S google-chrome
```

## 1password

{{ page.yay_needed }}

```zsh
# Install the package
yay -S 1password
```

## Visual Studio Code

{{ page.yay_needed }}

```zsh
# Install package 
yay -S visual-studio-code-bin
```

# Nice to have

## fish: friendly interactive shell
```zsh
# Install the fish package
sudo pacman -Syu fish
```

Close and reopen the terminal.

```zsh
# Add fish to the list of available shells
echo /usr/bin/fish | sudo tee -a /etc/shells

# Set fish as the default shell
chsh -s /usr/bin/fish
```

Logout and login to activate the change.

### Further infos

- [Official documentation](https://fishshell.com/docs/current/index.html#)

## Starship shell prompt
```zsh
# Install the starship package
# This might ask you to select a compatible font. I chose: ttf-firacode-nerd
sudo pacman -Syu starship

# Set starship as the default prompt for the fish shell
echo "starship init fish | source" >> ~/.config/fish/config.fish
```

Close and reopen the terminal.

### Further infos

- [Official documentation](https://starship.rs/de-DE/)

## Plymouth: Nice passphrase and splash screen

{{ page.yay_needed }}

```zsh
# Install package
yay -S plymouth

# Change HOOKS=(base udev ... encrypt ...) to HOOKS=(base udev plymouth ... plymouth-encrypt ...)
# Change MODULES=() to MODULES=(i915) # (Only on intel CPUs with HDPI screens)
sudo vim /etc/mkinitcpio.conf

# Regenerate image
sudo mkinitcpio -p linux

# Set GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash rd.udev.log_priority=3 vt.global_cursor_default=0"
sudo vim /etc/default/grub

# Regenerate grub config
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Set plymouth theme
sudo plymouth-set-default-theme -R spinfinity

# Start the plymouth display manager service for SDDM
sudo systemctl disable sddm
sudo systemctl enable sddm-plymouth

# Reboot and enjoy
sudo reboot now
```
