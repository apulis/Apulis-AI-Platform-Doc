0. 查看操作系统版本信息

`uname -m && uname -a && cat /etc/*release`

* 验证Linux 操作系统内核版本

`uname -r`

1. 设置root 密码

`sudo passwd root`

2. ssh server 允许root登录

修改ssh配置文件:

`sudo vim /etc/ssh/sshd_config`

后进入配置文件中修改`PermitRootLogin`后的默认值为`yes`

3. 更正dns

    ```
    vim /etc/resolv.conf 
    # 添加
    nameserver 192.168.253.9
    nameserver 8.8.8.8
    nameserver 114.114.114.114
    ```

4. 卸载 doker

    `apt-get autoremove docker docker-ce docker-engine  docker.io  containerd runc`

* 删除docker其他没有没有卸载
    ```
    dpkg -l | grep docker
    dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
    ```
* 卸载没有删除的docker相关插件

    `apt-get autoremove docker-ce-*`

* 删除docker的相关配置&目录
    ```
    rm -rf /etc/systemd/system/docker.service.d
    rm -rf /var/lib/docker
    ```
* 确定docker卸载完毕

    `docker --version`

5. 卸载 k8s
    ```
    kubeadm reset
    sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*   
    sudo apt-get autoremove  
    sudo rm -rf ~/.kube
    ```
