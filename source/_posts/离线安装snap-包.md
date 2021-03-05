---
title: 离线安装snap 包
category:
  - Tech
tags:
  - snap
mathjax: false
date: 2021-03-05 15:04:10
img:
---

> 国内有没有 Snap 源？
> 国内就没有一个能用的 snap 镜像源！
> 如题，snap 部署很方便，但是下载速度也太慢了吧?
> 问下 ubuntu 如何安装本地的 snap 包？
<!--more-->

demo with nodejs

```bash
curl -H 'Snap-Device-Series: 16' http://api.snapcraft.io/v2/snaps/info/node >> node.json
```

`node.json` 如下，ubuntu 对应找到 amd64 stable 对应的 download url

在有网络的机器上下载 `*.snap` 包

```bash
wget https://api.snapcraft.io/api/v1/snaps/download/MEd4V4HHFkCXBSz6UzVmKF2D2PmWcVwR_3292.snap

# 下载了 MEd4V4HHFkCXBSz6UzVmKF2D2PmWcVwR_3292.snap
```

到目标机器上安装 `*.snap` 包

```bash
sudo snap install --dangerous MEd4V4HHFkCXBSz6UzVmKF2D2PmWcVwR_3292.snap --classic
```
