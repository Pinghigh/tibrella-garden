---
title: 在 UEFI ArchLinux 下使用 ZFSBootMenu 作为引导器（可双启动）
date: 2022-11-28 16:32:35
tags: [ZFSBootMenu, ZFS, ArchLinux]
categories: ArchLinux
image: https://pic.imgdb.cn/item/63a8671208b683016397a5d0.jpg
description: zstd 下 zfs 根目录压缩率能上 2x，而且对于 gzip 来说性能更好，但是显然 grub 引导不了，其他的不好用，所以折腾了 ZBM
---

## 前言

写这篇博客的缘由挺简单的，当时 ClearArch 开发组讨论文件系统应当使用 ZFS 还是 F2FS，由于使用过 F2FS 作为根文件系统（它没有 Windows 下的驱动，透明压缩也不会提供额外的空间），同时查到[企鹅大大的一篇文章](https://qiedd.com/1386.html)，看到近 2x 的压缩率不禁心动，随后走上了 ZFS+zstd 作为根文件系统的折腾之路

企鹅大大使用 SysLinux 引导，我在服务器尝试安装结果无法启动；看到 [CachyOS](https://cachyos.org) 的仓库中有一个打了 ZFS+zstd 支持补丁的 grub，尝试，启动不了（后来询问了 CachyOS 的开发团队得知那个补丁是不起效果的），随后，我遇到了一个完美的解决方式（指 [eoli3n 的使用 ZFSBootMenu 的安装脚本](https://github.com/eoli3n/arch-config/blob/master/scripts/zfs/install/02-install.sh)）（如果想实机全盘单系统可以直接用他的安装脚本）

本文的目的是用 ZFSBootMenu 引导 zstd 压缩的 ZFS 文件系统根目录，同时附带双启动教程

## ZFSBootMenu 介绍

[官方仓库](https://github.com/zbm-dev/zfsbootmenu/) 介绍：

> ZFSBootMenu 是用于 root-on-ZFS 系统的 ZFS 引导加载程序，支持快照和本机全盘加密

其原理比较容易理解，总的来说：
1. 引导 ZFSBootMenu，他是一个 initramfs 映像
2. 找到并导入所有的 ZFS 存储池，然后挂载用户选择的根数据集
3. 用 `kexec`[^1] 将系统内核、initramfs 映像加载到内存中
4. 卸载所有 ZFS 数据集
5. 启动最终内核

总之，它能启动以 ZFS 为根目录的 Linux，速度不慢，适用环境广，是作为本文环境下引导器的不二之选

## 准备

确保你的电脑为 UEFI 启动，且已经分区完成，建立了根分区相应的子数据集

```SHELL
[ -d /sys/firmware/efi] && echo UEFI || echo BIOS
```

输入该命令，若输出 `UEFI`，则可以进行下一步，否则退出本教程。

### 挂载 ESP 分区

ESP 分区一般是磁盘头部 300M 左右的 FAT16/32 分区，同时有 EFI System 标记，可以通过 `fdisk -l` 查看各个分区的标记  
此处挂载点绝对不可以是 `/boot`，一般来说有两个其他选择：`/boot/efi` 和 `/efi`，我在这里选择了 `/efi`  
例如我的 ESP 分区是 /dev/nvme0n1p1
则
```SHELL
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

### 设置 ZFS Hook

```SHELL
# 先 arch-chroot
arch-chroot /mnt

# 进入后编辑 /etc/mkinitcpio 加入 zfs 支持，在 HOOKS=(.......) 中加入 zfs
vim /etc/mkinitcpio
# 比如正常安装的 arch 修改后应该长这样，确保其中有 zfs 即可
HOOKS=(base udev autodetect modconf block zfs filesystems keyboard fsck)

# 编辑后加载
mkinitcpio -P
```

## 安装 ZFSBootMenu

在这里我们使用预构建好的 EFI 文件~~因为我没玩明白ZFSBootMenu，经常出现换内核再构建就会炸引导的问题，同时不能做到启动时换内核~~

```SHELL
# /efi 换成你的 ESP 挂载目录
mkdir /efi/EFI
mkdir /efi/EFI/ZBM
wget https://github.com/zbm-dev/zfsbootmenu/releases/download/v2.0.0/zfsbootmenu-release-vmlinuz-x86_64-v2.0.0.EFI -O /efi/EFI/ZBM/zfsbootmenu.efi
# 访问不了可以把 https://github.com/ 换成 https://ghproxy.com/https://github.com/

# 配置
# 此处假设你的根目录的数据集为 zroot/ROOT/arch
# 这样可以引导 zroot/ROOT 下的任意子数据集，比如 zroot/ROOT/arch 上安装 arch，zroot/ROOT/void 上安装 voidlinux，这就实现了 Linux 的双启动
zfs set org.zfsbootmenu:commandline="rw" zroot/ROOT
```

## 配置 rEFInd

rEFInd 能够每次开机都搜索 EFI 分区下的 efi 文件，这样就不用更新引导文件了，同时插启动盘的时候也不需要进 bios 里调启动顺序，rEFInd 会自动搜索到
```SHELL
# 假设当前还在 chroot 里
pacman -Sy refind
refind-install
```
非常简单，这样就实现了 Windows/macOS(HFS+) 与 Linux on ZFS 的双启动
```SHELL
# 安装 nord 主题
pacman -U https://mirror.cachyos.org/repo/x86_64/cachyos/refind-theme-nord-1.1.0-1-any.pkg.tar.zst
```

## TODO
实验并整理整个安装流程

[^1]: `kexec` 是 Linux 内核的一种机制，它允许从当前运行的内核启动新内核。`kexec` 会跳过由系统固件（BIOS 或 UEFI）执行的引导加载程序阶段和硬件初始化阶段，直接将新内核加载到主内存并立即开始执行。