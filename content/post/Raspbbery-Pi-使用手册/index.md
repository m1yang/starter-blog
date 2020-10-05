---
title: 'Raspbbery Pi 使用手册'
subtitle:
summary: 
authors:
- miyang
tags:
- 树莓派
categories:
- Linux
date: "2017-10-17T10:08:04Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: false
---

**注：**推荐裸板，自己动手丰衣足食。[官方文档](https://www.raspberrypi.org/documentation/)

## 购买配置

**最小可选**

* **树莓派板子**`必选` (选最新最便宜的)
* **SD卡**`必选` 建议容量8G以上(起码也买个16G的吧)，TF卡又叫micro SD卡，现在一般都用TF卡
* **读卡器**`必选` 如果有别的方式烧录可以不买，比如说买的烧录好的(不推荐)
* **网线** `推荐` 尤其是在没显示屏的情况下，当然设置好wifi连接后就没必要了
* **电源** `推荐`听说树莓派最佳电源是2A，一般的安卓手机电源满足不了

**其他可选**

* **散热片** `推荐` 通常3片装
* **保护壳** `推荐`
* **风扇** `可选`
* **HDMI / DVI显示屏** `可选` 买不起显示屏，通过电脑wifi远程连接
* **视频线及转接口** `可选`
* **鼠标、键盘等** ``可选`` 

我的使用环境是**笔记本win10**加**wifi**远程连接，所以买的是：板子、TF卡、读卡器、电源、散热片、保护壳及网线

提示：连接上树莓派后关机不建议热插拔，命令行`sudo poweroff`

## 烧录篇

1. 下载系统，推荐[下载](https://www.raspberrypi.org/downloads/)`raspbian`系统。有`DESKTOP` 桌面版和`LITE` 最简安装。

2. Windows系统，下载烧录软件`Win32DiskImager`然后把TF卡插入读卡器，选好`Image File`和`Device`直接点`Write`就行了，其它选项可以忽略。

   **注** :重新烧录的情况下不要写入boot盘。

3. **重点** 烧录完成后，打开名为`boot` 的盘符，创建一个名为`ssh`的文件夹。

   这里是因为截至2016年11月发布的版本，默认情况下，Raspbian已禁用SSH服务器。

4. TF卡插入树莓派，**网线** 连接路由器，接通电源，红灯常亮就表示OK了

   (注意：这里pi还没有设置WIFI连接，没有网线是不行滴，除非你有显示屏可以连接使用)

5. 查找树莓派IP地址，不推荐端口扫描软件
   * 笔记本按`WIN+R`键输入`CMD`回车，在命令行输入`arp -a` 。

     这时候一般会有两个IP，不确定的话，可以再输`ipconfig`查看本机IP，那另一个就是树莓派的IP了。

     再不然，拔了树莓派的网线，输`arp -a`少了的那个就是树莓派的IP了。

   * 查看路由器的客户端列表。我是小米路由器，手机上点开`小米WiFi`一目了然。

6. 知道树莓派的IP后，就可以使用SSH连接了。使用`PuTTY` 、`xshell` 等都可以。这里使用免费的`PuTTY` ，连接上后，账号默认是`pi`，密码默认是`raspberry` ([centos7](http://mirror.centos.org/altarch/7/isos/armhfp/) 默认账号：root，密码:centos)。登录成功后会看到命令行提示符 `pi@raspberrypi ~ $`

## 设置篇

### 设置WIFI

**命令行设置WIFI**

- 获取可连接的网络`sudo iwlist wlan0 scan`

  注：很多时候附近可连接的网络比较多，建议指令存储到某个临时文件中

- 生成加密连接`wpa_passphrase "testing" "testingPassword"`

- 复制加密连接内容至`sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`

```shell
wpa_passphrase "testing" "testingPassword" >> /etc/wpa_supplicant/wpa_supplicant.conf
# 上面那个命令需要root权限，用下面那个
wpa_passphrase "testing" "testingPassword" | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf > /dev/null
```

这时文件中内容大概是这样：

```shell
network={
      ssid="testing" #连接的WIFI名称
      #psk="testingPassword" #连接的WIFI密码 	  
     psk=131e1e221f6e06e3911a2d11ff2fac9182665c004de85300f9cac208a6a80531
  }
```

注释部分可自行删除

查看是否连接完成，先让配置生效`sudo wpa_cli reconfigure` ，输入`ifconfig wlan0` 可查看wifi的连接(连网线的情况下是查看不了滴)。

**多网络连接**

修改配置`sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`

```shell
network={
    ssid="HomeOneSSID"
    psk="passwordOne"
    priority=1 #优先级
    id_str="homeOne" #连接标识
}

network={
    ssid="HomeTwoSSID"
    psk="passwordTwo"
    priority=2
    id_str="homeTwo"
}
```

**注：** 设置完成后重启即可，也可以等配置完了再重启连接，注意WIFI连接后IP地址可能会不同于有线连接的地址，需要重新设置连接的IP。

设置静态IP地址 `sudo vim /etc/dhcpcd.conf` 

### 登录配置工具 

在SSH控制台输入`sudo raspi-config` 

注：一般来说第一步都是拓展系统空间，但是我进入`配置工具`时并没有这个选项(应该是新版已经默认拓展了)。然后找了找，原来在`7 Advanced Options` 中，我们选择`ExpandFilesystem`

### 更改密码

**(Change User Password)** 

密码是需要修改的，毕竟默认密码是公开的，所以这里我们选择`Change User Password` 按流程输入两次密码就OK了

注：激活root

在命令行界面输入`sudo passwd root` 输入两遍密码后，再在命令行输`sudo passwd --unlock root` ，root用户便可以使用了。

命令`su -` 输入设置的密码，便可切换为root用户。

### 本地化

**(Localisation Options)** 

**tips** :键盘敲击选项的首字母可以快速定位到以该字母为首的选项旁。

时区本地化：`Localisation Options` -->`Change  Timezone` -->`Asia` --> `shanghai` 

wifi设置：`Change Wi-fi Country` -->`China` 

语言本地化：好好学英语，不设置。但是还是记录一下该怎么做。

首先获取中文字体，命令行`sudo apt-get install ttf-wqy-zenhei` ，然后命令`sudo raspi-config` 进入配置页面，选择`Localisation Options` -`Change Locale` 翻到最后一页再向上找`zh_CN. UTF-8` 敲空格 * 号标记选中后回车，再选择zh_CN，中文本地化设置就是这些步骤了。

## VNC

因为我使用的是LITE版没有桌面，所以略。。。(命令行用习惯了，桌面太多余)

[**启用VNC** 服务器](https://www.raspberrypi.org/documentation/remote-access/vnc/README.md)

先更新到最新版本的`VNC Connect`

```shell
sudo apt-get update
sudo apt-get install realvnc-vnc-server realvnc-vnc-viewer
```

命令行输入`sudo raspi-config` ，找到`Interfacing Options` ，在VNC选项中`enable` 选`yes` 

修改VNC配置文件

```shell
sudo nano /etc/init.d/vncserver
```

```shell
#!/bin/sh
### BEGIN INIT INFO
# Provides:          vncserver
# Required-Start:    $local_fs
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start/stop vncserver
### END INIT INFO
 
# More details see:
# http://www.penguintutor.com/linux/vnc
 
### Customize this entry
# Set the USER variable to the name of the user to start vncserver under
export USER='pi'
### End customization required
 
eval cd ~$USER
 
case "$1" in
  start)
    # 启动命令行。此处自定义分辨率、控制台号码或其它参数。
    su $USER -c '/usr/bin/vncserver -depth 16 -geometry 1024x768 :1'
    echo "Starting VNC server for $USER "
    ;;
  stop)
    # 终止命令行。此处控制台号码与启动一致。
    su $USER -c '/usr/bin/vncserver -kill :1'
    echo "vncserver stopped"
    ;;
  *)
    echo "Usage: /etc/init.d/vncserver {start|stop}"
    exit 1
    ;;
esac
exit 0
```

## FTP

`sudo apt-get install vsftpd`

`sudo service vsftpd start`

[pure-ftpd](https://www.raspberrypi.org/documentation/remote-access/ftp.md) 这是官方推荐的ftp server

## 配置VIM

命令行`sudo apt-get install -y vim`

然后我很鸡贼的从github上拷贝了个星星数很高的vim配置文档上来

## 更换树莓派镜像源

**中国科学技术大学** 

<http://mirrors.ustc.edu.cn/raspbian/raspbian/>

**清华大学** 

<http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/>

**大连东软信息学院源**

<http://mirrors.neusoft.edu.cn/raspbian/raspbian/>

**重庆大学源** 

<http://mirrors.cqu.edu.cn/Raspbian/raspbian/>

**阿里云镜像** 

http://mirrors.aliyun.com/raspbian/raspbian/  

对应中国地图，一般离你家近的网速快些，嫌麻烦直接用阿里就行

二、树莓派修改软件源的方法

编辑/etc/apt/sources.list文件 `sudo vim /etc/apt/sources.list`。

原文内容

```shell
deb http://archive.raspbian.org/raspbian/ stretch main contrib non-free rpi
deb-src http://archive.raspbian.org/raspbian/ stretch main contrib non-free rpi
```

deb: Debian软件包格式的拓展名

deb-src: 软件包源码文件(一般不看源码此项可注释掉)

更改其连接为国内源即可，例

`deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi`

编辑此文件后，命令行`sudo apt-get update`，更新软件列表。

## 安装node.js

安装Node.js 6:

```shell
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Alternatively, for Node.js 8:

```shell
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Optional**: install build tools

To compile and install native addons from npm you may also need to install build tools:

```shell
sudo apt-get install -y build-essential
```

## 安装git 

[git官网](https://git-scm.com/download/linux)

```shell
sudo apt-get install git
```

