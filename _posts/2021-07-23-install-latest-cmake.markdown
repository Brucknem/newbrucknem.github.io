---
layout: post
title:  "How to install the latest CMake version from source"
date:   2021-07-23 09:50:00 +0200
categories: Installation
tags: coding tutorials install
pin: false
---

Run the following `shell` script to install the latest CMake version.

The script will automatically check the for the latest CMake version and download the source from the [Github](https://github.com/Kitware/CMake/) repository. 

The script then compiles the source and creates a `.deb` via `checkinstall`. Follow the instructions from `checkinstall` to install the compiled package.

# Install
```shell
# Check Version
LOCATION=$(wget https://github.com/Kitware/CMake/releases/latest 2>&1 | grep Location | awk '{print $2}')
if [ "$SHELL" = "/usr/bin/zsh" ]; then
    VERSION=(${(@s:/:)LOCATION})
else
    IFS='/' read -ra VERSION <<< "$LOCATION"
fi
VERSION=$(echo ${VERSION[-1]})
VERSION=$(echo ${VERSION:1})
echo "Installing: CMake v$VERSION"

# Download
cd /tmp
CMAKE_EXTRACTED_DIR="cmake-$VERSION"
CMAKE_ARCHIVE="$CMAKE_EXTRACTED_DIR.tar.gz"
wget https://github.com/Kitware/CMake/releases/download/v$VERSION/$CMAKE_ARCHIVE
tar xfv $CMAKE_ARCHIVE

# Install
cd $CMAKE_EXTRACTED_DIR
./bootstrap
NUM_CPU_CORES=$(grep -c ^processor /proc/cpuinfo)
make -j $NUM_CPU_CORES

sudo apt update -y && sudo apt upgrade -y
sudo apt install checkinstall libssl-dev -y
sudo checkinstall --pkgname cmake
```

# Check Installation
```shell
cmake --version
```

# Uninstall
```shell
sudo dpkg -r cmake
```