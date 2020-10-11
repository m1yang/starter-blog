---
title: '需求分析-优惠券'
subtitle: 以优惠券为例
summary: 
authors:
- miyang
tags:
- test
categories:
- 工作
date: "2020-04-05T00:00:00Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: true
---

> 优惠券的逻辑十分复杂，但是理清楚了也可以很简单。

# 什么是优惠券

是给持券人的在购买商品或进行其他商业活动时某种特殊权利的优待券。可能是实物券，也可能是某种特殊约定的记号（如在互联网上采用的代码等）。—维基百科

这样的解释对我们了解优惠券帮助有限，下面深入解剖一下

![](https://static.slab.com/prod/uploads/posts/images/Bxt5Js0ogr-X0_ZQKbeFZQhC.png)

上图就是一张优惠券的样板，对此我们做一些术语上的统一：

| Coupon | 优惠券 |
| --- | --- |
| Code | 优惠码，用于兑换优惠券。一般一个Code用户只能兑换一次 |
| Description | 优惠券上的描述信息 |
| Generation method | 生成优惠券的方式 |
| Bulk （generation coupons） | 批量生成优惠券 |
| (create) Promo (code) | 生成一个有兑换次数限制的Code用于兑换优惠券 |
| Redemption (of coupon) | 兑换优惠券的条件 |
| （Coupon） Conditions | 使用优惠券的条件 |
| Batch | 一整批次的优惠券 |
| Version | 更新优惠券的版本 |

我们在admin后台来创建优惠券，有两种生成方式：1.Bulk 2.Promo 创建后整个优惠券就是一个Batch，Batch内每次更新都有一个对应优惠券的Version

创建的优惠券，用户需要符合Redemption才能兑换，兑换后优惠券上展示Code和Description。选择商品使用优惠券需要满足Conditions，才能被使用。

如果很难理解上面的内容，那么先略过，看下面的内容。

# 创建优惠券的页面

在admin优惠券创建分为3个页面

↓**Basic Information:mkt团队分析数据使用**

![](https://static.slab.com/prod/uploads/posts/images/SJuAJmBZm2lRI2h7gBsTEoYP.png)

↓**Code Setting：优惠券的一些基础设置**

![](https://static.slab.com/prod/uploads/posts/images/yE3gbANz2hkqyO_jX7glcNTf.png)

**↓Advanced Settings：优惠券的一些高阶设置**

![](https://static.slab.com/prod/uploads/posts/images/gAdgURRts7DxTxSB3Rto8il1.png)

**↓Discount Details：优惠券的折扣详情**

![](https://static.slab.com/prod/uploads/posts/images/RkWIT-ubRE8gPZZJ-vM5mef2.png)

这些页面按虽然是按编辑各个功能模块来划分的，但是直接这样看并不利于我们用户端测试理解，所以需要从如何使用优惠券的角度去细分上面3大页面。

# 优惠券模块细分

根据实际使用优惠券，划分为这几个模块：数据采集、生成方式、兑换条件、优惠券展示、使用条件、优惠方式。会将基本页面的字段分配到细分的各个模块，如果有字段重复，则表示在不同模块下起到不同的作用。（**Basic Information**基本上都是数据采集，不涉及功能，所以只划分出来不细讲）

## **生成方式（**Generation method**）**

**优惠券的生成方式**分两种： 一是批量生成优惠券 二是生成一个有兑换次数限制的Code用于兑换优惠券。涉及以下几个字段

![](https://static.slab.com/prod/uploads/posts/images/Ih-Vqy6K_osAbQBSkvCShfZu.png)

Batch Type就是生成方式选项

- 选择Coupon，则会是批量生成优惠券。后续三个字段控制Code的生成规则（随机的字母、数字、是否有固定的前缀）和优惠券的数量
- 选择Promo Code，则会是生成一个有兑换次数限制的Code。后续两个字段就是填写这个Code和兑换次数

虽然生成方式不同，但是都是根据数量决定了生成了多少张优惠券

TODO:在admin生成优惠券为防止操作意外，所以增加了审核功能，不同权限的账号设置优惠券有一定的额度限制。

## **兑换条件（**Redemption**）**

兑换条件是和生成方式强相关的，基本上是受到了相同字段的影响，但是额外还有一个兑换日期限制

![](https://static.slab.com/prod/uploads/posts/images/dd3ZHML7aNwJ3QYpoiNOHJ5z.png)

优惠券遵循一个基本的兑换规则：需要在可兑换时间段内，通过Code兑换优惠券，且**一个Code一个用户只能兑换一次**。但是根据生成方式的不同，优惠券的兑换逻辑也稍有差异，差异如下：

- 选择Coupon，兑换过后其他用户不能再使用该Code进行兑换。一个用户可以用不同Code兑换同一批次优惠券。
- 选择Promo Code，不同用户都可以通过该Code进行兑换。达到兑换次数后，其他用户不再能使用该Code进行兑换。

## 优惠券展示

在用户端主要有两个页面展示兑换后的优惠券：

- ↓个人中心，优惠券分**可用**和**不可用**两类，不可用的优惠券会提示其原因

![](https://static.slab.com/prod/uploads/posts/images/HjYOscqb8cZhuyJ-oWOwszC6.png)

- ↓支付页，展示的都是**可用优惠券**，但分**适用**和**不适用**两类，是否适用根据购买商品的相关信息来判断

![](https://static.slab.com/prod/uploads/posts/images/xTAIcYvyiChHM5G-tvTxTWqr.png)

对应admin配置字段

![](https://static.slab.com/prod/uploads/posts/images/kECV0tZ10kZkpW3BF-3Hrv7E.png)

Description目前并不关联具体商品信息，只能手动编辑需要呈现的描述。描述信息的多语言在admin的第一个页面控制。

![](https://static.slab.com/prod/uploads/posts/images/n6r92unfID6XdfHG-qjjdDHG.png)

增加对应语言选项后对应Description会增加所选语言的输入栏

## 使用条件（Conditions）

使用条件基本上是优惠券最为复杂的一块了，所以这里也拆成两部分来讲，一是用户限制 二是商品限制

### **用户限制**

![](https://static.slab.com/prod/uploads/posts/images/ErSQ-4bNLavv1_JVkxHoFnaE.png)

设置上述选项，会限制用户需要满足类似以下条件才能使用优惠券：支付方式、信用卡号、是否新用户、是否第一次下单、是否属于推广的城市、是否通过短信验证等。

第三页的用户使用平台也归为用户限制

![](https://static.slab.com/prod/uploads/posts/images/3ubN7jAWFJor-XMMrx8CMkd9.png)

还有一个比较特别的，通过(create) Promo (code)方式生成的优惠券有一个选项，**Repeat Use Per User**： **** 一批活动（0～all）发放的promo code 设置多次使用

对应场景：KFC会员卡，每个月可以用2次优惠购买早餐，1次只对应一个商品(unit)，下个月恢复优惠券的使用，直到优惠券到期。

注解：这是对promo code（优惠码）功能的拓展，是和**Batch coupon** 没有关系的，对**Batch coupon**来说如果一个活动用户想要再次享受优惠的话，不需要恢复使用过的优惠券，再兑换一张新的优惠券即可。

![](https://static.slab.com/prod/uploads/posts/images/-eCcGo5eRQecueImFmWcrMew.png)

这些都是用户可以通过调整自身情况来满足优惠券的使用条件

### **商品限制**

下面罗列的是商品属性相关来限制优惠券的使用

![](https://static.slab.com/prod/uploads/posts/images/GP3P7a93fWAWGnCZ-EzJ1sC1.png)

**↑Select Valid Activities** 虽然只有一个按钮，但是点开了以后的功能还是相当丰富的，差不多可以单独划出来一个页面来做了。这里将活动类型、活动属性、活动所属国家和城市等都作为使用条件来确保优惠券的正确投放。

↓有些薄利多销的活动可能本身就不适合使用优惠券，但是不能每次创建优惠券都去选择屏蔽一遍，所以设置了一个选项来控制这一类的活动是否能使用优惠券

（注意：**Affiliate Revenue Sharing** 使用这个优惠券的订单会同步到affiliate系统）

![](https://static.slab.com/prod/uploads/posts/images/Co2nFFftZYfneV25KDTcSVla.png)

**↓Usage Limit for Particular Activities ：**一批活动（0～all）有多少个unit是可以使用优惠的

对应场景：商家做推广活动，有400个商品（unit）是可以使用优惠券的，当有400个商品使用优惠券后其它的就作废了

注解：使用限制条件的核心在活动的unit上，**Bulk （generation coupons）**和**(create) Promo (code)**生成优惠券自身的逻辑不受影响。所以如果设置了时效，假如设置为1个月，那么1个月后**恢复的是unit使用数量，而不是使用过的优惠券**。

![](https://static.slab.com/prod/uploads/posts/images/6k6Nn9Rw6BCiDtknYQmvTidb.png)

铁路也会在这一块进行拓展，从这样细分的角度看Select Valid Activities、Usage Limit for Particular Activities以及后续的铁路业务是可以重构为一个统一的商品限制模块的。当然这也是比较理想的状态，现实是 后续会额外多个板块出来，再整理铁路附加的逻辑。

## 优惠方式

优惠券主要是两种优惠方式：一是 货币金额直接抵扣 二是 按一定百分比打折。都有满减和整批（Batch）优惠券金额限定，区别如下：

![](https://static.slab.com/prod/uploads/posts/images/U69wOfF1CHYTDhM6ew0BmUTS.png)

- 选择Cash，需要明确指定抵扣的货币及金额是多少
- 选择Percentage，设定百分比，但是可以设置抵扣金额上限

优惠券的金额抵扣可以精准到每个商品的单价，但是这里把商品限制和优惠价格计算耦合到了一起。总觉得这是个设计缺陷，所以还是按自己的理解来讲吧。

优惠券的活动，每个活动都有套餐和单价，这是个公有功能，而不是为单一类活动定制的功能，所以完全可以按照套餐的创建内容来设定优惠券的精准定位。

![](https://static.slab.com/prod/uploads/posts/images/vdDBjSrVPcYZnVWLZj3iehSl.png)

套餐和单价类型是所有活动公有的模块，所以指定是第几个套餐、什么类型，再加上商品限制就能实现优惠的精准定位了。

掌握了上述信息，在用户端，优惠券需要测试什么内容就能十分明确的在admin进行对应的配置了。

# 优惠券发布

优惠券的各个功能模块掌握后，就需要发布优惠券了。发布优惠券主要有两个功能 一是 优惠额度审核 二是优惠券历史版本

## 额度审核

![](https://static.slab.com/prod/uploads/posts/images/4sygW3eflXs5MR1U9p4VOsHK.png)

选择 `Code Type`不同选项在下图红框处的审批额度会不一样

![](https://static.slab.com/prod/uploads/posts/images/KtpBdkeW4l3T3RW8dxYPo3bG.png)

审核权限：在admin账户管理中添加角色，包括一级：优惠码高级管理，二级：优惠码区域经理，三级：大额度优惠码管理。

## 历史版本

![](https://static.slab.com/prod/uploads/posts/images/qCEvc9kN77L7OY41tAbAzKaa.png)

优惠券提交后就会生成相印的一个版本，每次提交修改也会生成相印的一个版本。

TODO:各个版本之间有优惠券使用数量和商品限制数量逻辑

如果设置优惠券的时候操作错误了，可以使用 Override 选项用新的一个版本覆盖之前版本，即使是用户已经兑换的优惠券。

Clone可以复制优惠券的配置来创建一个新的优惠券

# to be continued
