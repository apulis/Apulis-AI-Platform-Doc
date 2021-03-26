同步2个代码库的平台配置
===============================================================
**注意，新集群不要重新生成ClusterId**

在平台已启用的情况下，若更新版本库，如开源库，商业库之间。


1. 同步集群配置

```
cp /home/apulis/apulis_platform/src/ClusterBootstrap/config.yaml src/ClusterBootstrap/

# 同步集群ID
cp /home/apulis/apulis_platform/src/ClusterBootstrap/deploy/clusterID.yml src/ClusterBootstrap/deploy/

# 同步hosts
cp /home/apulis/apulis_platform/src/ClusterBootstrap/deploy/etc/hosts src/ClusterBootstrap/deploy/etc/hosts

# 同步节点密码

cp /home/apulis/github/apulis_platform/src/ClusterBootstrap/deploy/sshkey/rootuser deploy/sshkey/rootuser
cp /home/apulis/github/apulis_platform/src/ClusterBootstrap/deploy/sshkey/rootpasswd deploy/sshkey/rootpasswd

# 同步集群证书
cp /home/apulis/github/apulis_platform/src/ClusterBootstrap/deploy/sshkey/id_rsa* deploy/sshkey/
cp  /etc/kubernetes/admin.conf src/ClusterBootstrap/deploy/sshkey/admin.conf
chown $(id -u):$(id -g) src/ClusterBootstrap/deploy/sshkey/admin.conf
chmod 400 -R deploy/sshkey/

#  mkdir -p $HOME/.kube
#   sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
#  sudo chown $(id -u):$(id -g) $HOME/.kube/config
#


# 同步webui服务配置
cp /home/apulis/apulis_platform/src/dashboard/config/local.yaml  src/dashboard/config/local.yaml