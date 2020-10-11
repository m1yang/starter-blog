---
title: '自然语言编写UI自动化测试'
subtitle: 使用工具cypress、cucumber
summary: 
authors:
- miyang
tags:
- test
- cypress
- cucumber
categories:
- 测试
date: "2020-04-05T00:00:00Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: true
---

# 前言

> cucumber是bdd框架的一种，不管偏向技术还是业务，掌握bdd都可以让测试的核心技能-用例的编写，更贴近用户、更易于交流。
> cypress是类似selenium且更适用于web端的测试框架，可以深化测试对前端技术的了解,编写更有效的测试用例。而且，它的一些设计理念和最佳实践对其它端测试也有一定的借鉴作用。

# 大纲

> 主要分三部分来讲解：用例即代码、合理的分层、元素定位进阶
> **用例即代码** 介绍cucumber，讲feature自然语言编写的用例和代码的结合
> **合理的分层** 讲如何结合以前的PageObject和现在的feature
> **元素定位进阶** 讲如何更好的查找页面元素并保持高可用、易维护

cypress基础建议先阅读[Vant Ling](https://xxx.slab.com/users/mwyofduq) 写的slab，[Cypress UI自动化测试基础教程](https://xxx.slab.com/drafts/a16p8x61) 、[cypress UI自动化测试解决方案](https://xxx.slab.com/drafts/syh809p7) 或者[官方文档](https://docs.cypress.io/zh-cn/guides/overview/why-cypress.html#)。这里就不再赘述了。

## 用例即代码

以前写自动化，虽然说是需要有用例的支撑。但很多时候，代码都是脱离了测试用例，尽可能的把页面元素都点一遍，能传的值都传一遍，来编写自动化脚本的。这样有一个严重的问题，就是编写的脚本，根本不在测功能，只是机械重复的点击各个控件。

借助行为驱动开发，可以很好的控制代码的编写，不至于过分放飞自我。拿欧铁垂直页站点搜索栏的用例举个例子

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/dmKYin0x18ZRnIknfAxnMHlH.png)

对应cucumber的.feature文件可以这样描写测试用例

```
Feature: 欧铁垂直页站点搜索功能

  欧铁业务的主要入口

  Background:
    Given 进入欧铁垂直页

  Scenario Outline: 用户进行站点搜索
    When 搜索栏输入站点名称"<location>"
    Then 展示搜索结果"<result>"

    Examples:
    | location | result|
    | 巴 | 巴|
    | 巴黎 | 巴黎|
    | abcdefg | 找不到结果 |
```

(空格不知道怎么输入)。Background可以作为前置条件。Scenario Outline和Examples非常符合操作步骤一致，输入不同参数来校验结果的用例。

对应编写的代码

```
defineStep(`搜索栏输入站点名称{string}`, (location) => {
  selectors.xxx.type(location).click()
  selectors.xxx.as('suggestStation')
})

Then(`展示搜索结果{string}`, (result) => {
  cy.get('@suggestStation').should('contain', result)
});
```

(示例代码)。这样的方式，就迫使我们需要根据用例的操作步骤来编写代码了，而且自动化的用例也更易于和其他合作的人员沟通交流。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/JDcexjbp4UrA1N0pTfV_DkTG.png)

上述就是cypress结合cucumber的一个示例，它本身就如[官方文档](https://cucumber.io/docs/guides/overview/)介绍的说。 cucumber执行由Gherkin 语言写成具有标准规范的 .feature 文件。根据它的规范，编写对应执行的脚本，将用例和代码结合起来。

环境部署:[https://github.com/TheBrainFamily/cypress-cucumber-preprocessor](https://github.com/TheBrainFamily/cypress-cucumber-preprocessor)

了解cypress的cucumber插件:[https://github.com/TheBrainFamily/cypress-cucumber-example](https://github.com/TheBrainFamily/cypress-cucumber-example)

了解cucumber使用的Gherkin语言:[https://cucumber.io/docs/gherkin/reference/](https://cucumber.io/docs/gherkin/reference/)

## 合理的分层

cucumber这套框架本身推荐的一个目录结构

- integration：用来存放自然语言的用例文件
- support/step_definitions：用来存放代码实现的测试用例

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/AkiaLkMqb8A9t_k8AofivCJl.png)

目前我都存放在integration文件夹下面了，因为这样比较容易找到，后续再转换为上述结构。

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/0iIZx1pdHuIGF7QKyAUBJi-V.png)

公共使用的代码就存放在common目录下

大致的编写步骤

- support/step_definitions/pages内编写各个页面的元素定位
- support/step_definitions内编写各个页面操作步骤
- support/step_definitions/common内抽象可供公共使用的操作步骤
- integration内使用step_definitions中已定义好的步骤编写用例

这里编写用例很需要插件的支持，比如vscode中[Cucumber (Gherkin) Full Support](https://marketplace.visualstudio.com/items?itemName=alexkrechik.cucumberautocomplete)

在support/step_definitions文件夹下做元素定位和元素操作的分离，并与integration做自然语言和代码的分离。前者是加强了元素定位的复用，后者可以让技术人员和业务人员协同工作。

## 元素定位进阶

拿欧铁垂直页举例说明如果通过css selector，更好的进行页面元素的定位

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/G9cNVo-amnY7IiC6-WOZjBcM.png)

首先，定位整个搜索控件，在开发者模式下查看dom结构

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/xwG-y05lKHt03PvSk626dP3h.png)

点开之后能看到这个控件主要分为了三个部分：

- 搜索的标签-单程还是往返
- 搜索的主体
- 是否持有通票

这里主要就是想说，其实前端页面的布局肯定是有规律的，以前copy xpath、copy selector的方法就是偷了个懒，没让自己去熟悉前端的页面布局。如果能够有规划的去获取到页面元素，那么元素的定位和后期的维护都更容易把控。

### 元素定位小技巧

前端通过css来渲染特定的样式，利用这一点我们可以使用前端css style的元素定位来找到同一类的控件

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/AVWc7A0T7tbGm5_WL4V38qBW.png)

如上图所示，找到搜索下拉列表的css style，当然这时还不能直接使用这个元素定位。因为出发地、目的地都有下拉列表，直接使用就会出现定位到重复元素的问题。

解决这个问题，可以使用cypress提供的API，像[within](https://docs.cypress.io/zh-cn/api/commands/within.html#Syntax)，可以缩小元素定位的范围。如果将下拉列表的元素定位限制在出发地这个控件下，当然就不会受到目的地相同控件的影响了。能定位元素了，剩下的就是如何更好的对元素进行操作了。

出发地搜索下拉列表的定位就像

```
cy.get('.er_search .er_search_form') // 先确定是单程搜索
  .get('.from_div').type('Paris') // 再选择单程搜索栏，并输入Paris
  .get('.destinationList.choose_container.suggestStation > :not([style="display: none;"])') 
  .click() // 最后定位到下拉列表，并点击
```

上述只是一个简单的示例，真实的代码一般也不会这么去写

这样定位的大概思路就是：

1. 先找到前端页面渲染的方式
1. 再在这样的基础上，做好定位元素的前置条件
1. 准确定位到需要操作的控件后，再使用类似 :nth-child(n)的选择器来选择页面内的元素

这样的方法，大概能解决80%左右的元素定位问题

### 用代码来构建元素的定位

通常来说，做UI自动化都习惯用类似json的文件或者是key:value的数据结构，来存储元素定位。这样页面元素变动了，就只需要修改一个地方就可以了。

一开始也是想通过json来存储页面元素的，但是使用时发现json的可构造性不大，所以用js代码来存储页面元素

同样举个例子

```
// page.js文件
class SearchBar {
    // 定位元素是单程还是往返
    static isSearchOneway(bool) {
        let ele = '.er_search .er_search_form';
        if (bool === false) {
            ele = '.er_search .er_search_form.isReturn';
        }
        return cy.get(ele)
    }

    static ...
}

class ReviewBar ...
export { SearchBar, ReviewBar, … };
```

直接用构造函数来进行元素定位，增强可复用性。

```
// vertical_page.js文件
import selectors from 'page.js';
// 选单程
selectors.isSearchOneway(true).click()
// 选往返
selectors.isSearchOneway(false).click()
```

(上述代码是运行不了的，只是示例)，比起key:value的结构来，定位同一类型的元素，无论是可复用性还是易维护性方面都有提升。这样做的原因也是因为，前端是这样做样式渲染的，页面要改，也是一类一类的改。

# FAQ

Q:花费了大量时间写了脚本，前端页面一改全没了，一点效益都没有

A:更应该担心的是前端页面一成不变，因为这样就不知道，自己考虑的是否足够全面。编写的用例、脚本，它们的易用性、复用性、健壮性等等是不是到位了。

随着业务的发展，展示的页面肯定是会变动的，但是我们的用例、代码编写能力都会随着业务的改变而逐步提升的。

不要被网上的：“页面变动大，不适合做自动化。”“不要为了自动化而自动化。”这样的言论洗脑了。如果没有实践，不会知道这个页面到底变动大不大。而即使变动大，这个变动也不会不可控。

一个页面真的稳定了，说明不会再有什么改动能影响到它，这样的页面做自动化才真的会没有效益。



Q:UI自动化只能用来回归测试吗？自己都测过一遍了，还要再花时间写脚本，还是测已经验收了的功能，有什么用

A:（个人观点）只要是自动化测试，都只能用来做回归测试。道理很简单，一个全新的功能，代码都还没写，都没有可操作的对象，拿什么去自动化呢？心思不要杂，先通过自动化把回归做了。只出需求就能写自动化测试，目前的确也有这样的技术可以达到这个要求，但是我们更习惯称它为人工智能。

弄清楚了目标，就是通过写自动化测试来做回归测试。在这个基础上再看，为什么要做自动化测试。绝大部分原因是业务越来越广，每次新增一个功能，为避免产生负面影响，需要对之前的功能进行一次全量的回归，回归体量越来越大，新的功能也就难以轻易的添加了。

正因为是已经验收过的功能，该如何测试这些功能也明确下来了，那么为什么不让脚本去执行这些已知的操作步骤呢？



Q:编写脚本到产线上运行，需要多大的投入

A:编写一个模块，像欧铁搜索，前期投入时间大概在3～5天，框架熟练且公用工具慢慢积累起来后能缩短到1～3天。目前一个页面的功能块，十几个到几十个不等。所以前期做一条业务线的自动化，起码得投入60人/天。
