ms:1.0.1-basement-arm64
tf:1.15.0-basement-arm64

ubuntu:18.04-basement-arm64

/mntdlws/storage/resnet50_050/scripts/rank_table_1pcs.json

1 设置环境变量：
export PYTHONPATH=/usr/local/Ascend/ascend-toolkit/20.0.0.B035/arm64-linux_gcc7.3.0/opp/op_impl/built-in/ai_core/tbe:
export LD_LIBRARY_PATH=/usr/lib/aarch64-linux-gnu/hdf5/serial:/usr/local/Ascend/add-ons:/usr/local/Ascend/nnae/20.0.0.B035/arm64-linux_gcc7.3.0/fwkacllib/lib64:/usr/local/Ascend/driver/lib64/common:/usr/local/Ascend/driver/lib64/driver:/usr/local/lib:/usr/lib/
export TBE_IMPL_PATH=/usr/local/Ascend/ascend-toolkit/20.0.0.B035/arm64-linux_gcc7.3.0/opp/op_impl/built-in/ai_core/tbe
export PATH=/usr/local/Ascend/ascend-toolkit/20.0.0.B035/arm64-linux_gcc7.3.0/fwkacllib/ccec_compiler/bin/:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

2 进入Resnet50_HC路径下修改一些相对路径设置
cd /dlwsdata/storage/Resnet50_HC
vim run_8p.sh
将脚本中的json路径设好
export RANK_TABLE_FILE=/data/Resnet50_HC/hccl_8p.json

vim ./code/resnet50_train/configs/res50_32bs_8p.py
将数据集路径设好
'data_url': '/data/Resnet50_HC_data/data/resnet50/imagenet_TF',

3 执行./run_8p.sh
如果是拷贝到新环境，可能需要添加执行权限，chmod +x run_8p

* docker 执行
docker run -it --rm -v /mntdlws/storage/:/data/ -v /var/log/npu/:/var/log/npu/ --privileged tf:1.15 bash
pip3 install sympy

docker run -it --rm -v /mntdlws/storage/:/data/ -v /var/log/npu/:/var/log/npu/ --privileged ms:0.5.0 bash

source /pod.env && cd /data/resnet50_050/scripts/ && mkdir -p /var/log/npu/conf/slog/ && mkdir -p /var/log/npu/slog && cp slog.conf /var/log/npu/conf/slog/ && ./run_distribute_train_1pcs.sh resnet50 cifar10 rank_table_1pcs.json /data/resnet50_cifar10/cifar/ &&sleep infinity

* docker run mindspore:

`docker run -it --rm -v /mntdlws/storage/:/data/ -v /var/log/npu/:/var/log/npu/ --privileged ms:0.5.0 bash`

cd /data/resnet50_050/scripts/ && mkdir -p /var/log/npu/conf/slog/ && mkdir -p /var/log/npu/slog && cp slog.conf /var/log/npu/conf/slog/ && ./run_distribute_train_1pcs.sh resnet50 cifar10 rank_table_1pcs.json /data/resnet50_cifar10/cifar/

cd /data/resnet50_050/scripts/ && mkdir -p /var/log/npu/conf/slog/ && mkdir -p /var/log/npu/slog && cp slog.conf /var/log/npu/conf/slog/ && ./run_distribute_train_1_1pcs.sh resnet50 cifar10 rank_table_1_1pcs.json /data/resnet50_cifar10/cifar/

219.133.167.42:5000

-------------------------------------------------------------------------------------------------------------------

docker build -f ./Dockerfile-ubuntu1804-basement . -t ubuntu:18.04-basement
docker build -f ./Dockerfile-tfuser-arm64 . -t tf:1.15.0-basement-arm64
docker build -f ./Dockerfile-msuser-arm64 . -t ms:1.0.1-basement-arm64

docker tag ubuntu:18.04-basement harbor.sigsus.cn:8443/sz_gongdianju/apulistech/ubuntu:18.04-basement-arm64


    image:
      tensorflow:
        - image: apulistech/tensorflow-npu:1.15-20.1.RC1-arm
          desc: "tf-no-root"
      mindspore:
        - image: apulistech/mindspore-npu:1.0.1-20.1.RC1-arm
          desc: "ms-no-root"
           