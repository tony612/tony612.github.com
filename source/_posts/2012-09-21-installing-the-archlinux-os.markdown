---
layout: post
title: "Installing the archlinux OS"
date: 2012-09-21 01:43
comments: true
categories: [archlinux, linux, install]
---
参考自：老手看[这个](https://wiki.archlinux.org/index.php/Installation_Guide), [新手教程](https://wiki.archlinux.org/index.php/Beginners%27_Guide)

这个guides还是很细致的，不过就怕遇到一些莫名的问题就不好了。以及一些不符合自己电脑的东西

 
虽然在忙个rails的项目，不过还是抽了个空来搞一下一直想玩的arch，也当是从rails里切换出来，放松一下吧。

安装archlinux，版本是2012.09.07（好新～)

PS:其实还是有一段惨痛的波折的，本来是想在已有windows xp和ubuntu双系统的情况下，把ubuntu换成arch的，不过因为有一些原因导致安装不成功，最后不得不砍掉window，来拥抱arch了，不过也好，其实也都不用windows

###下载

首先当然是下载了，可以从这里下载，官方推荐用BitTorrent下载，但我这里几乎不能下载，就找到了国内的镜像，挨个点开，比较了一下，当时属北交的最快，十几分钟就搞定了。 当然，下载完了最好检查一下文件的完整性，用sha1sum archlinux-2012.XX.XX-XXXX.iso或md5sum archlinux-2012.XX.XX-XXXX.iso，会得到一个类似于这样的码`fa01ac8b4186c17cf7725e24c62c0e1891fcacc0 archlinux-2012.09.07-dual.iso`，然后和刚刚下载镜像的那个网站中的sha1sum或md5sum文件中的内容做对比看是不是一样的。

###U盘启动

然后就是做启动咯，选U盘的理由。。不解释了。 ：） 把U盘插上后，用lsblk来检测看时候U盘已经没有挂载，如下：

```

NAME MAJ:MIN RM SIZE RO MOUNTPOINT sda 8:0 0 465.8G 0 ├─sda1 8:1 0 200M 0 ├─sda2 8:2 0 48.8G 0 ├─sda3 8:3 0 1K 0 ├─sda5 8:5 0 61G 0 ├─sda6 8:6 0 150G 0 ├─sda7 8:7 0 166.8G 0 ├─sda8 8:8 0 14.5G 0 / ├─sda9 8:9 0 991M 0 [SWAP] └─sda10 8:10 0 23.6G 0 /home sr0 11:0 1 1024M 0 sdb 8:16 1 3.7G 0 └─sdb1 8:17 1 3.5G 0 /media/F40B-4DFF

```

其中MOUNTPOINT自然就是挂在点，这里的话sdb1是挂载的，然后因为要用到sdb（U盘所在），所以要保证它是不挂载的，这里貌似就是不挂载的，不过这里我为了安全起见，又把sdb1给取消了挂载，用`umount /dev/sdb1`。 另外说一下，那个树型结构是表示一种依赖关系的。 然后用`dd if=archlinux-2012.09.07-dual.iso of=/dev/sdb`（注意是sdb，不是sdb1，if应该是input file的意思），就能够把镜像写入U盘了，当然这是要覆盖U盘的。其他方法，以及在OS X和Windows上的方法见这里

到这里准备工作就做好了，然后就是正式的安装。

启动安装程序

把U盘插入电脑，开机或重启，在开始的界面按F12或其他的键，改变BIOS，选择USB XXXXX，然后就看到archlinux的字样了（很激动的有木有），选择Boot Arch Linux 然后经过一个系统的启动（有点像windows的PE系统），一些加载中的命令，正常的话就进入archlinux的文字界面，并且默认root登录，到这里还只是进入了系统，不过安装还没结束

###安装

上边一切顺利的话就是archlinux的命令行界面了。。因为archlinux是很简洁主义的，所以要自己去配置安装好多东西。开始吧。 键盘不用搞，略过。。

###网络

安装程序会自动配置有线链接，不过还是先ping一下看是否成功，比如ping -c 3 www.google.com，如果出现ping: unknown host，就是不通，要手动配置。（开始不行，然后准备往下走的时候想起来没插网线，晕～～） 因为懒得去配无线（这里要用无线的话还要另外配置，参考这里,当然即使这里配好了，安完后还是要再配置了无线才能用的，所以就不配咯。。）

###准备硬盘

终于到这比较关键的部门了。。一些重要的数据最好先备份，都懂的。。

这里本来想用那个据说是比MBR优越一些的GPT，见这里

```

Note: If you are not dual booting with Windows, then it is advisable to use GPT instead of MBR. GPT partitioning can only be done with gdisk orparted. Read GPT for the list of advantages.

```

**PS:不过这个就有点坑爹了，我照着这个来，结果最后都弄完了，一直开不了机，等删了windows才发觉，应该是选MBR的吧。不过因为我也没有再在双系统下试验，不知道在装了windows的情况下，选MBR是否可行，应该可以吧 ：）**

于是在尝试了N次之后，就乖乖地单系统了，用cfdisk，比gdisk好用多了。

然后就按照提示，建立不同的分区，注意把`boot`的分区加上Boot的flag，以及把swap设为`Linux swap / Solaris `，最好把分区对应的/dev/sdaX抄下来，后边会用到

说一下分区方案，仅供参考：200M的/boot，20G的/，1G的swap，10G的/var, 剩下有100多G都是/home的

然后确定没问题了就w写入了。

还要按照下边设定文件系统，注意swap的，boot也可以是ext4，不过据说ext2会好一点

```

# mkfs.ext2 /dev/sdaX(boot)
# mkfs.ext4 /dev/sdaX
# mkfs.ext4 /dev/sdaX
Swap：
# mkswap /dev/sdaX
# swapon /dev/sdaX
```

###挂载

linux的常识性东西了，用`mount /dev/sdaX /mnt`之类的去挂载，/mnt为/

boot，home，vara以及其他所有都要用如下方法

```

# mkdir /mnt/boot
# mount /dev/sdaX /mnt/boot
```

###选择镜像

编辑mirrorlist，来使快一点的镜像在上边。推荐用北交（njtu）或中科大（ustc）的。

```

# nano /etc/pacman.d/mirrorlist

```

用`Alt+6`去复制整行，用`Ctrl+U`去粘贴

###安装一些基础的程序

这里要安装一些基础的程序，base 和 base-devel，因为我下边想用vim,就多加了vim，当然i这里需要把网络配好

```

# pacstrap /mnt base base-devel vim

```

###生成fstab

```

# genfstab -p /mnt >> /mnt/etc/fstab
```

###Chroot并配置基本环境

这里就相当于要进入安装到硬盘上的环境进行操作，现在所处的环境还是用USB启动起来的一个内存中的环境（但是USB是拔不得的）

```

# arch-chroot /mnt
```

然后下边进行一下比如键盘，区域，时间的配置，具体参考最上边的链接。

####Install and configure a bootloader

然后是很重要的和启动有关的部分了，能不能启动起来和这个有很大关系。

我是BIOS的，以下也都是BIOS的，而且用了GRUB，因为这个比较熟悉。。

安装GRUB

```

# pacman -S grub-bios
# grub-install --target=i386-pc --recheck /dev/sda
# cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
```

配置GRUB，这里用了自动生成，还可以安装`os-prober`来自动检测系统，双系统的话很有用

```

# pacman -S os-prober
# grub-mkconfig -o /boot/grub/grub.cfg
```

####设置密码

用这个命令：

```

passwd

```

####退出

退出这个chroot的环境

```

exit

```

###umount 并重启

然后要umount那些设备

```

# umount  /mnt/{boot,home,var,a}

# reboot

```

终于结束了，如果正常的话重启之后就进入grub的引导了，可以选择安装的系统。



当然arch是需要去配置很多东西的，这里就先不说了，下次吧。。
