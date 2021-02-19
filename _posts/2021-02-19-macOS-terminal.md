---
layout: post
title: macOS Terminal 调优
---

## 前言

不管使用任何系统，终端永远是我最长使用的程序，干净、整洁、高效的终端使用体验是我的追求，在我的主力机MacBook Pro上，我也对Terminal做了很多的调优，从内而外的对Terminal进行了深度的优化

在调优前我曾翻阅过大量的文档，并对下面两个问题给出了答案

**要不要使用第三方Terminal？**

现在能在macOS上运行的终端程序非常多，比较出名的有iTerm2，我个人是坚持最小可用原则，当一个程序无法满足需求的时候才会去考虑使用其他程序，就目前的使用体验来看，系统中自带的Terminal已经能够为我提供完美的使用体验，不需要使用第三方终端程序

**要不要使用终端插件？**

如果你搜索zsh，有99%的搜索结果都会提到一个叫做[Oh-My-Zsh](https://ohmyz.sh)的终端插件，起初我感到这应该是一个非常不错的插件，在翻阅源码后我越发的感到这个并不适合我，用四个字形容就是：花里胡哨。

我认为理想的工作环境应该是简洁的，只关注最重要的事物，而这类的插件给我的感觉非常臃肿，我可能只需要一个小小的功能，但是它给我一堆附加功能，想想你只是需要削苹果，有必要买瑞士军刀吗？如果一个产品提供的内容有一半都是用不到的，那么除去可用的内容外都是浪费。

## 让Terminal变得好看点

基于此我开始对系统自带的Terminal进行调优，首先是视觉部分

![Terminal-main](./_posts/images/terminal_main.png)

每次打开后什么都没有（当然几乎也没关过），只有一个用颜色区分普通用户和root用户的标识，普通用户用的是粗体绿色，root用户用的是粗体红色。

设置内使用了[Solarized Dark](https://github.com/tomislav/osx-terminal.app-colors-solarized)色彩体系，但对默认提供的配置文件进行过改动，首先是字号使用了14号，这是一个涉及到命令执行安全的建议，因为在终端中运行程序是不具备撤销的，一旦因为输入错误造成了错误的命令执行，结果就是毁灭性的，所以调大字号方便看清命令的输入，唯一的缺点就是如果没有外接显示屏，单个Terminal程序还是挺占屏幕空间的。其次是开启粗体，并把光标调整到和Linux一样的红色Block

![Terminal-setting](./_posts/images/terminal_setting.pnv)

## Zsh调整

之前的macOS默认用的都是bash，到了Catalina后开始使用zsh作为默认shell，既然官方进行了调整就说明zsh有使用的良好价值，我也开始学习和了解这个shell

下面是我的.zshrc文件，针对每个项目都进行了详细的介绍

```bash
# .zshrc

# Zsh configure
# Autoload module
autoload -U compinit
autoload -U promptinit
autoload -U colors && colors

# Auto complete command
setopt completealiases
promptinit

# But we are hackers and
# hackers have black
# terminals witch green
# font colors!
# BY John Nunemaker
compinit
PROMPT="%{$fg_bold[green]%}>%{$reset_color%}"

# Freeze/Unfreeze Terminal
ttyctl -f

# History
# Clear duplicate history
setopt HIST_IGNORE_DUPS
HISTFILE=~/.zsh_history
HISTSIZE=1000
SAVEHIST=1000

# LANG
export LANG='en_US.UTF-8'

# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias ls='ls -GF' #macOS 中的ls默认没有color-auto，使用G配置
alias vi='vim'

# My private shell script lab
export PATH="$PATH:$HOME/.local/bin"

# Homebrew
export PATH="/usr/local/sbin:$PATH"
# To fixed brew doctor with pyenv config error
# https://github.com/pyenv/pyenv/issues/106
alias brew="env PATH=${PATH//$(pyenv root)\/shims:/} brew"

# Metasploit Framework
export PATH="/opt/metasploit-framework/bin:$PATH"

# Pyenv
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi

# Pyenv-virtualenv
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

# Add RVM to PATH for scripting
# Make sure this is the last PATH variable change.
export PATH="$PATH:$HOME/.rvm/bin"

# openJDK
export PATH="/usr/local/opt/openjdk/bin:$PATH"
export CPPFLAGS="-I/usr/local/opt/openjdk/include"
```

前面一部分是zsh的配置，主要是对色彩还有历史记录的控制，后半部分基本都是各种程序的PATH，出于隐私考虑删除了很多内容，基本的逻辑还是最小可用原则，简单、简洁

在使用macOS的时候，几乎没有切换到root账号的需求，所以没有对切换用户进行识别，在Linux环境中我会尝试识别当前用户，并切换到对应用户的工作环境

## Python和Ruby，我的最爱

接着是基础脚本程序Python和Ruby的改造，因为用这两个语言写的程序基本都会使用第三方库，为了避免和系统默认版本和库产生冲突，使用[Pyenv](https://github.com/pyenv/pyenv)配置自定义Python版本，使用[RVM](https://rvm.io/rvm/install)配置自定义Ruby版本，详细的配置说明可以在项目官网查看，写的已经非常详细了

**有一点需要注意的是**：不管处于什么情况，**永远**不要把自定义的Python或Ruby版本设置为系统级，我最近的电脑总是出现重启报错，经过排查后发现就是因为把Ruby设置为自定义下载的，而且因为项目需要更新gems很勤，导致系统调用错误重启报错

可以使用下面两个办法进行解决：

1. Python：使用`Pyenv local VERSIONS`可以在项目目录里生成一个.python-version文件，文件内记录着当前项目中的Python版本，进入目录会自动切换

2. Ruby：使用`rvm --create --ruby-version VERSIONS@GEMSET`在项目目录里配置专属Ruby版本，进入目录会自动切换

## 使用软链接备份

这点是我的一个使用习惯，我会把所有日常使用的目录和文件设置软链接，连接到一个自动更新repostory里，这就保证了即使机器出现不可逆故障，我仍然可以在任何一台机器上快速恢复生产力

## 安全

安全方面主要是命令执行的部分，有这么几点：

1. 永远不要把网页中复制的命令直接粘贴到Terminal，必须使用文本编辑器检查是否存在错误命令以及回车

2. 尽可能减少`rm -rf`命令的使用，上周就错误的删除过一个文件夹，幸好远端留了备份

3. 尽可能不做免密，即使需要也要使用hash加密，虽然我从没让别人操作过我的电脑，但万一对吧

关于安全方面能写的还有很多，总的来说就是保持谨慎、保持距离

## 结语

Terminal是我最喜欢的程序，我之所以不喜欢Windows的很大原因就是因为到目前为止都没有一个使用体验超过Terminal的终端，如果动动键盘就能搞定的事情还要去动鼠标，这很不符合我的习惯，当我开始理解shell的时候，我也理解了Linux世界的美好，这点对我非常重要

写完后我又回过头阅读了一遍，好像是什么都没写的样子哈，不过也确实是这样，我做的事情也只是把Terminal变得更简单、简洁了而已，在遵循少即是多原则的基础上，我挺喜欢现在正在使用的Terminal^_^
