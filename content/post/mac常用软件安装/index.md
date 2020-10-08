---
title: '常用软件安装'
subtitle:
summary: 
authors:
- miyang
tags:
- mac
categories:
- os
date: "2020-04-05T00:00:00Z"
lastmod: "2020-04-17T00:00:00Z"
featured: false
draft: true
---
> Homebrew是一款自由及开放源代码的软件包管理系统，用以简化macOS系统上的软件安装过程。

# 批量安装

一般使用一台新的电脑，一开始总要装太多软件，所以写了个脚本来自动安装必要软件。脚本很简单，大概就是判断是否是mac电脑，是就安装上homebrew，通过homebrew来装上必要的软件。

```
#!/bin/bash

# 判断是否为MacOS，是则安装homebrew
if [ "$(uname)" = "Darwin" ] ; then
  # MacOS
  if ! type brew >/dev/null 2>/dev/null; then
    echo "---start install homebrew---"
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  else
    echo "---start upgrade homebrew---"
  fi
else
  # Linux
  echo "homebrew The missing package manager for macOS"
fi

# Make sure we’re using the latest Homebrew.
brew update

# Upgrade any already-installed formulae.
brew upgrade

# 常用的命令行工具安装
brew install vim
brew install jmeter

#常用的工具包安装
brew cask install java
brew cask install iterm2
brew cask install google-chrome
brew cask install shadowsocksx-ng
brew cask install postman
brew cask install sequel-pro
brew cask install xmind-zen
brew cask install wechat
brew cask install wechatwork
brew cask install slack
brew cask install zeplin
brew cask install charles

brew cleanup
```

将文件上传到了Google driver [brew.sh](https://accounts.google.com/ServiceLogin?service=wise&passive=1209600&continue=https://drive.google.com/open?id%3D1VXa2Mb1WdCNVELjJta3mefCv_PnxklMD&followup=https://drive.google.com/open?id%3D1VXa2Mb1WdCNVELjJta3mefCv_PnxklMD) ，下载下来在终端执行类似 `sh brew.sh`	这样的指令就行（注意下载路径）。

国内下载速度缓慢，可以更换源（mirror）

```
# 手动敲指令
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

brew update
brew doctor
```

// 指定版本安装可以参考问题题：[Homebrew install specific version of formula?](https://stackoverflow.com/questions/3987683/homebrew-install-specific-version-of-formula) （问题太陈旧，只有点参考价值）

# 命令行工具

不同苹果电脑版本，需要通过不同的方式安装命令行工具，通常执行

```
xcode-select -p
xcode-select --install
```

如果安装失败，就需要在苹果开发者站点上下载Command Line Tools，地址如下：https://developer.apple.com/downloads/index.action

![](https://static.slab.com/prod/uploads/d9aeaycl/posts/images/vDPSvyhkmhUWh6q7BFtOx7ti.png)

根据对应的系统版本安装Command Line Tools，安装好了Command Line Tools再执行上述脚本。

FAQ：[https://medium.com/@vschroeder/xcode-select-error-tool-xcodebuild-requires-xcode-but-active-developer-directory-fb7d3408c80b](https://medium.com/@vschroeder/xcode-select-error-tool-xcodebuild-requires-xcode-but-active-developer-directory-fb7d3408c80b)



# 推荐安装

有想要安装的软件可以在Githubawesome系列里找：

[https://github.com/jaywcjlove/awesome-mac/blob/master/README-zh.md](https://github.com/jaywcjlove/awesome-mac/blob/master/README-zh.md)

AppStore中的应用也可以通过mas来安装，文档：

[https://github.com/mas-cli/mas](https://github.com/mas-cli/mas)

homebrew文档：[https://docs.brew.sh/Formula-Cookbook.html](https://docs.brew.sh/Formula-Cookbook.html)

后续如果有想要安装的软件就可以直接在命令行查找下载了，例如需要下载alfred

```
brew search alfred
```

然后根据搜索结果下载对应软件

```
brew cask install alfred3
```

通过homebrew安装的命令行工具或者是软件需要更新

```
brew upgrade && brew cask upgrade
```

一些能提供生产力的工具：

- [Alfred](https://www.alfredapp.com/) - 效率神器。 
- [Spectacle](https://www.spectacleapp.com/) - 简单的移动和调整大小的窗口，和可定制的键盘快捷键
- [CheatSheet](https://www.mediaatelier.com/CheatSheet/) - CheatSheet 是一款 Mac 上的非常实用的快捷键快速提醒工具。
- [Snipaste](https://zh.snipaste.com/) - 一个简单但强大的截图工具。
- [HandShaker](http://www.smartisan.com/apps/handshaker) - Mac 电脑上也可以方便自如地管理您在 Android 手机中的内容。
- [Visual Studio Code](https://code.visualstudio.com/) - 微软推出的免费/开源编辑器。
- [Wunderlist](https://www.wunderlist.com/?ncr=1) - 奇妙清单跨平台的任务管理器与移动应用程序。
- [ProcessOn](https://www.processon.com/) - 流程图、思维导图、原型图... 中文友好，免费保存 5 个文件。

顺手安利一些Chrome拓展

- cVim - 脱离鼠标操作Chrome浏览器
- Grammarly for Chrome - 英语语法校验
- JSON-handle - JSON格式化
- Octotree - 阅读GitHub的文档树
- Tab Snooze - 延后浏览网页
- uBlock Origin - 过滤广告
- Workona - 浏览器标签管理
- 掘金 - 每日资讯。
- 沙拉查词 - 划词翻译。
- IE Tab - 在Chrome中运行Internet Explorer
- OneTab - 将所有标签页转换成一个列表



# to be continued
