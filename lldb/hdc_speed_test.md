## run hdc_std client parallelly
- 启动3个hdc client分别下载test.data数据
- test.data数据大小
```
-rw-r--r-- 1 root root 117440512 2017-08-08 13:32 /data/data/test.data
```

```go
package main

import (
	"log"
	"os"
	"os/exec"
	"sync"
)

var command string = "hdc_std file recv /data/data/test.data /tmp"

func execHdc() {
	cmd := exec.Command("sh", "-c", command)
	cmd.Stderr = os.Stderr
	cmd.Stdout = os.Stdout
	cmd.Stdin = os.Stdin

	err := cmd.Run()
	if err != nil {
		log.Fatal(err)
	}
}

func main() {
	var wg sync.WaitGroup

	for i := 0; i < 3; i++ {
		wg.Add(1)
		go func() {
			execHdc()
			wg.Done()
		}()
	}

	wg.Wait()
}
```

- result:
```
FileTransfer finish, Size:117440512 time:21212ms rate:5536.51kB/s
FileTransfer finish, Size:117440512 time:21213ms rate:5536.25kB/s
FileTransfer finish, Size:117440512 time:21218ms rate:5534.95kB/s

real    0m21.269s
user    0m0.015s
sys     0m0.026s
```

## download one file which size is 3 times of test.data by one client
- 下载命令
```
hdc_std shell 
 > cat test.data test.data test.data > test.data.3
time hdc_std file recv /data/data/test.data.3 /tmp
```
- 结果
```
FileTransfer finish, Size:352321536 time:29314ms rate:12018.88kB/s

real    0m29.344s
user    0m0.000s
sys     0m0.006s
```

## try load module parallelly
- modify loading LoadAllCurrentModule by ThreadPool with 3 workers
```diff
diff --git a/lldb/source/Plugins/DynamicLoader/POSIX-DYLD/DynamicLoaderPOSIXDYLD.cpp b/lldb/source/Plugins/DynamicLoader/POSIX-DYLD/DynamicLoaderPOSIXDYLD.cpp
index 3146c2ae296..ac90e2f7c48 100644
--- a/lldb/source/Plugins/DynamicLoader/POSIX-DYLD/DynamicLoaderPOSIXDYLD.cpp
+++ b/lldb/source/Plugins/DynamicLoader/POSIX-DYLD/DynamicLoaderPOSIXDYLD.cpp
@@ -23,6 +23,7 @@
 #include "lldb/Target/ThreadPlanRunToAddress.h"
 #include "lldb/Utility/Log.h"
 #include "lldb/Utility/ProcessInfo.h"
+#include "llvm/Support/ThreadPool.h"
 
 #include <memory>
 
@@ -607,22 +608,26 @@ void DynamicLoaderPOSIXDYLD::LoadAllCurrentModules() {
   m_process->PrefetchModuleSpecs(
       module_names, m_process->GetTarget().GetArchitecture().GetTriple());
 
+  llvm::ThreadPool pool(llvm::hardware_concurrency(3));
   for (I = m_rendezvous.begin(), E = m_rendezvous.end(); I != E; ++I) {
-    ModuleSP module_sp =
-        LoadModuleAtAddress(I->file_spec, I->link_addr, I->base_addr, true);
-    if (module_sp.get()) {
-      LLDB_LOG(log, "LoadAllCurrentModules loading module: {0}",
-               I->file_spec.GetFilename());
-      module_list.Append(module_sp);
-    } else {
+    pool.async([&](const FileSpec &file_spec, addr_t link_addr, addr_t base_addr) {
       Log *log(GetLogIfAnyCategoriesSet(LIBLLDB_LOG_DYNAMIC_LOADER));
-      LLDB_LOGF(
-          log,
-          "DynamicLoaderPOSIXDYLD::%s failed loading module %s at 0x%" PRIx64,
-          __FUNCTION__, I->file_spec.GetCString(), I->base_addr);
-    }
+      ModuleSP module_sp =
+          LoadModuleAtAddress(file_spec, link_addr, base_addr, true);
+      if (module_sp.get()) {
+        LLDB_LOGF(log, "LoadAllCurrentModules loading module: %s", file_spec.GetFilename().GetCString());
+        module_list.Append(module_sp);
+      } else {
+        LLDB_LOGF(
+            log,
+            "DynamicLoaderPOSIXDYLD:: failed loading module %s at 0x%" PRIx64,
+             file_spec.GetCString(), base_addr);
+      }
+     }, I->file_spec, I->link_addr, I->base_addr);
   }
 
+  pool.wait();
+
```
- Before change, cost **21s**,   after merge patch, cost **15s** while attach