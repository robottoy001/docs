fix longtime waiting while attach remote process

### related issue 
https://gitee.com/openharmony/third_party_llvm-project/issues/I5IG24?from=project-issue

### Description:
lldb take longtime(almost >15s) to attach remote process.after investigating, we found the root casue: 
1. lots of reconnection in communication between host & device, because hdc client execute command in temporary session.
2. we should replace "getprop ro.build.version.sdk" by "param get const.ohos.apiversion" to get the correct sdk version, otherwise return INVALID_SDK_VERSION.

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

