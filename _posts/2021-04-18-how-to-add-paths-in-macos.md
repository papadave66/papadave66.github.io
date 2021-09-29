---
layout: post
title: "如何在MacOS中添加PATH和MANPATH"
categories: tech-share
author: "papadave"
---
在其他的操作系统中，例如发行版Linux。可以在 `~/.profile` 添加PATH<br>
而在macOS Catalina中，如果查看 `/etc/profile` 的话，可以发现Apple 使用了个叫`path_helper`的东西<br>
用户可以使用sudo 权限，直接在 `/etc/paths` 和 `/etc/manpaths` 中添加路径即可,每个路径一行
