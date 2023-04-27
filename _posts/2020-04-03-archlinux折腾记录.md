---
title:  "archlinux折腾记录"
author: 船长
date:   2020-04-03 20:44:36 +0530
categories: Linux Archlinux 折腾
layout: post
---

### 具体参考 archlinux wiki

### 笔记本显卡驱动安装 
* [NVIDIA Optimus (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/NVIDIA_Optimus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
* 使用`NVIDIA Optimus`最简单，在要使用32位的程序的时候，比如`steam` `deepin TIM` 要先安装32位显卡驱动程序`lib32-nvidia-utils`
* 查看NVIDIA 显卡的 PCI 地址。PCI 地址是提到 NVIDIA 的输出行的前7个字符，看起来像 01:00.0. 在 xorg.conf 中，需转换为 #:#:# 格式；例如 01:00.0 应该写成 1:0:0
```bash
lspci | grep -E "VGA|3D"
```
* 修改`/etc/X11/xorg.conf`
```bash
Section "Module"
	Load "modesetting"
EndSection

Section "Device"
	Identifier "nvidia"
	Driver "nvidia"
	BusID "2:0:0"
	Option "AllowEmptyInitialConfiguration"
EndSection
```
* 在`~/.xinitrc`的最开头添加一下内容,方便调用`startx`启动桌面
```bash
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
* `lightdm` `gdm` 都需要单独设置显卡

### lightdm
```bash
sudo pacman -S lightdm
sudo pacman -S lightdm-gtk-greeter
sudo pacman -S lightdm-gtk-greeter-settings
```

### bash美化
* 系统信息显示工具 `neofetch` `screenfetch`

### ntfs 挂载
* 安装`ntfs-3g`

### deb包安装
* [AUR (en) - debtap](https://aur.archlinux.org/packages/debtap/)
* [AUR (en) - Page Not Found](https://aur.archlinux.org/debtap.git)

### 运行安卓
* [下载 \| 北京麟卓信息科技有限公司](https://www.linzhuotech.com/index.php/home/index/down.html)
* [GitHub - Genymobile/scrcpy: Display and control your Android device](https://github.com/Genymobile/scrcpy)

### 安装rambox 
* 主要为安装微信,从aur仓库里面安装

### gnome快捷键
* 打开文件夹 `nautilus --new-window` `Super+E`
* 打开终端 `gnome-terminal` `Ctrl+Alt+T`

### 安装teamviewer
* 启动后台服务`sudo systemctl enable teamviewerd`
* `sudo systemctl start teamviewerd`

### 安装keepass
* `sudo pacman -S keepass`
* import from url `https://dav.jianguoyun.com/dav/pass/keepass.kdbx`
* ios版本: ikeepass 奇密
* 需要下载中文包[Translations - KeePass](https://keepass.info/translations.html),解压
* 新建`/usr/share/keepass／Languages`文件夹，将`Chinese_Simplified.lngx`文件拷贝过来
* `Tools/Options/Interface`取消到`Force usage of system font(Unix Only)`
* `Tools/Options/Interface` `Select List Font`选择中文

### 安装输入
* 安装输入`ibus`
* 安装`ibus-libpinyin`,不要安装`lbus-pinyin`没有维护了

### 安装samba,解析主机名称
* Samba 通过 Microsoft's NetBIOS 提供主机名称解析。这只需要安装 samba 并启用 nmbd.service 服务。运行 Windows、 OS X、或者运行着 nmbd 的 Linux，将能找到你的机器。
* 编写配置文件`/etc/samba/smb.conf`,`netbios name`就是主机名称
```bash
workgroup = WORKGROUP
netbios name = archlinux
```
* 启动`smb`服务
```bash
sudo systemctl enable smb
```
* 启动`nmb`服务
```bash
sudo systemctl enable nmb
```

### 安装农历
* [AUR (en) - ccal](https://aur.archlinux.org/packages/ccal/)
* `ccal -u`用utf-8显示农历
* `ccal -u -月 -年` `ccal -u 9 2019`

### 有意思的火车命令
* `sudo pacman -S sl`

### 类似mspaint的轻量级绘画应用
`gpaint` ,需要在`aur`中安装