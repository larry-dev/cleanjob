# cleanjob
一个golang实现的分布式计划任务内库，依托于 "github.com/robfig/cron/v3"的单机实现扩展了一个分布式实现

## 目录

- [安装](#install)
- [Example](#example)

## Install

Install the package with:

```bash
go get github.com/larry-dev/cleanjob
```

Import it with:

```go
import "github.com/larry-dev/cleanjob"
```

## Example

Please check the example folder for details.

```go
package main

import (
	"flag"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/larry-dev/cleanjob"
	"time"
)

var (
	addr string
)

func init() {
	flag.StringVar(&addr, "addr", ":8080", "服务端口号")
	flag.Parse()
}
type TestJob struct {
}
func (j *TestJob) Execute(ex *cleanjob.ExecuteContext) error {
	fmt.Println(ex.Job.String())
	time.Sleep(10 * time.Second)
	fmt.Println("执行完成")
	return nil
}

type Req struct {
	Ex   string `json:"ex"`
	Key  string `json:"key"`
	Data string `json:"data"`
}
func main() {
	men, err := cleanjob.NewRedisStore("127.0.0.1:6379")
	if err != nil {
		panic(err)
	}
	mange := cleanjob.NewManage(men)

	mange.On("game", &TestJob{})
	app := gin.Default()
	app.POST("job/add", func(c *gin.Context) {
		var req Req
		if err := c.BindJSON(&req); err != nil {
			return
		}
		job := cleanjob.NewJob(req.Ex, req.Key)
		job.WithData([]byte(req.Data))
		job.WithCronTime(time.Now().Add(10 * time.Second))
		if err := mange.Save(job); err != nil {
			c.JSON(500, gin.H{
				"err": err.Error(),
			})
			return
		}
		c.JSON(200, gin.H{})
	})
	app.POST("job/del", func(c *gin.Context) {
		var req Req
		if err := c.BindJSON(&req); err != nil {
			return
		}
		job := cleanjob.NewJob(req.Ex, req.Key)
		job.WithData([]byte(req.Data))
		job.WithCronTime(time.Now().Add(10 * time.Second))
		if err := mange.Delete(job); err != nil {
			c.JSON(500, gin.H{
				"err": err.Error(),
			})
			return
		}
		c.JSON(200, gin.H{})
	})
	mange.Run()
	app.Run(addr)
}

```

