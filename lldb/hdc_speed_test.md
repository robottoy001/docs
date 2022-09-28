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

