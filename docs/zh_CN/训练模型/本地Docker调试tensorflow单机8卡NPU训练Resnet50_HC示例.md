```hccl_8p.json
{
"group_count": "1",
"group_list": [
{
    "group_name": "worker",
    "device_count": "8",
    "instance_count": "1",
    "instance_list": [{"devices":[ 
    {"device_id":"0","device_ip":"192.168.10.3" },
    {"device_id":"1","device_ip":"192.168.20.3" },
    {"device_id":"2","device_ip":"192.168.30.3" },
    {"device_id":"3","device_ip":"192.168.40.3" },
    {"device_id":"4","device_ip":"192.168.10.4" },
    {"device_id":"5","device_ip":"192.168.20.4" },
    {"device_id":"6","device_ip":"192.168.30.4" },
    {"device_id":"7","device_ip":"192.168.40.4" } ],
    "pod_name":"another","server_id":"127.0.0.1"}]
}
],
"status": "completed"
}
```

docker run -it -v /root/sample_tf:/data/ 
--device=/dev/davinci0 
--device=/dev/davinci1 
--device=/dev/davinci2 
--device=/dev/davinci3 
--device=/dev/davinci4 
--device=/dev/davinci5 
--device=/dev/davinci6 
--device=/dev/davinci7 
--device=/dev/davinci_manager 
--device=/dev/devmm_svm 
-v /var/log/npu/conf/slog/slog.conf:/var/log/npu/conf/slog/slog.conf 
-v /var/log/npu/slog/container/7:/var/log/npu/slog 
-v /var/log/npu/profiling/container/7:/var/log/npu/profiling 
-v /var/log/npu/dump/container/7:/var/log/npu/dump 
-v /var/log/npu/docker_slog_7:/usr/slog 
--device=/dev/hisi_hdc 
-w /data/Resnet50_HC tf:1.15-0728 bash

