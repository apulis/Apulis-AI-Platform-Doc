# aiarts无网一键部署文档(不含ceph部署)

## 1. 配置说明 & 示例

| 主机名   | 配置  | 计算设备 | 操作系统       | 子网IP      | 描述           |
| -------- | ----- | -------- | -------------- | ----------- | -------------- |
| master01 | 1C16G |          | ubuntu 18.04.4 | 172.16.1.31 | k8s master节点 |
| master02 | 1C16G |          | ubuntu 18.04.4 | 172.16.1.32 | k8s master节点 |
| master03 | 1C16G |          | ubuntu 18.04.4 | 172.16.1.33 | k8s master节点 |
| worker01 | 1C16G | 1 GPU    | ubuntu 18.04.4 | 172.16.1.34 | GPU woker节点  |
| worker02 | 6C64G | 8 NPU    | ubuntu 18.04.1 | 172.16.1.35 | NPU woker节点  |

其中：

1. master和worker需在同一个子网或VPC，且不会与其他设备有IP冲突。
2. worker节点计算设备配置，驱动需在部署平台前安装好
   - 若设备为NVIDIA GPU，则驱动版本不低于430
   - 若设备为Huawei NPU, 则驱动版本不低于 1.72.T2.100.B020

## 2. 安装准备

### 2.1 安装要求

- **建议**

  推荐所有机器重新安装操作系统后再安装aiarts平台

- **操作系统**

  普通节点：ubuntu 18.04 server

  NPU节点：ubuntu 18.04.1 server （小版本号必须为1）

- **安装盘**

  最好使用移动ssd制作，mount在主master的机器上

- **用户配置**

  root用户，或具备sudo权限的非ROOT用户

  **所有机器允许root登录**

  ```
  vim /etc/ssh/sshd_config
  ...
  
  PermitRootLogin yes （如果不是yes，则应改为yes）
  ...
  
  
  重启ssh服务： service sshd restart
  ```

  

- **硬件配置**

  - worker节点：需禁用SecureBoot（如不禁用，将导致GPU驱动无法升级）
  - 所有节点：配置并允许SSH连接、关闭防火墙
  - GPU节点：预装好驱动，NVIDIA GPU
  - NPU节点：预装好驱动，华为NPU请参考 A910驱动安装手册

- **步骤说明**：执行时如提示无权限，则使用sudo权限执行

### 2.2 配置节点Hostname

 将各个节点的主机名， 配置方法（依master01为例子）：

- 编辑/etc/hostname，更新内容为master01
- 设置hostname立即生效：sudo hostnamectl set-hostname master01

### 2.3 编辑节点/etc/hosts

需要为**部署的mater节点**配置短域名解析。（其他节点不需要，脚本会自动copy）

```shell
vim /etc/hosts
```

例：

```
127.0.0.1      localhost

172.16.1.31    master01
172.16.1.31    master01.sigsus.cn

172.16.1.31    harbor.sigsus.cn # 这个要重点注意

172.16.1.32    master02
172.16.1.32    master02.sigsus.cn

172.16.1.33    master03
172.16.1.33    master03.sigsus.cn

172.16.1.34    worker01
172.16.1.34    worker01.sigsus.cn

172.16.1.35    worker02
172.16.1.35    worker02.sigsus.cn
```

### 2.4 在主master中创建两个空目录用于存放NFS和docker harbor数据

- 这两个目录需要创建在有200G以上的磁盘下，若**空间不足**会导致集群**无法安装**。
- 也可将这两个目录创建在不同的磁盘下，安装部署测试时推荐存放docker harbor目录有150G以上
- 只需在主master的机器上创建，其他机器无需操作。

例：

```shell
cd /home
mkdir nfs harbor
```

### 2.5 编辑节点/etc/hosts

编辑安装盘中config目录下的install_config.json

```shell
vim {安装盘目录}/config/install_config.json
```

例：

```
{
    "_comment_storage": "please check readme for further information about storage setting",
        "storage":{
                "type":"nfs",                           
                "path":"/home/nfs",                     # nfs存放目录
                "mountcmd":""
        },
    "HARBOR_STORAGE_PATH": "/home/harbor",              # docker harbor存放目录 
    "DOCKER_HARBOR_LIBRARY": "sz_gongdianju",           # docker harbor库名
    "alert_host": "smtp.test.com:25",                   # 告警服务smtp服务器
    "alert_smtp_email_address": "test_smtp@test.com",   # 告警服务邮件发送的邮箱
    "alert_smtp_email_password": "Apulis123",           # 告警服务邮件发送的邮箱密码
    "alert_default_user_email": "receiver@test.com",    # 告警服务接收的邮箱
    "HARBOR_ADMIN_PASSWORD": "Apulis123",               # docker harbor admin用户的密码
	"language": "zh",
	
	"worker_nodes": [                                   # worker节点
        {                                           
            "host": "worker01",                         # host为短域名
            "gpuType": "gpu",                           # gpuType为计算设备类型gpu或npu
            "vendor": "nvidia"                          # vendor为设备厂商nvidia或huawei
        },
        {
            "host": "worker02",
            "gpuType": "npu",
            "vendor": "huawei"
        }
     ],
     "_comment_extra_master_nodes": "if there is no extra_master_nodes, leave this array empty",
     "extra_master_nodes": [                         # 除主master外的其他master节点 
         {
             "host": "master02"                      # host为短域名
         },
         {
             "host": "master03"
         }
      ],
      "kube_vip": "172.16.1.30"                       # 集群vip，可与集群节点的ip相同，但不能															 和局域网内其他机器有ip冲突
      "secret_key_for_password": "123abcABC!@#",
      "platform_name": "依瞳人工智能平台",               # 平台显示名称，请按实际需要修改
      "enable_vc": true,                              # true显示vc功能，false则隐藏
      "i18n": true,                                   # 可填true、zh-CN、en-US 是否启用多语言                                                         配置，填 true 表示启用全部语言，选填                                                           zh-CN 或 en-US 可指定一种语言
      "db_archtype": "amd64",                         # 可以填入"arm64"或者"amd64"
      "enable_avisuals": true                         # 是否部署可视化建模
}
```

- install_config.json中各项配置均可按实际修改，其中 "NFS_STORAGE_PATH" 和 "HARBOR_STORAGE_PATH" 是步骤2.4中提前创建好的目录，这两个目录在安装前必须为空。
- 如果没有worker节点，则 "worker_nodes" 需要为**空数组**，即 "worker_nodes": [] 。
- 如果没有其他master节点，则 "extra_master_nodes" 需要为**空数组**，即 "extra_master_nodes": [] 。
- 除 "worker_nodes" 和 "extra_master_nodes"外，其他配置项都是**必填项**。

### 2.6 确认时区和时间

确认集群所有节点机器的时区和时间。要求集群所有节点机器的时区和时间一致。（时间可以相差几秒）

如果时间相差太大，会导致k8s的证书失效，aiarts部署失败。请手动修改集群所有节点机器的时区和时间。

```
root@master01:~# date
Sat Nov 14 16:26:10 EST 2020

修改时区： tzselect命令
root@master01:~# tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the time zone using the Posix TZ format.
#? 4


Please select a country whose clocks agree with yours.
 1) Afghanistan           18) Israel                35) Palestine
 2) Armenia               19) Japan                 36) Philippines
 3) Azerbaijan            20) Jordan                37) Qatar
 4) Bahrain               21) Kazakhstan            38) Russia
 5) Bangladesh            22) Korea (North)         39) Saudi Arabia
 6) Bhutan                23) Korea (South)         40) Singapore
 7) Brunei                24) Kuwait                41) Sri Lanka
 8) Cambodia              25) Kyrgyzstan            42) Syria
 9) China                 26) Laos                  43) Taiwan
10) Cyprus                27) Lebanon               44) Tajikistan
11) East Timor            28) Macau                 45) Thailand
12) Georgia               29) Malaysia              46) Turkmenistan
13) Hong Kong             30) Mongolia              47) United Arab Emirates
14) India                 31) Myanmar (Burma)       48) Uzbekistan
15) Indonesia             32) Nepal                 49) Vietnam
16) Iran                  33) Oman                  50) Yemen
17) Iraq                  34) Pakistan
#? 9


Please select one of the following time zone regions.
1) Beijing Time
2) Xinjiang Time
#? 1


The following information has been given:

        China
        Beijing Time

Therefore TZ='Asia/Shanghai' will be used.
Selected time is now:   Mon Nov 16 15:23:13 CST 2020.
Universal Time is now:  Mon Nov 16 07:23:13 UTC 2020.
Is the above information OK?
1) Yes
2) No
#? 1


You can make this change permanent for yourself by appending the line
        TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai


修改时间：
date -s 2020-11-16   （修改日期，会将时间修改到凌晨零点，需再执行下面的命令修改时间）

date -s 17:35:15
```



### 2.7 修改标准时间。（此步骤为非必要步骤，试情况操作）

要么将**所有的机器**都改为UTC标准时间，要么将**所有的机器**都改为CST标准时间，**要统一。**

一、将时间改为UTC标准时间。

执行命令：

```
dpkg-reconfigure tzdata
```

然后选择None of the above。

最后选则UTC。

再执行date查看时间以确认。



二、将时间改为CST标准时间。

执行命令：

```
dpkg-reconfigure tzdata
```

然后选择Asia

最后选则Shanghai

再执行date查看时间以确认。

## 3.执行安装

```
cd {安装盘目录}

./install_DL.sh
然后根据脚本提示操作。
```





### 3.1检查配置是否正确，正确则输入yes继续安装

root@master01:/home/pan_1106# **./install_DL.sh**
Hardware Architecture: x86_64
Hardware Architecture: x86_64
directory: /{安装盘路径} file: install_DL.sh path: /{安装盘路径}/install_DL.sh
system: ubuntu version: "18.04"
Install directory: /home/dlwsadmin/DLWorkspace
Cluster Name:  DLWorkspace
################################
 Please check if every config is correct

 * storage type has been set to : nfs
 * storage path has been set to : /home/nfs
 * storage mount cmd has been set to :
 * harbor storage path has been set to : /home/harbor
 * docker library name has been set to : sz_gongdianju
 * harbor admin password has been set to : Apulis123
 * smtp server host has been set to : smtp.test.com:25
 * smtp server email has been set to : test_smtp@test.com
 * smtp server password has been set to : Apulis123
 * smtp default receiver has been set to : receiver@test.com
################################
Are these config correct? [ yes / (default)no ]   **yes**



### 3.2检查集群信息是否正确，正确则输入yes继续安装

################################
now begin to deploy node account
################################
You have config follwing extra master nodes:

1. master02
2. master03
You have config follwing worker nodes:
1. worker01:
* gpu type: gpu
* vendor: nvidia

4. worker02：

* gpu type: npu
* vendor: huawei

Are these configs correct? [ yes / (default)no ]   **yes**



### 3.3输入各机器的ssh root的密码

Set up node master02 ...
Node is: , master02
root@master02's password:      **（会提示在这里输入）**
Your identification has been saved in /home/dlwsadmin/.ssh/id_rsa.
Your public key has been saved in /home/dlwsadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:YcFYKRJ9NRJgujdYTNkMbMyjI1kPBBQKIXJjZtNzlus dlwsadmin@master02
The key's randomart image is:
+---[RSA 2048]----+
|=o@=o*=@=+o      |
|+* o*=# *o .     |
|.  o.Xo=o        |
|  o o+o. .       |
|   .ooo S        |
|     .E.         |
|                 |
|                 |
|                 |
+----[SHA256]-----+
dlwsadmin ALL = (root) NOPASSWD:ALL
Connection to master02 closed.
OK
Set up node master03 ...
Node is: , master03
root@master03's password:      **（会提示在这里输入）**
Your identification has been saved in /home/dlwsadmin/.ssh/id_rsa.
Your public key has been saved in /home/dlwsadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:JQAAtlMrpg0RXULKEFEP7tGKLO9rJZiWK7DRqn1uR34 dlwsadmin@master03
The key's randomart image is:
+---[RSA 2048]----+
|BX*+o..          |
|+++=.  .         |
|o*o.o   . .      |
|+=oo     o       |
|+==     S        |
|*+o . .          |
|o+oo o           |
|++. o o E        |
|+.+=.. .         |
+----[SHA256]-----+
dlwsadmin ALL = (root) NOPASSWD:ALL
Connection to master03 closed.
OK
Set up node worker01 ...
Node is: , worker01
root@worker01's password:    **（会提示在这里输入）**
Your identification has been saved in /home/dlwsadmin/.ssh/id_rsa.
Your public key has been saved in /home/dlwsadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:OqdbGbjjbc8Byta+hR/Yz5ub7XDE68bDNmxJm/lRgCU dlwsadmin@worker01
The key's randomart image is:
+---[RSA 2048]----+
|            E .  |
|             +   |
|            . .  |
|       .     . . |
|      . S     o .|
|     . = B   ....|
|      O B = .=o* |
|     o O.+ = B# .|
|      +o+o+ BB++.|
+----[SHA256]-----+
dlwsadmin ALL = (root) NOPASSWD:ALL
Connection to worker01 closed.

Set up node worker02 ...
Node is: , worker02
root@worker02's password:    **（会提示在这里输入）**
Your identification has been saved in /home/dlwsadmin/.ssh/id_rsa.
Your public key has been saved in /home/dlwsadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:OqdbGbjjbc8Byta+hR/Yz5ub7XDE68bDNmxJm/lRgCU dlwsadmin@worker02
The key's randomart image is:
+---[RSA 2048]----+
|            E .  |
|             +   |
|            . .  |
|       .     . . |
|      . S     o .|
|     . = B   ....|
|      O B = .=o* |
|     o O.+ = B# .|
|      +o+o+ BB++.|
+----[SHA256]-----+
dlwsadmin ALL = (root) NOPASSWD:ALL
Connection to worker02 closed.

OK
################################
deploy node complete
################################
Congratulation! config file loaded completed.
Now complete reamain setting

### 3.4是否将master机器用作worker？

yes则将集群内**所有master**机器作为worker使用，适合用于单点部署。（只有一个机器，既当master又当worker）

如果集群中有**多台master，建议填写no**。

Do you want to use master as worknode? [yes|no]
[no] >>>  **no**

Not setup Up Master as a worknode.

READY

press any key to continue installing    **(按任意键继续)**

### 3.5接受协议
```
press any key to continue installinguseradd: user 'dlwsadmin' already exists
User already exists...
./install_DL.sh: line 169: break: only meaningful in a \`for', \`while', or `until' loop
Done to crate 'dlwsadmin'
dlwsadmin ALL = (root) NOPASSWD:ALL

Welcome to DLWorkspace 2020.06

In order to continue the installation process, please review the license
agreement.

=====================================

End User License Agreement - Apulis

请务必仔细阅读和理解此Apulis Platform软件最终用户许可协议（“本《协议》”）中规定的所有权利和限制。
在安装本“软件”时，您需要仔细阅读并决定接受或不接受本《协议》的条款。除非或直至您接受本《协议》的全
部条款，否则您不得将本“软件”安装在任何计算机上。本《协议》是您与依瞳科技之间有关本“软件”的法律协议。
本“软件”包括随附的计算机软件，并可能包括计算机软件相关载体、相关文档电子或印刷材料。除非另附单独的
最终用户许可协议或使用条件说明，否则本“软件”还包括在您获得本“软件”后由依瞳科技不时有选择所提供的任
何本“软件”升级版本、修正程序、修订、附加成分和补充内容。您一旦安装本“软件”，即表示您同意接受本《协
议》各项条款的约束。如您不同意本《协议》中的条款，您则不可以安装或使用本“软件”。

本“软件”受中华人民共和国著作权法及国际著作权条约和其它知识产权法和条约的保护。本“软件”权利只许可使
用，而不出售。

至此，您肯定已经详细阅读并已理解本《协议》，并同意严格遵守各条款和条件。

=====================================

Copyright 2019-2020, Apulis, Inc.

All rights reserved under the MIT License:

Do you accept the license terms? [yes|no]
[no] >>>    **yes**
```


### 3.6选择安装步骤

初始安装一般选择第1步即可，如果在安装途中突然退出，则可以选择对应的步骤继续安装部署

```

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!
!   Start to work on the master. Hostname is:  master01
!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Install DLWS On Ubuntu...
WARNING:
       DLWorkspace is only certified on 18.04, 19.04, 19.10
1 . check_docker_installation
2 . check_k8s_installation
3 . install_necessary_packages
4 . copy_bin_file
5 . prepare_storage_path
6 . set_up_k8s_cluster
7 . install_harbor
8 . load_docker_images
9 . push_docker_images_to_harbor
10 . set_up_password_less
11 . install_source_dir
12 . prepare_k8s_images
13 . setup_node_environment
14 . init_cluster_config
15 . init_cluster
16 . deploy_services
Choose a step to start from: >>    **1**
```


### 3.7安装盘内的docker镜像推送完毕后，需要确认继续

All docker images are loaded from install disk ...
Remove all untagged images ...
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N]    **y**

接下来的步骤会比较耗时，耐心等待即可



### 3.8检查各机器的nfs是否成功挂载

ssh -q -o "StrictHostKeyChecking no" -o "UserKnownHostsFile=/dev/null" -i ./deploy/sshkey/id_rsa "dlwsadmin@worker02.sigsus.cn" "sudo rm /mntdlws/jobfiles; sudo rm /mntdlws/storage; sudo rm /mntdlws/work; sudo rm /mntdlws/namenodeshare; sudo rm -r /dlwsdata; sudo mkdir -p /dlwsdata; sudo mkdir -p /mntdlws/jobfiles; sudo chmod ugo+rwx /mntdlws/jobfiles; sudo mkdir -p /mntdlws/storage; sudo chmod ugo+rwx /mntdlws/storage; sudo mkdir -p /mntdlws/work; sudo chmod ugo+rwx /mntdlws/work; sudo mkdir -p /mntdlws/namenodeshare; sudo chmod ugo+rwx /mntdlws/namenodeshare; if [ ! -e /dlwsdata/jobfiles ]; then sudo ln -s /mntdlws/jobfiles /dlwsdata/jobfiles; fi; if [ ! -e /dlwsdata/storage ]; then sudo ln -s /mntdlws/storage /dlwsdata/storage; fi; if [ ! -e /dlwsdata/work ]; then sudo ln -s /mntdlws/work /dlwsdata/work; fi; if [ ! -e /dlwsdata/namenodeshare ]; then sudo ln -s /mntdlws/namenodeshare /dlwsdata/namenodeshare; fi; "
Please check if all nodes have mounted storage using below cmds:
    **cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap**
    **source /home/dlwsadmin/DLWorkspace/python2.7-venv/bin/activate**
    **./deploy.py execonall "df -h"**

If the storage havnt mounted yet, please try:
    **./deploy.py --verbose --force mount**
    or
    **./deploy.py execonall "python /opt/auto_share/auto_share.py"**

Please press any key to continue:>>



不要退出此时的安装脚本程序，新开一个终端执行命令：

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap
source /home/dlwsadmin/DLWorkspace/python2.7-venv/bin/activate
./deploy.py execonall "df -h"
```



#### 3.8.1如果所有的节点都有形如下面的输出，则说明挂载成功。

如果挂载成功，则按任意键继续部署。如果挂载不成功，请看步骤3.8.2继续操作。

Filesystem           Size  Used Avail Use% Mounted on

master01:/mnt/local  234G   61G  161G  28% /mntdlws

```
from console
Exec on all: ['master01.sigsus.cn', 'master02.sigsus.cn', 'master03.sigsus.cn', 'worker01.sigsus.cn']
Warning: Permanently added 'master01.sigsus.cn,172.16.1.31' (ECDSA) to the list of known hosts.
Node: master01.sigsus.cn
Filesystem           Size  Used Avail Use% Mounted on
udev                 7.8G     0  7.8G   0% /dev
tmpfs                1.6G  2.6M  1.6G   1% /run
/dev/nvme0n1p2       234G   61G  161G  28% /
tmpfs                7.8G     0  7.8G   0% /dev/shm
tmpfs                5.0M     0  5.0M   0% /run/lock
tmpfs                7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1       511M  4.6M  507M   1% /boot/efi
tmpfs                1.6G     0  1.6G   0% /run/user/0
master01:/mnt/local  234G   61G  161G  28% /mntdlws  ****这个就是挂载成功的标志****
tmpfs                1.6G     0  1.6G   0% /run/user/1001

Warning: Permanently added 'master02.sigsus.cn,172.16.1.32' (ECDSA) to the list of known hosts.
Node: master02.sigsus.cn
Filesystem           Size  Used Avail Use% Mounted on
udev                 7.7G     0  7.7G   0% /dev
tmpfs                1.6G  1.7M  1.6G   1% /run
/dev/nvme0n1p2       117G   18G   94G  16% /
tmpfs                7.8G     0  7.8G   0% /dev/shm
tmpfs                5.0M     0  5.0M   0% /run/lock
tmpfs                7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1       511M  4.6M  507M   1% /boot/efi
tmpfs                1.6G     0  1.6G   0% /run/user/0
master01:/mnt/local  234G   61G  161G  28% /mntdlws  ****这个就是挂载成功的标志****
tmpfs                1.6G     0  1.6G   0% /run/user/1001

Warning: Permanently added 'master03.sigsus.cn,172.16.1.33' (ECDSA) to the list of known hosts.
Node: master03.sigsus.cn
Filesystem           Size  Used Avail Use% Mounted on
udev                 7.8G     0  7.8G   0% /dev
tmpfs                1.6G  1.8M  1.6G   1% /run
/dev/nvme0n1p2       468G   17G  428G   4% /
tmpfs                7.8G     0  7.8G   0% /dev/shm
tmpfs                5.0M     0  5.0M   0% /run/lock
tmpfs                7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1       511M  4.6M  507M   1% /boot/efi
tmpfs                1.6G     0  1.6G   0% /run/user/0
master01:/mnt/local  234G   61G  161G  28% /mntdlws  ****这个就是挂载成功的标志****
tmpfs                1.6G     0  1.6G   0% /run/user/1001

Warning: Permanently added 'worker01.sigsus.cn,172.16.1.34' (ECDSA) to the list of known hosts.
Node: worker01.sigsus.cn
Filesystem           Size  Used Avail Use% Mounted on
udev                 7.8G     0  7.8G   0% /dev
tmpfs                1.6G  1.5M  1.6G   1% /run
/dev/nvme0n1p2       234G   13G  209G   6% /
tmpfs                7.8G     0  7.8G   0% /dev/shm
tmpfs                5.0M     0  5.0M   0% /run/lock
tmpfs                7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1       511M  4.6M  507M   1% /boot/efi
tmpfs                1.6G     0  1.6G   0% /run/user/0
master01:/mnt/local  234G   61G  161G  28% /mntdlws  ****这个就是挂载成功的标志****
tmpfs                1.6G     0  1.6G   0% /run/user/1001

Warning: Permanently added 'worker02.sigsus.cn,172.16.1.35' (ECDSA) to the list of known hosts.
Node: worker02.sigsus.cn
Filesystem           Size  Used Avail Use% Mounted on
udev                 7.8G     0  7.8G   0% /dev
tmpfs                1.6G  1.5M  1.6G   1% /run
/dev/nvme0n1p2       234G   13G  209G   6% /
tmpfs                7.8G     0  7.8G   0% /dev/shm
tmpfs                5.0M     0  5.0M   0% /run/lock
tmpfs                7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1       511M  4.6M  507M   1% /boot/efi
tmpfs                1.6G     0  1.6G   0% /run/user/0
master01:/mnt/local  234G   61G  161G  28% /mntdlws  ****这个就是挂载成功的标志****
tmpfs                1.6G     0  1.6G   0% /run/user/1001
```



#### 3.8.2如果nfs挂载失败，则需要手动挂载。

不要退出此时的安装脚本程序，新开一个终端执行命令：

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap

source /home/dlwsadmin/DLWorkspace/python2.7-venv/bin/activate

./deploy.py --verbose --force mount

或

./deploy.py execonall "python /opt/auto_share/auto_share.py"
```

执行完毕后，再执行下面的命令进行检查。

```
./deploy.py execonall "df -h"
```

检查挂载成功后，按任意键继续安装部署。

### 3.9安装结束后，平台所有服务均启动正常，状态为running，且安装不会自行中途退出，则平台部署完成。

```
kubectl get po -A    # 查看k8s所有pod状态
```

### 3.10目前部署成功后，平台不会自行启动knative服务，需手动启动。

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap

./deploy.py kubernetes start knative
```

## 4.配置gpu资源

打开平台，在专家系统页面中配置gpu资源

例：

http://172.16.1.30/expert   账号密码： admin、123456

左侧导航栏---->虚拟集群---->修改cpu、npu的配置数量







## FAQ:


1. 通过域名访问 endpoint
   在Restfullapi 的配置修改
   ```bash
   vim /etc/在Restfullapi/config.yaml
   # 设置
   endpoint_use_private_ip:false
   ```

UBUNTU系统安装盘在RAID ON模式下会认不到NVME固态硬盘，需要改为AHCI模式。

### 1.获取安装盘

安装盘目前都存放在harbor服务器上10.31.3.222:/data/InstallPanBackup。

请根据cpu架构和ubuntu系统进行选择。

目前已有并经过测试的安装盘有：

amd64_Ubuntu_18.04.1_LTS  

amd64_Ubuntu_18.04.3_LTS  

arm64_Ubuntu_18.04.1_LTS

### 2.如果进行安装部署测试，为了缩短时间，可以将docker-images下关于算法的镜像先剔除掉

pytorch:1.5.tar

tensorflow:1.14.0-gpu-py3.tar

tensorflow:2.3.0-gpu-py3.tar

tensorflow:1.15.2-gpu-py3.tar

mxnet:2.0.0-gpu-py3.tar

kfserving-pytorchserver:1.5.1-gpu.tar

kfserving-pytorchserver:1.5.1.tar

atc:0.0.1-amd64.tar

tensorflow-serving:1.15.0-gpu.tar

tensorflow-serving:2.2.0-gpu.tar

ubuntu:18.04-amd64.tar

tensorflow-serving:2.2.0.tar

tensorflow-serving:1.15.0.tar

### 3.harbor部署完成之后，可以在harbor.yaml中查看密码

```
vim /opt/harbor/harbor.yml
```

###4.目前部署成功后，平台不会自行启动knative服务，需手动启动。

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap

./deploy.py kubernetes start knative
```



### 5.部署volcano时，pull了错误的镜像

例： master是x86架构的cpu，却pull了arm的镜像，harbor.sigsus.cn:8443/sz_gongdianju/vc-scheduler:v0.0.1-arm64。

原因是部署脚本中，执行了

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap/deploy/services/volcanosh

kubectl create -f volcanosh-arm64.yaml
```

修复方法：

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap/deploy/services/volcanosh

kubectl delete -f volcanosh-arm64.yaml

kubectl delete -f volcanosh.yaml


kubectl create -f volcanosh.yaml
```



### 6.kfserving与knative启动不正常

检查namespace状态

```
root@master:~# kubectl get ns
NAME               STATUS        AGE
custom-metrics     Active        3d7h
default            Active        3d7h
istio-system       Active        3d7h
kfserving-pod      Terminating   3d7h
kfserving-system   Terminating   3d7h
knative-serving    Terminating   3d7h
kube-node-lease    Active        3d7h
kube-public        Active        3d7h
kube-system        Active        3d7h
volcano-system     Active        3d7h
```

如果发现有一直都是Terminating的namespace，则执行

```
kubectl get namespace $NAMESPACE -o json > $NAMESPACE.json
sed -i -e 's/"kubernetes"//' $NAMESPACE.json
kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f ./$NAMESPACE.json

**$NAMESPACE 替换为kfserving-pod、kfserving-system、knative-serving 依次执行

```

更新DLWorkspace最新的代码，然后执行

```
./deploy.py kubernetes start kfserving knative
```



### 7.配置harbor开机自启

```
vim /lib/systemd/system/harbor.service



############以下是vim的内容############



[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor
[Service]
Type=simple
Restart=on-failure
RestartSec=5
ExecStart=/usr/local/bin/docker-compose -f /opt/harbor/docker-compose.yml up
ExecStop=/usr/local/bin/docker-compose -f /opt/harbor/docker-compose.yml down
[Install]
WantedBy=multi-user.target



############以上是vim的内容############
```



然后执行 

```
systemctl enable harbor 
```



### 8.重启worker机器后发现node状态为NotReady，docker ps也发现没有容器在运行

（1）先查看kubelet状态

```
root@worker01:~# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since Wed 2020-12-02 09:47:29 CST; 3s ago
     Docs: https://kubernetes.io/docs/home/
  Process: 18117 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 18117 (code=exited, status=255)

```



```
root@worker01:~# journalctl -xeu kubelet
Dec 02 09:48:22 worker01 kubelet[18984]: Flag --cgroup-driver has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for mor
Dec 02 09:48:22 worker01 kubelet[18984]: Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more
Dec 02 09:48:22 worker01 kubelet[18984]: Flag --cgroup-driver has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for mor
Dec 02 09:48:22 worker01 kubelet[18984]: Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more
Dec 02 09:48:23 worker01 kubelet[18984]: I1202 09:48:23.007341   18984 server.go:417] Version: v1.18.6
Dec 02 09:48:23 worker01 kubelet[18984]: I1202 09:48:23.008279   18984 plugins.go:100] No cloud provider specified.
Dec 02 09:48:23 worker01 kubelet[18984]: I1202 09:48:23.008342   18984 server.go:838] Client rotation is on, will bootstrap in background
Dec 02 09:48:23 worker01 kubelet[18984]: I1202 09:48:23.023655   18984 certificate_store.go:130] Loading cert/key pair from "/var/lib/kubelet/pki/kubelet-client-current.pem".
Dec 02 09:48:23 worker01 kubelet[18984]: I1202 09:48:23.024875   18984 dynamic_cafile_content.go:167] Starting client-ca-bundle::/etc/kubernetes/pki/ca.crt
Dec 02 09:48:23 worker01 kubelet[18984]: I1202 09:48:23.402823   18984 server.go:647] --cgroups-per-qos enabled, but --cgroup-root was not specified.  defaulting to /
Dec 02 09:48:23 worker01 kubelet[18984]: F1202 09:48:23.403702   18984 server.go:274] failed to run Kubelet: running with swap on is not supported, please disable swap! or set --fail-swap-on flag to false. /proc/swaps contained: [Filename
Dec 02 09:48:23 worker01 systemd[1]: kubelet.service: Main process exited, code=exited, status=255/n/a
Dec 02 09:48:23 worker01 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```



（2）查看日志后，发现需要关闭swapoff，执行：

```
日志：Dec 02 09:48:23 worker01 kubelet[18984]: F1202 09:48:23.403702   18984 server.go:274] failed to run Kubelet: running with swap on is not supported, please disable swap! or set --fail-swap-on flag to false. /proc/swaps contained: [Filename
```

**swap on is not supported**

```
swapoff -a    #在每一台worker上执行

或在master上执行：
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap
./deploy.py --verbose execonall sudo swapoff -a
```



（3）编辑/etc/fstab，注释掉swap

```
vim /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sdc3 during installation
UUID=b2883de6-e182-4e8d-b25a-1adec5011489 /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sdc1 during installation
UUID=E155-C28A  /boot/efi       vfat    umask=0077      0       1
## swap was on /dev/sdc2 during installation
#UUID=7b1c99e2-991c-4745-9e6c-9c5aa373212f none            swap    sw              0       0
```



（4）kubectl get nodes即ready



### 10.没有创建DLWSCluster数据库

bug出现的原因：restfulapi容器中有包没有安装好。



（1）进入restfulapi容器：

```
kubectl exec -it restfulapi-\**** bash

top # 查看是否有apache进程
```

如果有apache相关的进程，则执行： 

```
pkill apache
```

安装相关的依赖包（需要有网）

```py
pip install --ignore-installed mysql-connector-python
```

运行初始化脚本：

```
bash ./run.sh
```



### 11.起job的时候一直在调度中，报错configmap "dlws-scripts" not found

生成configmap

```
cd /home/dlwsadmin/DLWorkspace/YTung/src/ClusterBootstrap/services/jobmanager2

./pre-render.sh

kubectl create -f dlws-scripts.yaml
```





### 外出部署的一些建议：

1、安装盘中复制预置模型数据（model_data），来源于算法组同事，里面包含数据集、预置模型、训练脚本等。

2、一款ssh连接工具，一定是免安装版本的，如mobaXterm。

3、mysql数据库连接工具，如navicat，试用版的也可以。

4、chrome浏览器.exe安装包，32位和64位都带上







## **.关于制作安装盘

如果需要制作安装盘，需要在符合要求的服务器下进行制作：

①操作系统要和目标部署的机器相同。（ubuntu18.04.4、ubuntu18.04.1，请确保连最小的版本号也要一样。ubuntu18.04.4、ubuntu18.04.1的安装盘依赖是不相同的）

②已经安装docker

```
apt install docker.io
```

③需要有docker源、apt源

```
##docker源，包含kubernetes##
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo gpg --keyserver keyserver.ubuntu.com --recv-keys BA07F4FB #对安装包进行签名
sudo gpg --export --armor BA07F4FB | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
            distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
            curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
```



```
##阿里云apt源##
vim /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse


sudo apt-get update
```

④连接10.31.3.222的harbor

```
vim /etc/hosts

在hosts中添加 10.31.3.222    harbor.sigsus.cn
```

⑤已经安装pip和pip3

```
apt install python-pip

apt install python3-pip
```

