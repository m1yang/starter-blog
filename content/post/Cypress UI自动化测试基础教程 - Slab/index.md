---
title: 'Cypress 基础'
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
## 一、安装

### 创建项目

```
mkdir CyAutotest ##创建一个CyAutotest目录
touch package.json
vim package.json
```

编辑package.json文件

```
{
}
```

### 安装Cypress

```
cd CyAutotest
npm install cypress --save-dev
或者
yarn add cypress --dev
```

### 启动Cypress

```
npx cypress open
或者
yarn run cypress open
```

## 二、编写第一个测试

### 创建测试文件

1. 在文件夹**cypress/integration**创建第一个脚本文件：**klook_spec.js**
1. 查看已经启动的GUI界面，会发现多了一个刚才创建的**klook_spec.js**文件
1. 点击**klook_spec.js**文件，Cypress会自动启动浏览器，但是在浏览器中什么都没有

### 执行测试用例及查看结果

1. 在上面已经启动浏览器的情况下，在文件中写入测试代码

```
describe('My First Test', function() { 
  it('Does not do much!', function() { 
    expect(true).to.equal(true) 
  }) 
})	
```

1. 保存之后，浏览器会自动执行这个脚本。

![](https://static.slab.com/prod/uploads/posts/images/8uO4X9TuRm1KhxmLTB7rg1H3.png)

1. 创建一个失败的测试脚本，查看失败结果

```
describe('My First Test', function() { 
  it('Does not do much!', function() { 
    expect(true).to.equal(false) 
  }) 
})
```

![](https://static.slab.com/prod/uploads/posts/images/8ghYMfGKsqLnIxkE4iCxF3w6.png)

### 测试用例基本结构

```
describe('描述这个测试脚本', function() {
  it('描述这个测试用例1', function() {
    //执行测试的用例
  })
  it('描述这个测试用例2', function() {
    //执行测试的用例
  })
  it('描述这个测试用例3', function() {
    //执行测试的用例
  })
})
```

### 例子：客路首页

```
describe('测试客路系统', function() {
  it('测试进入客路首页', function() {
    cy.visit('https://www.klook.com')
  })
})
```

注：

```
1.describe('测试客路系统', function() {}
创建一个测试的函数
2.it('测试进入客路首页', function() {}
创建一个测试用例的方法
3.cy.visit('https://www.klook.com')
执行测试用例步骤，打开客路系统
```

前面的代码已经介绍如何打开浏览器以及在浏览器中输入测试的URL，在页面进行简单的操作

```
describe('测试客路系统', function() {
  it('测试进入客路首页', function() {
    cy.visit('https://www.klook.com')
    #在页面中点击登录按钮
    cy.get('#login').click()
    #断言页面url中的确是登录页
    cy.url().should('include','/signin')
    #对执行结果的页面进行截图的操作
    cy.screenshot('my-image')
  })
})
```

## 三、添加测试报告

安装依赖包

```
npm install macha&&mochawesome&&mochawesome-merge
```

打开cypress.json文件，添加下面内容

```
"reporter": "mochawesome",
  "reporterOptions": {
    "overwrite": false,
    "html": false,
    "json": true,
    "timestamp": "mmddyyyy_HHMMss"
  },
```

执行命令，生成报告

```
cypress run
```
