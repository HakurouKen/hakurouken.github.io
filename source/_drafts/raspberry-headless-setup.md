---
title: 树莓派的无头初始化
tags:
  - raspberry pi
  - mac
---

## Step.1 从树莓派官网下载镜像

在[树莓派官网](https://www.raspberrypi.org/downloads/raspbian/)可以下载最新的镜像，为了后续省去安装一些软件，我们可以下载附带必要软件的完整包。

## Step.2 格式化 SD 卡

使用 [SD Memory Card Formatter](https://www.sdcard.org/downloads/formatter_4/index.html)对 SD 卡格式化，这个软件同时提供 Mac 版和 windows 版。

## Step.3 写入镜像

使用 [balenaEtcher](https://www.balena.io/etcher/)将树莓派镜像写入 SD 卡。在 windows 上，类似的可以使用 [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)

## Step.4 配置 WIFI / ssh

为了使树莓派能够自动连接 WIFI，我们需要将一个 `wpa_supplicant.conf` 放在树莓派 boot 的根目录下。`wpa_supplicant.conf` 的配置内容如下：

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CN

network={
  ssid="YOUR_WIFI_SSID"
  psk="YOUR_WIFI_PASSWORD"
  key_mgmt=WPA-PSK
}
```

新版本的树莓派，默认关闭了 ssh 功能，为了开启 ssh ，我们需要在 boot 的根目录下，创建一个空的 `ssh` 文件即可。

在完成上述步骤之后，我们所有的前置工作均已经准备完成。接下来只需要将 SD 卡插入树莓派并接电开机，树莓派即会自动连接到指定 SSID 的 WIFI。我们登录路由器，即可看到给树莓派分配的 IP。 ssh 连接树莓派的默认账号是 `pi`，密码是 `raspberry`。

## Step.5 修改用户名/密码

可以通过 `sudo raspi-config`，也可以使用 `adduser` 和 `passwd` 命令直接修改。

## Step.6 locales

同样也可以在 `raspi-config` 中设置，也可以通过 `update-locale` 命令设置。我们也可以手动设置 locale 的值，可以参考 [Debian 的文档](https://wiki.debian.org/Locale)。

## Step.7 国内更新源

树莓派的更新源，涉及两个位置：

```bash
/etc/apt/sources.list
/etc/apt/sources.list.d/raspi.list
```

分别对应 **apt 软件源** 和 **树莓派的系统镜像源**。

树莓官方网站中，给出了一系列的 raspbian [软件镜像源](https://www.raspbian.org/RaspbianMirrors)，但是并没有包括全部的。这里建议使用阿里云的源：http://mirrors.aliyun.com/raspbian/raspbian/ 。

阿里云没有 archive.raspberrypi.org 的源，我们可以选择：

切换到清华的源 https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/

或者中科大的源 https://mirrors.ustc.edu.cn/archive.raspberrypi.org/

更新源中的 `stretch` / `jessie` / `wheezy` ，分别对应树莓派系统 raspbian 的发行版，和 Debian 的版本一一对应，参考[Debian 的发行版本](https://www.debian.org/releases/)。

在修改完成之后，执行：

```bash
sudo apt update && sudo apt upgrade -y
```

来更新。

## Step.8 安装基础软件

这一步因人而异，这里仅针对个人情况，做一个简单记录。

```bash
# 安装 vim
sudo apt install vim
# 安装 zsh
sudo apt install zsh
# 安装 oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# 安装 nvm/nodejs
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | zsh
# node
nvm install
# nginx
sudo apt install nginx
# php
sudo apt install php
```
