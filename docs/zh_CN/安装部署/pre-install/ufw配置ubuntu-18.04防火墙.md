1. 安装防火墙

   `sudo apt install ufw`

2. 查看UFW状态

   `sudo ufw status verbose`

3. 配置允许访问的应用

    `ufw allow ssh`

4. 启用 UFW

    `ufw enable`

5. 禁止访问端口
    ```
    ufw deny 2049/tcp
    ufw deny 2049/tcp
    ```

6. 查看UFW 允许列表

    `ufw app list`

7. 允许子网内所有的 IP，你可以 CIDR 的格式来配置

   `sudo ufw allow from 192.168.100.33/24`
8. master节点防火墙配置示例
```bash
root@atlas03:~/docker-build# ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
2049/tcp                   DENY        Anywhere
9300/tcp                   DENY        Anywhere
3399/tcp                   ALLOW       Anywhere
3399/udp                   ALLOW       Anywhere
22/udp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
80/udp                     ALLOW       Anywhere
6443/udp                   ALLOW       Anywhere
6443/tcp                   ALLOW       Anywhere
111/tcp                    ALLOW       Anywhere
111/udp                    ALLOW       Anywhere
2049/udp                   ALLOW       Anywhere
13025/tcp                  DENY        Anywhere
13025/udp                  DENY        Anywhere
1110/udp                   ALLOW       Anywhere
1110/tcp                   ALLOW       Anywhere
2049                       DENY        Anywhere
111                        ALLOW       Anywhere
13025                      ALLOW       Anywhere
Anywhere                   ALLOW       192.168.100.23
Anywhere                   ALLOW       192.168.100.25
Anywhere                   ALLOW       192.168.100.0/24
3399                       ALLOW       Anywhere
22                         ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
2049/tcp (v6)              DENY        Anywhere (v6)
9300/tcp (v6)              DENY        Anywhere (v6)
3399/tcp (v6)              ALLOW       Anywhere (v6)
3399/udp (v6)              ALLOW       Anywhere (v6)
22/udp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
80/udp (v6)                ALLOW       Anywhere (v6)
6443/udp (v6)              ALLOW       Anywhere (v6)
6443/tcp (v6)              ALLOW       Anywhere (v6)
111/tcp (v6)               ALLOW       Anywhere (v6)
111/udp (v6)               ALLOW       Anywhere (v6)
2049/udp (v6)              ALLOW       Anywhere (v6)
13025/tcp (v6)             DENY        Anywhere (v6)
13025/udp (v6)             DENY        Anywhere (v6)
1110/udp (v6)              ALLOW       Anywhere (v6)
1110/tcp (v6)              ALLOW       Anywhere (v6)
2049 (v6)                  DENY        Anywhere (v6)
111 (v6)                   ALLOW       Anywhere (v6)
13025 (v6)                 ALLOW       Anywhere (v6)
3399 (v6)                  ALLOW       Anywhere (v6)
22 (v6)                    ALLOW       Anywhere (v6)

```
参考链接：
1. [使用UFW配置防火墙](https://blog.csdn.net/bigdata_mining/article/details/80699180)
2. [ubuntu ufw详解](https://help.ubuntu.com/community/UFW)
3. [配置子网](https://www.jianshu.com/p/e2951fa4ce3a )