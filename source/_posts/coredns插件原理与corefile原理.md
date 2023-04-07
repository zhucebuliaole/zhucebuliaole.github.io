---
title: coredns插件原理与corefile原理
date: 2023-03-06 20:43:23
tags:
---
# 什么是coredns？
coredns是谷歌开发并开源的一款dns解析服务器。其使用golang开发，提供了插件跨平台等特性。
# coredns插件
## 注册插件
coredns框架会在初始化时对插件进行注册，注册过后的插件可以被配置和调用。这里用 log 插件举例说明, 在 init() 内注册 log 的 setup 装载方法, 然后使用 AddPlugin 注册一个 plugin.Plugin 方法. 其目的就是要实现中间件那种调用链。其调用遵循一个调用链的倒叙。  
代码位置: `plugin/log/setup.go`  
``` go
// 注册 log 插件, 把 log 的 setup 装载方法注册进去.
func init() { plugin.Register("log", setup) }

func setup(c *caddy.Controller) error {
    // Logger 实现了 plugin.Handler 接口了, 注册一个 plugin.Plugin 方法.
    dnsserver.GetConfig(c).AddPlugin(func(next plugin.Handler) plugin.Handler {
        return Logger{Next: next, Rules: rules, repl: replacer.New()}
    })

    return nil
}
```
引入plugin时你需要在plugin.cfg 中引入。  

```
external :
dump:github.com/miekg/dump
official :
sign:sign
```

## 调用插件
coredns在注册server时会倒序遍历已经注册的plugin（这也意味着plugin的顺序是非常重要的）。  
下图所示，当dns query到来之时，会按倒序调用plugin直到到达application。  
![plugin](https://s2.loli.net/2023/03/06/uJXzYbTO49LcIA2.png)
# corefile 的配置
Corefile是coerdns的配置文件，它定义了：  
1. 什么server监听什么端口和协议
2. 每个服务器负责哪个**zone**
3. 每个server中会加载哪些plugin
其格式为：

```
ZONE:[PORT] {
    [PLUGIN]...
}
```

这里的**ZONE** 定义此服务器的区域。 可选的 PORT 默认为 53，或 -dns.port 标志的值。  
**PLUGIN** 定义了我们要加载的插件。 这也是可选的，但是没有插件的服务器将只为所有查询返回 SERVFAIL。   
当运行corefile的时候，你可以通过`conf`参数指定corefile，若未找到corefile，则会加载`whoami plugin`。


你无法为同一个zone的相同端口注册一样的server。比方说如下的两个注册会报错：
```
. { }
. { }
```
error:  
```
2017/07/23 20:39:10 cannot serve dns://.:53 - zone already defined for dns://.:53
```
# coredns如何处理查询

不同的端口对应不同的server，换句话说，相同的**端口**，但是不同的**zone**处于同一个server。
如若我们写了如下的corefile：  
```
coredns.io:5300 {
  file /etc/coredns/zones/coredns.io.db
}

example.io:53 {
  errors
  log
  file /etc/coredns/zones/example.io.db
}

example.net:53 {
  file /etc/coredns/zones/example.net.db
}

.:53 {
  errors
  log
  health
  rewrite name foo.example.com foo.default.svc.cluster.local
  kubernetes cluster.local 10.0.0.0/24
  file /etc/coredns/example.db example.org
  forward . /etc/resolv.conf
  cache 30
}
```
如下图所示：  
![query-processing](https://s2.loli.net/2023/03/06/kFA3RXZLwqv7iSo.png)
由于我们在corefile中注册了两个端口，则生成了两个server。根据zone的不同，coredns会使用不同的插件链进行处理(插件链的顺序取决于注册顺序)。如果没有`zone` 匹配，`SERVFAIL` 则返回。  
如果一个查询命中了`Corefile`中的某个区域`（Zone）`，那么默认情况下，CoreDNS将停止搜索其他插件，并直接返回匹配的结果。这是因为，如果一个查询命中了特定的区域，那么其他插件可能无法提供更准确的答案，因此继续搜索其他插件可能是浪费资源。

但是，你可以通过使用`fallthrough`指令来改变这种默认行为。`fallthrough`指令告诉CoreDNS在匹配了一个区域之后继续搜索其他插件。使用`fallthrough`指令可以在某些情况下提供更灵活的查询策略，但也可能会导致查询时间增加。  
