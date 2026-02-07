# Arch Install
## Install Process

### Initial Setup
#### Keyboard Layout

```
loadkeys us
```

将大写锁定和esc更换
```
vim keys.conf
```

在配置文件中
```
keycode 1 = Caps_Lock
keycode 58 = Escape
``` 
加载配置
```
loadkeys keys.conf
```

### Internet

查看网络设备
```
ip link
```

打开网络设备
```
ip link set <network-device> up
```

重新查看网络设备
```
ip link
```

搜索wifi
```
iwctl wlan0 scan
```

过滤扫描结果，pipe输入到grep（搜索工具），显示ESSID
```
iwlist wlan0 scan | grep ESSID
```

设置互联网连接配置文件
```
wpa_passphrase <wifi-name> <wifi-password> > internet.conf
```

编辑互联网连接配置文件
```
vim internet.conf
```

通过配置文件连接wifi
```
wpa_supplicant -c internet.conf -i wlan0 &
```

<!-- 查看是否有连接
```
ip a
``` -->

```
ping www.bilibili.com
```

分配动态ip地址
```
dhcpcd &
```

<!-- ```
ping www.bilibili.com
```

连接wifi
```
iwctl
station wlan0 connect <wifi-name>
输入密码
```

退出软件
```
exit
``` -->

### Timezone
同步时间
```
timedatectl set-ntp true
```

### Disk Partition
#### Disk Information
查看磁盘信息
```
fdisk -l
```

进入磁盘
```
fdisk /dev/nvme1n1
```



查看磁盘，p代表列出完整路径，f代表详细信息
```
lsblk -pf
```

- 分区：
树状目录结构
1. / 根分区，ext4格式，大小根据个人需求，一般20G以上
2. /boot EFI分区，FAT32格式，大小512M，存放启动相关文件，efi system，和根目录不同，需要一个单独的硬盘分区
3. [swap] 交换分区，大小根据内存大小，一般为内存的1-2倍，与虚拟内存相关，分为交换分区和交换文件两种，交换分区需要单独的磁盘分区，而交换文件则是在根目录下创建一个文件来充当交换空间，这里设置交换文件
   因为RAM为32G，所以设置交换文件为64G


？ 这里还要再看
- 分区工具：
使用cfdisk，这里的磁盘是nvme1n1，nvme代表NVMe协议的固态硬盘，1代表第一个NVMe设备，n1代表第一个分区，要根据之前lsblk -pf结果选择
```
cfdisk /dev/nvme1n1
```
、
1. 启动分区：efi system，1G，
2.  



- 格式化分区
再次查看分区
```
lsblk -pf
```

### Pacman
#### Pacman Config
编辑pacman配置文件
```
vim /etc/pacman.conf
```

在配置文件中

去除Color注释，得到彩色报告
```
Color
```

更改软件源，core extra community
在vim中，按下gf可以跳转到文件路径，编辑mirrorlist文件，把China服务器粘到顶部（？ vim宏操作）

### Arch mount
创建挂载点
```
mount /dev/nvme1n1p2 /mnt
```

看一下mnt目录
```
ls /mnt
```

制作文件夹
```
mkdir /mnt/boot
```

挂载efi分区
```
mount /dev/nvme1n1p1 /mnt/boot
```

查看 mnt目录
```
ls /mnt
```

安装
```
pacstrap /mnt base linux linux-firmware
```

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot
进入新系统环境
```
arch-chroot /mnt
```

### Timezone
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### Locale
编辑locale.gen文件，去除zh_CN.UTF-8的注释
```
exit
```

```
vim /mnt/etc/locale.gen
```

找en_US.UTF-8和zh_CN.UTF-8，去掉注释

回到系统，生成本地化内容
```
arch-chroot /mnt
locale-gen
exit
```

编辑locale.conf文件，设置系统语言环境
```
vim /mnt/etc/locale.conf
```
在文件中添加
```
LANG=en_US.UTF-8
LANG=zh_CN.UTF-8
```

### Hostname
编辑hostname文件，设置主机名
```
vim /mnt/etc/hostname
```

在文件中添加
```
WZM
```

编辑hosts文件，设置主机名和IP地址的映射关系
```
vim /mnt/etc/hosts
```

在文件中添加
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	WZM.localdomain	WZM
```

### Password
```
arch-chroot /mnt
```

设置root用户密码
```
passwd
```
输入新密码，确认新密码

### Bootloader
安装grub引导程序
```
pacman -S grub efibootmgr intel-ucode os-prober
```

创建系统引导文件夹
```
mkdir /boot/grub
```

安装grub引导程序到efi分区
```
grub-mkconfig > /boot/grub/grub.cfg
```

确定系统类型
```
uname -m
```

安装grub引导程序到efi分区
```
grub-install --target=x86_64-efi --efi-directory=/boot 
```

### Internet
安装网络管理工具
```
pacman -S wpa_supplicant dhcpcd
```

重启
```
exit
```

```
killall wpa_supplicant dhcpcd
```

重启系统
```
reboot
```
拔掉usb，进入新系统

## Foundmental Setup

### Foundamental Software
更新系统
```
pacman -Syu
```

安装基础软件，vim是文本编辑器，man是帮助文档工具
```
pacman -S vim man
```

```
pacman -S base-devel
```
### User

创建新用户
```
useradd -m -G wheel wzm
```

更改用户密码
```
passwd wzm
```

增加用户sudo权限
把vim编辑器链接到vi命令
```
ln -s /usr/bin/vim /usr/bin/vi
```

编辑sudoers文件，增加wheel用户组的sudo权限
```
visudo
```

在文件中找到
```
%wheel ALL=(ALL) ALL
```
并取消注释

推出root用户
```
exit
```

登录新用户
```
wzm
```

测试sudo
```
sudo pacman -Syyu
```

