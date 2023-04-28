---
title: 在启用 ZSTD 压缩的 ZFS 根分区上安装 ArchLinux 并配置 Windows 双启动
date: 2022-12-15 19:46:43
tags: [ZFSBootMenu, ArchLinux, ZFS]
categories: Linux
image: https://pic.imgdb.cn/item/639b2768b1fccdcd36e13360.jpg
top_img: https://pic.imgdb.cn/item/639b27a7b1fccdcd36e1f24d.jpg
description: 终于填上这个坑了！为了 zfs 折腾了好久！(经过虚拟机实验，完美安装)
---

[[ZFSBootMenu 基础使用]] 和这篇文章的最后部分相同。
本方案已经被实际测试过，完全可用，但是建议在有 Linux 基础的情况下折腾 ZFS，因为没准有某个地方我会忘记修改...

## 总览
分区大概长这样
```TEXT
分区1 300M FAT16 EFI
分区2 128G ZFS   ArchLinux
分区3 128G NTFS  Windows
分区N **G  **FS  DATA // 其他分区
```
然后 ZFS 池里面长这样
```TEXT
zroot
 ├─ROOT
 |  ├─voidlinux // 实际上可以在这个池里安装多个 Linux，本处计划把所有 Linux 根分区数据集放在 zroot/ROOT/ 里
 |  └─archlinux
 └─data
    └─home // 存放 /home
```
ZFS池那块看不懂没关系，只需要跟着文章做就行

## 安装前
准备一个有 ZFS 支持的 archiso，这里我们提供两个方案
### CachyOS LiveCD（推荐）
从 [CachyOS 的 Sourceforge 界面](https://sourceforge.net/projects/cachyos-arch/files/gui-installer/) 下载带 GUI 的 LiveCD（它的 CLI 安装器没 ZFS 支持），然后扔进 U 盘重启进入即可
```SHELL
sudo modprobe zfs # 加载 ZFS 模块

sudo su # 切换到 root 用户

timedatectl set-ntp true  # 同步时间

vim /etc/pacman.d/mirrorlist # 改镜像站
# 开头添加
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
```
### archiso-zfs
此处使用 [eoli3n 的 archiso-zfs](https://github.com/eoli3n/archiso-zfs) 项目

首先下载一个[官方的 ArchLinux LiveCD](https://archlinux.org/download/)，然后重启进入
#### 联网
```SHELL
iwctl # 进入 iwctl 命令行界面
```
下面的命令在 `iwctl` 中输入
```SHELL
device list # 列出可用设备
# 假设上面列出的设备是 wlan0
station wlan0 scan
station wlan0 connect SSID # 连接名为 SSID 的网络
exit     
```
验证联网
```SHELL
ping www.baidu.com
timedatectl set-ntp true  # 同步时间
```
#### 加载 ZFS 模块
```SHELL
curl -s https://raw.githubusercontent.com/eoli3n/archiso-zfs/master/init | bash -x
```
如果连接 github 有问题，则运行下面的命令替代
```SHELL
curl -s https://ghproxy.com/https://raw.githubusercontent.com/eoli3n/archiso-zfs/master/init | bash -x
```
执行过程因为网络原因会比较慢

#### 改镜像站

```SHELL
vim /etc/pacman.d/mirrorlist
# 开头添加
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
```
至此，环境准备完毕，下面的操作不受 livecd 影响

## 格式化

此处认为你已经分好区了，总体需要三个分区

* ESP（EFI） 分区，300M，打了 ESP 标签（可以用 `cfdisk` 打），此处举例为 `/dev/nvme0n1p1`
* Linux 分区（ZFS池），30+G，此处举例为 `/dev/nvme0n1p2`
* Windows 分区（如果不需要安装 Windows 就不用建），40+G，此处举例为 `/dev/nvme0n1p3`

#### 创建 ZFS 池

`ashift = 12` 代表 4096 字节扇区大小，9 代表 512 扇区大小，13 代表 8192 字节扇区大小，在 4096/8192 扇区大小的 SSD 上设置 9 会导致性能损失，在 512 字节扇区大小的硬盘上设置 12/13会导致容量损失，扇区大小可以通过 fdisk -l（不准）或者 diskgenius 这类工具查看  
`dedup=on` 为去重功能，可能会占用较大 RAM，低配机子可以把这行删掉  
`compression=zstd` 为压缩功能，zstd 目前来看压缩率和性能损失比较平衡，在意性能可以改为快速压缩算法 lz4（gzip 压缩率和性能都比不上 zstd）  
其他照做即可
```SHELL
zpool create -f -o ashift=12           \
             -O acltype=posixacl       \
             -O relatime=on            \
             -O xattr=sa               \
             -O dnodesize=legacy       \
             -O normalization=formD    \
             -O mountpoint=none        \
             -O canmount=off           \
             -O devices=off            \
             -O compression=zstd       \
             -O dedup=on               \
             -R /mnt                   \
             zroot /dev/nvme0n1p2             
```
创建数据集
```SHELL
zfs create -o mountpoint=none zroot/data
zfs create -o mountpoint=none zroot/ROOT
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/archlinux
zfs create -o mountpoint=/home zroot/data/home
```
测试zpool是否能够导入导出
```SHELL
zpool export zroot
zpool import zroot -R /mnt  
```
挂载zpool
```SHELL
zfs mount zroot/ROOT/archlinux
zfs mount -a
```
设置启动数据集
```SHELL
zpool set bootfs=zroot/ROOT/archlinux zroot
```
设置zpool缓存
```SHELL
zpool set cachefile=/etc/zfs/zpool.cache zroot
mkdir /mnt/etc
mkdir /mnt/etc/zfs
cp /etc/zfs/zpool.cache /mnt/etc/zfs/zpool.cache
```
查看是否有挂载上
```SHELL
df -h
```
输出
```TEXT
zroot/ROOT/archlinux   30G  128K   30G   1% /mnt
```

#### 格式化 EFI 分区

注意此处挂载点不可以设置成 `/boot`！[^1]
```SHELL
mkfs.vfat /dev/nvme0n1p1
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi 
```

## 安装

这部分需要你在 `/etc/pacman.conf` 里添加 [archzfs](https://github.com/archzfs/archzfs/wiki) 源或者按照[一刀斩的博客](https://blog.yidaozhan.top/2022/08/11/arch-linux-upgrade-to-x86-64-v3-microarchitecture/)配置 CachyOS 源，此处不再赘述

#### 安装基本软件包

如果是 Intel 的 CPU 就把 `amd-ucode` 换成 `intel-ucode`

此处使用 `zfs-linux` 包有可能会因为版本不相同然后挂掉，所以如果你添加了 CachyOS 的软件源，那么我推荐你使用 CachyOS 的自定义内核（它的内核和 zfs 模块同时编译打包，就不会出现版本不统一的问题）（最主要还是因为安装 `zfs-linux` 还得启用 v3 源），有多种任务调度器可选，比如 BMQ PDS TT 等，还有 LLVM LTO 编译的版本

使用 CachyOS 内核的话直接改包即可，比如我想用 `linux-cachyos-pds`，那么把下面 `linux linux-headers zfs-linux` 换成 `linux-cachyos-pds linux-cachyos-pds-headers linux-cachyos-pds-zfs` 即可

如果你不想用 CachyOS 的内核，也不想因为 zfs 模块和 linux 内核版本不统一而滚挂，那么可以使用 dkms 模块，把 `zfs-linux` 替换为 `zfs-dkms` 即可，这个的缺点是构建 dkms 模块时会风扇狂转（理论上任何内核都可以用这个当 zfs 内核模块）
```SHELL
pacstrap /mnt base linux-firmware linux linux-headers zfs-linux base-devel neovim os-prober amd-ucode openssh wget networkmanager zfs-utils 
```
#### 基础安装
```SHELL
# 配置 /etc/fstab
genfstab -U /mnt >> /mnt/etc/fstab
# 验证 /etc/fstab
cat /mnt/etc/fstab

# chroot 进 ArchLinux
arch-chroot /mnt

# 设置zpool缓存
zpool set cachefile=/etc/zfs/zpool.cache zroot

# 设置时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
date

# 设置语言
echo -e "\nen_US.UTF-8 UTF-8\nzh_CN.UTF-8 UTF-8\nzh_TW.UTF-8 UTF-8" >> /etc/locale.gen

# 应用配置
locale-gen

# 设置默认语言
echo -e "\nLANG=en_US.UTF-8\n" >> /etc/locale.conf

# 设置root密码
passwd

# 添加非 root 用户
pacman -S fish # 如果不想用 fish 可以跳过这一步并把下一步的 fish 字段换成 bash
useradd -m -G wheel -s /bin/fish user # user 替换为你的用户名
passwd user # 设置密码
sed -i "s|#%wheel ALL=(ALL) ALL|%wheel ALL=(ALL) ALL|g" /etc/sudoers # 添加 sudo 权限


# 添加 multilib 和 archlinuxcn 仓库 
arch='$arch'
echo -e "\n[multilib]\nInclude = /etc/pacman.d/mirrorlist\n\n[archlinuxcn]\nServer = https://mirrors.bfsu.edu.cn/archlinuxcn/$arch" >> /etc/pacman.conf
```
#### 配置内核钩子
重点部分！
编辑 `/etc/mkinitcpio.conf`，直接在 `HOOKS=(......)` 里面加上 `zfs` 
```SHELL
nvim /etc/mkinitcpio.conf
HOOKS=(base udev autodetect modconf block zfs filesystems keyboard fsck)
```
编辑完直接借用 ArchLinux 的极为人性化的脚本生成即可
```SHELL
mkinitcpio -P
```
#### 设置 zfs 服务以及网络管理服务
```SHELL
systemctl enable zfs.target
systemctl enable zfs-import-cache
systemctl enable zfs-mount
systemctl enable zfs-import.target
systemctl enable sshd
systemctl enable NetworkManager
```

## 配置 ZFSBootMenu 引导
（需要在 chroot 中完成）  
重点部分！  
详细了解可以看我之前那篇[在 UEFI ArchLinux 下使用 ZFSBootMenu 作为引导器（可双启动）](https://pinghigh.github.io/2022/11/28/2022-11-28-ZFSBootMenu/)  
在这里我们使用预构建好的 EFI 文件~~因为我没玩明白ZFSBootMenu，经常出现换内核再构建就会炸引导的问题，同时不能做到启动时换内核~~

首先下载预构建好的 ZBM efi 引导文件
（访问不了可以把 https://github.com/ 换成 https://ghproxy.com/https://github.com/）
```SHELL
mkdir /efi/EFI
mkdir /efi/EFI/ZBM
wget https://github.com/zbm-dev/zfsbootmenu/releases/download/v2.0.0/zfsbootmenu-release-vmlinuz-x86_64-v2.0.0.EFI -O /efi/EFI/ZBM/zfsbootmenu.efi
```
再进行一个简单的配置
```SHELL
zfs set org.zfsbootmenu:commandline="rw" zroot/ROOT
```
这样就做到了启动（如果没有双启动的需求没必要进行下一步了）
## 配置 rEFInd 以实现与 Windows 双启动
安装 refind
```SHELL
pacman -Sy refind git
refind-install
```
这样重启之后就能双启动了  
但是你还可以美化一下，安装 nord 主题（非必要）
```SHELL
pacman -U https://mirror.cachyos.org/repo/x86_64/cachyos/refind-theme-nord-1.1.0-1-any.pkg.tar.zst
```
## 结束
退出 chroot
```SHELL
exit
```
解挂载
```SHELL
umount /mnt/efi
zfs umount zroot/ROOT/archlinux
zfs umount -a
```
导出 zfs 池
```SHELL
zpool export zroot
```
（导出没成功也没关系，直接重启即可）
重启
```SHELL
reboot
```
然后你的 archlinux 就可以正常使用了

## 后记

还是在年前完成了虚拟机上的测试，能够完美安装，原来写的还是有点低级错误的  
Hyper-V 上安装可能会涉及到 pacman 检测不到架构之类的问题，因此启用 archlinuxcn 源需要你在 `/etc/pacman.conf` 改 `Architecture = auto` 中的 `auto` 为你的架构，如 `x86_64` `x86_64-v3`  
另外不启用 CachyOS-v3 源，只启用 CachyOS 源也可以安装 CachyOS 的优化内核，但是没有 lto 版本  
以及我个人宣布，这是本博客 2022 年的最佳博文  

[^1]: 原因是 ZFSBootMenu 需要从 `/boot` 加载内核，而 ZBM 的 initramfs 启动时大概不会挂载 ESP 分区（我之前这么干寄了