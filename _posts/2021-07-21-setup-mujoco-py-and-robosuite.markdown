---
layout: post
title:  "How to install mujoco-py and robosuite"
date:   2021-07-21 10:20:00 +0200
categories: Install
tags: coding tutorials install
pin: false
---

# Prerequisites

1. Go to the [MuJoCo website](https://www.roboti.us/license.html) and obtain a license.
2. Download and extract MuJoCo

```shell
cd /tmp
wget https://www.roboti.us/download/mujoco200_linux.zip
mkdir -p $HOME/.mujoco/mujoco200
unzip mujoco200_linux.zip
mv mujoco200_linux/* $HOME/.mujoco/mujoco200
```
3. Download the license and place it in `~/.mujoco/mjkey.txt`

```shell
mv $HOME/Downloads/mjkey.txt $HOME/.mujoco/
```

# Installation

```shell
# Install dependencies
sudo apt update -y && sudo apt upgrade -y

sudo apt install libosmesa6-dev libgl1-mesa-glx libglfw3 libglew-dev -y
sudo ln -s /usr/lib/x86_64-linux-gnu/libGL.so.1 /usr/lib/x86_64-linux-gnu/libGL.so

sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update -y
sudo apt-get install python3.6 -y
sudo apt-get install python3.6-dev  
sudo apt install patchelf
sudo apt purge python-virtualenv -y
sudo apt purge python3-virtualenv -y
sudo pip3 install virtualenv 

# Create virtualenv
virtualenv -p /usr/bin/python3.6 venv
source venv/bin/activate

# Install 
python -m pip install -I testresources
python -m pip install -I fasteners

python -m pip install -I --no-use-pep517 mujoco-py
python -m pip install -I robosuite
```

# Setup

## Command Line
```shell
if [ "$SHELL" = "/usr/bin/zsh" ]; then
    SHELL_TYPE="zsh"
else
    SHELL_TYPE="bash"
fi

echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/home/$USER/.mujoco/mujoco200/bin" >> ~/.$SHELL_TYPE"rc"
echo "export LD_PRELOAD=\$LD_PRELOAD:/usr/lib/x86_64-linux-gnu/libGLEW.so" >> ~/.$SHELL_TYPE"rc"

source ~/.$SHELL_TYPE"rc"
```

## PyCharm
Add the following to the `Environment variables` in the run configs.
```
;LD_LIBRARY_PATH=/home/brucknem/.mujoco/mujoco200/bin;LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libGLEW.so
```

### To apply it to all future created run configs:
- Open any run config
- Click on lower left: `Edit configuration templates...`
- Add the above to the `Environment variables`


# References

- [mujoco-py](https://github.com/openai/mujoco-py)
- [robosuite](https://github.com/ARISE-Initiative/robosuite)
