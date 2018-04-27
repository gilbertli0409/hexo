---
title: ArchLinux安装指南
date: 2018-04-27 22:08:09
tags: [ArchLinux, Linux, OS]
---




<font size="5">ArchLinux</font>是一个轻量极的linux发行版，体积短小精悍。不提供图形版的安装界面，所有安装过程都要通过命令行来操作，非常具有可玩性。是用来学习的最佳良伴。

下面来分享如何安装ArchLinux:

# 安装准备

<font size="5">Arch Linux</font> 能在任何内存空间不小于 512MB 的 x86_64 兼容机上运行。用 base 组内的软件包进行的基本安装将占用小于 800MB 的存储空间。由于安装过程中需要从远程存储库获取软件包，机器将需要一个有效的互联网连接。

根据 Category:Getting and installing Arch 中所述，下载并引导安装介质。启动完成后将会自动以 root 身份登录虚拟控制台并进入zsh命令提示符。类似 systemctl(1) 的常规命令都可以用 Tab 自动补全。

如果你想切换至其它的虚拟终端来干点别的事, 例如使用 ELinks 来查看本篇指南，使用 Alt+arrow 快捷键。可以使用 nano，vi 或 vim 编辑配置文件。

键盘布局
控制台键盘布局 默认为us（美式键盘映射）。如果您正在使用非美式键盘布局，通过以下的命令选择相应的键盘映射表：

# loadkeys layout

将layout转换为您的键盘布局, 如fr, uk, dvorak或be-latin1. 这里有国家的二位字母编码表。使用命令ls /usr/share/kbd/keymaps/**/*.map.gz 列出所有可用的键盘布局。

Console fonts 位于 /usr/share/kbd/consolefonts/, 设置方式参考 setfont(8).

# 验证启动模式
如果以在 UEFI 主板上启用 UEFI 模式, Archiso 将会使用 systemd-boot 来启动 Arch Linux。可以列出 efivars 目录以验证启动模式:

> ls /sys/firmware/efi/efivars
如果目录不存在，系统可能以 BIOS 或 CSM 模式启动，详见您的主板手册。

连接到因特网
守护进程 dhcpcd 已被默认启用来探测有线设备, 并会尝试连接。如需验证网络是否正常, 可以使用 ping:

> ping -c 3 archlinux.org

若发现网络不通,利用 systemctl stop dhcpcd@,TAB 停用 dhcpcd 进程，详情请查看 网络配置文档.
对于无线连接,iw(8), wpa_supplicant(8) 和 netctl 等工具已被提供. 详情查看无线网络配置.

systemd-timesyncd 更新系统时间：

> timedatectl set-ntp true
用 timedatectl status 检查服务状态.详情阅读 Time (简体中文).

# 建立硬盘分区
磁盘若被系统识别到，就会被分配为一个块设备，如/dev/sda。识别这些设备，使用lsblk或fdisk。输出中以rom, loop 或 airoot 结尾的可以被忽略。

# fdisk -l
对于一个选定的设备，以下的分区是必须要有的:

一个根分区（挂载在根目录） /.
如果 UEFI 模式被启用,你还需要一个 EFI 系统分区.
Swap 可以在一个独立的分区上设置，也可以直接建立 交换文件.
如需修改分区表,使用 fdisk 或 parted. 查看Partitioning (简体中文)以获得更多详情.

如果需要需要创建多级存储例如 LVM、LUKS 或 RAID，请在此时完成。

格式化分区
当分区配置好了, 这些分区应立即被格式化并使用一个合适的文件系统. 例如，如果你想将/dev/sda1格式化成ext4, 使用这个命令:

# mkfs.ext4 /dev/sda1
详情参见 文件系统 和 swap (简体中文)。

挂载分区
首先将根分区挂载[broken link: invalid section]到 /mnt，例如：

# mount /dev/sda1 /mnt
如果使用多个分区，还需要为其他分区创建目录并挂载它们（/mnt/boot、/mnt/home、……）。

# mkdir /mnt/boot
# mount /dev/sda2 /mnt/boot
genfstab 将会自动检测挂载的文件系统和 swap 分区。

安装
选择镜像
编辑 /etc/pacman.d/mirrorlist，选择您的首选 mirror. 这个 mirror 列表也将通过 pacstrap 被复制并保存在到系统中，所以请确保设置正确。

安装基本系统
执行 pacstrap 脚本，安装 base 组：

# pacstrap /mnt base
这个组并没有包含全部 live 环境中的程序，有些需要额外安装，例如btrfs-progs。packages.both 页面包含了它们的差异。

如果您想通过 AUR (简体中文) 或者 ABS (简体中文) 编译安装软件包,需要装上 base-devel：

# pacstrap -i /mnt base base-devel
使用 -i 选项时会在实际安装前进行确认。此章节会给您安装好最基本的 Arch 系统，其它软件以后会用 pacman (简体中文) 安装得到。第一个 initramfs 会在新系统的启动路径生成和安装，请确保 ==> Image creation successful.

配置系统
Fstab
用以下命令生成 fstab 文件 (用 -U 或 -L 选项设置UUID 或卷标)：

# genfstab -U /mnt >> /mnt/etc/fstab
强烈建议 在执行完以上命令后，后检查一下生成的 /mnt/etc/fstab 文件是否正确。

Chroot
Change root 到新安装的系统：

# arch-chroot /mnt
时区
设置 时区:

# ln -sf /usr/share/zoneinfo/zone/subzone /etc/localtime
例如：

# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
建议设置时间标准 为 UTC，并调整 时间漂移:

# hwclock --systohc --utc
Locale
本地化的程序与库若要本地化文本，都依赖 Locale, 后者明确规定地域、货币、时区日期的格式、字符排列方式和其他本地化标准等等。在下面两个文件设置：locale.gen 与 locale.conf.

/etc/locale.gen是一个仅包含注释文档的文本文件。指定您需要的本地化类型，只需移除对应行前面的注释符号（＃）即可，建议选择帶UTF-8的項：

# nano /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
接着执行locale-gen以生成locale讯息：

# locale-gen
/etc/locale.gen 生成指定的本地化文件，每次 glibc 更新之后也会运行 locale-gen。

创建 locale.conf 并提交您的本地化选项：

Tip: 将系统 locale 设置为en_US.UTF-8，系统的 Log 就会用英文显示，这样更容易问题的判断和处理。用户可以设置自己的 locale，详情参阅Locale#Per user[broken link: invalid section].
# echo LANG=en_US.UTF-8 > /etc/locale.conf
警告: 不推荐在此设置任何中文locale，或导致tty乱码。
另外，如果你需要修改键盘布局[broken link: invalid section], 并想让这个设置持续生效，编辑 vconsole.conf(5)，例如:

/etc/vconsole.conf
KEYMAP=de-latin1
主机名
要设置 hostname，将其添加 到 /etc/hostname, myhostname 是需要的主机名:

# echo myhostname > /etc/hostname
建议添加对应的信息到hosts(5):

/etc/hosts
127.0.0.1	localhost.localdomain	localhost
::1		localhost.localdomain	localhost
127.0.1.1	myhostname.localdomain	myhostname
网络配置
对新安装的系统，需要再次设置网络。具体请参考 Network configuration (简体中文) 和

对于 无线网络配置，安装 软件包 iw, wpa_supplicant，dialog 以及需要的 固件软件包.

Initramfs
如果修改了 mkinitcpio.conf，用以下命令创建一个初始 RAM disk：

# mkinitcpio -p linux
Root 密码
设置 root 密码:

# passwd
安装引导程序
启动加载器页面介绍了可用选项和配置方法。包括 GRUB (简体中文) (BIOS/UEFI), systemd-boot (简体中文) (UEFI) 和 Syslinux (简体中文) (BIOS)等.

Intel CPU 也需要安装 intel-ucode 并根据 Microcode 配置 boot loader.

重启
输入 exit 或按 Ctrl+D 退出 chroot 环境。

可选用 umount -R /mnt 手动卸载被挂载的分区：这有助于发现任何“繁忙”的分区，并通过 fuser(1) 查找原因。

最后，通过执行 reboot 重启系统：systemd 将自动卸载仍然挂载的任何分区。不要忘记移除安装介质，然后使用root帐户登录到新系统。





# 安装并配置 bootloader

这里采用的bootloader是Grub；安装 grub 包，并执行 grub-install 已安装到 MBR：
> pacman -S grub 

> grub-install --target=i386-pc --recheck /dev/sdb

注意：须根据实际分区自行调整 /dev/sdb, 切勿在块设备后附加数字，比如 /dev/sdb1 就不对。
由于我的硬盘上还有另外一个操作系统windows 7，为了检测到该系统并写到grub启动项中，还需要做下面的操作。

> pacman -S os-prober 

> grub-mkconfig -o /boot/grub/grub.cfg
