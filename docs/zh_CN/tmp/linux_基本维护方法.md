Linux基本维护方法
------------------------------------------------------------
1. 查看更新记录
    `less /var/log/apt/history.log`
    ```log
    root@atlas02:~# less /var/log/apt/history.log
    Start-Date: 2020-09-04  15:34:17
    Commandline: apt install nmap
    Requested-By: apulis_admin (1004)
    Install: liblinear3:arm64 (2.1.0+dfsg-2, automatic), nmap:arm64 (7.60-1ubuntu5), liblua5.3-0:arm64 (5.3.3-1ubuntu0.18.04.1, automatic)
    End-Date: 2020-09-04  15:34:19

    Start-Date: 2020-09-07  15:46:16
    Commandline: apt install cpu-checker
    Install: cpu-checker:arm64 (0.7-0ubuntu7)
    End-Date: 2020-09-07  15:46:17

    Start-Date: 2020-09-08  17:15:08
    Commandline: apt-get install nfs-kernel-server nfs-common portmap
    Upgrade: rpcbind:arm64 (0.2.3-0.6, 0.2.3-0.6ubuntu0.18.04.1)
    End-Date: 2020-09-08  17:15:11

    Start-Date: 2020-09-10  14:47:11
    Commandline: apt upgrade
    Install: gcc-10-base:arm64 (10.1.0-2ubuntu1~18.04, automatic), motd-news-config:arm64 (10.1ubuntu2.10, automatic), python3-netifaces:arm64 (0.10.4-0.1build4, automatic), linux-headers-4.15.0-117-generic:arm64 (4.15.0-117.118, automatic), linux-image-4.15.0-117-generic:arm64 (4.15.0-117.118, automatic), linux-modules-4.15.0-117-generic:arm64 (4.15.0-117.118, automatic), libnetplan
    ```

2. 配置apt安装源
  ```
  cp /etc/apt/sources.list  /etc/apt/sources.list.save
  vim /etc/apt/sources.list

  # 将http://archive.ubuntu.com和http://security.ubuntu.com替换成http://repo.huaweicloud.com，可以参考如下命令：
  sed -i "s@http://.*archive.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list
  sed -i "s@http://.*security.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list

  # 详细链接参考如下
  deb https://repo.huaweicloud.com/ubuntu-ports/ bionic main restricted universe multiverse
  deb-src https://repo.huaweicloud.com/ubuntu-ports/ bionic main restricted universe multiverse
  deb https://repo.huaweicloud.com/ubuntu-ports/ bionic-security main restricted universe multiverse
  deb-src https://repo.huaweicloud.com/ubuntu-ports/ bionic-security main restricted universe multiverse
  deb https://repo.huaweicloud.com/ubuntu-ports/ bionic-updates main restricted universe multiverse
  deb-src https://repo.huaweicloud.com/ubuntu-ports/ bionic-updates main restricted universe multiverse
  deb https://repo.huaweicloud.com/ubuntu-ports/ bionic-backports main restricted universe multiverse
d
  ```
3. 查看可升级的版本
  `do-release-upgrade -c`
  
  Ubuntu强制升级命令

  `sudo update-manager -d`

4. 查询dns
  ```
  dig baidu.com @114.114.114.114
  nslookup baidu.com
  ```

* 查看是否有GPU资源
lspci | grep -i NVIDIA


5. docker 维护

* docker 清楚缓存

```bash
docker system prune --volumesdocker
```

6. 登陆samba共享目录
  * 列出某个IP地址所提供的共享文件夹

  `smbclient -L 198.168.0.1 -U username%password`
  * 像ftp客户端一样使用smbclient

  ```bash
  smbclient //192.168.0.1/public  -U username%password
  # 执行smbclient命令成功后，进入smbclient环境，出现提示符：smb:/>

  # 这里有许多命令和ftp命令相似，如cd 、lcd、get、megt、put、mput等。通过这些命令，我们可以访问远程主机的共享资源。
  ```smbclient //192.168.0.1/publi

  * 直接一次性使用smbclient命令

  ```bash
  smbclient -c "ls"  //192.168.0.1/public  -U username%password
  # 和
  smbclient //192.168.0.1/public  -U username%password
  smb:/>ls
  # 功能一样的。
  ```

  * 创建一个共享文件夹

  ```bash
  smbclient -c "mkdir share1" //192.168.0.1/public -U username%password
  如果用户共享//192.168.0.1/public的方式是只读的，会提示NT_STATUS_ACCESS_DENIED making remote directory /share1
  ```

  * 挂载samba到本地
  
  ```bash
  # 挂载命令
  mount -t cifs //10.31.3.222/haiyuan.bian /mnt/samba -o username=haiyuan.bian,password=haiyuan
  # /etc/fstab方式挂载
  //10.31.3.222/haiyuan.bian                 /mnt/samba              cifs   username=haiyuan.bian,password=haiyuan  0  0
  ```


7. 查看主板序列号
  ```
  dmidecode | more

  ...

  Handle 0x0002, DMI type 2, 15 bytes
  Base Board Information  主板型号/主板信息
          Manufacturer: Intel Corporation
          Product Name: DG43GT
          Version: AAE62768-300
          Serial Number: BTGT9340022N

  ```

   
8. 查看是否为SSD硬盘
  *因为SSD是非转动盘，如果返回结果为0说明是SSD硬盘，如果返回结果为1，说明是转动盘HDD类的硬盘。* 
  ```
  cat /sys/block/sda/queue/rotational   #表明sda这块硬盘是固态硬盘(SSD)
  0
  ```

7. 配置 GIT 编码

```git
git config --global core.quotepath false   
git config --global gui.encoding utf-8  
git config --global i18n.commit.encoding utf-8  
git config --global i18n.logoutputencoding utf-8  
export LESSCHARSET=utf-8
```

8. 设置iptables禁止外网访问
设置只允许指定ip地址访问指定端口
1、在tcp协议中，禁止所有的ip访问本机的1521端口。
iptables -I INPUT -p tcp --dport 1521 -j DROP
2、允许192.168.1.123访问本机的1521端口
iptables -I INPUT -s 192.168.1.123 -p tcp --dport 1521 -j ACCEPT
最后，保存当前规则
/etc/rc.d/init.d/iptables save
service iptables restart 

```
iptables -I INPUT -p tcp --dport 13025 -j DROP
iptables -I INPUT -s 192.168.100.23 -p tcp --dport 130
iptables -I INPUT -s 192.168.100.33 -p tcp --dport 130
# 保存Iptables设置
iptables-save > /etc/iptables-rules
ip6tables-save > /etc/ip6tables-rules
```

9. 删除5天前的文件

find /oss/bt_backup -mtime +6 -name "*.tar.gz" | xargs -I {} rm -rf {}

find ./ -mtime +6 -name "*" | xargs -I {} rm -rf {}


问题描述:
误删/var/lib/docker/overlay2 下的某些镜像文件重新拉取镜像,和创建容器出现报错
docker: Error response from daemon: error creating overlay mount /var/lib/docker/overlay2* no such file or directory
问题分析:
docker 拉取相同镜像会读取本地缓存存在的话就不会重新拉取,这时我们删除了/var/lib/docker/overlay2对应镜像的配置目录 所以拉取时或者创建容器时由于数据的丢失而创建失败
问题解决:
彻底清空无效镜像的工作目录和缓存
使用docker system prune -a 命令


/home/train_use


如果同学们需要在 arm64 环境编译或使用容器可以在 worker 节点 121.37.54.25 这台服务器上操作；使用worker 节点存储空间；这样可以避免影响到平台，不与平台争用空间，疑惑影响docker的运行状态。


10. 删除15天以前的日志
find . -mtime +15 -type f | xargs rm -rf

11. 清理系统空间
https://blog.csdn.net/m0_37407756/article/details/79903

* 创建用户账号

* 更改用户名
  + 查看已有的系统用户
  cat /etc/passwd
  + 修改用户名
  sudo usermod -l NEW_USERNAME OLD_USERNAME
  *usermod: user john is currently used by process 1532*
   pkill -u john 1532
  + 修改用户ID
  usermod -u [NEW_USER_ID] [USERNAME]

  * Refer:[How to Change or Rename Username and UID in Linux](https://linux4one.com/how-to-change-or-rename-username-and-uid-in-linux)

 [How to Set Default User for Windows Subsystem for Linux (WSL) Distro in Windows 10](https://www.tenforums.com/tutorials/128152-set-default-user-windows-subsystem-linux-distro-windows-10-a.html?__cf_chl_jschl_tk__=b305e32f33302d6b90943e58c6fcbe8a17205400-1606648843-0-AZ8A5bSXqTqM8jsc8fmKaC3GBPmbPSgOgKzEkScEbIXZd2jHdl1es2Xq3cxo9NnvhR5kqxlHrUQRyWQIJPmSnwi4swQVzgcQRWPnUd3cA5makja240zXKIcHm2KZLTTS5RJ7QHouULd2u8v2YHKKjgC2oIfJJVZTsUQatNKLuKDJGtXw4MDmvYUeg6OFOldl-4qUGremwBOCAyrMOa_-kb-m6jbs7PrPdEGgbhqe8QlO3S-3Wvd2oIHxQ2Fv31thRDd7axRRxeFgnL29RhYFzI1RHFerQcBVu-gOJsQGbm2vkh_ZMAnBVygsoQeur92zmKPNi7rMOAVdWktSEAezhfSnpGalFBAIxfmFnAecidKcFeiXx4diI8GJnnJ_5jdqSrreSnRfPKZ7uSKOSx_d_iTF4r3xGkZ28U8EKp_wznAP) 
 [linux delete user](https://www.cyberciti.biz/faq/linux-remove-user-command/)

 userdel -r -f HwHiAiUser

 [creater user accout](https://www.tecmint.com/add-users-in-linux/)
useradd -u 1680 -m  HwHiAiUser

12. 断电续传

`rsync -P --rsh='ssh -p 22' home.tar root@192.168.205.34:/home/home.tar`

rsync -P -r --rsh=ssh  testops@192.168.1.222:/data/InstallPanBackup/amd64_Ubuntu_18.04.1_LTS .

13. 配置静态路由

  ```
  ifconfig enp189s0f0 192.168.1.181 netmask 255.255.255.0 up
  ip route add default via 192.168.1.1 dev enp5s0 proto static

  ip route add default via 192.168.1.1 dev enp189s0f0 proto static

  ip route del default via 172.168.1.1 dev enp6s0f1 proto static

  ip route del default via 172.168.1.1 dev enp197s0 proto static

  apt-get update && apt-get autoremove
  ```

16. atp brokern 处理

  ```
  apt-get update
  apt-get autoremove  -y
  apt --fix-broken install  -y 
  dpkg --configure -a
  # 如果是python相关包有问题，则执行，否则不用
  # apt-get purge python* && apt-get autoclean && apt-get install python*
  apt install  -y python3.6
  ```
  若出现lsb_release -a 文件不存在
  apt-get install -y lsb-core

17. X Server打开firefox 中文乱码

ubuntu 支持的语言设置

locale -a
```
hadoop@dataeyes2:~$ locale -a
C
C.UTF-8
en_US.utf8
POSIX
```
* 安装语言包
apt-cache search language-pack-zh # 根据搜索的情况安装

sudo apt-get install -y language-pack-zh-hans # 安装

* 增加语言设置
```
sudo vim /etc/default/locale

# vim /etc/default/locale 输入以下内容
LANG="en_US.UTF-8"
LANGUAGE="zh_CN.utf8"
LC_ALL="zh_CN.utf8"
```
*系统需要注销重新登录*

```
sudo apt-get install -y ttf-wqy-microhei  ttf-wqy-zenhei  xfonts-wqy  #文泉驿-微米黑 文泉驿-正黑 文泉驿-点阵宋体

```
*关闭并重新打开firefox*
使用支持XServer的终端模拟器，比如`MobaXterm`;

新打开一个本地对话，加上 -x 选择，如下：

```
ssh -X root@192.168.1.23 # 输入密码登录
firefox             # 打开远程firefox
Ctrl C              # 关闭或打开 
killall firefox     # 如果再次打开提示有已经运行的，则尝试消除可能运行的所有firefox
firefox                
```