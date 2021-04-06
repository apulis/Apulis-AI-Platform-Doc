Apulis AI Platform Installation & Deployment Guide
===================================================

* Pre Info

1. Nodes:             1 More nodes connected in one Lan network
2. AnsiblePoinht:     Set master nodes as admin & controller point
3. OS:                Ubuntu server 18.04.4 amd64 or Ubuntu server 18.04.1 arm64
4. InstallerPackages: The k8s or platform pod container tar packages should be saved and prepared before start
5. Suggession:        Hope to connect to internet

# Design cluster network

- 管理节点：集群的master作为ansible的管理节点
- 集群节点（被管理节点）：

  - 119.147.212.162 -p 50006（worker01）（atlas01)（arm64)  192.168.1.183（内网）
  - 119.147.212.162 -p 50007（worker02）（atlas02）(arm64）192.168.1.107（内网）
  - 119.147.212.162 -p 50018 （master)（amd64)    192.168.1.113（内网）

# Pre-installs

## 配置免密登录

- 1、登录管理节点（192.168.1.113）并初始化生成SSH Key

  `ssh-keygen -t ed25519 -C "testops@apulis.com"`
- 2、将管理节点的公钥拷贝到所有被管理节点的机器上：

  ```
  # ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.113 习惯怕配置管理节点在master节点
  cp -f ~/.ssh/id_*.pub ~/.ssh/id_rsa.pub
  cp -f ~/.ssh/id_%S  ~/.ssh/id_rsa

  chmod 644 ~/.ssh/id_rsa.pub
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  sshpass -p Aiperf@2025  scp  ~/.ssh/id_rsa.pub root@192.168.1.198:~/.ssh/ 
  sshpass -p Aiperf@2025  scp  ~/.ssh/id_rsa.pub root@192.168.1.225:~/.ssh/
  sshpass -p Aiperf@2025  scp  ~/.ssh/id_rsa.pub root@192.168.1.235:~/.ssh/

  ```

ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.198
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.196
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.235

```

* 验证管理节点能否ssh免密登录到所有被管理节点：

```

ssh root@192.168.1.198 rm -rf /home/dlwsadmin
ssh root@192.168.1.183 rm -rf /home/dlwsadmin
ssh root@192.168.1.107 rm -rf /home/dlwsadmin

```

NFS 路径配置
roles/storage/defaults


## 管理节点安装ansible

- 1、配置华为安装源然后安装ansible>=2.9： 
  ```
  sudo cp -a /etc/apt/sources.list /etc/apt/sources.list.bak && sudo sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list && sudo sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list
  sudo apt update &&  sudo apt install -y software-properties-common && sudo apt-add-repository --yes --update ppa:ansible/ansible && sudo apt install -y ansible
  ansible --version
  ```

**:warning:注意：**

1. 使用上面的apt方式安装ansible的时候，需要注意apt的源，测试过程中发现使用华为apt源安装ansible会有点问题。
2. 安装好ansible后，需要特别注意ansible的版本，由于ansible版本更新升级很快，使用旧的版本ansible会导致有些模块功能用不了。**最好是2.9或者2.10等以上的版本。**

## 所有被管理节点安装python和pip

使用python3和pip3，需要在InstallationYTung/hosts文件中指定python3的解释器路径，如：

- 1、查看python3路径

```

which python3
vim hosts

192.168.1.113 ansible_python_interpreter=/usr/bin/python3

```

:warning:备注：后面应该会准备好所有的依赖包（包括apt包和python包），不再需要联网下载package。


## 拉取部署代码

```

cd /home && git clone https://apulis-gitlab.apulis.cn/apulis/InstallationYTung.git && cd InstallationYTung && git checkout v1.5.0

```

## 修改配置文件

### 修改hosts文件

hosts文件的路径是：InstallationYTung/hosts

下面是云下atlas部署时使用的hosts文件，主要修改内容如下：

- 1、在 [etcd] 下填写etcd的安装节点（ip）
- 2、在 [kube-master] 下填写k8s集群的master节点（ip）
- 3、在 [kube-worker] 下填写k8s集群的worker节点（ip）。有多个worker节点时，直接换行填写即可。
- 4、在 [nfs-server] 下填写nfs安装节点（ip）
- 5、在 [harbor] 下填写harbor的安装节点（ip)

```sh
# 'etcd' cluster should have odd member(s) (1,3,5,...)
# variable 'NODE_NAME' is the distinct name of a member in 'etcd' cluster
[etcd]
192.168.1.113 NODE_NAME=etcd1 

# master node(s)
[kube-master]
192.168.1.113

# work node(s)
[kube-worker]
192.168.1.183
192.168.1.107

[cluster:children]
kube-master
kube-worker

[nfs-server]
192.168.1.113

[harbor]
192.168.1.113 NEW_INSTALL=yes SELF_SIGNED_CERT=yes

```

### 修改all.yaml文件

all.yaml文件的路径：InstallationYTung/group_vars/all.yaml

下面是云下atlas部署时使用的all.yaml文件，主要修改的内容为：

- 1、项目名称：PROJECT_NAME: "yunxia"，这个名称将作为集群harbor保存镜像的项目名称
- 2、集群harbor域名：HARBOR_DOMAIN: "harbor.yunxia.cn"
- 3、每个镜像的name和tag（包括基础镜像和服务镜像）。基础镜像的name和tag改动不多，一般来说，沿用之前的基础镜像name和tag，以及使用之前的镜像tar包就行了。服务镜像一般要修改下，一定要确保imageName、imageTag与推送到集群harbor上的一致。

```yaml
# Project Name
PROJECT_NAME: "GPU-Cluster"

PLATFORM_NAME: "Apulis"

# the name of cluster
CLUSTER_NAME: "DLWS"

# -------- Additional Variables (don't change the default value right now)---
# Binaries Directory
i18n: "zh-CN"

bin_dir: "/opt/kube/bin"

# CA and other components cert/key Directory
ca_dir: "/etc/kubernetes/ssl"

# Deploy Directory (aiarts workspace)
base_dir: "{{ lookup('env', 'PWD') }}"

# resource directory (include apt, images, dlws code and other package)
resource_dir: "{{base_dir}}/resources"

# --------- Main Variables ---------------
CONTAINER_RUNTIME: "docker"

# Network plugins supported: calico, flannel, kube-router, cilium, kube-ovn
CLUSTER_NETWORK: "weavenet"

# Service proxy mode of kube-proxy: 'iptables' or 'ipvs'
PROXY_MODE: "iptables"

# K8S Service CIDR, not overlap with node(host) networking
SERVICE_CIDR: "10.68.0.0/16"

# Cluster CIDR (Pod CIDR), not overlap with node(host) networking
CLUSTER_CIDR: "172.20.0.0/16"

# NodePort Range
NODE_PORT_RANGE: "20000-40000"

# Cluster DNS Domain
CLUSTER_DNS_DOMAIN: "cluster.local"

# harbor domain value
HARBOR_DOMAIN: "harbor.sigsus.cn"

# harbor https port
HARBOR_HTTPS_PORT: 8443

# CPU Architecture
arch_map:
  i386: "386"
  x86_64: "amd64"
  aarch64: "arm64"
  armv7l: "armv7"
  armv6l: "armv6"

thirdparty_images:
  grafana:
    name: "apulistech/grafana"
    tag: "6.7.4"
  grafana-zh:
    name: "apulistech/grafana-zh"
    tag: "6.7.4"
  a910-device-plugin:
    name: "apulistech/a910-device-plugin"
    tag: "devel3"
  alibi-explainer:
    name: ""
  atc:
    name: "apulistech/atc"
    tag: "0.0.1"
  visualjob:
    name: "apulistech/visualjob"
    tag: "1.0"
  tensorflow:
    name: "apulistech/tensorflow"
    tag: "1.14.0-gpu-py3"
  pytorch:
    name: "apulistech/pytorch"
    tag: "1.5"
  mxnet:
    name: "apulistech/mxnet"
    tag: "2.0.0-gpu-py3"
  ubuntu:
    name: "apulistech/ubuntu"
    tag: "18.04"
  bash:
    name: "bash"
    tag: "5"
  k8s-prometheus-adapter:
    name: "directxman12/k8s-prometheus-adapter"
    tag: "v0.7.0"
  tensorflow-serving:
    name: "apulistech/tensorflow-serving"
    tag: "1.15.0"
  tensorrtserver:
    name: ""
  kfserving-pytorchserver:
    name: "apulistech/kfserving-pytorchserver"
    tag: "1.5.1"
  knative-serving:
    name: ""
  kfserving-logger:
    name: ""
  golang:
    name: "golang"
    tag: "1.13.7-alpine3.11"
  prometheus-operator:
    name: "jessestuart/prometheus-operator"
    tag: "v0.38.0"
  coredns:
    name: "k8s.gcr.io/coredns"
    tag: "1.6.7"
  etcd:
    name: "k8s.gcr.io/etcd"
    tag: "3.4.3-0"
  kube-apiserver:
    name: "k8s.gcr.io/kube-apiserver"
    tag: "v1.18.2"
  kube-controller-manager:
    name: "k8s.gcr.io/kube-controller-manager"
    tag: "v1.18.2"
  kube-proxy:
    name: "k8s.gcr.io/kube-proxy"
    tag: "v1.18.2"
  kube-scheduler:
    name: "k8s.gcr.io/kube-scheduler"
    tag: "v1.18.2"
  pause:
    name: "k8s.gcr.io/pause"
    tag: "3.2"
  mysql-server:
    name: "mysql/mysql-server"
    tag: "8.0"
  postgresql:
    name: "postgres"
    tag: "11.10-alpine"
  nvidia-device-plugin:
    name: "nvidia/k8s-device-plugin"
    tag: "1.11"
  kube-vip:
    name: "plndr/kube-vip"
    tag: "0.1.8"
  alertmanager:
    name: "prom/alertmanager"
    tag: "v0.20.0"
  node-exporter:
    name: "prom/node-exporter"
    tag: "v0.18.1"
  prometheus:
    name: "prom/prometheus"
    tag: "v2.18.0"
  redis:
    name: "redis"
    tag: "5.0.6-alpine"
  weave-kube:
    name: "weaveworks/weave-kube"
    tag: "2.7.0"
  weave-npc:
    name: "weaveworks/weave-npc"
    tag: "2.7.0"
  xgbserver:
    name: ""
  sklearnserver:
    name: ""
  onnxruntime-server:
    name: ""
  vc-scheduler:
    name: "vc-scheduler"
    tag: "v0.0.1"
  vc-webhook-manager:
    name: "vc-webhook-manager"
    tag: "v0.0.1"
  vc-controller-manager:
    name: "vc-controller-manager"
    tag: "v0.0.1"
  ascend-k8sdeviceplugin:
    name: "ascend-k8sdeviceplugin"
    tag: "v0.0.1"
  reaper:
    name: ""

apulis_images:
  apulisvision:
    name: "apulistech/apulisvision"
    tag: "1.2.1"
  aiarts-backend:
    name: "apulistech/aiarts-backend"
    tag: "v1.5.0-rc4"
  aiarts-frontend:
    name: "dlworkspace_aiarts-frontend"
    tag: "v1.5.0-rc4"
  custom-user-dashboard-backend:
    name: "dlworkspace_custom-user-dashboard-backend"
    tag: "v1.5.0-rc4"
  custom-user-dashboard-frontend:
    name: "dlworkspace_custom-user-dashboard-frontend"
    tag: "v1.5.0-rc4"
  data-platform-backend:
    name: "apulistech/dlworkspace_data-platform-backend"
    tag: "latest"
  gpu-reporter:
    name: "apulistech/dlworkspace_gpu-reporter"
    tag: "latest"
  image-label:
    name: "apulistech/dlworkspace_image-label"
    tag: "latest"
  init-container:
    name: "apulistech/dlworkspace_init-container"
    tag: "latest"
  openresty:
    name: "apulistech/dlworkspace_openresty"
    tag: "latest"
  repairmanager2:
    name: "apulistech/repairmanager2"
    tag: "latest"
  restfulapi2:
    name: "apulistech/restfulapi2"
    tag: "v1.5.0-rc4"
  webui3:
    name: "dlworkspace_webui3"
    tag: "v1.5.0-rc4"
  job-exporter:
    name: "apulistech/job-exporter"
    tag: "1.9"
  nginx:
    name: "apulistech/nginx"
    tag: "1.9"
  node-cleaner:
    name: "node-cleaner"
    tag: "latest"
  watchdog:
    name: "apulistech/watchdog"
    tag: "1.9"
  istio-proxy:
    name: "apulistech/istio-proxy"
    tag: "latest"
  istio-pilot:
    name: "apulistech/istio-pilot"
    tag: "latest"
  knative-serving-webhook:
    name: "apulistech/knative-serving-webhook"
    tag: "latest"
  knative-serving-queue:
    name: "apulistech/knative-serving-queue"
    tag: "latest"
  knative-serving-controller:
    name: "apulistech/knative-serving-controller"
    tag: "latest"
  knative-serving-activator:
    name: "apulistech/knative-serving-activator"
    tag: "latest"
  knative-serving-autoscaler:
    name: "apulistech/knative-serving-autoscaler"
    tag: "latest"
  knative-net-istio-webhook:
    name: "apulistech/knative-net-istio-webhook"
    tag: "latest"
  knative-net-istio-controller:
    name: "apulistech/knative-net-istio-controller"
    tag: "latest"
  kfserving-manager:
    name: "apulistech/kfserving-manager"
    tag: "latest"
  kfserving-storage-initializer:
    name: "apulistech/kfserving-storage-initializer"
    tag: "latest"
  kfserving-kube-rbac-proxy:
    name: "apulistech/kfserving-kube-rbac-proxy"
    tag: "latest"
  mlflow:
    name: "apulistech/mlflow"
    tag: "v1.0.0"

```

### 修改cluster.yaml文件

cluster.yaml文件的路径是：InstallationYTung/group_vars/cluster.yaml

下面是云下atlas部署时使用的cluster.yaml文件，主要修改的内容为：

- 1、集群的kube vip地址：kube_vip_address: "192.168.1.113" （使用master的ip即可）

```yaml
MASTER_AS_WORKER: true

container_mount_path: /dlwsdata
physical_mount_path: /mntdlws

manifest_dest: "/root/build"
user_name: dlwsadmin
default_cni_path: /opt/cni/bin

# kube vip address
kube_vip_address: "192.168.1.113"

# Network plugins supported: calico, flannel, kube-router, cilium, kube-ovn
CLUSTER_NETWORK: "weavenet"

# Service proxy mode of kube-proxy: 'iptables' or 'ipvs'
PROXY_MODE: "iptables"

# K8S Service CIDR, not overlap with node(host) networking
SERVICE_CIDR: "10.68.0.0/16"

# Cluster CIDR (Pod CIDR), not overlap with node(host) networking
CLUSTER_CIDR: "172.20.0.0/16"

# NodePort Range
NODE_PORT_RANGE: "20000-40000"

# Cluster DNS Domain
CLUSTER_DNS_DOMAIN: "cluster.local"

# cluster api address
cluster_api_address: "{{ kube_vip_address if kube_vip_address is defined else groups['kube-master'][0]}}"

# 平台类型，决定了可以启动多少服务
PLATFORM_MODE: "express"

datasource: postgres

modes:
  preview:
    - storage
    - network
    - a910-device-plugin
    - nvidia-device-plugin
    - postgres
    - restfulapi2
    - custom-user-dashboard
    - jobmanager2
    - custommetrics
    - monitor
    - nginx
    - openresty
    - webui3
    - aiarts-backend
    - aiarts-frontend
    - mlflow
    - image-label
    - data-platform
    - volcanosh
    - istio
    - kfserving
    - knative
  express:
    - storage
    - network
    - a910-device-plugin
    - nvidia-device-plugin
    - postgres
    - restfulapi2
    - custom-user-dashboard
    - jobmanager2
    - custommetrics
    - monitor
    - nginx
    - openresty
    - webui3
    - aiarts-backend
    - aiarts-frontend
    - mlflow
    - image-label
    - data-platform
    - volcanosh
    - istio
    - kfserving
    - knative
  professional:
    - storage
    - network
    - a910-device-plugin
    - nvidia-device-plugin
    - postgres
    - restfulapi2
    - custom-user-dashboard
    - jobmanager2
    - custommetrics
    - monitor
    - nginx
    - openresty
    - webui3
    - aiarts-backend
    - aiarts-frontend
    - mlflow
    - image-label
    - data-platform
    - volcanosh
    - istio
    - kfserving
    - knative

# the unified image tag
image_tag: "v1.2.0"
```

### 修改harbor.yaml文件

harbor.yaml文件的路径是：InstallationYTung/group_vars/harbor.yaml

下面是云下atlas部署时使用的harbor.yaml文件，可修改内容为：

- 1、集群harbor存储数据的路径：HARBOR_LOCATION: "/data"  （建议修改成存储空间较大的路径）

```
HARBOR_LOCATION: "/data"
```

### 检查管理节点能否ping通所有的被管理节点

在InstallationYTung路径下，执行以下命令，检查管理节点能否ping通所有的被管理节点：

```
ansible all -i hosts -m ping

The authenticity of host '192.168.1.113 (192.168.1.113)' can't be established.
ECDSA key fingerprint is SHA256:5q6sr8SMfMco15RtB/cbVAIaTs3EvaT3GXq5KvZzOCg.
Are you sure you want to continue connecting (yes/no)? 192.168.1.183 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.1.107 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

```

### 导入bin包

```
sudo mount -t cifs //192.168.1.85/public /mnt/usb -o username=sheare,password='',vers=2.0
```

将事先准备好的bin包拷贝到管理节点的
cd /home/InstallationYTung
ln -s   /mnt/usb/ansible-install/bin  bin

:warning:这个bin包，是通过执行InstallationYTung/tools目录下的ext-bin-amd64.sh和ext-bin-arm64.sh来联网下载准备的
**建议直接使用之前的准备好的bin包**

### 导入harbor离线安装包

将事先准备好的harbor离线安装包拷贝到管理节点的InstallationYTung/resources/images路径下：

rm -r resources/images
ln -s /mnt/usb/ansible-install/v.1.5.0.bak/images resources/images

rm -r resources/images-pushed
ln -s /mnt/usb/ansible-install/v.1.5.0.bak/images-pushed resources/images-pushed

InstallationYTung/resources/目录下，存在images和images-pushed两个目录：

- images：存放即将要上传到集群的的镜像的tar包，以及harbor的离线安装包。
- images-pushed：在后面执行部署harbor的步骤之后， images目录下的镜像tar包会被load->tag->create(manifest)->annotate(manifest)->push(manifest)，推送到harbor之后，images目录下的镜像tar包会被mv到images-pushed（已推送）目录下。

:warning:备注：

> - 1、镜像的tar包的名称并不重要，重要的是镜像tar包被load之后，镜像name与tag是跟InstallationYTung/group_vars/all.yaml里面的一致的。当然，tar包名称最好能标识镜像name和tag。
> - 2、**怎么保证镜像的tar包里面的镜像name与tag是跟InstallationYTung/group_vars/all.yaml里面的一致？**
> - 3、打包成镜像tar包的镜像名称必须带有/amd64或者/arm64（根据镜像的架构类型来写），可查看上图。

# 执行部署脚本

**确保下面所有的ansible-playbook命令都是在`InstallationYTung`路径下执行。**

按顺序执行下面的命令：

cd /home/InstallationYTung
ansible-playbook -i hosts 01.prepare.yaml

```Result:
192.168.1.107              : ok=21   changed=5    unreachable=0    failed=0    skipped=17   rescued=0    ignored=0
192.168.1.113              : ok=24   changed=13   unreachable=0    failed=0    skipped=15   rescued=0    ignored=0
192.168.1.183              : ok=21   changed=5    unreachable=0    failed=0    skipped=17   rescued=0    ignored=0
localhost                  : ok=42   changed=25   unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
```

ansible-playbook -i hosts 02.etcd.yaml

```Result
192.168.1.113              : ok=10   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

ansible-playbook -i hosts 03.docker.yaml

ansible-playbook -i hosts 04.kube-master.yaml

ansible-playbook -i hosts 05.kube-worker.yaml

ansible-playbook -i hosts 06.kube-init.yaml

ansible-playbook -i hosts 07.storage.yaml

* FAQ：
  > NFS 路径：InstallationYTung/roles/storage/defaults
  >
  > config:  /etc/exports
  >

ansible-playbook -i hosts 08.harbor.yaml

* FAQ: [iptables: No chain/target/match by that name](https://stackoverflow.com/questions/31667160/running-docker-container-iptables-no-chain-target-match-by-that-name)
  > sudo iptables -t filter -F
  > sudo iptables -t filter -X
  > systemctl restart docker
  >
* images 缺少： mv images-push 到images, harbor从images目录加载镜像

ansible-playbook -i hosts 09.network.yaml

ansible-playbook -i hosts 10.aiarts-service.yaml

```

:warning:注意：

- 1、每一步的执行结果需要确保failed=0：



- 2、被ignore掉的报错可以忽略：



- 3、执行03.docker.yaml一步之后，需要检查docker有没有安装成功

```

systemctl status docker

```

- 4、执行06.kube-init.yaml一步之后，需要检查kubernetes集群节点：

```

kubectl get nodes

```

- 5、执行08.harbor.yaml之后，集群harbor被搭建起来了，harbor用户名和密码如下：
- 用户名：admin
- 密码：可在InstallationYTung/credentials/harbor.pwd中查看
- 6、10.aiarts-service.yaml是用来启动平台所有的服务的，执行完此步没有报错后，**部署就完成了**。接下来就是去测试平台的服务正不正常，以及检查各种pod的状态等。

- 7、ansible部署遵循幂等性，每一个步骤都可以重复执行。

# 更新镜像

**通过manifest方式推送新的镜像到集群harbor的方式：**

1. 修改InstallationYTung/group_vars/all.yaml文件中对应的镜像name和tag
2. 将新的镜像的tar包放在resources/images目录下
 1. tar包的名称没有限制（尽量写规范点，能够标识包里的imageName,tag,镜像架构等）
 2. tar包里面的镜像的名称有限制：
    1. 必须带有/amd64或者/arm64来表示镜像的架构类型，如：
       harbor.sigsus.cn:8443/sz_gongdianju/apulistech/tensorflow-serving**/amd64**:1.15.0
       harbor.atlas.cn:8443/sz_gongdianju/apulistech/tensorflow-npu**/arm64**:1.15-20.1.RC1
    2. 镜像name和tag必须与all.yaml一致
3. 执行`ansible-playbook -i hosts 08.harbor.yaml`（这个playbook会帮助我们推送新的镜像到集群harbor中）

更新了镜像之后，我们可能需要重拉镜像来**重启某个服务**，方式是：

```sh
ansible-playbook -i hosts 93.aiarts-restart.yaml -e sn=serviceName
```

- `serviceName`指的是服务的名称，可在`InstallationYTung/manifests/services`目录下查看：

![image-20210112184254409](ansible部署依瞳平台文档.assets/image-20210112184254409.png)

# 遇到的问题

- 1、端口映射场景报错：

  ![image-20210113190440305](ansible部署依瞳平台文档.assets/image-20210113190440305.png)

  该场景有待继续验证。。
- 2、python netaddr包安装失败（为避免这类型问题，后面的安装部署会准备好所有的python包，不再联网下载）：

  ```sh
  TASK [kube-master : 创建 kubernetes 证书签名请求] ****************************************************
  fatal: [192.168.1.110]: FAILED! => {"changed": false, "msg": "AnsibleError: An unhandled exception occurred while templating '{{ SERVICE_CIDR | ipaddr('net') | ipaddr(1) | ipaddr('address') }}'. Error was a <class 'ansible.errors.AnsibleFilterError'>, original message: The ipaddr filter requires python's netaddr be installed on the ansible controller"}
  ```
- 3、etcd服务启动不了或者kube-worker一直不能转为Ready状态：

  ```sh
  TASK [kube-worker : 轮询等待node达到Ready状态] *******************************************************
  FAILED - RETRYING: 轮询等待node达到Ready状态 (8 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (8 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (7 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (7 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (6 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (6 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (5 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (5 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (4 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (4 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (3 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (3 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (2 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (2 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (1 retries left).
  FAILED - RETRYING: 轮询等待node达到Ready状态 (1 retries left).
  fatal: [192.168.1.114]: FAILED! => {"attempts": 8, "changed": true, "cmd": "/opt/kube/bin/kubectl get node 192.168.1.114|awk 'NR>1{print $2}'", "delta": "0:00:00.048050", "end": "2021-01-13 18:34:04.558100", "rc": 0, "start": "2021-01-13 18:34:04.510050", "stderr": "Error from server (NotFound): nodes \"192.168.1.114\" not found", "stderr_lines": ["Error from server (NotFound): nodes \"192.168.1.114\" not found"], "stdout": "", "stdout_lines": []}
  fatal: [192.168.1.115]: FAILED! => {"attempts": 8, "changed": true, "cmd": "/opt/kube/bin/kubectl get node 192.168.1.115|awk 'NR>1{print $2}'", "delta": "0:00:00.103329", "end": "2021-01-13 18:34:05.406824", "rc": 0, "start": "2021-01-13 18:34:05.303495", "stderr": "Error from server (NotFound): nodes \"192.168.1.115\" not found", "stderr_lines": ["Error from server (NotFound): nodes \"192.168.1.115\" not found"], "stdout": "", "stdout_lines": []}

  ```

  这个问题，很大可能是因为之前安装过kubernetes集群，没有reset干净的原因，解决办法：

  ```
  kubeadm reset -f
  modprobe -r ipip
  lsmod
  rm -rf ~/.kube/
  rm -rf /etc/kubernetes/
  rm -rf /etc/systemd/system/kubelet.service.d
  rm -rf /etc/systemd/system/kubelet.service
  rm -rf /usr/bin/kube*
  rm -rf /etc/cni
  rm -rf /opt/cni
  rm -rf /var/lib/etcd
  rm -rf /var/etcd
  ```

  这个清除工作，后面应该会加入到ansible中脚本中，不再需要手动清除。
- 4、提示/home/dlwsadmin/.bashrc不存在

  ```
  TASK [prepare : 写入环境变量$PATH] ***************************************************
  ok: [192.168.1.110] => (item=/root)
  ok: [192.168.1.114] => (item=/root)
  ok: [192.168.1.115] => (item=/root)
  ok: [192.168.1.110] => (item=/home/dlwsadmin)
  failed: [192.168.1.114] (item=/home/dlwsadmin) => {"ansible_loop_var": "item", "changed": false, "item": "/home/dlwsadmin", "msg": "Destination /home/dlwsadmin/.bashrc does not exist !", "rc": 257}
  failed: [192.168.1.115] (item=/home/dlwsadmin) => {"ansible_loop_var": "item", "changed": false, "item": "/home/dlwsadmin", "msg": "Destination /home/dlwsadmin/.bashrc does not exist !", "rc": 257}
  ```

  手动解决办法：

  ```
  touch /home/dlwsadmin/.bashrc
  chown dlwsadmin:dlwsadmin /home/dlwsadmin/.bashrc
  ```

  这个问题，需要进一步优化ansible脚本。

# 算法镜像包的准备

由于事先没有准备好镜像tar包放到InstallationYTung/resources/images路径下（以后建议采用这种方式），本次推送算法镜像到集群harbor是通过手动方式：

```sh
docker load -i mindspore-1.0.1-npu-arm.tar
docker tag mindspore:1.0.1-npu-arm harbor.yunxia.cn:8443/yunxia/apulistech/mindspore-npu:1.0.1-20.1-arm
docker push harbor.yunxia.cn:8443/yunxia/apulistech/mindspore-npu:1.0.1-20.1-arm

docker load -i nni-mindspore.tar
docker tag nni:1.9-mindspore-npu-arm harbor.yunxia.cn:8443/yunxia/apulistech/nni-mindspore-npu:1.0.1-20.1-arm
docker push harbor.yunxia.cn:8443/yunxia/apulistech/nni-mindspore-npu:1.0.1-20.1-arm

docker load -i nni-tensorflow.tar
docker tag nni:1.9-tensorflow-npu-arm harbor.yunxia.cn:8443/yunxia/apulistech/nni-tensorflow-npu:1.15.0-20.1-arm
docker push harbor.yunxia.cn:8443/yunxia/apulistech/nni-mindspore-npu:1.0.1-20.1-arm
```

* docker login
  /opt/kube/bin/docker login harbor.atlas.cn:8443 -u admin -p 8XBaGd64LXaqYHpd
