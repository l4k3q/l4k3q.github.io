---
title: "docker deployment guide on ubuntu 22.04 LTS"
tags:
  - deployment
  - docker
date: 2026-09-15 22:56:00 +0800 # 可选：覆盖文件名里的日期
---

### Before You Begin

```Bash
curl -fsSL https://get.docker.com -o get-docker.sh | bash
```

this command essentially downloads a script from the internet and executes it immediately with root privileges, the installation process is a black box and not controllable

you don't even know what exactly happens at each step, which makes the environment non-auditable and version management hard

**ONLY use this for quick testing** 

**NOT for production**

### Prerequisites

**System:** Ubuntu 22.04 LTS (Jammy Jellyfish), 64-bit version

**Permissions:** Root or sudo privileges are required

**Minimum Kernel Version:** 3.10 (satisfied by default on Ubuntu 22.04)

### Install Docker

#### Unstall potential old Docker versions

try removing existing older versions to avoid conflicts

```Bash
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get autoremove
```

Note: even if these packages are not installed on the system, running this command will not produce errors and helps ensure a clean environment

#### Install required dependencies

```Bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

these packages ensure that the system can securely download and verify official Docker packages.

#### Add Docker’s official GPG key

```Bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

use the Alibaba Cloud mirror:

```Bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

#### **Add the Docker** **repository**

```Bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

use the Alibaba Cloud mirror:

```Bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Install Docker Engine, CLI, containerd and Docker Compose

```Bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Verify the installation

Start up the Docker service and enable it to start on boot

```Bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Test Installation

```Bash
sudo docker --version
sudo docker run hello-world
sudo docker compose version
```

### Configure a Registry Mirror

1. Edit the Docker configuration file

```Bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json 
```

1. Add the following content

```JSON
{
  "registry-mirrors": [
    "https://mirror.baidubce.com",
    "https://docker.m.daocloud.io"
  ]
}
```

1. Restart Docker service

```Bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

1. Verify the registry mirror configuration

```Bash
docker info | grep -A 5 "Registry Mirrors"
```