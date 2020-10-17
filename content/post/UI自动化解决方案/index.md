---
title: 'UI自动化解决方案'
subtitle:
summary: 
authors:
- admin
tags: []
categories: []
date: "2020-04-05T00:00:00Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: true
---
## 前言

如果你是新手，请先看之前的文档。这份文档的目的是在使用cypress的过程中，提供解决方案，不提供cypress的教程方式。关于解决方案中，不懂的地方，可以私聊我们一起讨论

[Cypress UI自动化测试基础教程](https://xxx.slab.com/drafts/a16p8x61)

## 问题及解决方案

### 1. xxx谷歌机器人验证问题，添加谷歌白名单。

在visit的url中添加headers

```
cy.visit(url,{
  headers:{
    "x-px-access-token":"",
}
})
```

### 2.关于cypress拿不到页面的text问题

在使用的过程中，会发现，当你想校验值是否为期待值的时候。cypress提供了两个解决方案

解决方案1:直接通过断言获取的值

```
cy.get(".bottom.t_white.m_main > h1")
  .should('have.text', 'YOURS TO EXPLORE')
```

解决方案2:通过获取到的元素值，再进行校验

```
cy.get('header.title').first().should(($div) => {
    const text = $div[0].text()
    console.log(text)
    expect(text).to.eq("Popular Destinations")
    expect(text).to.include('Destinations')
    expect(text).not.to.include('xxx')
})
```

### 3.设置mobile web浏览器代理

在默认情况下，cypress提供的API只在Web运行，如何设置mobile web。

解决方案：

打开Mweb目录下的`cypress.json`，添加以下一段参数

```
{
  ...   #这是省略其他与本次解决方案无关的参数
。
  "viewportWidth": 375,
  "viewportHeight": 667,
  "userAgent":"Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1"
}
```

### 4.解决报告生成在统一位置

背景：在cypress里面，每个测试报告都会生成在对应的项目目录下，对于Web端和Mweb端的测试报告，需要统一起来了。

解决方案：在cypress.json的文件下，对mochawesome的报告路径进行修改，存放在统一的文件夹中。路径是根据自己的项目，自定义路径`"reportDir": "../results"`,

```
"reporter": "mochawesome",
  "reporterOptions": {
    "reportDir": "../results",
    "overwrite": false,
    "html": false,
    "json": true,
  },
```

### 5.将失败测试用例的截图放到报告中

背景：每一次的执行测试用例，都有可能出现执行失败的情况，那么，如何将失败的截图放到报告中。

解决方案：去到对应项目目录下的`cypress/support/`	打开index.js

```
Cypress.on('test:after:run', (test, runnable) => {
  if (test.state === 'failed') {
    const screenshotFileName = `${runnable.parent.title} -- ${test.title} (failed).png`
    addContext({ test }, `../Web/cypress/screenshots/${Cypress.spec.name}/${screenshotFileName}`)
  }
  });
```

### 6. cypress中使用xpath定位元素

cypress默认情况是使用css的定位元素方式，在使用起来就已经足矣。在一些情况需要用到xpath去定位元素，可以在`cypress/support/`	打开index.js

```
// Import commands.js using ES2015 syntax:
import './commands'
require('cypress-xpath')
```

### 7.保存token方法

很多时候，需要使用到token保持到登录态，可以使用cypress提供的接口请求方法保存token，然后在测试用例中使用

去到`cypress/support` 打开commands.js，请求登录接口，获取token数据，其中url放在`cypress.json`	中作为统一的请求

```
Cypress.Commands.add("login", (email, password) => {
    let md5 = MD5(password).toString();
    cy.request({
            url: "/xos_api/v1/usrcsrv/user/login/web",
            method: "POST",
            form: true,
            body: {
                username: email,
                password: md5
            }
        }).its('body')
        .then((body) => {
            cy.setCookie('_pt', body.result.token)
        })
});
```

在测试中使用，拿到已经保存下来的token

```
describe("测试登录", () => {
    beforeEach(()=>{
      cy.login("你的账号","你的密码")
      Cypress.Cookies.preserveOnce("_pt")
    })
    it("测试登录的标题", () => {
        cy.get_url("/")
        cy.url()
            .should('include', "xxx")
    })
})
```

### 8.如何测试多种语言

多种语言可能是最在测试中，比较麻烦的一个环节了。因为在断言的时候，你无法通过一种语言去检查你测试的结果是否准确，而是需要在不同语言下，断言的结果也是对应的语言。

解决方案有两种：

第一种就是通过接口返回的方式，去断言对应的结果。

第二种就是将需要断言的对应期望结果写到文件中，测试的时候再去读取。

两种方法各有所好，看自己喜欢用哪种就用哪种吧

第一种，这里就拿获取城市页的数据做例子：

```
Cypress.Commands.add("get_city", (aid, language) => {
    cy.request({
        url: '/xos_api/v1/usrcsrv/city/' + aid,
        headers: {
            "Accept-Language": language,
            "x-px-access-token": "crVsiOTO6xOhLKgYXtrwFsSoTqekOqvbpmA9ctRPZYlUzzJxvPaifeisNbpjYTIu",
        }
    }).then((respone) => {
        return respone
    })
})
```

这里的接口会返回城市接口的数据，然后，就可以在测试用例中使用这份数据

```
const page = require('../../../pageObject/xxx-city-page.json')
const aid = "2"

it(`检查城市名字与data数据中保存的城市名字是否一致 ${langs}`, () => {
            const lans = langs.split('-').join('_')
            cy.get_city(aid, lans)
                .then((respone) => {
                    const city_name = respone.body.result.activities_filter.city_name
                    cy.get_url('/' + langs + '/city/' + aid)
                    cy.get(page.web_city_name)
                        .should('have.text', city_name)
                })
        })
```

这里使用到js的字符转换的方法，const lans = langs.split(&#39;-&#39;).join(&#39;_&#39;)

第二种方法，看起来会很冗余。

```
const lang = [{ "en-AU": 'value' },
    { "en-CA": 'value' },
    { "en-HK": 'value' },
    { "en-IN": 'value' },
    { "en-MY": 'value' },
    { "en-NZ": 'value' },
    { "en-PH": 'value' },
    { "en-SG": 'value' },
    { "en-US": 'value' },
    { "en-GB": 'value' },
    { "zh-CN": 'value' },
    { "zh-HK": 'value' },
    { "zh-TW": 'value' },
    { "ja": 'value' },
    { "ko": 'value' },
    { "th": 'value' },
    { "vi": 'value' },
    { "id": 'value' },
]
```

这里的数据，就可以在测试中里中使用了

```
lang.forEach((langs) => {
        it(`TTD页面多语言调试 ${Object.keys(langs)[0]}`, () => {
            cy.klkVisit('url' + Object.keys(langs)[0])
                console.log(Object.values(langs)[0])
                console.log(Object.keys(langs)[0])
        })
```
