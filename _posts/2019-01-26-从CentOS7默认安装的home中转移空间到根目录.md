---
author: zyqzyq
date: 2019-01-26 10:16:32+00:00
layout: post
title: 从CentOS7默认安装的/home中转移空间到根目录/
key: 20190126
tags:
- centos
---

转自 [https://blog.csdn.net/evandeng2009/article/details/49814097](https://blog.csdn.net/evandeng2009/article/details/49814097)

# 一、基础概念

Cent0S 7默认启用LVM2（Logical Volume Manager），把机器的一块硬盘分为两个区sda1和sda2，其中分区sda1作为系统盘/boot挂载，少量空间；sda2作为一个物理卷并且完全作为逻辑卷组VG(Volume Group）centos，在这个逻辑卷组centos中建立三个逻辑卷LV（Logical Volume）root和home还有swap，分别挂载到根目录/和/home以及swap。而两个分区sda1和sda2上都建立了文件系统XFS，文件系统XFS作为RedHat的默认文件系统也有它的考虑，成为继ext3，ext4之后的主流文件系统。

几个概念的关系：M个物理硬盘HD或者物理硬盘中的分区一起组建为一个逻辑卷组VG及存储池，在卷组VG中创建N个逻辑卷LV，在一个逻辑卷LV中创建文件系统比如xfs。物理硬盘/分区、逻辑卷有最小基本寻址单元，CentOS7默认的大小为4MB，二者一一对应，类似于链接或者变量引用，但是一个二者关系并非一直不变，因为物理硬盘可能发生变化而逻辑卷自动调整。创建卷组和逻辑卷，会类似于创建分区一样在磁盘开始位置写入卷的信息VGDA（卷组描述符区域，Volume Group Descriptor Area）用于识别。逻辑卷的好处在于屏蔽物理底层支撑，可自由扩展变更，而不用担心硬盘或者分区的物理空间局限，也就不会存在为了扩展分区大小而去备份/扩展分区重新格式化硬盘等问题。



HD/分区--通过pvcreate->PV--通过vgcreate(vgchange)/vgextend-->VG--通过lvcreate/lvextend-->LV--通过mkfs-->FS--通过xfs_growfs等-->df磁盘生效

但是关键点在于，CentOS 7默认安装时/home占用太多空间，根目录相较而言就小得多(只有50G)，而OpenStack安装以及存储的东西都在根目录下。上传几个镜像说不定就把你的根目录空间耗尽。不像其他文件系统ext3，ext4或者reiserfs等，有命令（resize2fs，resize_reiserfs）直接支持缩小文件系统的大小，**默认安装的xfs支持扩展增大但是不支持缩小空间！**



# 二、步骤概览

所以下面要做的步骤大概为：（直接root用户登录系统，本机或者ssh root过去，如果使用当前普通用户会遇到点不必要的麻烦）

1. 备份/home/用户文件，要是没啥内容则忽略这步（为什么非要这个/home，删掉直接用root？还是保留它，它存在也有道理的，再说生产环境还是不要只用root）
2. umount /home 卸载并lvremove删除这个home逻辑卷，释放它的空间，vgdisplay查看卷组可用空间大小
3. lvcreate新建一个新的home卷，并在其上mkfs建立xfs文件系统，(分配挂载到/home - 不用更改/etc/fstab，重启即可，) 拷贝回来之前的内容

(这个时候空余的空间随便你分配，可以再建立别的逻辑卷，或者直接空闲下来以后使用，也可以直奔主题的走下面的第四步)

4. 把之前的home逻辑卷释放并分配新卷home之后剩下的空间，lvextend分配给root卷，并用命令xfs_growfs扩展它的文件系统空间



# 三、相关命令

看看大概的命令有哪些

#### pv

> pvchange   pvck       pvcreate   pvdisplay  pvmove     pvremove   pvresize   pvs        pvscan

##### vb

1. vgcfgbackup    vgchange       vgconvert      vgdisplay      vgextend       vgimportclone  vgmknodes      vgremove       vgs
2. vgsplit        vgcfgrestore   vgck           vgcreate       vgexport       vgimport       vgmerge        vgreduce       vgrename
3. vgscan

##### lv

1. lvchange     lvcreate     lvextend     lvmchange    lvmdiskscan  lvmetad      lvmsar       lvremove     lvresize     lvscan
2. lvconvert    lvdisplay    lvm

# 四、默认安装情况

##### df -h //查看磁盘使用情况

##### cat /etc/fstab

##### vgdisplay //查看逻辑卷组情况

##### lvdisplay //查看逻辑卷情况，默认三个，root、home和交换空间swap

# 五、操作步骤

#### 备份/home中的用户数据

> [root@localhost /]# mkdir /backup && mv /home/* /backup
> [root@localhost /]# ls /home/

#### 卸载这个/home并删除逻辑卷home

> umount /home

#### df -h //查看磁盘情况

> Filesystem               Size  Used Avail Use% Mounted on
> /dev/mapper/centos-root   50G   17G   34G  33% /
> devtmpfs                 3.9G     0  3.9G   0% /dev
> tmpfs                    3.9G   84K  3.9G   1% /dev/shm
> tmpfs                    3.9G  9.0M  3.9G   1% /run
> tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
> /dev/sda1                494M  133M  362M  27% /boot

#### lvremove /dev/centos/home //删除逻辑卷home

> Do you really want to remove active logical volume home? [y/n]: y
>   Logical volume "home" successfully removed

#### vgdisplay //查看卷组可用空间

#### 新建一个卷home，fdisk格式化为8e格式，文件系统还是搞为xfs（同样挂载到/home）

> lvcreate -L 50G -n home centos //L表示大小，默认单位为M；n表示卷名；这里的centos是CentOS7安装系统的时候就默认建立好的卷组名

> WARNING: xfs signature detected on /dev/centos/home at offset 0. Wipe it? [y/n]: y
>   Wiping xfs signature on /dev/centos/home.
>   Logical volume "home" created.

#### lvdisplay //查看逻辑卷home

#### vgdisplay //再次查看卷组空间大小

#### [# vgchange -ay centos  //可选步骤：激活卷组centos，使得这个新建的home逻辑卷生效（用vgchange而不用lvchange）]

>   3 logical volume(s) in volume group "centos" now active

##### mkfs -t xfs /dev/centos/home //在新建的逻辑卷home上建立xfs文件系统

##### mount /dev/centos/home /home/ //把这个新逻辑卷home挂到之前的文件夹/home中去，直接重启用fstab来挂载也行

##### df -h //现在来查看磁盘使用情况

##### mv /backup/* /home/ //再把之前拷出来的东西拷回新建的/home中，拷贝完成就可以直接用这个普通用户来桌面登录系统了，不用重启

##### 最后再把释放出来多余的空间分配给root卷并xfs_growfs扩展文件系统

lvextend -L +823G /dev/centos/root //把剩下的823G现在分配给root卷，剩下那点渣渣空间让它闲着；+号表示在原来的基础上额外增加，不要加好则设定为具体额度

  Size of logical volume centos/root changed from 50.00 GiB (12800 extents) to 873.00 GiB (223488 extents).
  Logical volume root successfully resized

##### lvdisplay //查看逻辑卷和卷组情况



#### df -h //不使用xfs_growfs扩展文件系统，磁盘是不认得多的空间的

##### xfs_growfs /dev/centos/root //扩展root卷

##### df -h //再看root大小已经生效



# 六、命令汇总

简单总结下，就是下面几条命令：

1. mkdir /backup
2. mv /home/* /backup/
3. umount /home
4. lvremove /dev/centos/home
5. lvcreate -L 50G -n home cents
6. mkfs -t xfs /dev/centos/home
7. mv /backup/* /home/
8. lvextend -L +xxxG /dev/centos/root
9. xfs_growfs root
10. rm -rf /backup

