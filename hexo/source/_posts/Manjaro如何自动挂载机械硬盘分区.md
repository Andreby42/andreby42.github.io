---
title: Manjaro如何自动挂载机械硬盘分区
date: 2019-11-24 14:19:43
tags: [Linux]
---

Manjaro如何自动挂载机械硬盘分区<!--more-->

* 找到要挂载的硬盘的分区

```
sudo fdisk -l
```

* 安装 ntfs-3g

```
su root pacman -Sy ntfs-3g
```

* 查看机械硬盘对应的分区的UUID

```
ls -l /dev/disk/by-uuid/

lrwxrwxrwx 1 root root 10 11月 24 14:15 488F7D9C353C6AC0 -> ../../sdb1
lrwxrwxrwx 1 root root 10 11月 24 14:15 4CBE55FEBE55E0CE -> ../../sdc3
lrwxrwxrwx 1 root root 10 11月 24 14:15 52E8632FE8631091 -> ../../sdb2
lrwxrwxrwx 1 root root 10 11月 24 14:15 5fb9899e-3d1c-457e-a56f-b5897afb52f0 -> ../../sda5
lrwxrwxrwx 1 root root 10 11月 24 14:15 6EA4AF17A4AEE0B7 -> ../../sdb3
lrwxrwxrwx 1 root root 10 11月 24 14:15 72F8E430F8E3EFF1 -> ../../sdc1
lrwxrwxrwx 1 root root 10 11月 24 14:15 8072a077-8a2a-48e3-b673-fb9e065e528c -> ../../sda4
lrwxrwxrwx 1 root root 10 11月 24 14:15 aa585056-a791-49ca-88b3-9be7591a845b -> ../../sda3
lrwxrwxrwx 1 root root 10 11月 24 14:15 ace96684-1f71-4c5d-8570-1bb7394d0de6 -> ../../sda6
lrwxrwxrwx 1 root root 10 11月 24 14:15 b54b7ab8-f8cf-4e21-ba81-8d7d04bbda28 -> ../../sda7
lrwxrwxrwx 1 root root 10 11月 24 14:15 C14D-581B -> ../../sda1


```

上述的 sdb1sdb2 sdb3就是要设置自动挂载的机械硬盘分区

* 编辑/etc/fstab文件

  该文件是系统启动文件 编辑的时候一定 要慎重

  * 设置好sdb1的挂载点

    ```
    /home/andy/DISKS/Amuse/
    ```

  * 编辑`/etc/fstab`文件

    ```
    sudo vim /etc/fstab
      1 # /etc/fstab: static file system information.                                         
      2 #                                                                                     
      3 # Use 'blkid' to print the universally unique identifier for a device; this may       
      4 # be used with UUID= as a more robust way to name devices that works even if          
      5 # disks are added and removed. See fstab(5).                                          
      6 #                                                                                     
      7 # <file system>             <mount point>  <type>  <options>  <dump>  <pass>          
      8 UUID=8072a077-8a2a-48e3-b673-fb9e065e528c /boot          ext4    defaults,noatime,discard 0 2
      9 UUID=C14D-581B                            /boot/efi      vfat    defaults,noatime 0 2 
     10 UUID=aa585056-a791-49ca-88b3-9be7591a845b swap           swap    defaults,noatime,discard 0 2
     11 UUID=5fb9899e-3d1c-457e-a56f-b5897afb52f0 /var           ext4    defaults,noatime,discard 0 2
     12 UUID=ace96684-1f71-4c5d-8570-1bb7394d0de6 /              ext4    defaults,noatime,discard 0 1
     13 UUID=b54b7ab8-f8cf-4e21-ba81-8d7d04bbda28 /home          ext4    defaults,noatime,discard 0 2
     14 tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0
     15 UUID=488F7D9C353C6AC0                     /home/andreby/DISKS/Amuse/ ntfs defaults 0 0
    ```

    如上图 第15行就是新加入的挂载项

    * UUID是刚刚查询出来的
    * /home/mazo/data表示挂载点
    * ntfs表示格式，小写
    * 0 0表示开机不检查磁盘。

    修改完成后重启，使用`df -h`  查看挂载情况

    

    ```
    ...
    /dev/sdb1       327G   42G  285G   13% /home/andreby/DISKS/Amuse/
    ...
    ```

    **DONE**