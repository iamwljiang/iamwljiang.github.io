---
layout: post
title:  "数据查询方式"
categories: bigdata
tags:  query distributed 
author: iamwljiang
---

* content
{:toc}

本文将介绍分布式系统中常见的数据查询方法，也以此作为一个记录，后续如果接触新的内容会初步更新此文档。


#客户单查询

所谓的客户端查询是查询的控制逻辑在客户端进行控制，负责串联整个查询工作流。

### 客户端简单查询

抛开客户端缓存而言，数据路由信息的维护绝大多数而言不会放在客户端实现，因此需要从服务端获取进行定位，大致过程如下：

* 根据查询的具体key，发送请求到维护路由表的服务端，获取key的信息或位置

* 根据查询的key所在的具体数据维护位置发起读取请求

* 实际key对应数据维护的服务器获取实际数据返回给客户端


### 客户端复杂查询

客户端如果需要进行范围或做全局扫描而言，则和简单查询会有所差异，差异在于需要持续获取路由表信息，以及客户端需要对数据进行过滤汇聚，相当于组织一次之后在返回数据给调用方。

## 服务端查询

### 服务端简单查询

这个很好理解，对于单机系统，比如mysql，客户端只是发起请求，但是真正执行的是mysql服务器。当然mysql对于查询而言会自己解析请求，优化查询任务，执行等。


### 服务端并行查询
和单机的区别是在于对一个多机的系统中，也会存在轻客户端重服务端场景。
像solr，greenplum，hadoop mr等，服务端接受客户端请求，生成查询树，逐步执行，汇聚，之后才返回数据给客户端。这里还涉及一个性能问题，是单机接受查询任务还是多机接受查询任务，是否在并行查询的同时可以支持高并发处理。





