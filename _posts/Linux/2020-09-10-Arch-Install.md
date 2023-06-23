---
title: Arch Linux安装
categories:
  - Linux
tags:
  - Linux
  - Arch
pin: true
---

Arch 是一个十分干净简洁的 Linux 发行版，日常使用不吃硬件十分流畅。采用滚动更新方式，安装之后更新就行，没有类似重装的升级，个人感觉这种平滑升级很适合类似实验室多人共用机器的场景，随时可以升级，挑个晚上重启下机器就行。Arch 的社区可能是一众 Linux 发行版中最繁荣的。Arch Wiki 基本能解答所有系统相关的问题，里面的内容对于一些 Arch 下游的 Linux 发行版(比如 Manjaro)也有帮助。AUR(Arch User Repository)提供了大量的软件安装脚本，基本上装所有的东西都只需要一行命令。总结起来就是简单且强大。

下文所有折叠的块都是可选步骤，主要包括 ssh 连接，mdadm raid 和 btrfs 文件系统，不需要的话直接跳过。

<details>
  <summary markdown='span'>
  折叠块长这样
  </summary>

折叠起来的都是可选步骤，不需要直接跳过

</details>

# 硬件需求

- CPU：x86 架构，绝大多数电脑都是。ARM 架构有专门的版本
- RAM：512M 以上
- 硬盘：2G 以上
- 网：可以 wifi，插网线更简单
- U 盘：做虚拟机不需要。8G 肯定够用，镜像不到 1G。读写速度主要影响做启动盘的时间，安装过程中感觉联网下载比较多，从 U 盘中读写比较少

# iso

[Arch 官方镜像列表](https://archlinux.org/download/)里有所有的 iso 下载地址，国内从[清华源](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/)下一般比较快。文件名是 `archlinux-年月日-x86_64.iso`，蓝色那行

![image](https://user-images.githubusercontent.com/29757093/171315041-335255a7-8c44-4b4d-86f8-c6f03b5ba055.png)

做虚拟机不需要做启动盘，直接用 iso 即可。在物理机器上安装的话[Etcher](https://www.balena.io/etcher/)非常好用。使用简单，大都一次成功。选盘的时候注意别选错了！！！否则可能把正在用的盘格掉。Etcher 一般会把不像 U 盘的设备折叠起来，U 盘一般会是最小的一个盘。手上有之前做的启动盘也建议做一个新的。Arch 更新比较频繁，用旧的启动盘可能会出一些软件兼容的小问题，最新的镜像可以避免麻烦。

# 开始安装

插入做好的启动盘，重启电脑选择 U 盘作为启动媒体，一般是按住 F1，F2, F5，F10，F12 中的一个键同时按电源键。启动后选择 Arch Linux install medium。

![image](https://user-images.githubusercontent.com/29757093/139161890-baedb7c9-3080-4c44-bd33-9d514bbe4ce3.png)

之后会进到一个命令行，开头的提示符是 root@archiso，如下

![image](https://user-images.githubusercontent.com/29757093/139161941-9e1e6abc-4e50-4797-b15d-8216001e2866.png)

<!-- todo live 字体 terminus font

```shell
setfont ter-132n
``` -->

# 准备步骤

## 键位

国内的键盘一般都不需要改键位，如果需要可以参考[官方教程](https://wiki.archlinux.org/title/installation_guide#Set_the_console_keyboard_layout)

```shell
ls /usr/share/kbd/keymaps/**/*.map.gz # 查看所有键盘布局
loadkeys [键盘布局名] # 不带 .map.gz
```

## 联网

安装过程中需要联网下软件。可以 ping 一个网站检查网络连接

```shell
ping baidu.com
# ctrl c 停止
```

如果看到 Name or service not know 之类的报错就是目前没网。

![image](https://user-images.githubusercontent.com/29757093/139162328-17fe7acf-ddf2-4c69-a537-e9afaba1e19d.png)

直接插网线最简单，不需要进行额外操作。连 wifi Arch 推荐用[iwctl](https://wiki.archlinux.org/title/Iwd#Connect_to_a_network)。

```shell
iwctl device list # 列出所有网络接口
ip link set [网络接口] up # 我的叫 wlan0。先打开硬件，关闭硬件是down
iwctl # 进入iwctl
device list # 列出所有网络接口，一般会有一个lo是环回的，不是这个。需要用的设备大概叫wlan0
station [网络接口] scan
station [网络接口] get-networks # 获取所有wifi
station [网络接口] connect [wifi name] # 之后输密码
quit # 退出iwctl
```

连接完成重新 ping 一下，这时候应该看到能 ping 通。

![image](https://user-images.githubusercontent.com/29757093/139162642-68f2027f-d4cb-4e5f-9689-c36443c51323.png)

连接 8021x 校园网[1](https://unix.stackexchange.com/questions/145366/how-to-connect-to-an-802-1x-wireless-network-via-nmcli/334675) [2](https://www.reddit.com/r/archlinux/comments/pb3r0f/cannot_connect_to_college_wifi_using/)

## ssh

<details>
  <summary markdown='span'>
  可选步骤。用另一台电脑ssh到正在装的电脑上可以复制命令，方便一点
  </summary>

启动盘的 live 系统不能复制，一些命令手打比较麻烦。可以考虑用另一台机器 ssh 到要安装的机器上，方便一点。

```shell
# 最新的arch iso已经带openssh了，不需要装
# pacman -Syy openssh # 安装ssh
systemctl start sshd # 启动ssh服务
passwd root # 给root设置密码
ip a # 查看机器ip

# 在另一台机器上
ssh root@[上面ip a看到的ip]
```

</details>

## 硬盘分区

硬盘上创建了文件系统才能用。首先检查电脑是不是用了 uefi

```shell
ls /sys/firmware/efi/efivars
```

如果说没有这个路径那就是没用 uefi，下面分区的时候跳过 uefi 分区的部分。如果像下图一样出了一堆文件就是用了 uefi，需要做一个 uefi 分区。

![image](https://user-images.githubusercontent.com/29757093/155045718-cd58db21-29a2-404e-8d9c-6da3589d608a.png)

除了 uefi 分区至少还需要一个 root 分区，一般还会做一个 swap。一些特殊用途的系统比如服务器可能/srv 或者/var 下会存巨多的文件，这样可以给这个路径单开一个分区放到一个合适的盘上单独管理。做分区的过程都很类似，这个教程做三个分区：uefi，root 和 swap。

```shell
lsblk # 查看机器硬盘情况
```

![image](https://user-images.githubusercontent.com/29757093/153557189-0d2d5652-8e60-46ba-b131-30a88af07957.png)

这里 sda 是 U 盘启动盘，nvme0n1 和 nvme1n1 是 ssd。

进入 gdisk 开始创建分区

```shell
gdisk /dev/[盘号] # 注意这块就写到盘，不要写到分区p1p2这种的
```

![image](https://user-images.githubusercontent.com/29757093/153557435-8e429654-fb00-438a-a823-20fa5acae7cb.png)

```shell
d # 之后打一个数字分区号删除它，多次执行直到提示没有分区
```

![image](https://user-images.githubusercontent.com/29757093/153557471-6bd00602-e2d6-411b-8080-bf5704277e4b.png)

首先做 efi 分区，如果上面没有 uefi 就跳过这段，直接创建下面的主分区

```shell
n # 创建分区
# 回车，默认分区号
# 回车，默认起始位置
+512M # 大小512M
ef00 # 修改分区类型为efi分区
```

![image](https://user-images.githubusercontent.com/29757093/153557577-b1c4017e-2ee4-4de0-bbaf-d70669d8b47f.png)

创建主分区

```shell
n # 创建新分区
# 回车，默认分区号
# 回车，默认起始位置
-8G # 这个8G就是给swap留下的大小，一般swap跟内存大小相同。如果有两块盘可以都做一个swap分区，分别内存一半大小
# 回车，分区类型默认就是Linux文件系统
```

![image](https://user-images.githubusercontent.com/29757093/153557905-270aafcc-1db9-4c6e-9831-3f16b2303940.png)

创建 swap

```shell
n
# 回车，默认分区号
# 回车，默认起始位置
# 回车，直接占满剩下的容量
8200 # Linux swap
```

![image](https://user-images.githubusercontent.com/29757093/153557958-ae7e3ef2-4779-41fb-9842-fc2018b15c25.png)

分区创建完成，看一下目前磁盘情况，应该有三个分区，类型分别是 EFI，Linux 和 Linux swap。

```shell
p
```

![image](https://user-images.githubusercontent.com/29757093/153558007-fab52bfa-32a7-40b5-b5a1-850bbf366157.png)

如果没问题就把修改写入磁盘

```shell
w # 确认无误后写入
Y # 确认
lsblk # 再次查看分区情况
```

![image](https://user-images.githubusercontent.com/29757093/153558144-8da25103-5029-4eac-973d-b8143aa35641.png)

之后需要在分区上创建文件系统

```shell
# mkfs.vfat /dev/[uefi分区] # 没有uefi分区跳过这行
mkfs.fat -F32 /dev/[uefi分区] # 没有uefi分区跳过这行
mkswap /dev/[swap分区]
swapon /dev/[swap分区]
# swapon 也可以写多个swap分区，比如 swapon /dev/nvme0n1p2 /dev/nvme1n1p3，这样多个swap分区应该会像raid 0一样做stripping加快速度
```

主分区文件系统有三种选择，绝大多数情况下最简单的 ext4 是最合适的，跑下面这一行之后直接到[安装 Arch](#安装Arch)一节就可以。

```shell
mkfs.ext4 /dev/[主分区]
```

如果想提升一点读写速度可以做[软件 raid](#raid)，如果想要 snapshot 功能可以用[btrfs](#btrfs)文件系统。

## Raid

<details> <summary markdown='span'>可选步骤。软件Raid可以提升一些读写速度</summary>
之前在搜教程的时候看到一个Arch + Raid 0经验贴下面的的[评论](https://forum.level1techs.com/t/arch-linux-install-with-2-nvmes-in-raid-0/147268/2)，笑了一下午。必须放在这 /笑哭

![image](https://user-images.githubusercontent.com/29757093/152065902-cb1b40ca-3005-48c9-9415-0cd39a9e38f4.png)

个人在存储技术方面可以说没有任何经验，下面的背景部分只是记录一些自己在调研过程中的理解和想法。

基础的软件 Raid 大概有两条路线，一种是基本的 ext4 文件系统+软件 Raid+逻辑卷管理，另一种是直接用 zfs，btfs 这种带 Raid 支持的文件系统。就我的在 2 块 SSD 的笔记本上加快读写速度的场景下似乎第一种更合适。

网上冲浪的过程中感觉 btfs 风评差很多，在 Raid 这种追求持续在线和稳定的用例下开发团队似乎并不重视软件质量。评价用 btfs 不是会不会丢数据的问题，只是什么时候丢数据。zfs 功能很多，除了自带 Raid 以外 copy on write 带来的灵活创建备份点和快速格式化很大的存储听起来很有用。不过不是做 NAS 盘比较小 ext4 还能应付，我也没有备份系统的习惯最多备份文件，所以功能上二者没有决定性的差别。而且因为开源协议的问题 zfs 不能合进 linux 内核，自己做 iso 比较耗时间。综上选了第一条路线。

Raid 的实现分为三大类：硬件 Raid，软件 Raid 和主板 Raid(fakeraid)。硬 Raid 用专门的芯片和独立于 uefi 的固件实现，性能最好，对主板固件和操作系统来说一个硬 Raid 的阵列就是一块盘。软件 Raid 和 fakeraid 都依赖 CPU 进行运算，理论上性能差别不是很大，一般没有硬件 Raid 就会采用软件 Raid，fakeraid 没人推荐。Raid 有多种级别，具体可以参考[这篇](https://www.prepressure.com/library/technology/raid)。笔记本大概就是 Raid 0 和 Raid 1,主要着眼提速，Raid 0 用一倍的故障率换速度，Raid 1 用一半的空间换容错。我做的是 Raid 0 不过其他级别流程上区别不大。

做 Raid 要分区，因为就算一个厂商一个型号的盘大小也会有一些不同。如果阵列有容错，换盘的时候软件 Raid 要求换进来的盘和之前的盘大小完全相同。如果直接用整块盘做 Raid，之前的盘还偏大就很不巧了。

做 Arch 至少要两个，一般有三个分区，分别是 uefi，主分区和可选的 swap。软件 Raid 依赖操作系统，而 uefi 分区在进操作系统之前就要用，所以这个分区要么不 Raid 要么 Raid 1。linux 支持多个 swap，如果有两个 swap 在两个盘上默认就会用类似 Raid 0 的方式读写，所以 swap 也没必要放进 Raid。这样就只需要给系统分区做 Raid。如果继续对系统分区细分，只想对存数据的部分进行 Raid 安装过程和不配 Raid 完全相同。系统做完之后配 Raid 挂载就可以。

废话结束，综上我的两块 SSD Raid 0 分区布局如下

- SSD 1
  - UEFI 分区：512M
  - 系统分区：剩下的空间取个整数
  - swap 分区： 1/2 swap 大小
- SSD 2
  - 系统分区：和 SSD 1 上的系统分区一样大
  - swap 分区： 1/2 swap 大小

格盘的细节参考下一节，记一下 mdadm 的内容

```shell
# 删除旧记录
mdadm --misc --zero-superblock /dev/drive

# 创建Raid 0
mdadm --create /dev/md0 --level=stripe --raid-devices=2 /dev/drive[1-2]
# 或者盘分开写
mdadm --create /dev/md0 --level=stripe --raid-devices=2 /dev/drive1 /dev/drive2

# 查看mdadm运行状态，raid细节
cat /proc/mdstat
mdadm -E /dev/drive[1-2]
mdadm --detail /dev/md0

```

Raid 做完之后如下步骤，替换成自己的设备文件

1. 在分区 1 上 `mkfs.fat -F32 /dev/[uefi分区]`
2. 在分区 3 和 5 上分别 `mkswap /dev/[swap分区]`
3. `swapon /dev/[SSD 1上的swap分区] /dev/[SSD 2上的swap分区]`
4. 在 md0 里创建一个 ext4 分区，`mkfs.ext4 /dev/raid 0里的分区`，后面 mount 的时候也 mount 这个分区

Raid 部分已经跑起来了，在挂载主分区之后，arch-chroot 之前和之后还有几行命令需要执行。

</details>

## btrfs

<details>
  <summary markdown='span'>
  可选步骤。btrfs 提供 raid 和快照功能，做起来比较复杂不推荐新手用
  </summary>

btrfs 和 ext4 一样是一个文件系统，负责管理存在盘上的文件。btrfs 的主要竞品应该是 zfs，这两个文件系统融合了传统解决方案 ext4+软件 Raid+逻辑卷管理的大部分功能。主要的优点是提供快速和不怎么占额外空间的快照，此外可以做跨盘的文件系统并开启 raid。总体上来说速度更快，功能更多。个人在用 btrfs 的过程中还没遇到什么大问题，但是感觉和 zfs 相比 btrfs 风评差很多，据说主要是因为 bug 比较多可能丢数据，而且开发者社区貌似赶不上 zfs。但是 zfs 因为开源协议冲突不能合入 linux 内核，安装过程比 btrfs 麻烦一些。研究明白 btrfs 之后这篇会加做 zfs 的过程。

上一节已经做好了 root，uefi 和 swap 三个分区，这里从 mkfs.btrfs 开始。类似逻辑卷，btrfs 的文件系统可以跨盘，详情参考[btrfs wiki](https://btrfs.wiki.kernel.org/index.php/Using_Btrfs_with_Multiple_Devices)，下面是官方给的一些常用例子

```shell
# Create a filesystem across four drives (metadata mirrored, linear data allocation)
mkfs.btrfs -d single /dev/sdb /dev/sdc /dev/sdd /dev/sde # 简单跨盘，不raid

# Stripe the data without mirroring, metadata are mirrored
mkfs.btrfs -d raid0 /dev/sdb /dev/sdc # 相当于raid0

# Use raid10 for both data and metadata
mkfs.btrfs -m raid10 -d raid10 /dev/sdb /dev/sdc /dev/sdd /dev/sde

# Don't duplicate metadata on a single drive (default on single SSDs)
mkfs.btrfs -m single /dev/sdb
```

比如我的两块 ssd raid

```shell
mkfs.btrfs -f -d raid0 /dev/nvme0n1p1 /dev/nvme1n1p2
```

![image](https://user-images.githubusercontent.com/29757093/153563174-7aba4e07-390e-451e-b607-33e1355a97c9.png)

做好 btrfs 文件系统之后就是挂载和创建子卷。这里的子卷结构是[Arch Wiki](https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout)的推荐加上一点个人理解。

直接用 btrfs 创建和管理快照比较麻烦，openSUSE 社区的 snapper 比较推荐。snapper 在快照的时候不会包含子卷里挂的子卷。根目录必须是一个子卷，所以如果其他子卷挂在根目录下，快照根目录子卷只会包含根目录里不是子卷的目录，这些根目录下的子卷需要分别创建快照，比较麻烦。综上用的子卷结构是整个系统基本上都直接扔到根目录的子卷里，给不希望回滚的目录单独创建子卷挂上去。比如数据库的数据一般在/var 目录下，ftp 和 web 的文件一般在/srv 下，显卡驱动一类第三方软件一般在/opt 下，临时文件一般在/tmp 下。这样快照+回滚的时候不会影响到这些子卷里的内容。

特殊一点的是快照和 swap，快照需要单独用一个子卷，否则会快照到快照。btrfs 不支持快照 swap 文件，所以单独开一个子卷放 swap 文件，不一定用得上，但是反正他也不占地方。之后把 home 目录单独做子卷拿出去。整体结构如 Arch wiki 的这个图

![image](https://user-images.githubusercontent.com/29757093/155220812-a00bbf82-db57-4229-86aa-509760b8f77e.png)

<!-- todo 研究 snapper alternative -->

开干

```shell
part_name=[btrfs的任意一个分区名字]
mount /dev/${part_name} /mnt # ${part_name} 是bash变量，前面写的 [btrfs的任意一个分区名字] 会被放到 ${part_name}
btrfs subvolume create /mnt/@
btrfs su cr /mnt/@home
btrfs su cr /mnt/@swap # swap文件推荐放进单独的子卷
btrfs su cr /mnt/@.snapshots
```

挂载子卷，一些参数的解释

- noatime： 不写 accesstime，速度更快
- compress： zlib 最慢，压缩最率高；lzo 最快，压缩率最低；zstd 和 zlib 兼容，压缩率和速度适中，可以调压缩等级
- space_cache=v2：将文件系统中空闲的 block 地址放在缓存里，创建新文件的时候可以立即开始往里写
- nodatacow：禁用 cow，新数据直接覆盖
- ssd：开启针对 ssd 的优化
- discard=async：大概是异步进行 trim，可以降低读延迟

```shell
umount /mnt
mount -o noatime,compress=lzo,space_cache=v2,ssd,discard=async,subvol=@ /dev/${part_name} /mnt
mkdir /mnt/{home,swap,.snapshots}
mount -o noatime,compress=lzo,space_cache=v2,ssd,discard=async,subvol=@home /dev/${part_name} /mnt/home
mount -o noatime,compress=lzo,space_cache=v2,ssd,discard=async,subvol=@.snapshots /dev/${part_name} /mnt/.snapshots
mount -o nodatacow,ssd,subvol=@swap /dev/${part_name} /mnt/swap
```

</details>

<!-- prettier-ignore -->
## 安装Arch

安装需要联网下载，选一个快的镜像可以节省很多时间

```shell
pacman -Syy # 更新pacman数据库
pacman -S reflector
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bk # 备份镜像列表
reflector -c "CN" -l 20 -n 10 --sort rate --save /etc/pacman.d/mirrorlist
```

todo 检查
bash-completion

![image](https://user-images.githubusercontent.com/29757093/155212372-31f34bc2-dfac-49f4-9c06-e36867f9082c.png)

```shell
mount /dev/[主分区] /mnt # 挂载主分区
pacstrap /mnt base linux linux-firmware linux-headers vim base-devel opendoas grub efibootmgr git
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

<details>
  <summary markdown='span'>
  Raid
  </summary>

把 mdadm 配置写入文件

```shell
mdadm --detail --scan >> /mnt/etc/mdadm.conf
```

</details>

切换到新装好的系统

```shell
arch-chroot /mnt
```

<details>
  <summary markdown='span'>
  btrfs
  </summary>

```shell
pacman -S btrfs-progs grub-btrfs

vim /etc/mkinitcpio.conf
# MODULES=(btrfs)
mkinitcpio -p linux
systemctl enable fstrim.timer
```

</details>

<details>
  <summary markdown='span'>
  Raid
  </summary>

在新系统里装 mdadm，和修改一个配置文件。Raid 所有配置完成，下面正常安装就行。

```shell
pacman -S mdadm

vim /etc/mkinitcpio.conf
HOOKS=(base udev autodetect keyboard modconf block mdadm_udev filesystems fsck) # 在HOOKS这行添加 mdadm_udev

mkinitcpio -p linux
```

</details>

## 校准时间

```shell
# timedatectl list-timezones # 显示所有时区，按q退出
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localetime
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp true # 开启联网时间校准
hwclock --systohc
```

todo
echo "KEYMAP=[键盘布局]" >> /etc/vconsole.conf

## 语言

```shell
vim /etc/locale.gen
# 按 / 进入查找，输入en_US，回车下一条结果
# 找到 en_US.UTF-8 UTF-8 这一行
# 按i编辑，删除前面的 #

# 按两次esc进入命令模式
# 按 / 进入查找，输入zh_CN，回车下一条结果
# 找到 zh_CN.UTF-8 UTF-8 这一行
# 按i编辑，删除前面的 #

# 两下esc，输入:wq保存退出
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

![image](https://user-images.githubusercontent.com/29757093/139164535-05d4f355-2975-4df0-a53d-99a438fee4fa.png)

## 设置网络

```shell
hostname=[随便选一个hostname] # 不建议用自己的名字之类的个人信息，这个至少局域网里是能看到的
echo ${hostname} >> /etc/hostname
cat /etc/hostname
```

```shell
echo "
127.0.0.1 localhost
::1 localhost
127.0.0.1 ${hostname}.localdomail ${hostname}
" >> /etc/hosts
```

## 安装 grub

systemd-boot

uefi 系统

```shell
mkdir /boot/efi
mount /dev/[uefi分区] /boot/efi
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi # 注意那个x是小写的
grub-mkconfig -o /boot/grub/grub.cfg
```

非 uefi 系统

```shell
pacman -S grub
grub-install /dev/[主分区]
grub-mkconfig -o /boot/grub/grub.cfg
```

## 设置密码

```shell
passwd ＃ 设置密码
```

## 网络

前面设置的网络连接只在这次安装过程中生效，还需要给刚装好的系统装联网软件。下面只写基本的连接 wifi 的部分，DSL，移动网络之类的连接可以参考[这篇详细教程](https://linuxhint.com/arch_linux_network_manager/)

```shell
pacman -S wpa_supplicant wireless_tools networkmanager network-manager-applet
pacman -S nm-connection-editor
systemctl enable NetworkManager.service
systemctl disable dhcpd.service # 如果说dhcpd not found也没关系，目标就是把他关了
systemctl enable wpa_supplicant.service
```

这里就可以重启进入只有命令行的系统了，可以选择现在重启看一下前面的步骤是不是做的有问题，下面的步骤在进入系统后做，或者也可以不重启直接继续装。

## 添加用户

一般日常使用不会直接用 root 账户，创建一个用户帐户。

```shell
username=[用户名]
useradd -m ${username}
# 添加一行
echo "permit persist ${username} as root" >> /etc/doas.conf # 允许 用户名 作为root执行，persist是输入一次密码之后一段时间不用再输入
mv /usr/bin/sudo /usr/bin/sudo-bk
ln -s /usr/bin/doas /usr/bin/sudo
passwd ${username} # 设置新用户密码
```

## 桌面

<details>
  <summary markdown='span'>
  Btrfs snapshot
  </summary>

桌面会装很多包，如果用了 btrfs，可以在安装桌面之前做一下 snapshot。详细步骤参考[这里](https://linhandev.github.io/posts/Arch-Apps/#btrfs)

</details>

### xfce4

lightdm-slick-greeter

```shell
pacman -S xorg xfce4 xfce4-goodies lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
systemctl enable lightdm
```

220 多个包

### KDE

```shell
pacman -S xorg plasma plasma-wayland-session kde-applications
systemctl enable sddm.service
```

![image](https://user-images.githubusercontent.com/29757093/155061233-99de26b9-e02c-41e3-ae7d-a7c2050b0723.png)

800 多个包

有时候新安装桌面可能遇到登录循环的情况，开机后正常输入用户名密码，结果回车登录之后又回到输入密码界面。这种情况可能是因为没有 /home/[用户名] 目录或者用户没有这个目录的权限，创建试一下。

```shell
mkdir /home/[用户名]
chown -R [用户名]:[用户名] /home/[用户名]
```

安装完成，重启进入系统

```shell
# 按 ctrl+D 退出 chroot
reboot
```

重启之后应该就能看到一个登陆界面，登陆进去看到桌面就是安装成功了！如果安装过程中有任何问题欢迎在下方留言。有关一些常用软件的安装在[下一篇文章](https://linhandev.github.io/posts/Arch-Apps/)中记录。

(swap btrfs: truncate -s 0 /swap/swapfile; chattr +C /swap/swapfile; btrfs property set /swap/swapfile compression none; dd if=/dev/zero of=/swap/swapfile bs=1G count=2 status=progress; chmod 600 /swap/swapfile; mkswap /swap/swapfile; swapon /swap/swapfile; vim /etc/fstab； /swap/swapfile none swap defaults 0 0 )

参考资料：

[官方安装教程](https://wiki.archlinux.org/title/installation_guide)

[Arch 安装教程 (不带 swap)](https://itsfoss.com/install-arch-linux/)

[安装教程 (带 swap)](https://frontpagelinux.com/tutorials/how-to-install-arch-linux-installation-guide/)

[iwctl 连接 wifi](https://wiki.archlinux.org/title/Iwd)

[mdadm+arch](https://www.serveradminz.com/blog/installation-of-arch-linux-using-software-raid/)

[Arch Raid](https://wiki.archlinux.org/title/RAID)

[Arch Linux BTRFS Install](https://www.youtube.com/watch?v=7ituCCKXmMM&t=1143s&ab_channel=EF-LinuxMadeSimple)
