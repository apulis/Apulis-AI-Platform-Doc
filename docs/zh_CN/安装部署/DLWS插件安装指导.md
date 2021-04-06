# DLWS插件安装指导

## 使用前说明

目前支持 x86 平台插件安装

* 使用环境： Ubuntu 18.04 ， Python 3.6+

* 支持的插件列表

    + addon_custom_user_dasboard_backend
    + addon_custom_user_group_dashboard

* 版本支持

    + DLWorksapce release-0.0.2_beta
    + YTung-inner release-0.0.1_beta

## 安装说明


1. 创建 python 虚拟环境。（如果确定要在本地全局安装，请跳过这一步）

    ```
    virtualenv yt-env
    source yt-env/bin/active

    ```
2. 同步 github 工程代码库

    ```
    git clone -b release_0.0.1-beta https://<username>:<userpasswd>@github.com/apulis/YTung-inner.git
    ```
3. 安装依赖包
    ```
    cd YTung-inner/deploy/
    pip3 install -r requirements.txt
    ```
4. 导入将要安装插件的集群配置文件  

    ```
    ./deploy.py config restore_cluster {your_cluster_config_tar_file}
    ```
     
5. 更新插件信息配置  
    文件：`YTung-inner/deploy/addon.yaml`  
    文件字段说明：
    ```
    插件REPO名称（只包含目录名）：
        repo：插件repo的https链接，暂不支持ssh访问方式 
        branch：分支名称
        port: 3088

    ## 示例：
    # addon_custom_user_dasboard_backend:
    #     repo: https://6b4f0598ca28b2c24d636f3289bd9949e267bf6f@github.com/apulis/addon_custom_user_dasboard_backend
    #     branch: master
    #     port: 3083

    ```

6. 执行部署插件

    - `./deploy.py addon pull {your_addon_repo_name}`  
    - `./deploy.py addon build {your_addon_repo_name}`  
    - `./deploy.py addon deploy {your_addon_repo_name}`  

    说明：
    1. {your_addon_repo_name} 表示插件的repo名称, 仅包含目录  
    2. **在第三个步骤-addon deploy-之前，如果插件存在配置文件，则需更改配置文件的对应参数**。配置文件路径：

       `../addon/your_addon_name_dir/.config/local.config`

7. 重启Nginx

   `./deploy.py config nginx`

8. 导出插件接口

    `./deploy.py addon export`  

## 使用约定
- 配置文件目录  
  root_dir_of_your_addon_repo/.config/local.config  
    
- 配置文件在容器中的位置
  /etc/app/config/local.config  

## 代码目录说明
```
   .
   ├── deploy 				   # 插件部署文件夹
   ├── main 				   # YTung平台主工程
   ├── addon 				   # 插件源代码目录，部署过程生成
   ├── util 				   # 部署辅助代码
   └── README.md               # 部署步骤说明文档
```

## 待处理列表
- 支持 插件 禁用/删除操作
- 支持 插件 部署状态或失败log回传
- 支持批处理插件部署
- 支持插件列表管理  

## 问题调试


* 查看部署后的插件 pod 状态

    `kubectl get pods`

* 查看异常状态插件 log

    `kld <插件pod名称>`

* 插件拉取到本地的目录为

    `YTung-inner/addon/`

* 插件 build 后的的 services 配置

    `YTung-inner/deploy/<插件库名称>/service.yaml`

* 插件部署生成的路由配置

    `YTung-inner/deploy/deploy/nginx/addon.conf`
    
    - 确认或者更新插件配置

        `vim ../addon/addon_custom_user_dasboard_backend/.config/local.config`

* 更新证书

    `cp DLWorkspace/src/ClusterBootstrap/deploy/sshkey ./deploy/sshkey/admin.conf `

**备注：**

*插件服务端口统一在 YTung-inner/deploy/addon.yaml 中配置*

[YTung-inner 开源库](https://github.com/apulis/YTung-inner/tree/release_0.0.1)