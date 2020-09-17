---
title: Ubuntu下搭建GitLab
date: 2020-04-16 16:04:17
categories: Linux
tags:
- Linux
- Ubuntu
---

## 1. 安装依赖

`sudo apt-get install -y curl openssh-server ca-certificates`

安装Postfix服务用来发送通知邮件，使用其他SMTP服务器可以跳过此步。

`sudo apt-get install -y postfix`

## 2. 安装GitLab

添加镜像源：

```bash
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
echo "deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu bionic main" > /etc/apt/sources.list.d/gitlab-ce.list
sudo apt-get update
```

安装GitLab：

`sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ce`

将`https://gitlab.example.com`替换为自己的链接，GitLab将自动从[Let's Encrypt](https://docs.gitlab.com/omnibus/settings/ssl.html#lets-encrypthttpsletsencryptorg-integration)获取证书。

## 3. 浏览器访问

第一次访问时，将提示重置密码，默认账户名为`root`。

## 4. 管理工具：gitlab-ctl

启动/停止/重启/强制停止/状态

`gitlab-ctl start/stop/restart/kill/status`

重新获取HTTPS证书

`gitlab-ctl renew-le-certs`

清除所有数据

`gitlab-ctl cleanse`

卸载

`gitlab-ctl uninstall`
