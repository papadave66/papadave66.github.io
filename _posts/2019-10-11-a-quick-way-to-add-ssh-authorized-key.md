---
layout: post
title: "一个快速添加ssh authorized key的方法"
categories: tech-share
author: "papadave"
---
如何快速添加authorized key? <br>
之前很多操作都是使用```ssh```登陆 到目标主机，再对应账户的```~/.ssh/authorized_keys``` 里手动添加你的public key<br>
但是最近闲着无聊火星地发现了一个命令：
```ssh-copy-id```<br>
这个指令实际上也是个shell脚本。源码可以在 ``` cat `which ssh-copy-id ```找到，感兴趣的朋友可以自行研究