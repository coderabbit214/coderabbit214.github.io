# Golang GC调优


# Golang GC调优

## 问题

在一个服务中需要开多协程大量请求外部接口获取数据，使用docker进行部署，会出现` exited with code 137 `OOM问题，检查代码后发现并无内存泄漏问题，我在代码中每一千次请求添加一个`runtime.GC()`，进行手动GC，判断为GC不及时导致的OOM。

## 核心代码

```go
package main

import (
	"sync"
	"time"
)

type Info struct {

}

var infoChan chan Info

func main() {
	// 可认为长度无限
	urls := []string{"xx"}
	infoChan = make(chan Info)
	ch := make(chan struct{}, 100)
	wg := &sync.WaitGroup{}

	for _,url := range urls {
		wg.Add(1)
		ch <- struct{}{}
		go do(url, ch, wg)
	}

	go readInfo()

	wg.Wait()
	time.Sleep(6 * time.Second)
}

func do(url string, ch chan struct{}, wg *sync.WaitGroup) {
	defer wg.Done()
	// 请求第三方接口 获取数据
	
	infoChan <- Info{
		
	}
	
	<-ch
}
func readInfo() {
	infoList := make([]Info, 0)
	timer := time.NewTimer(5 * time.Second)
	for {
		select {
		case info := <-infoChan:
			infoList = append(infoList, info)
			if len(infoList) > 100 {
				// Save
        infoList = make([]Info, 0)
				timer.Reset(5 * time.Second)
			}
		case <-timer.C:
			if len(infoList) > 0 {
				//Save
        infoList = make([]Info, 0)
			}
			timer.Reset(5 * time.Second)
		}
	}
}

```



## GOGC

https://tip.golang.org/doc/gc-guide#GOGC

*Target heap memory = Live heap + (Live heap + GC roots) \* GOGC / 100*

Go为开发者只暴露了一个调节GC的参数，这个参数可以通过在启动Go之前指定环境变量`GOGC`来控制，他控制的是触发GC的阈值，是一个百分比。当Go新创建的对象所占用的内存大小，除以上次GC结束后保留下来的对象占用内存大小，所得到的比值大于`GOGC`设置的阈值时，就会触发一次GC。如果没有指定这个变量，则默认值是100。也就是说，默认情况下，当目前占用内存是上次GC结束后占用内存的一倍时，才会触发一次GC。

## 调优

优化思路: 降低触发GC的阈值，避免等待GC的对象占用过多的内存

以下为测试时`GOGC`设置不同时第一次发生GC的内存占用

### `GOGC=100`

![100](https://images-jsh.oss-cn-beijing.aliyuncs.com/coderabbit/2024/03/21/20240321-183224.png)

### `GOGC=50`

![50](https://images-jsh.oss-cn-beijing.aliyuncs.com/coderabbit/2024/03/21/20240321-183242.png)

### `GOGC=25`

![25](https://images-jsh.oss-cn-beijing.aliyuncs.com/coderabbit/2024/03/21/20240321-183302.png)

## 结论

通过上述数据分析，可以根据容器内存限制选择合适的`GOGC`参数进行设置。
