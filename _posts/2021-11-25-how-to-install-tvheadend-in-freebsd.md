---
layout: post
title: "如何在FreeBSD安装tvheadend 以及搜索DVB-C电视"
categories: tech-share
author: "papadave"
---
### 前期准备
如果你想通过服务器接收数字电视信号，你需要提前准备以下的东西。
1. DVB-C 电视棒 我选择的是USB版本的。
2. F母头转TV公头（非必须，取决你的有线电视进户线接口）

我的USB电视棒是这个样子:<br>
![DVB-C dongle](https://www.linuxtv.org/wiki/index.php/File:Astrometa-dvb-t2.png)
但是不同的是我买的这款，外壳带有DVB-C字样<br>
<br>
接下来是编译环节。推荐通过ports编译安装tvheadend 和 webcamd。<br>
使用root权限运行以下命令：
`cd /usr/ports/multimedia/tvheadend` `CPP="gcc -E" CC=gcc CFLAGS="-Os --pipe" CXX=g++ CXXFLAGS="-Os --pipe" make install` <br>
`cd /usr/ports/multimedia/webcamd` `CPP="gcc -E" CC=gcc CFLAGS="-Os --pipe" CXX=g++ CXXFLAGS="-Os --pipe" make install` <br>
安装好后将tvheadend用户添加到webcamd组 `pw groupmod webcamd -m`<br>
<br>
接好电视棒，输入`usbconfig` 应该能看到如下类似结果：<br>
```
ugen1.2: <vendor 0x8087 product 0x0024> at usbus1, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)  
ugen1.3: <astrometadvbt2 dvbt2> at usbus1, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=ON (500mA)          
ugen1.4: <SCM Microsystems Inc. SCR3310 v2.0 USB SC Reader> at usbus1, cfg=0 md=HOST spd=FULL (12Mbps) pwr=ON (100mA)                        
```

记录下你的电视棒前面的ugen开头的字符串，此例为 ugen1.3 <br>
在 `/etc/rc.conf` 下 添加以下配置：<br>

```
#tvheadend                   
tvheadend_enable="YES"       
tvheadend_flags="-C" 
cuse_load="YES"              
webcamd_enable="YES"         
webcamd_0_flags="-d ugen1.3"
```
接下来运行 `kldload cuse4bsd` `service webcamd start` `service tvheadend start` <br>
在`http://IP_ADDRESS:9981` 进行配置，由于之前在rc.conf 里设置了`tvheadend_flags="-C"`,可以进行初始化配置
Language 可以选择中文。第二步允许的网络如果你想以后外网访问就写0.0.0.0/0，如果内网访问就写内网地址，注意使用CIDR格式<br>
配置用户名和密码。再下一步可以跳过，直接点下一步到最后。<br>
<br>
然后回到 `/etc/rc.conf` 里，将`tvheadend_flag`那行用#注释掉,运行`service tvheadend restart`<br>
<br>
### 测试DVB-C电视棒
首先确定webcamd服务是否正常运行<br>
1.编译安装ports的 `multimedia/w_scan2`
2.运行`w_scan2 -f c >> w_scan.log` 
3.如果显示`main:4785: FATAL: ***** NO USEABLE CABLE CARD FOUND. *****` 则可能需要停用tvheadend服务再尝试，否则可能webcamd的配置不正确
如果一切正常，则可以看到搜索结果<br>
<br>
<br>

### 在tvheadend中配置DVB-C相关
1. `cd /usr/local/share/dtv-scan-tables/dvb-c` 复制其中一份，命名为zh-something
2. 修改里面的channel.你只需要写一个就行。样例如下：<br>
```
[CHANNEL]                                   
        DELIVERY_SYSTEM = DVBC/ANNEX_A      
        FREQUENCY = 315000000               
        SYMBOL_RATE = 6875000               
        INNER_FEC = NONE                    
        MODULATION = QAM/64                 
        INVERSION = AUTO                                                             
```
然后重启tvheadend service。并回到tvheadend webUI<br>
1. DVB输入-TV adapter-选中你的DVB-C设备 点击启用，并且取消勾选idle scan
2. 在network选项卡里点击添加，type写DVB-C Network，network name 随便，
pre-defined muxes选刚才你创建的zh-something，字符集选GB2312,时区选UTC+8 点击save
3. 回到DVB输入-TV adapter选项卡，选中DVB-C设备,network选上一步的名字。点击save
4. DVB输入-Muxes选项卡里应该出现所有的频率了，然后回到network选项卡里点击强制扫描，就开始搜索节目了
5. DVB输入-services 里 应该有所有的频道了。点击Map services- Map all service
6. 频道/EPG 选项卡里应该有所有的频道了。
<br>
到此为止已经基本完成。如果需要下载m3u列表，需要访问http://ip:9981/playlist下载。<br>
并且在每个链接的http://后面添加你的`tvheadend用户名:tvheadend密码@`保存即可<br>
由于没有配置CA卡，只能看免费的电视节目
