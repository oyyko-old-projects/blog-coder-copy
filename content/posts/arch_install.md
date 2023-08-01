---
title: Archlinux/Manjaro Install Guide
date: 2023-07-05
tags: [os, linux, archlinux, manjaro]
toc: true
---
manjaro I3新电脑配置

假设已经安装好了manjaro i3，接下来要做的是：


## 基本装机 按顺序执行以下操作
### 换源
编辑`/etc/pacman.d/mirrorlist` 
内容改为
```             
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```
### 加入archlinuxcn源并安装yay
在 `/etc/pacman.conf` 文件末尾添加以下两行
```
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
之后通过以下命令安装 archlinuxcn-keyring 包导入 GPG key。
```
sudo pacman -Sy archlinuxcn-keyring
```

之后就可以用yay 安装各种软件了



记得先`sudo pacman -Syyu`一下。

### 科学上网
由于安装某些软件的时候，yay会从国外下载，所以要先科学上网才能尽情安装想要的软件。
```
yay -S clash-for-windows-bin
```
之后导入已有的订阅。
之后在终端中设置(例如加入到`~/.zshrc`中)：
```
export https_proxy=http://127.0.0.1:7890;
export http_proxy=http://127.0.0.1:7890;
export all_proxy=socks5://127.0.0.1:7890;
```

对于i3wm等无法设置系统级代理的桌面环境，请把chrome的启动改为
```
bindsym $mod+F2 exec google-chrome-stable --proxy-server="socks5://127.0.0.1:7890"
```
这样chrome也可以自动使用代理上网了。

### 安装输入法
```
yay -S fcitx5-im
yay -S base-devel
yay -S fcitx5-rime
yay -S rime-cloverpinyin
```

配置fcitx5的环境变量：

```text
sudo vim /etc/environment
```

内容为：

```text
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
```

创建并写入rime-cloverpinyin的输入方案：

```text
nano ~/.local/share/fcitx5/rime/default.custom.yaml
```

内容为：

```text
patch:
  "menu/page_size": 5
  schema_list:
    - schema: clover
```

安装中文维基百科词库：

```text
yay -S fcitx5-pinyin-zhwiki-rime
```

设置fctix5自动启动：

```
nano ~/.i3/config
```

加入

```
exec --no-startup-id fcitx5
```



### 安装geek字体

```
yay -S nerd-fonts-jetbrains-mono
yay -S ttf-jetbrains-mono
```

之后把终端的字体改为该字体。例如更改Konsole的配置，如果默认终端不是Konsole建议改为Konsole。

### 安装常用软件

```
sudo pacman -S ntfs-3g # 使系统可以识别 NTFS 格式的硬盘
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei # 安装几个开源中文字体。一般装上文泉驿就能解决大多 wine 应用中文方块的问题
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra # 安装谷歌开源字体及表情
```

```
yay -S typora-free
yay -S visual-studio-code-bin
yay -S google-chrome
```

一些常用命令的Rust版本：

```
yay -S fd bat exa ripgrep procs dust 
```

### 安装latex
```
sudo pacman -S texlive-most texlive-lang
```

配置vscode-latexworkshop使用xelatex为默认

https://blog.csdn.net/Haulyn5/article/details/124128533

修改settings.json

```
"latex-workshop.latex.tools": [

        {
            "name": "latexmk",
            "command": "latexmk",
            "args": [
                "-xelatex",
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%"
            ],
            "env": {}
        },
```

### 安装zsh

```
sudo pacman -S zsh zsh-autosuggestions zsh-syntax-highlighting zsh-completions
```

更改默认shell

```
chsh -s /usr/bin/zsh
```

对于Konsole需要修改它的配置文件来改变它启动的时候默认使用的shell

由于manjaro i3有自带的zsh配置已经够用，因此这里我们暂时不使用oh my zsh

### 配置zsh为manjaro风格
安装了archlinux之后如果想把zsh配置为manjaro风格，可以安装`yay -S manjaro-zsh-config`这个包。
之后把`~/.zshrc`更改为：

```
# Use powerline
USE_POWERLINE="true"
# Source manjaro-zsh-configuration
if [[ -e /usr/share/zsh/manjaro-zsh-config ]]; then
  source /usr/share/zsh/manjaro-zsh-config
fi
# Use manjaro zsh prompt
if [[ -e /usr/share/zsh/manjaro-zsh-prompt ]]; then
  source /usr/share/zsh/manjaro-zsh-prompt
fi
```

### 输出炫酷neofetch 

```
yay -S lolcat neofetch
```

```
neofetch | lolcat
```

你会得到炫酷的manjaro/Archlinux图案和你的电脑的基本信息显示。

### GIT配置

```
git config --global  user.name "111" 
git config --global  user.email "111@111.111"   
```

配置好的gitconfig文件在`~/.gitconfig`

### github ssh  配置

生成新 SSH 密钥

可在本地计算机上生成新的 SSH 密钥。 生成密钥后，可以将密钥添加到您在 GitHub.com 上的帐户，以启用通过 SSH 进行 Git 操作的身份验证。

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

之后

```
cat ~/.ssh/id_ed25519.pub
```

然后在github网站上加入SSH KEY即可。

### 配置SSH 使用代理

安装netcat

```
sudo pacman -S netcat                                                                    
:: 有 2 个软件包可提供 netcat ：
:: 软件仓库 extra
   1) gnu-netcat
:: 软件仓库 community
   2) openbsd-netcat

输入某个数字 ( 默认=1 ): 2
正在解析依赖关系...
正在查找软件包冲突...

软件包 (1) openbsd-netcat-1.219_1-1

下载大小：      0.02 MiB
全部安装大小：  0.05 MiB

:: 进行安装吗？ [Y/n] 

```

需要下载openbsd版本的

之后

修改 `~/.ssh/config` 文件

```
Host github.com
    User git
    ProxyCommand nc -v -x 127.0.0.1:7890 %h %p
```

这样git使用ssh方式的时候就可以走代理加速了。

### 注释掉archlinuxcn源

装机的时候用一下可以了。由于manjaro毕竟和archlinux有区别，日常使用不需要打开这个源。可能会导致软件版本出现不兼容。

## 杂项

### 默认应用
默认应用的设置文件：vim ~/.config/mimeapps.list, 可以在里面更改默认浏览器
### I3配置
配置文件在 ~/.i3/config



## I3快捷键

```
MOD+9 锁屏
MOD+数字 切换工作区
MOD+Return 打开终端
MOD+F2 打开浏览器
```

可以在`~/.i3/config`中查看快捷键。

