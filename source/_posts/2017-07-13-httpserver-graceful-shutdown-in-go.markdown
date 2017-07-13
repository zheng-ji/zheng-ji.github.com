---
layout: post
title: "HTTPServer 优雅关闭"
date: 2017-07-13 09:04
comments: true
categories: Programe
description: graceful shutdown
---

和同事聊到了服务在需要关闭的时候该如何优雅退出，顺藤摸瓜挖掘了Go1.8的特性。Go 1.8起新增了优雅退出 HTTPServer 的特性，也就是大家经常提到的 GraceFul ShutDown。

```go
// src/net/http/server.go
// Shutdown gracefully shuts down the server without interrupting any active connections. Shutdown works by first closing all open listeners, then closing all idle connections, and then waiting indefinitely for connections to return to idle and then shut down. If the provided context expires before the shutdown is complete, then the context's error is returned.

func (srv *Server) Shutdown(ctx context.Context) error {
    atomic.AddInt32(&srv.inShutdown, 1)
    defer atomic.AddInt32(&srv.inShutdown, -1)

    srv.mu.Lock()
    lnerr := srv.closeListenersLocked()
    srv.closeDoneChanLocked()
    srv.mu.Unlock()

    ticker := time.NewTicker(shutdownPollInterval)
    defer ticker.Stop()
    for {
        if srv.closeIdleConns() {
            return lnerr
        }
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
        }
    }
}
```


从文档注释得知，server.Shutdown 首先关闭所有 active 的 listener，以及所有处于 idle 状态的 Connections，然后无限等待那些处于 active 状态的 connection 变为 idle 状态后，关闭他们，Server退出。

如果有一个 Connection 依然处于 active 状态，那么 server 将一直 block 在那里。
Shutdown 接受一个 Context 参数，调用者可以通过 Context 传入一个等待的超时时间。
一旦超时，Shutdown 将直接返回。对于仍然处理 active 状态的Connection，就无能为力了。
所以 Shutdown 超时时间尽量要比链接处理的时间长。

了解完原理，我们用例子来感受一下这个特性。

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"time"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("/", func(c *gin.Context) {
		time.Sleep(3 * time.Second)
		log.Printf(http.StatusOK, "Handle request success")
	})

	srv := &http.Server{
		Addr:    ":8080",
		Handler: router,
	}

	go func() {
		if err := srv.ListenAndServe(); err != nil {
			log.Printf("listen: %s\n", err)
		}
	}()

	quit := make(chan os.Signal)
	signal.Notify(quit, os.Interrupt)
	<-quit
	log.Println("Shutdown Server ...")

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := srv.Shutdown(ctx); err != nil {
		log.Fatal("Server Shutdown:", err)
	}
	log.Println("Server exist")
}
```

代码中，每个请求都等待3秒才完成，使用信号来捕捉程序退出。退出时，HTTPServer 等待5秒来"善后"。我发起`curl localhost:8080`来测试，随即按下 Ctrl+C 退出程序，结果显示，服务器坚持在处理完这个 HTTP 请求才退出。

```
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
Handle Request success
[GIN] 2017/07/12 - 20:30:47 | 200 |  3.000385597s | 127.0.0.1 |   GET     /
^C  //终端输入Ctrl+C
Shutdown Server ...
listen: http: Server closed
Handle Request success //在接收到关闭信号时，依然保证正在处理的请求正常处理完
[GIN] 2017/07/12 - 20:30:53 | 200 |  3.000360362s | 127.0.0.1 |   GET     /
Server exist
```
