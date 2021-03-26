AIArts FAQ
-------------------------------------------------------------------------------
1. 通过域名访问 endpoint
   在Restfullapi 的配置修改
   ```bash
   vim /etc/在Restfullapi/config.yaml
   # 设置
   endpoint_use_private_ip:false
   ```

UBUNTU系统安装盘在RAID ON模式下会认不到NVME固态硬盘，需要改为AHCI模式。