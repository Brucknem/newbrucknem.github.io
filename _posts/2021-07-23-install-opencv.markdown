---
layout: post
title:  "How to install OpenCV"
date:   2021-07-23 10:55:00 +0200
categories: Installation
tags: coding tutorials install
pin: false
---

Run the following `shell` script to install the latest [OpenCV](https://opencv.org/) to `Ubuntu 20.04`.
The script will install OpenCV with support for the `contrib` modules, `CUDA`, `cuDNN`, `OpenGL`, `Python3` and much more. See the script for all enabled options.

The script will download the OpenCV `core` from the [OpenCV](https://github.com/opencv/opencv) repository and the OpenCV `contrib` modules from the [OpenCV contrib](https://github.com/opencv/opencv_contrib) repository. 

The script then compiles the source and creates a `.deb` via `checkinstall`. Follow the instructions from `checkinstall` to install the compiled package.

# Prerequisites

OpenCV relies heavily on [CUDA](https://developer.nvidia.com/cuda-zone) and [cuDNN](https://developer.nvidia.com/cudnn).
If you don't have CUDA and/or cuDNN installed, please see this [CUDA and cuDNN installation guide]({% post_url 2021-07-23-install-cuda %}).

The script relies on [CMake](https://cmake.org/). If you don't have CMake installed, please see this [CMake installation guide]({% post_url 2021-07-23-install-latest-cmake %}).

The script relies on [Ceres Solver](http://ceres-solver.org/index.html). If you don't have Ceres Solver installed, please see this [Ceres Solver installation guide]({% post_url 2021-07-23-install-ceres-solver %}).

# Install
```shell
# Install Dependencies
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y g++ wget unzip

sudo apt install liblz-dev libtiff5 libtiff5-dev libtiff-tools libwebp6
sudo apt-get install vlc ubuntu-restricted-extras

sudo apt install liblz-dev -y
sudo apt-get install python3-dev python3-numpy python3-pip -y
sudo apt-get install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev -y
sudo apt-get install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev -y
sudo apt-get install libpng-dev -y
sudo apt-get install libjpeg-dev -y
sudo apt-get install libopenexr-dev -y
sudo apt-get install libtiff-dev -y
sudo apt-get install libwebp-dev -y
sudo apt-get install qt5-default -y

sudo apt install libavcodec-dev libavformat-dev libswscale-dev libavresample-dev -y
sudo apt install libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev  -y  
sudo apt install libfaac-dev libmp3lame-dev libvorbis-dev -y
sudo apt-get install libgtk-3-dev -y
sudo -H pip3 install -U pip numpy

sudo apt-get install libtbb-dev -y
sudo apt-get install libprotobuf-dev protobuf-compiler -y
sudo apt-get install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen -y
sudo apt-get install libdc1394-22 libdc1394-22-dev libxine2-dev libv4l-dev v4l-utils -y
sudo ln -s -f /usr/include/libv4l1-videodev.h /usr/include/linux/videodev.h

# Download
cd /tmp
wget -O opencv.zip https://github.com/opencv/opencv/archive/master.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/master.zip
unzip opencv.zip
unzip opencv_contrib.zip

# Install
BUILD_DIR="opencv_build"
mkdir $BUILD_DIR
cd $BUILD_DIR

CUDA_ARCH_BIN="75" # You may replace this with your GPU architecture
cmake \
  -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-master/modules \
  -DCMAKE_BUILD_TYPE=Release \
  -DOPENCV_ENABLE_NONFREE=ON \
  -DBUILD_opencv_python3=ON \
  -DINSTALL_TESTS=ON \
  -DINSTALL_C_EXAMPLES=ON \
  -DINSTALL_PYTHON_EXAMPLES=ON \
  -DWITH_CUDNN=ON \
  -DOPENCV_DNN_CUDA=ON \
  -DWITH_CUDA=ON \
  -DCUDA_ARCH_BIN=$CUDA_ARCH_BIN \
  -DWITH_CUBLAS=ON \
  -DENABLE_FAST_MATH=ON \
  -DCUDA_FAST_MATH=ON \
  -DWITH_OPENGL=ON \
  -DWITH_GSTREAMER=ON \
  -DOPENCV_GENERATE_PKGCONFIG=ON \
  -DWITH_TBB=ON \
  -DWITH_V4L=ON \
  -DWITH_QT=ON \
  -DWITH_OPENGL=ON \
  -DOPENCV_GENERATE_PKGCONFIG=ON \
  -DOPENCV_PC_FILE_NAME=opencv.pc \
  -DBUILD_EXAMPLES=ON \
  ../opencv-master

NUM_CPU_CORES=$(grep -c ^processor /proc/cpuinfo)
cmake --build . -j $NUM_CPU_CORES

sudo apt install checkinstall libssl-dev -y
sudo checkinstall --pkgname opencv
```

# Uninstall
```shell
sudo dpkg -r opencv
```
