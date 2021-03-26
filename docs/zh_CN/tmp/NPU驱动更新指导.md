NPU驱动更新指导
===========================================================================

* 背景： 容器中支持普通用户调用NPU，需要将HwHiAiUser的用户ID保持与所有主机的HwHiAiUser的用户ID一致
* 目标： 支持普通用户即（no-root）账户调用NPU做模型训练
* 影响： 需要重启Atlas机器，网络、设备将会中断不可访问
* 方案来源： 华为atlas产品线

推荐方法
---------------------------------------------------------------------------
0. 确认Linux内核已更新

1. 查看各机器的HwHiAiUser的用户ID

   `id HwHiAiUser `

2. 如果各机器的HwHiAiUser的用户ID都一样比如都是`1001`,则只需切换到HwHiAiUser登录后再将工具包安装一遍

```

sudo chgrd -R HwHiAiUser driver/ # 假定工具包所在目录为 driver/
cd driver/
ls -l 
-rwxr-xr-x  1 root  HwHiAiUser   90853156 Nov 13 18:24 A800-9000-npu-driver_20.1.0_ubuntu18.04-aarch64.run*
-rwxr-xr-x  1 root  HwHiAiUser     904104 Nov 13 18:24 A800-9000-npu-firmware_1.75.22.0.220.run*
-rwxr-xr-x  1 root  HwHiAiUser  672112709 Nov 13 18:25 Ascend-cann-nnae_20.1.rc1_linux-aarch64.run*
-rwxr-xr-x  1 root  HwHiAiUser   20752615 Nov 13 18:25 Ascend-cann-nnrt_20.1.rc1_linux-aarch64.run*
-rwxr-xr-x  1 root  HwHiAiUser 1486640805 Nov 13 18:27 Ascend-cann-toolkit_20.1.rc1_linux-aarch64.run*
-rwxr-xr-x  1 root  HwHiAiUser    2090382 Nov 13 18:27 Ascend-fwk-tfplugin_20.1.rc1_linux-aarch64.run*
-rwxr-xr-x  1 root  HwHiAiUser  285250987 Nov 13 18:27 Ascend-mindstudio-toolbox_20.1.rc1_linux-aarch64.run*
./Ascend-cann-nnae_20.1.rc1_linux-aarch64.run --install 
./Ascend-cann-nnrt_20.1.rc1_linux-aarch64.run --install 
./Ascend-cann-toolkit_20.1.rc1_linux-aarch64.run --install 
./Ascend-fwk-tfplugin_20.1.rc1_linux-aarch64.run --install 
./Ascend-mindstudio-toolbox_20.1.rc1_linux-aarch64.run --install 

cd /home/HwHiAiUser/Ascend/ascend-toolkit/20.1.rc1/arm64-linux/aicpu
 ./Ascend910-aicpu_kernels-1.75.22.0.220.run --run 
 # 输入 n  重启
# 安装后可以在该目录下查看到安装文件
cat /home/HwHiAiUser/Ascend

```

3. 如果各机器的HwHiAiUser的用户ID不相同，则需要卸载重启之后再安装
```

sudo chgrd -R HwHiAiUser driver/ # 假定工具包所在目录为 driver/
cd driver/

./Ascend-cann-nnae_20.1.rc1_linux-aarch64.run --uninstall 
./Ascend-cann-nnrt_20.1.rc1_linux-aarch64.run --uninstall 
./Ascend-cann-toolkit_20.1.rc1_linux-aarch64.run --uninstall 
./Ascend-fwk-tfplugin_20.1.rc1_linux-aarch64.run --uninstall 
./Ascend-mindstudio-toolbox_20.1.rc1_linux-aarch64.run --uninstall 
./A800-9000-npu-driver_20.1.0_ubuntu18.04-aarch64.run --uninstall 
./A800-9000-npu-firmware_1.75.22.0.220.run --uninstall

# 卸载完成后重启机器
reboot

```
4. 删除并重新指定ID创建HwHiAiUser


```bash
userdel -r -f HwHiAiUser
cat /etc/group
cat /etc/passwd
useradd -u 1680 -s /bin/bash -m HwHiAiUser 
passwd HwHiAiUser 
# Ascend@910!@
 chgrp -R HwHiAiUser /root/c75
 mv /root/c75  /home
 cd /home/c75
  
```


### 2.2. 升级内核
scp -P 22 cmake-3.5.2.tar.gz root@122.224.218.53:/root/c75

```bash
uname -sr
apt update

apt remove -y Linux-headers-4.15.0-29
apt remove -y Linux-image-4.15.0-29-generic  
apt remove -y linux-modules-4.15.0-29-generic 
apt remove -y Linux-modules-extra-4.15.0-29-generic  
apt remove -y Linux-tools-4.15.0-29-generic  
apt remove -y linux-headers-4.15.0-29-generic

apt remove -y Linux-headers-4.15.0-45
apt remove -y Linux-image-4.15.0-45-generic  
apt remove -y linux-modules-4.15.0-45-generic 
apt remove -y Linux-modules-extra-4.15.0-45-generic  
apt remove -y Linux-tools-4.15.0-45-generic  
apt remove -y linux-headers-4.15.0-45-generic


apt remove -y Linux-headers-4.15.0-124
apt remove -y Linux-image-4.15.0-124-generic  
apt remove -y linux-modules-4.15.0-124-generic 
apt remove -y Linux-modules-extra-4.15.0-124-generic  
apt remove -y Linux-tools-4.15.0-124-generic  
apt remove -y linux-headers-4.15.0-124-generic


apt install -y Linux-headers-4.15.0-99 
apt install -y Linux-image-4.15.0-99-generic 
apt install -y linux-modules-4.15.0-99-generic 
apt install -y Linux-modules-extra-4.15.0-99-generic 
apt install -y Linux-tools-4.15.0-99-generic
apt install -y linux-headers-4.15.0-99-generic

reboot

#降级(内核升级若系统异常可降级，再升级)
sudo dpkg -l | grep 4.15.0-0415
apt remove xxx (dpkg --purge xxx )
```

### 2.4. cmake3.5.2编译安装
scp -P 22 cmake-3.5.2.tar.gz root@122.224.218.53:/root/c75


```bash
wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz --no-check-certificate
tar -zxvf cmake-3.5.2.tar.gz
cd cmake-3.5.2
./bootstrap --prefix=/usr
make
make install
cmake --version
```

### 2.5. gcc/g++7.3.0编译安装

scp -P 22 gcc-7.3.0.tar.gz root@122.224.218.53:/root/c75

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz
rm -rf /tmp/*
apt-get install -y bzip2
tar -zxvf gcc-7.3.0.tar.gz
cd gcc-7.3.0

wget http://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.1.0.tar.bz2
wget http://gcc.gnu.org/pub/gcc/infrastructure/mpfr-3.1.4.tar.bz2
wget http://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.0.3.tar.gz
wget http://gcc.gnu.org/pub/gcc/infrastructure/isl-0.16.1.tar.bz2

./contrib/download_prerequisites
./configure --enable-languages=c,c++ --disable-multilib --with-system-zlib --prefix=/usr/local/gcc7.3.0
# 通过grep -w processor /proc/cpuinfo|wc -l查看cpu数，示例为15，用户可自行设置相应参数。
make -j120 
make install 
```

* E: Sub-process /usr/bin/dpkg returned an error code (1)
sudo  apt remove --purge -y python3-lib2to3  python3-distutils  python3-dev  python3-virtualenv  python3-pip  dh-python  virtualenv
sudo  apt install  -y python3-lib2to3  python3-distutils  python3-dev  python3-virtualenv  python3-pip  dh-python  virtualenv