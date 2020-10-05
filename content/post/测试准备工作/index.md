---
title: '测试准备工作'
subtitle:
summary: 
authors:
- miyang
tags:
- test
categories:
- 测试
date: "2020-04-05T00:00:00Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: true
---
# 准备工作

第一步：命令行执行 `ssh-keygen` 生成属于自己的私钥和公钥，具体内容可参考 [生成 SSH 公钥](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5) 。这样就有了  `.ssh`	目录统一配置服务器登录相关的信息了。

第二步：找相关人员获取测试环境连接密钥，注意密钥需通过邮件发送。一般下载的密钥是开发权限，所以得先修改文件使用权限。举例

```
chmod 400 ~/.ssh/Ops-Test-Connect-DB.pem
```

第三步：尝试连接数据库，参照[文档](https://klook.slab.com/posts/%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83%E7%9B%B4%E8%BF%9E%E6%B5%8B%E8%AF%95%E6%95%B0%E6%8D%AE%E5%BA%93%E6%96%87%E6%A1%A3-mfdfr3ls)

第四步：找相关人员授权jump机登录权限，web页面登录有具体文档，命令行登录参照[文档](https://klook.slab.com/posts/%E7%BB%95%E8%BF%87%E4%BA%8C%E6%AC%A1%E9%AA%8C%E8%AF%81%EF%BC%8C%E7%9B%B4%E8%BF%9E%E6%B5%8B%E8%AF%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%96%87%E6%A1%A3-ciwqp352)

# 宿主机信息

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/HC_ooI0jm5ZkZoGNSJWMOWK0.png)

目前测试环境一共两台主机，对应分配了容器 t1~t80 。假如自己搭建环境分配的容器是t1，就进入测试环境-1，可以输入 docker ps	查看正在运行的容器。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/h8OuYoHRLGil2D_hyo1vpl1z.png)

进入对应的容器，可以输入指令 `t1` ，因为已经做了 `alias t1='docker exec -ti t1 bash'`	这样的配置。

如果没有看到分配的容器，可以输入 `docker ps -a` ，查看所有容器进程，即使容器没有在运行中。没有自己的容器，就重新创建，容器状态为exited，输入 `docker start t1`	重启容器。

一般来说在宿主机上不用太多操作，都交给专门的人来维护。

目前我们通过orion平台来创建测试环境

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/gYOwZ2bmiuJuV6l44tGe_OY-.png)



# 容器信息

## 常用目录：

```
/opt/testing-config # 测试环境配置目录
/opt/release # 前后端各个项目目录
/srv # 前端项目目录
/tmp # 临时文件，后端日志存放目录
/etc/nginx # 服务端口、路由配置目录
```

## 常用指令：

```
cat /etc/nginx/backend/klook-railway.conf | grep /v1/reappserv -5 # 查看请求接口占用了哪个端口
grep -hr "usrcsrv\s" /etc/nginx/backend/ -A 5 # 查看整个目录下的文件是否有配置对应接口
netstat -plnt|grep 10437 # 查看端口监听的服务
ps -aux|grep reappserv # 查看服务启动的相关配置项
tailf /tmp/reappserv.log # 查看服务日志
supervisorctl status # 后端服务状态
kcli --help # 服务部署指令
pm2 status # 前端服务状态

num=`echo $KL_TESTING_ENV_NAME | tr -cd "[0-9]"`; cd /opt/testing-config/init/ && git checkout master && git pull && bash /opt/testing-config/init/start_nginx.sh $KL_TESTING_ENV_NAME.test.klook.io admin$num.test.klook.io # 更新nginx
```

# 定位错误

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/n1jbmxR7ORR4NfdnKXDjKY7M.png)

通常这种情况是后端服务没能请求到

登录上去查看可以看到，类似以下这种

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/lx3yWfJ_QYFBBChdsS1-jj2g.png)

因为某种原因导致后端服务启动失败了，所以前端请求不到对应的接口。这里是因为找不到docker_mysql5.7，查看 `/etc/hosts`	发现确实少了这样一个配置，加上去就可以了。

**具体操作步骤拆分：**

## 接口请求

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/GxULCDmmxAblpLy6Q5te-mhH.png)

web页面通过开发者工具(option + command + i)，查看页面请求的接口，如上图 `/v3/usrcsrv/`，再登录到对应环境中

- 找到报错接口监听在哪个端口，通过grep指令筛选nginx 里 backend目录下所有的请求端口，例如 &quot;usrcsrv&quot;

```
grep -hr "usrcsrv\s" /etc/nginx/backend/ -A 5
```

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/v-wbgvzd-EgD0nYMY8bp9BnT.png)

## 服务进程

- 通过端口号找到进程，会得到进程号及对应服务

```
netstat -plnt|grep 10003
```

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/6zi-90orerde0wZM0h9136QV.png)

## 启动信息

- 查看服务启动信息

```
ps -aux|grep 20132
```

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/KbdYiPHW4NT1P22BsfciB6Yt.png)

如果知道服务名称，也可以直接

```
ps -aux|grep usercommsrv
```

从进程启动命令中，可以知道服务的配置文件 `/opt/release/klook-appapi/usercommsrv/service.conf` 和服务的日志目录 `/opt/release/klook-appapi/usercommsrv/logs`

## 查看日志

- 查找错误

如果服务没能启动，可以直接执行启动文件，这样会直接把日志打印到终端

```
/opt/release/klook-appapi/usercommsrv/usercommsrv
```

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/atOSWDntHiDVbb9wd_t0qEB7.png)

如果服务是启动的，可以查看日志

```
tailf /opt/release/klook-appapi/usercommsrv/logs/usercommsrv.INFO
```

在前端再次请求接口，就能获取到请求响应的信息了

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/dFUZLwvWOj2Sh_PBe7lBgXH1.png)

类似的，前端能请求到接口，但是接口报错，这样的情况，也可以通过上述步骤排查错误

当我们对接口及后端服务极其熟悉之后，定位到是后端错误，直接通过`/tmp/`	目录下的日志来定位错误

```
tailf /tmp/usercommsrv.log
```

根据日志提供的信息，再进一步判断是哪里出错了

## 最佳实践

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/GxULCDmmxAblpLy6Q5te-mhH.png)

把请求中的 **Request URL:** http://t2.klook.io/v3/usrcsrv/xxx-xxx-xxx 和

**X-Klook-Request-Id:**cde9940 ，发给负责这个服务的后端开发

如果这还不够，那么

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/Y0Utelrg135V7MiakhupvD-g.png)

把这一整个复制发过去

# to be continued
