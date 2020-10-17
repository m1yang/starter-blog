---
title: '接口自动化测试'
subtitle: '借助工具postman'
summary: '讲解postman如何通过csv文件实现接口自动化的测试'
authors: [admin]
tags: []
categories: [测试]
date: "2020-04-05T00:00:00Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: true
---

> 讲解postman如何通过csv文件实现接口自动化的测试，以优惠券接口作为示例
> 主要讲解：
> 一、了解需求数据流，抓包导入postman
> 二、编写脚本解决接口间数据依赖问题
> 三、分析数据依赖的情况

背景：优惠券新增铁路&amp;交通的校验逻辑

# 获取接口

通常来说，就是进入需要抓接口的页面，打开浏览器的开发者模式--&gt;选择network--&gt;勾选preserve log--&gt;过滤类型选择XHR，然后在页面上走完创建优惠券的流程

以下是具体操作步骤：

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/PYY6-Dmzt237ofAEI1HJUMjG.png)

提交后，能看到抓取的请求信息

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/8gEdYO2a4uN8X4tY6diGgPTI.png)

能够得到3个请求，一个create是创建优惠券的，一个approve是审核通过优惠券的，最后一个是获取优惠券信息的。在接口层面不做进一步校验的话，获取优惠券信息的接口可以先忽略。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/tkXZbSFLJcrhZ9ZsfhPQr2_e.png)

将需要的接口复制，导入到postman。上述两个接口，就能直接在postman中直接创建优惠券了。而这次的需求中，新增了policy内容，所以将优惠券加入policy选项还需要继续抓包。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/aX9E0sjPE5Rbkp2t4HFGiCHT.png)

在创建policy的过程中，能观察到，供应商信息会提前发请求获取数据，而线路信息同样需要请求数据，这一部分属于静态数据获取的范围内，如果数据量不大的话，可以直接拿固定的数据而不需要去请求接口获取。上述接口中找到创建policy的接口，查看它的传参信息。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/5nW00eZXa3tml0vnaeKo0RF1.png)

从接口信息中可以知道，除了routes的获取，其他静态信息可以直接使用一些固定的数据。同时要从优惠券创建接口中找出policy是如何添加进去的。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/2uAeLQwSkf6s_DMpVgIrn-Dx.png)

transfer_policy_info中通过绑定policy_id来将policy信息和优惠券绑定起来，这个信息可以从新旧接口传值比对得出，更好是直接找开发了解。将上述接口导入postman并整理后大概会是以下5个接口，配置policy依赖数据、创建policy、创建优惠券、优惠券审核以及生成优惠券code的定时任务。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/cVEC0m5EWMtE9PlysLMK_Pny.png)

依次调用这几个接口就能生成一个绑定了policy的优惠券了

有一个关键点需要注意的是admin是Google OAuth校验，目前没有一个很elegant的方法获取登录态

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/05ubIkY9qJ06EADDDnnJtZ9f.png)

所以强行获取cookies里面的认证信息，保存的postman里面临时使用。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/nSZKrRo-npYsKCLhWHkwVEmF.png)

# 数据依赖

一个接口需要传递的参数，很多时候都来源于其他接口的响应结果，这些数据一般分两类，一是基本上不会有太多改变的静态数据，一是会根据请求实时变化的动态数据。

静态数据的依赖问题很好解决，取一部分会使用的用文本存储起来就OK。但是动态数据，尤其是类似于ID这样的字段，无法提前预设、存储，只能实时的获取，就需要通过脚本来实时获取了。

来看这次需求最主要的接口，policy创建接口，[相关文档](https://orion.xxx.io/interface/interfaces/detail/21/497/6922/show)

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/Zex88Vtl0FYIoH8Q2FdpartG.png)

## 静态数据

从文档和对应前端页面，可以知道只有 name 字段是随便输的。departure_date、passenger_num这样的字段，前端通过下拉框限定了输入。seat_classes、operator_ids这样的字段有对应的接口可以请求到数据。这些字段的数据基本上可以提前准备，写入到csv文件中

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/QvAUxMVXs14Az-brCczgo61a.png)

以{{variableName}}的形式使用变量

## 动态数据

从请求体中，可以看到routes字段内传的origin的值是ID的形式，这样的话，即使提前准备，也会十分麻烦，需要我们把paris、london这样的值通过 `/v1/readminserv/staticdata/cities/search` 接口找到对应ID提前存储下来。为了简化这一部，所以写了脚本自动转换站点为站点ID，脚本：[collection](https://drive.google.com/open?id=1KEcavu-deN15_3hVJ-8TN0aJo752F1tb)、数据文件：[csv](https://drive.google.com/open?id=13hAny_yOryhygfs5pd0matZszE6gJQ1g)

代码中主要有这么几个知识点比较有意思

### Postman作用域

使用postman的pm.sendRequest函数时发现数据只能在函数内部生效，父级和全局的变量都无法生效。所以这里做了一个类似读写分离的操作，先复制一个routes数据为routesID

```
var _ = require("lodash");
routesID = _.cloneDeep(routes);
```

然后在pm.sendRequest函数内部将站点转换为ID，并存储在环境变量中

```
    pm.sendRequest(REQ, function(err, res) {
        let puid = res.json().result.cities[0].id;
        routesID = routesID.map(d => {
            switch(true){
                case d.origin === cityName:
                    d.origin = puid;
                    break;
                case d.destination === cityName:
                    d.destination = puid;
                    break;
            }
            return d;
        });
        pm.environment.set("routesID", JSON.stringify(routesID));
    });
```

这样就可以读取routes的数据，不修改routes的情况下，将ID写入另一个变量并存入环境变量中去（修改了routes，下次调用接口时就会拿ID查ID了），当然这里代码写的不是很严谨，还有很多可以优化的地方，但是主要是体现postman作用域的问题。

### 读取数据文档和环境变量

这一块是因为会出现有需要接口能支持读取csv变量，如果没有就要能读取环境变量的情况。

```
switch(true){
    case typeof(pm.iterationData.get("routes")) !== "undefined" && pm.iterationData.get("routes") !== "null":
        routes = JSON.parse(pm.iterationData.get("routes"));
        break;
    case typeof(pm.environment.get("routes")) !== "undefined":
        routes = JSON.parse(pm.environment.get("routes"));
        break;
}
```

所以通过js的typeof函数判断变量是否为undefined来确定是否能获取到数据。能取到数据就用，不能就不用。

### 支持的数据格式

调试参数的时候老是报错，因为如果你存环境变量里的值不是字符串。那么就需要

```
pm.environment.set("variableName", JSON.stringify("xxx"));
```

postman环境变量只存字符串，所以你要用环境变量赋值，经常需要时不时解析一下才能用，不然你的变量就存的是字符串了

```
variable = JSON.parse(pm.environment.get("variableName"));
```

# 使用总结

Postman并不适合连续调用多个API及复杂参数传递的问题，也就是我们常说的场景用例。重要的是有些接口场景涉及定时任务，这更是灾难。但是对于简单的单个API调用，查看响应是否预期，还是挺方便的。还有是，不做接口自动化，只是通过postman来调接口造数据也是很方便的。

建议是**写一些简单的脚本，把collection积累起来。**像批量生成优惠券、批量下单，慢慢积累起来后还是能提提高不少生产力的。

接口自动化需要能解决一些复杂问题，有一定灵活性，还要能提升测试效率的话，最好还是从**代码下手，开发使用什么语言开发，测试使用什么语言测试。**

当然，现有的API测试框架已经很成熟了，还可以通过配置文件来添加用例。我们在orion系统上也有类似的框架。这一块感兴趣的话推荐阅读 [httprunner](https://cn.httprunner.org/)、[httptest](https://github.com/qiniu/httptest) ，都有不错的文章教程。
