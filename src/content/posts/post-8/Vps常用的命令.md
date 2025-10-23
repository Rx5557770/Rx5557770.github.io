
---
title: Vps常用的命令
published: 2025-10-23
description: ''
image: ''
tags: ['code']
category: 代码日常
draft: false
lang: zh-CN
---
    初始化
```bash
sudo apt update
sudo apt upgrade -y
```

1panel
```shell
bash -c "$(curl -sSL https://resource.fit2cloud.com/1panel/package/v2/quick_start.sh)"
```

warp
```shell
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh [option] [lisence/url/token]
```

docker
```shell
# 删除或注释掉 backports 相关的源
sudo sed -i '/bullseye-backports/d' /etc/apt/sources.list /etc/apt/sources.list.d/*
sudo sh get-docker.sh

# 安装 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker --version
docker-compose --version
```

openlist
```shell
curl -fsSL https://res.oplist.org/script/v4.sh > install-openlist-v4.sh && sudo bash install-openlist-v4.sh
```

lnmp
```shell
wget https://soft.lnmp.com/lnmp/lnmp2.2.tar.gz -O lnmp2.2.tar.gz && tar zxf lnmp2.2.tar.gz && cd lnmp2.2 && ./install.sh lnmp
```