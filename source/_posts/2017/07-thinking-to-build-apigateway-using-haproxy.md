---
title: 谈一谈使用 HAProxy 构建 API 网关服务的思路
date: 2017-06-17
tags: 
    - APIGateway
    - HAProxy
---

HAProxy 作为一个优秀的反向代理 ( Reverse Proxy ) 和负载均衡器 ( Load balancer ) ，有着广泛的应用场景，本文将主要介绍使用 HAProxy 构建 API 网关和 Service Router 的思路以及相关 HAProxy 特性，希望对读者有所启发~

在开始之前，我们先回顾并想一想，为什么我们需要 API 网关和 Service Router 呢，它的背景是什么？

## 背景

我们知道，当项目变大，开发人员变多，项目复杂度增加，任何一个人都很难清楚每一个细节，沟通协作成为开发效率的瓶颈，任何一个改动都可能牵一发而动全身，项目风险变大，迭代缓慢，导致开发速度跟不上企业发展需要。

为了解决这个问题，聪明的人们提出 SOA 的思想，把一个大的项目拆分成一个个独立的服务，服务与服务之间通过统一的方式进行通信，实现了服务的解藕，各个服务可以独立开发部署，近两年流行微服务则更进一步，拆的更细更激进。当然，需要明确的是，SOA 还解决了其他问题，但这不是本文的重点。

服务拆分解决了协作与开发效率的瓶颈，但也带来了新的问题，那就是服务管理的复杂度，具体涉及服务发现和服务网关（或者说 API 网关），API 网关正是本文的主题，进一步的服务发现也需要 API 网关支撑才行。  

有了这样的背景，我们来看如何利用 HAProxy 构建 API 网关吧~ 


## HAProxy 基本配置

工欲善其事，必先利其器，首先让我们先了解一下 HAProxy 的基本配置~

**配置文件结构**

HAProxy 的配置文件一般名为 haproxy.cfg，默认位于 /etc/haproxy 目录下，HAProxy 的配置文件有多个 Section 组成，总共有4种类型的 Section，它们分别是：

* global - 进程级别的参数，比如设定日志路径和用户组
* defaults - HAProxy 作为代理默认的一些参数，比如连接超时时间
* frontend - 定义如何接收和转发请求
* backend - 定义后端处理请求的服务器列表

**ACL（Access Control Lists）**

作为一个反向代理，我们需要根据请求内容，比如 Host 或者 Http Header 动态的选择请求转发的后端服务器，而 ACL 则是实现该需求的手段。

ACL 一般使用形式如下：

```    
acl is_rethinkphp hdr(Host) -i rethinkphp.com
```

这里的 is_rethinkphp 是 acl 的名称，hdr(Host) 表示从 Header 中提取 Host 用于比较，-i 表示忽略大小写，整条 acl 的含义是判断一个请求的 Host 是否是 [http://rethinkphp.com][1]。

这条 ACL 规则可以写成这样:
    
```    
acl is_rethinkphp hdr(Host),lower rethinkphp.com
```

其中 lower 是一个 converter，可以理解为一个函数，把 Host 转成小写再用于比较。

**监听网络请求**

有了上面的基础，我们就可以定义一个 frontend 来监听网络请求了：
    
```
frontend http_in
    bind *:80
    mode http # 代理服务器模式
    timeout http-keep-alive 1000
    acl is_rethinkphp hdr(Host) -i rethinkphp.com
    use_backend bk_rethinkphp if is_rethinkphp
```

这里我们监听了所有 80 端口的网络请求，并设置代理服务器模式为 http，然后判断如果请求的 Host 是 [http://rethinkphp.com][1]，就使用 bk_rethinkphp 这个 backend，表示匹配的请求都会转发到 bk_rethinkphp 所定义的后端服务器。

**定义后端服务**

网络监听和转发规则都设定好了，我们直接看后端 backend 的定义吧：
    
```
backend bk_rethinkphp
    mode http
    option forwardfor
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk GET /
    server server01  172.16.1.101:7789 check
    server server02  172.16.1.102:7789 check 
```

这里我们是 backend 关键字定义了名为 bk_rethinkphp 的后端服务器组，里面包含了 server01 和 server02 两台服务器，并通过 check 参数开启了健康检查，到这里一个 HAProxy 基本配置基本完成了，server01 和 server02 会接收来自 [https://rethinkphp.com][1] 的 http 请求。


## 路由功能实现

在上文中，我们实现了简单的按照域名进行请求转发的功能，但在真实的场景中，一个 API 网关可能需要处理成百上千的服务，也需要根据 URL 进行更灵活的转发配置，如果都一条条的编写 ACL 规则，难免过于繁杂，不好维护。这个时候我们可以使用 HAProxy 提供的 map 及强大的正则功能改造 frontend，从而实现基于 URL 的路由转发。核心配置如下：
    
```
frontend http-in
    bind *:80
    mode http
    timeout http-keep-alive 1000
    acl is_found base,map_reg(routes.map) -m found
    use_backend %[base,map_reg(routes.map)] if is_found
```

其中 routes.map 是一个包含两列的 key value 格式文件，内容如下：  

```
docs.rethinkphp.com/.*   bk_docs
www.rethinkphp.com/.*    bk_main
```

这样我们就可以直接在 routes.map 通过正则表达式建立 url 与具体后端服务器的对应关系，维护变得简单许多。

在这个例子中，base 是包含 host 和 path 的一个字符串，然后我们使用 map_reg 这个 converter，它的作用就是读取指定 key value 格式文件，并使用正在表达式对 key 进行匹配，如果匹配成功，则返回对应的 value，然后再进行下一步的比较。  

## Socket Commands

HAProxy 有一个非常独特的功能，那就是它的 Socket Commands，我们可以在 global section 中开启该功能： 
    
```
stats socket /run/haproxy.sock mode 660 level admin
```

有了该功能，我们可以通过 Socket 的方式动态控制 HAProxy 的行为，包括如下几个方面：

* 查询服务器运行状态以及各种统计信息
* 开启与禁用 frontend
* 配置后端服务地址或者修改相关信息
* 修改 map 和 acl 信息，可实现动态修改路由

有了这些功能，我们可以在不 reload HAProxy 服务器的情况下，修改部分配置。

遗憾的是，目前 HAProxy 还不支持 backend 和 server 的动态添加，涉及相关的修改只能修改配置文件并 reload 服务器。

## 服务化

利用好上文介绍的功能，我们就已经能够配置好一台 HAProxy 作为 API 网关，但配置都需要手动操作，容易出错不说也不方便业务方使用。

所有，更好的方式是把 API 网关作为一个独立的服务，该服务通过对外提供 API 接口来进行管理控制，更进一步，我们还能够与 CI/CD 集成，做到无缝的持续交付，提高协作效率，降低人为操作风险。

[HAproxy Router][2] 是我正在尝试开发的一个 HAProxy 服务化管理工具，目前（2017年5月30日）还处于起步阶段，感兴趣的读者可以关注，我会在后续的文章中分享进展，也欢迎提出你的想法和建议~

[1]: https://rethinkphp.com
[2]: https://github.com/rethinkphp/haproxy-router
