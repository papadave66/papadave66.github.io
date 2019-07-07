---
layout: post
title: "如何解决WIN10下挂载NFS乱码"
date: 2019-07-07
categories: tech-share
author: "papadave"
---

    在使用windows 10的时候，想访问NAS的资源，于是试着挂载了nfs。但是文件名乱码，经过搜索后找到了如下解决方法：

    1.在搜索框搜索language 或语言，点击 **编辑语言和键盘选项**
    ![search_language_at_windows_search](/images/2019-07-07-1.png)

    2.点击 **管理语言设置**
    ![click_language_settings](/images/2019-07-07-2.png)

    3.点击 **管理** 选项卡，选择 **更改系统区域设置**
    ![click_change_system_locate](/images/2019-07-07-3.png)

    4.勾选 **Beta版：使用Unicode UTF-8 提供全球语言支持**
    ![set_unicode](/images/2019-07-07-4.png)

    现在重新启动你的电脑，就没有乱码啦，是不是很方便。
