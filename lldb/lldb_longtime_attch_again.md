load current modules of target parallelly to save time

### related issue 
https://gitee.com/openharmony/third_party_llvm-project/issues/I5IG24?from=project-issue
https://gitee.com/openharmony/third_party_llvm-project/issues/I5IFO9?from=project-issue


### Description:
lldb take longtime(almost >15s) to attach remote process. The root casue: 
- download module file(.so related to process which we debug) in low speed transmission
  After lldb host client connect the lldb server and attach to specific application, it will download the related so file from remote-device via platform special channel(hdc_std file recv).But the download speed is 8-10M/byte by single client, so if we download 129M files, we cost 12-15s. if we use multiple client (etc, 3) , speed will be increased to almost 30%.
  more investment, see [here](https://github.com/robottoy001/docs/blob/main/lldb/hdc_speed_test.md)
- lots of time to reading  sdk version (fix in [PR !70](https://gitee.com/openharmony/third_party_llvm-project/pulls/70))
1. lots of reconnection in communication between host & device, because hdc client execute command in temporary session.
2. we should replace "getprop ro.build.version.sdk" by "param get const.ohos.apiversion" to get the correct sdk version, otherwise sdk version is INVALID_SDK_VERSION and return 0

### How to reproduce 
```
# server side
# unix address: /data/data/patch.sock 
./lldb-server platform --server --listen unix-abstract:///data/data/patch.sock

# In client side, we can read lldb commands from .lldbinit or interactively
# explictly load init when start: ./lldb -s ~/.lldbinit

platform select remote-ohos
platform connect unix-abstract-connect:///data/data/patch.sock
attach 1503 # 1503 is pid of the appspwan application
```

### log before fixing


### log after fixing 
there is no reconnection in batch reading action. 

### need more testing on stability to check if loading is correct




