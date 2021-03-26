ClamAV
-----------------------------------

* 在Ubuntu安装ClamAV 

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install clamav clamav-daemon -y
```

* 更新病毒库

    `sudo freshclam`

* 在指定目录进行病毒查杀

    `sudo clamscan -r /home`


* 不输出扫描的所有文件，只输出被感染的文件

    `sudo clamscan -r --bell -i /`

* 定时扫描(自动执行病毒查杀)
    + 启用后台服务
        ```bash
        1. 启动clamav服务，这一个命令是你如果按章pyclamd调用clamav时必须要提前执行的方法。
        sudo /etc/init.d/clamav-daemon start
        2. 自动更新病毒库的。
        sudo /etc/init.d/clamav-freshclam start
        ```

    + 设置定时扫描
        ```bash
        # 设置cron任务：每天午夜扫描FTP目录。
        $ sudo crontab -e
        00 00 * * * clamscan -r /My_FTP
        
        ```
FAQ
----------------------------------------------------------------

1. `LibClamAV Error: cli_loaddb(): No supported database files found in /var/lib/clamav/
ERROR: Can't open file or directory`

```bash
sudo touch /var/lib/clamav/clamd-socket
#vscan:vscan改成你的用户名和组名
sudo chown root:root /var/lib/clamav/clamd-socket 
sudo freshclam -v

```

2. 递归扫描目录

`加个 -r 进行递归扫描 -i 只列出受感染的文件： clamscan -r -i /home`

3. 更新病毒库抛错 `sudo sudo freshclam
ERROR: /var/log/clamav/freshclam.log is locked by another process
ERROR: Problem with internal logger (UpdateLogFile = /var/log/clamav/freshclam.log).
ERROR: initialize: libfreshclam init failed.
ERROR: Initialization error!`


    *Remove potentially bad AV installations:*

    ```bash
    sudo apt-get remove clamav 
    sudo apt-get remove clamtk 
    sudo apt-get remove freshclam
    sudo apt-get clean
    sudo apt-get autoremove
    ```

4. 更新病毒库超时
    ```bash
    sudo chmod  +w /etc/clamav/freshclam.conf 
    sudo vim /etc/clamav/freshclam.conf
    # 将默认超时30s 改为 120s
    ConnectTimeout 120 # 30
    ReceiveTimeout 120 # 30
    sudo chmod  -w /etc/clamav/freshclam.conf
    # 重新更新
    sudo freshclam -v
    ```

**参考链接**

1. ClamAV ：http://www.clamav.net/documents/installing-clamav
2. 源码链接 ： http://www.clamav.net/downloads/production/clamav-0.102.1.tar.gz
