# lldb 调试器

标签: lldb 调试

---

## lldb介绍
## lldb日志开关
```
log enable lldb all -T -f /path/to/log

```

## lldb 调试流程
### 链接调试服务器
在设备端先启动lldb-server，对于通讯协议区分unix和tcp套接字方式，
以tcp套接字为例
```
# server side
./lldb-server platform --server --listen "*:5566"

# client size, in .lldbinit or interactively
# explictly load init when start: ./lldb -s ~/.lldbinit

platform select remote-ohos
platform connect connect://:5566

```

以unix套接字为例
```
# server side
# unix address: /data/data/patch.sock 
./lldb-server platform --server --listen unix-abstract:///data/data/patch.sock

# client size, in .lldbinit or interactively
# explictly load init when start: ./lldb -s ~/.lldbinit

platform select remote-ohos
platform connect unix-abstract-connect:///data/data/patch.sock

```
### 测试与服务器链接速度
```
process plugin packet speed-test

```

