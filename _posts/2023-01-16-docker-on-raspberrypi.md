---
layout: post
title:  "Docker and Portainer on Raspberry Pi"
date:   2023-01-16 19:00:00 +0100
categories: "Raspberry Pi"
tags: tutorials raspberrypi docker
pin: false
toc: true
---

# Prerequisites

- At least a Raspberry Pi **3** with 4GB RAM
- Better a Raspberry Pi **4** with 4GB RAM

# Install Docker

```shell
# Download Docker install script
curl -fsSL https://get.Docker.com -o get-Docker.sh

# Install Docker via the script
sudo sh get-Docker.sh

# Add your user to the allowed Docker users
sudo usermod -aG docker $USER

# Reload the group settings
newgrp docker

# Optional: Check the installation
docker run hello-world
```

# Run Portainer

```shell
# Set the port you want to reach Portainer
PORTAINER_PORT=9443

# Create a persistent volume for the Portainer settings
docker volume create portainer

# Run Portainer
docker run -d --restart always -p 8000:8000 -p $PORTAINER_PORT:9443 --name portainer  -v /var/run/docker.sock:/var/run/docker.sock -v portainer:/data cr.portainer.io/portainer/portainer-ce:latest

# Optional: Check if Portainer is running
docker ps

# Open Portainer
echo "Portainer is reachable via https://192.168.178.72:$PORTAINER_PORT"

# Note: Without a certificate you will see a warning that the connection is not secure when opening in the browser
```