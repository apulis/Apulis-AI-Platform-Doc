NFS加载新硬盘
------------------------------------------------------------------

已经配置，并启用nfs，但是共享目录空间已用完，挂载新硬盘到共享空间。

1. 查看磁盘状态
  ```
  fdisk -l

  Disk /dev/nvme1n1: 1.5 TiB, 1600321314816 bytes, 3125627568 sectors
  Units: sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes

  ```
* 对新硬盘分区
  ```
  sudo fdisk /dev/sdb
    n # n 为新建分区，详细命令使用 --help

    # 按默认回车
  ```
* 格式化硬盘

  `sudo mkfs -t ext4 /dev/nvme1n1`

* 临时挂载
  ```
  mkdir -p /mnt/local # 创建挂载目录
  sudo mount -t ext4 /dev/nvme1n1 /mnt/local # 挂载硬盘分区，-t ext4 非必须
  ```
* 永久挂载

若要每次重启不需要重新挂载，还需要进行下面的操作：

+ 查看磁盘分区的UUID

`sudo blkid /dev/nvme1n1`

+ 配置sudo vim /etc/fstab：

`UUID=3d0ada40-b1f0-4fd7-be6e-3c2fdc9da180  /mnt/local      ext4     defaults        0      0`

2. 查看挂载的目录

  `df -h`

3. 取消挂载

  `umount /mnt/local`

**备注：**
如果k8s集群的服务使用的是nfs存储，nfs 依赖挂载目录；此时需要先把相关服务全部停掉。
然后将挂载目录`/mnt/local`下的内容备份或清除，避免已有的目录冲突

  ```
  mv /mnt/local /mnt/local-bak
  mkdir -p /mnt/local
  ```

再将磁盘挂载到该目录，并将备份文件同步过去

  ```
  mount /dev/nvme1n1  /mnt/local 

  rsync -avz /mnt/local-bak /mnt/local
  ```

**因为共享目录已有文件，故不能重新作为nfs的共享目录，利用nfs配置通过重启系统配置共享目录,然后检查nfs，挂载目录是否正常。**

  `reboot` 


