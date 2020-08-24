---
title: win10使用docker和virtualBox冲突的过程
date: 2020-07-31 20:11:23
tags: docker
---

# win10使用docker和virtualBox冲突的过程
> 本文并没有解决 win10 同时使用 docker 和 virtualBox 时起冲突的问题。只是记录一下解决的过程。
> 系统版本：Windows 10 专业版 2004
> 操作系统版本：19041.329
> CPU：i5-4590。内存：16GB

<!--more-->

## 过程
首先，我本机中装有 Docker for Windows ，Hyper-v已经开启。尝试用VMware时，可以成功安装并启动 Centos7 系统。但是无法和 win10 主机共享文件夹。

同事的电脑安装 virtualBox 可以正常使用，我就用他的版本来尝试了一下。发现因为本机的  Hyper-v 已经开启的缘故，virtualBox 无法成功安装开机。关闭了 Hyper-v 后发现还是显示 VT-X 没有启动，同事的电脑没有安装过 docker，可以成功的启动。

检查后关闭了 Windows 功能中的`适用于 Linux 的 Windows 子系统`，然后还是依旧不能成功启动 virtualBox 安装的虚拟机。

检查网络适配器，已经只剩下 virtualBox 的和机器本身的网卡。目前进度卡在这里，决定放弃在我本机上用 virtualBox 安装 Centos7 使用。