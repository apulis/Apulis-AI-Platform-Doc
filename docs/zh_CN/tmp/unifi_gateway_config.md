testop_lab_gateway
---------------------------------------------------

测试实验室网关配置

1. unifi gateway 安装

* 预置条件

    + unifi gateway 硬件必须先接入网络中
    + 需要单独的节点案子gateway 的控制器`controller`，controller 要接入Internet
    + 需要在[unifi cloud](https://account.ui.com/login?redirect=https%3A%2F%2Funifi.ui.com%2Fdashboard) 申请帐号
    + unifi 初始帐号为 ubnt/ubnt or root/ubnt. 
    + [使用controller设置Gateway的登陆帐号](https://help.ui.com/hc/en-us/articles/204909374-UniFi-Accounts-and-Passwords-for-Controller-Cloud-Key-and-Other-Devices)




* unifi 接入网组网实例：

https://help.ui.com/hc/en-us/articles/204950304-UniFi-Troubleshooting-Device-Adoption

* unifi ports

https://help.ui.com/hc/en-us/articles/218506997

* unifi compose：

8443 端口不能访问
https://github.com/linuxserver/docker-unifi-controller

* docker run

docker run  -d --privileged   --name=unifi-controller  --net=host  -e PUID=1000   -e PGID=1000   -e MEM_LIMIT=1024M   -p 3478:3478/udp   -p 10001:10001/udp   -p 8080:8080   -p 8443:8443   -p 1900:190/udp   -p 8843:8843   -p 8880:8880   -p 6789:6789   -p 5514:5514  -v /usr/local/unifi:/config    --restart unless-stopped   linuxserver/unifi-controller

2. unifi gateway 配置和应用

* 设置静态WAN口

* LAN 配置

* 防火墙配置（白名单，黑名单）

* 端口转发配置

* 网络服务DHCP DNS 等配置