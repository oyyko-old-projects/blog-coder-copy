---
title: "New Macbook Setup"
date: 2023-10-04T10:41:00-07:00
tags: [mac, macbook]
toc: true
math: false
---
## Install HomeBrew
see https://brew.sh/.

## 增加软件仓库
```bash
brew tap homebrew/cask-fonts 
brew tap homebrew/cask
```

在增加了homebrew/cask之后，安装cask中的软件不需要`brew install --cask`, 只需要直接`brew install`即可。

> 但是官方不建议这么做，官方做法是使用`brew install --cask XXX`

## Use homebrew to install Apps
```bash
## Install Rust CLI Apps
brew install bat fd wget bottom lsd ripgrep
## Install cask Apps
brew install --cask google-chrome visual-studio-code wechat qq iterm2 iina
```

## Install snap
With Snap App, you can use `Command ⌘ + 1` to call the first app in your dock.

从App Store下载。这是免费的。

## Install Flow
Flow可以定时提醒自己休息。从App Store下载或者brew安装都可以。

## Tex
安装MacTex即可。

```bash
brew install --cask mactex
```

## 下载字体
```bash
brew install font-jetbrains-mono-nerd-font
brew install font-jetbrains-mono
```

## 配置iterm2和zsh
在`~/.zshrc`中配置`alias ins="brew install"`

```bash
ins zsh-fast-syntax-highlighting
ins zsh-autosuggestions
ins zsh-completions
ins zsh-history
```
安装了之后需要在zshrc中配置才能生效
如下
```bash

source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh

if type brew &>/dev/null; then
  FPATH=$(brew --prefix)/share/zsh-completions:$FPATH
  zstyle ':completion:*' matcher-list '' 'm:{a-zA-Z}={A-Za-z}' 'r:|=*' 'l:|=* r:|=*'
  autoload -Uz compinit
  compinit
fi

source /opt/homebrew/opt/zsh-fast-syntax-highlighting/share/zsh-fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
```
## 安装zoxide
zoxide会记忆你最近访问的目录。例如你最近访问过`~/aaa/bb/banana`,那么不管你当前处在哪一个目录，你都可以用`z ba`来跳转到banana目录下吗。`z b`, `z ba`,`z ban`都是可以的。
安装方法：
`ins zoxide`
之后在zshrc中配置
`eval "$(zoxide init zsh)"`

## 安装p10k
参考https://github.com/romkatv/powerlevel10k

## 使终端中的option ⌥ + ← 可以实现跨单词跳转
在zshrc中配置
```bash
bindkey "\e\e[D" backward-word
bindkey "\e\e[C" forward-word
```

## 使用espanso
espanso是一个键盘快捷键映射软件。例如在任何地方输入:date会自动变为当前的日期。输入:name会自动显示自己的姓名等。



