Job资源分配
====================================================


* Job资源分配： 默认最小分配 1 核CPU,100M内存，无上限；

- grafana 关于job 的监控都是该Job的使用情况；需要注意的是当前job的资源使用是没有上限限制的；
- grafana 监控中Job Status与Node Status单位粒度不一致。
    + Job Status中 CPU使用率为核心使用数，内存是MB
    + Node Status中 CPU使用率为百分比，内存是GB

- 在后台配置文件可以修改 src/Jobs_Templete/pod.yaml.template中resources.limits
- 配置后重启jobmanager 组件就可以了

