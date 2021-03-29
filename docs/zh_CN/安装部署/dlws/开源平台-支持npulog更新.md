开源平台更新-支持Log,预置环境变量
================================================================

1. 同步更新代码库

2. 更新编译 restfulapi2, webui3 

3. 更新配置

configmap指令：
cd /home/HwHiAiUser/apulis_platform/src/ClusterBootstrap/services/jobmanager2
kubectl delete -f dlws-scripts.yaml
rm dlws-scripts.yaml
./pre-render.sh
kubectl create -f dlws-scripts.yaml

4. 重启服务

./deploy.py --verbose kubernetes stop jobmanager2 restfulapi2
./deploy.py --verbose kubernetes stop webui3

wait... 

./deploy.py --verbose kubernetes start jobmanager2 restfulapi2
./deploy.py --verbose kubernetes start webui3


* 查看更新后的log
cd /var/log/npu/slog
tail -n 2000 device-1/device-1_20201104102535071.log | grep -i ERROR
