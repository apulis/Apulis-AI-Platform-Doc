FAQ
-------------------------------------------------------------------------------
1.  `deploy.py 4063 line：AttributeError: 'module' object has no attribute 'FullLoader'`
    ```
    # 在部署环境中重新安装 PyYAML
    pip install --ignore-installed PyYAML
    ```

2. 强制 Kill Job方法：
    ```
    kubectl get pods                     # 查看僵死pod
    kubectl delete po --force <pod id>   # 强制 kill pod 
    docker ps |grep mind* <tf>           # 查看响应的容器
    docker top <docker id>               # 查看容器内的进程
    kill -9 <process id>                 # 逐个kill容器中所有进程
    kubectl delete --grace-period=0  --force --namespace=system pod mysql-84fbb8bfc6-lhlnh
    ```

3. 使用私有镜像库

    1. 在提交 job 的 Advanced ——> Custom Docker Registry 中配置私有镜像

    2. 在每个节点的host系统的 docker 配置文件 `/etc/docker/daemon.json` 中指定私有镜像库

    3. 在返回 job 提交页面的 docker image 配置项中指定私有镜像库的镜像，前缀必须加私有镜像库地址


4. `image pull backoff`

    1. 当更新或安装平台时，出现该错误可以在（配置/私有）docker hub 查看镜像是否编译并 push 成功
    2. `kubectl describe po -n <namespace> <podname> `查看错误详情，如果时相关依赖镜像拉取失败则会重试的


5. 登录页返回未认证或空白，需要修正前端配置的链接, 注意https或端口配置
    ```
    # vim /etc/WebUI/local.yaml

    clusters:
      sandbox03-master:
          restfulapi: "http://altas02.ascend.cn/apis"
          title: Grafana-endpoint-of-the-cluster
          workStorage: work
          dataStorage: data
          grafana: "http://altas02.ascend.cn/endpoints/grafana/"
          prometheus: http://altas02.ascend.cn:9091
    userGroup:
      type: custom
      domain: "http://altas02.ascend.cn"
      backEndPath: /custom-user-dashboard-backend
      frontEndPath: /custom-user-dashboard
    ```

6. NPU 节点重启之后，需要重新配置

    ```
    ./deploy.py --background --sudo runscriptonall scripts/npu/npu_info_gen.py
    ./deploy.py --verbose kubernetes <stop/start> monitor
    ```
7. 重启平台或集群pod，服务仍旧没有回复
    ```
    # 首先要stop pod
    ./deploy.py --verbose kubernetes stop <要重启的pod或服务>
    
    # 查看pod状态已经terminal
    kubectl get pods -w -o wide

    # 再start pod
    ./deploy.py --verbose kubernetes start <要重启的pod或服务>

    ```
8. 如果遇到 dlwsadmin 登录失败，提示sshkey权限太高

   `chmod 400 -R deploy/sshkey/`

9. 重启监控组件如grafana,或openstry,如果立即stop/start，pod的服务会因主机端口占用启用失败；此时可以增加wait选项等待stop操作完成

   `kubectl wait --for=delete pod/openstry* --timeout=60s -n kube-system`

   或者等待其状态：
   
   `kubectl wait --for=condition=Terminal pod/grafana`

   *参考：* [k8s调试方法](https://github.com/apulis/Diamond/wiki/k8s%E8%B0%83%E8%AF%95%E6%96%B9%E6%B3%95)

10. weave 路由冲突(Weave Network overlaps with existing route on host)

    *如果节点存在多个路由，或有其他k8s集群的路由配置，有 10.32.0.0 相似网段路由会出些冲突*

    * 如果这些路由不使用，则直接清理
    * 指定一个可用的范围
    `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.32.0.0/16"`
    *参考：* [weave github issues](https://github.com/kubernetes/kubernetes/issues/45419)
            [k8s weavea ddon](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options)
    
11. Restfullapi 启用master_private_ip: <121.37.54.27>

    * 在/etc/RestfulAPI/config.yaml  增加 master_private_ip

    * 重启master weavenet pod

12. 节点之间22端口不通，隐射了其他端口

    1) 增加配置config.yaml 

        `ssh_port: 22`

    2) 重新编译init-container
        ```
        # 修改config.yml中ssh_port: 隐射的端口 非 22
        ./deploy.py --verbose rendertemplate ../docker-images/init-container/ssh_config/sshd/sshd_config.template ../docker-images/init-container/ssh_config/sshd/sshd_config
        ./deploy.py --verbose docker push init-container
        ```

13. 当为网络环境不允许配置公网域名，则需要配置IP的方式
    * 修改restfullapi的endpoint的IP链接 `vim /etc/RestfulAPI/config.yaml`
    <img src="../../images/config_images/restfullapi配置.png" width = "600" height = "300" alt="域名配置" />
    配置master_private_IP后，重启jobmanager2 restfulapi2

    * 修改webui3的IP链接 `vim /etc/WebUI/local.yaml`
    <img src="../../images/config_images/webui3配置.png" width = "600" height = "300" alt="域名配置" />
    保存配置后，重启webui3

14. IP访问方式下，nginx 没有起来
    ```bash
    kubectl get pods | grep nginx
    kubeectl exec -it nginx-xxxx bash
    ps aux # 查看nginx的进程是否起来
    ```
    将 `/etc/nginx/conf.d/default.conf` 中第5行 `server name`删除，然后再启动nginx
    

15. 修改镜像拉取策略
    ```bash
    vim ../Jobs_Templete/pod.yaml.template
    # 如果需要直接使用本地镜像修改Always 为 IfNotPresent
    initContainers:
    - name: init
        imagePullPolicy: Always
    ```

16. endpoint 中ssh链接
    
    `ssh -p 32199 admin@121.37.54.27 [Password: tryme2017]`
    要登录atlas主机的,因为使用了主机上保存的sshkey<br>
    如果不登录atlas主机，在其他设备的终端下登录，可以这样，将 `-i /dlwsdata/work/admin/.ssh/id_rsa` 证书去掉，使用提示的密码 tryme2017 就可以
    如：`ssh -p 32199 admin@121.37.54.27` 在这一步输入密码：`tryme2017`
