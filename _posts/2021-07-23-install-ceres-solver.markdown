---
layout: post
title:  "How to install the Ceres Solver"
date:   2021-07-23 10:25:00 +0200
categories: Installation
tags: coding tutorials install
pin: false
---

Run the following `shell` script to install the [Ceres Solver](http://ceres-solver.org/index.html) to `Ubuntu 20.04`.

The script will download the source from the [Github](https://github.com/Kitware/CMake/) repository. 

The script then compiles the source and creates a `.deb` via `checkinstall`. Follow the instructions from `checkinstall` to install the compiled package.

# Prerequisites

The script relies on [CMake](https://cmake.org/). If you don't have CMake installed, please see this [CMake installation guide]({% post_url 2021-07-23-install-latest-cmake %}).

# Install
```shell
# Install Dependencies
sudo apt update -y && sudo apt upgrade -y

sudo apt-get install libgoogle-glog-dev libgflags-dev -y
sudo apt-get install libatlas-base-dev -y
sudo apt-get install libeigen3-dev -y
sudo apt-get install libsuitesparse-dev -y

# Download
cd /tmp
CERES_VERSION="ceres-solver-2.0.0"
CERES_ARCHIVE="$CERES_VERSION.tar.gz"
wget http://ceres-solver.org/$CERES_ARCHIVE
tar xfv $CERES_ARCHIVE

# Install
cd $CERES_VERSION
mkdir build
cd build
NUM_CPU_CORES=$(grep -c ^processor /proc/cpuinfo)
cmake ..
cmake --build . -j $NUM_CPU_CORES

sudo apt install checkinstall libssl-dev -y
sudo checkinstall --pkgname ceres-solver
```

# Uninstall
```shell
sudo dpkg -r ceres-solver
```