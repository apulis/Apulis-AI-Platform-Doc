
* 本地数据库连接：

    `mysql -h127.0.0.1 -P 3306 -uroot -p<PASSWORLD>`

* 常用 k8s kubectl 命令

    ```   
    kubectl get po # 查看目前所有的pod
    kubectl get rs # 查看目前所有的replica set

    kubectl describe po my-nginx # 查看my-nginx pod的详细状态
    kubectl describe rs my-nginx # 查看my-nginx replica set的详细状态
    kubectl describe deployment my-nginx # 查看my-nginx deployment的详细状态
    kubectl get eventskubectl get events查看相关事件
    ```

* 查看任务：

    `kubectl get deployment`

* 删除任务

    `kubectl delete deployment my-nginx ` 

* 查看集群

    `kubectl get pods -o wide -n kube-system ##  kube-system 为用户空间"namespace"`

* 查看 pod 环境变量：

    `kubectl exec -it 35abba0b-6f16-4336-9d2e-a9a5326c209e -- env`


* 设置kubectl 快捷键
    ```
    Vim ~/.bashrc

    alias kdpk='kubectl describe pod -n kube-system'
    alias kdpd='kubectl describe pod -n default'
    alias kdpc='kubectl describe pod -n custom-metrics'

    alias kdn='kubectl describe node'
    alias kddsk='kubectl describe ds -n kube-system'
    alias kddpk='kubectl describe deployment -n kube-system'

    alias kld='kubectl logs -n default --tail 20'
    alias klc='kubectl logs -n custom-metrics --tail 20'
    alias klk='kubectl logs -n kube-system --tail 20'

    alias kek='kubectl exec -n kube-system -it' 
    alias ked='kubectl exec -n default -it'  
    alias kec='kubectl exec -n custom-metrics -it' 

    alias kgda='kubectl get deployment --all-namespaces'
    alias kgpa='kubectl get po --all-namespaces'
    alias kgpa2='kubectl get pod -o=custom-columns=Namespace:.metadata.namespace,NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces'
    alias kgca='kubectl get configmap --all-namespaces'
    alias kgn='kubectl get nodes'
    alias kgdsa='kubectl get ds --all-namespaces'

    alias kgpyd='kubectl get pod -n default -o yaml'
    alias kgpyk='kubectl get pod -n kube-system -o yaml'

    alias kdtpd='kubectl delete pod -n default'
    alias kdtpk='kubectl delete pod -n kube-system'
    alias kdtdsk='kubectl delete ds -n kube-system'

    Source ~/.bashrc
    ```

* 指定容器资源占用：

    `docker run -it --rm ivotron/stress-ng:v0.07.29-arm64v8  /usr/bin/stress-ng --cpu 2 --cpu-load 100 --metrics-brie`

* 查看 pods

    `kgpa | grep mysql`

* 查看pods 详情： 

    `kdpk mysql-xwmtw`

* 查看 pod 错误log ：

    `kld addon-custom-user-dasboard-backend-vvfds`

* 确认服务器中是否有nvidia显卡的命令

    `lspci -vnnn | perl -lne 'print if /^\d+\:.+(\[\S+\:\S+\])/' | grep VGA | grep -i NVIDIA`

* 查看 npu 接口状态

    `for i in {0..7}; do hccn_tool -i ${i} -link -g;done`

* 查看 npu 使用状态

    `npu-smi info`

* 查看 npu 监控文件状态

    `cat /var/log/npu/npu_smi/device0`


* 等待pod状态
   

   1. Wait for the pod "busybox1" to contain the status condition of type "Ready".

        `kubectl wait --for=condition=Ready pod/grafana`

   2. Wait for the pod "busybox1" to be deleted, with a timeout of 60s, after having issued the "delete" command.

       ```
       kubectl delete pod/busybox1
       kubectl wait --for=delete pod/busybox1 --timeout=60s
       ```

   * 示例，重启监控组件如grafana,或openstry,如果立即stop/start，pod的服务会因主机端口占用启用失败；此时可以增加wait选项等待stop操作完成

      `kubectl wait --for=delete pod/openstry* --timeout=60s -n kube-system`
      
* 查看并释放端口
   ```bash
   netstat -tln | grep 8080
   lsof -i:8060
   ```

**参考：**
   [kubectl_wait](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#wait)