	替换
　　:s/vivian/sky/ 替换当前行第一个 vivian 为 sky
　　:s/vivian/sky/g 替换当前行所有 vivian 为 sky
　　:n,$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky
　　:n,$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky
　　n 为数字，若 n 为 .，表示从当前行开始到最后一行
　　:%s/vivian/sky/(等同于 :g/vivian/s//sky/) 替换每一行的第一个 vivian 为 sky
　　:%s/vivian/sky/g(等同于 :g/vivian/s//sky/g) 替换每一行中所有 vivian 为 sky
　　可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符
　　:s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/
　　:%s+/oradata/apras/+/user01/apras1+ (使用+ 来 替换 / )： /oradata/apras/替换成/user01/apras1/



**参考**
-----------------------------------------------------------------------------

1. [vim字符串替换](https://www.cnblogs.com/black/p/5171633.html)


    image:
    tensorflow:
        - image: apulistech/ubuntu:18.04-amd64
        desc: "测试1"
        category: "normal"
        - image: apulistech/tensorflow:2.3.0-gpu-py3
        desc: "测试2"
        category: "normal"
        - image: apulistech/tensorflow:1.15.2-gpu-py3
        desc: "测试4"
        category: "normal"
    horovod:
        - image: apulistech/horovod:0.20.0-tf2.3.0-torch1.6.0-mxnet1.6.0.post0-py3.7-cuda10.1
        desc: "horovod分布式"
        category: "normal"
    mxnet:
        - image: apulistech/mxnet:2.0.0-gpu-py3
        desc: "测试7"
        category: "normal"
    pytorch:
        - image: apulistech/pytorch:1.5
        category: "normal"
        desc: "测试6"
    mindspore:
        - image: apulistech/mindspore:tcy-test
        desc: "mindspore"
        category: "normal"
        - image: apulistech/mindspore:1.1.0-amd64-gpu-nni
        desc: "mindspore"
        category: "normal"
    hyperparameters:
        - image: apulistech/mindspore:tcy-test
        desc: "mindspore 1.10"
        category: "hyperparameters"
        - image: apulistech/horovod:0.20.0-tf2.3.0-torch1.6.0-mxnet1.6.0.post0-py3.7-cuda10.1-nni1.9
        desc: "horovod-nni-distribute"
        category: "hyperparameters"


---



DltsUrl: http://192.168.1.3:80/apis
TrackingUrl: http://localhost:9010/api/2.0/mlflow
image:
  tensorflow:
    - image: apulistech/ubuntu:18.04-amd64
      desc: "测试1"
      category: "normal"
    - image: apulistech/tensorflow:2.3.0-gpu-py3
      desc: "测试2"
      category: "normal"
    - image: apulistech/tensorflow:1.15.2-gpu-py3
      desc: "测试4"
      category: "normal"
  horovod:
    - image: apulistech/horovod:0.20.0-tf2.3.0-torch1.6.0-mxnet1.6.0.post0-py3.7-cuda10.1
      desc: "horovod分布式"
      category: "normal"
  mxnet:
    - image: apulistech/mxnet:2.0.0-gpu-py3
      desc: "测试7"
      category: "normal"
  mindspore:
    - image: apulistech/mindspore:tcy-test
      desc: "mindspore"
      category: "normal"
    - image: apulistech/mindspore:1.1.0-amd64-gpu-nni
      desc: "mindspore"
      category: "normal"
  hyperparameters:
    - image: apulistech/mindspore:tcy-test
      desc: "mindspore 1.10"
      category: "hyperparameters"
    - image: apulistech/horovod:0.20.0-tf2.3.0-torch1.6.0-mxnet1.6.0.post0-py3.7-cuda10.1-nni1.9
      desc: "horovod-nni-distribute"
      category: "hyperparameters"


InteractiveModeJob: false
PrivateRegistry: harbor.sigsus.cn:8443/sz_gongdianju/
imagesave:
  k8sconf: /root/.kube/config
  sshkey: /root/.ssh/id_rsa
  sshuser: dlwsadmin
  sshport: 22

