---
title: Golang中的服务限流
date: 2021-07-12 16:32:41
tags: Golang, 系统架构
categories: 系统架构
---


## 服务限流

服务限流指的是设置服务能支持得最大并发访问量，超过阈值则拒绝；限流阈值得设置通常分两种类型一种是Qps，一种是Concurrency。限流通常使用令牌桶来实现。

令牌桶算法是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。大小固定的令牌桶可自行以恒定的速率源源不断地产生令牌。当有新的请求的到来时需要先申请令牌桶取一定数量的令牌。取不到时返回错误，以此来实现限流。

<!--more-->

## Golang中的限流包time/rate

### 构造一个限流器

```golang
limiter := NewLimiter(10, 1);
```



```
1.第一个参数是 r Limit。代表每秒可以向 Token 桶中产生多少 token。Limit 实际上是 float64 的别名。
2.第二个参数是 b int。b 代表 Token 桶的容量大小。
```

用r来限制qps，用b来限制concurrentcy。


### Allow/AllowN


```golang
func (lim *Limiter) Allow() bool
func (lim *Limiter) AllowN(now time.Time, n int) bool
```


