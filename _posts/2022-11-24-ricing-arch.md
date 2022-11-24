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

### Further infos

- [Official documentation](https://github.com/alacritty/alacritty)

## Yay

See [how to install yay]({% post_url 2022-11-24-install-yay %})

# Important

## Google Chrome

[Yay](#yay) has to be installed!

```zsh
# Install Google Chrome
yay -S google-chrome
```

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

# Nice to have
tdb!