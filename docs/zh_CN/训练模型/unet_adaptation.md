# UNET 裸机环境变量配置


```bash
#mindspore 
# control log level. 0-DEBUG, 1-INFO, 2-WARNING, 3-ERROR, default level is WARNING.
export GLOG_v=2

# lib libraries that the run package depends on
export LD_LIBRARY_PATH=/usr/local/Ascend/add-ons/:/home/HwHiAiUser/Ascend/ascend-toolkit/latest/fwkacllib/lib64:/usr/local/Ascend/driver/lib64/common:/usr/local/Ascend/driver/lib64/driver:/home/HwHiAiUser/Ascend/ascend-toolkit/latest/opp/op_impl/built-in/ai_core/tbe/op_tiling

export TBE_IMPL_PATH=/home/HwHiAiUser/Ascend/ascend-toolkit/latest/opp/op_impl/built-in/ai_core/tbe            # TBE operator implementation tool path
export ASCEND_OPP_PATH=/home/HwHiAiUser/Ascend/ascend-toolkit/latest/opp                                       # OPP path
export PATH=/home/HwHiAiUser/Ascend/ascend-toolkit/latest/fwkacllib/ccec_compiler/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin                 # TBE operator compilation tool path
export PYTHONPATH=${TBE_IMPL_PATH} 
```