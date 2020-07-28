---
title: "Homebrew on Linux (linux 中安装 Homebrew)"
date: 2020-07-28T18:06:47+08:00
draft: false
summary: "在Linux(Debian|Ubuntu|Fedora|CentOS|RedHat)中安装 Homebrew"
---
[HomeBrew 官网文档](https://docs.brew.sh/Homebrew-on-Linux "HomeBrew 官网文档")

```shell
# Debian or Ubuntu
sudo apt-get install build-essential curl file git
git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
mkdir ~/.linuxbrew/bin
ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
eval $(~/.linuxbrew/bin/brew shellenv)

#Fedora, CentOS, or Red Hat
sudo yum groupinstall 'Development Tools'
sudo yum install curl file git
sudo yum install libxcrypt-compat # needed by Fedora 30 and up
git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
mkdir ~/.linuxbrew/bin
ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
eval $(~/.linuxbrew/bin/brew shellenv)
```