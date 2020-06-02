---
layout: post
title:  "Linux之磁盘配额"
date:   2020-06-02 14:34:00 +0800
categories: Linux服务器搭建与管理
tags: Linux
comments: 1
---
磁盘配额：磁盘配额是系统对用户能使用磁盘资源的控制(或者说限制).在Linux中,磁盘配额可以对用户的空间使用情况,文件数量(实际上是inode的数量，文件数量是限制inode的结果)进行限制。如果超出此范围则用户不能再往磁盘里写入数据。
限制原因：资源有限
能限制谁：普通用户或用户组


## 磁盘配额

### 磁盘配额步骤

1. 修改**/etc/fstab**文件,加入磁盘配额选项(rw,usrquota,grpquota)
2. **重启**系统或重新挂载文件系统
3. 生成磁盘配额文件 `quotacheck -avug -mf`
5. 启用配额功能 `quotaon -avug`
6. 编辑用户的磁盘配额 `edquota -u wz`

### 磁盘配额详细步骤

```shell
#查看当前系统分区挂载信息
[root@localhost Desktop]# mount
/dev/sda2 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
vmware-vmblock on /var/run/vmblock-fuse type fuse.vmware-vmblock (rw,nosuid,nodev,default_permissions,allow_other)
#1.修改fstab文件
[root@localhost dev]# vi /etc/fstab 

[root@localhost dev]# reboot

[root@localhost dev]# mount
/dev/sda2 on / type ext4 (rw,usrquota,grpquota)#(可以看到多出了磁盘配额选项)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
vmware-vmblock on /var/run/vmblock-fuse type fuse.vmware-vmblock (rw,nosuid,nodev,default_permissions,allow_other)
#2.生成磁盘配额文件(可以看到,执行quotacheck命令后系统的根目录下生成了aquota.group,aquota.user这两个文件)
[root@localhost Desktop]# quotacheck -avug -mf
...略...

[root@localhost Desktop]# ls /
aquota.group  aquota.user  bin  boot  dev  etc  home  lib  lost+found  media  mnt  opt  proc  root  sbin  selinux  srv  sys  tmp  usr  var
#3.启用磁盘配额功能
[root@localhost Desktop]# quotaon -avug
/dev/sda2 [/]: group quotas turned on
/dev/sda2 [/]: user quotas turned on
#4.编辑用户磁盘配额
[root@localhost Desktop]# edquota -u wz
```

至此,磁盘配额完毕,下面进行测试

```shell
#查看限额使用情况
[root@localhost Desktop]# repquota -auvs
...略...
#添加25M大小的aaa文件测试
[wz@localhost Desktop]#dd if=/dev/zero of=aaa bs=1M count=25
```

