---
layout: post
title: "Go写Benchmark"
date: 2013-12-08 21:54
comments: true
categories: Programe 
---

相比写C++/C，看到Go语言简直是感动，怀念那段写Go的时光。好久没有写Go了， 此次回眸，更觉得他可爱使用，这种感觉类似我第一次使用Python.最近在看一个Go的[IpLoc模块](https://github.com/slene/iplocll)，觉得帅!


用到的模块
----
Go 的 testing
使用的时候要 
```
import (
    . testing
)
```

demo source
-----------

```
func testSpeed () {
    r := Benchmark(func(b *B) {
        ips := []string{
            "0.0.0.0",
            "127.0.0.1",k
            "169.254.0.1",
            "192.168.1.1",
            "10.0.0.0",
            "255.255.255.255",
            "112.226.155.1",
            "121.18.72.0",
            "6.18.72.0",
            "200.18.72.0",
        }
        for i := 0; i < b.N; i++ {
            for _, ipAddr := range ips {
                iploc.GetIpInfo(ipAddr)
            }
        }
    })
    fmt.Println(r)
}
```

返回值
-----------
返回值,显示的是执行操作的次数，以及每次消耗的时间
```
200000         11494 ns/op
```


