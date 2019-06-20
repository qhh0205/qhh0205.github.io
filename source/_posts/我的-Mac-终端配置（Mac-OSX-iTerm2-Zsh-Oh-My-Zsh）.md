---
title: 我的 Mac 终端配置（Mac OSX + iTerm2 + Zsh + Oh-My-Zsh）
date: 2018-03-18 09:24:33
categories: Mac
tags: 终端
---

**相关工具介绍**

1. **iTerm2**：Mac 下 Terminal 终端的替代品，拥有更多强大的功能，想了解更多请戳 [iTerm2 官网](https://www.iterm2.com/)；
2. **zsh**：Linux 的一种 shell 外壳，和 bash 属于同类产品；
3. **Oh-My-Zsh**：用来管理 zsh 的配置，同时还有很多社区贡献的主题配置以及好用的插件可供使用，了解更多请戳 [Oh-My-Zsh 官网](http://ohmyz.sh/) ；

**配置方案总览**

1. [iTerm2](https://www.iterm2.com/) 终端工具；
2. iTerm2 [Solarized Dark Higher Contrast 配色方案](https://github.com/mbadolato/iTerm2-Color-Schemes/blob/master/schemes/Solarized%20Dark%20Higher%20Contrast.itermcolors)；
3. [Monaco for Powerline 字体](https://github.com/supermarin/powerline-fonts)；
4. zsh （Mac 系统自带，无需安装）；
5. [Oh-My-Zsh](http://ohmyz.sh/)；
6. [Oh-My-Zsh powerlevel9k 主题](https://github.com/bhilburn/powerlevel9k)；

**最终效果：**
![z-mac-terminal-config-sample1](https://gist.githubusercontent.com/qhh0205/5570934d25a627dd9e9629a8ceeb415c/raw/8f5b3e5ece35629f04fcf66fbc94338e730c3bcd/z-mac-terminal-config-sample1.png)
![z-mac-terminal-config-sample2](https://gist.githubusercontent.com/qhh0205/5570934d25a627dd9e9629a8ceeb415c/raw/8f5b3e5ece35629f04fcf66fbc94338e730c3bcd/z-mac-terminal-config-sample2.png)
![z-mac-terminal-config-sample3](https://gist.githubusercontent.com/qhh0205/5570934d25a627dd9e9629a8ceeb415c/raw/8f5b3e5ece35629f04fcf66fbc94338e730c3bcd/z-mac-terminal-config-sample3.png)
#### 具体配置步骤
##### 1. 安装 iTerm2 终端工具：
打开 [iTerm2 官网](https://www.iterm2.com/) 直接点击 Download 下载并安装即可。
##### 2. 安装 iTerm2 [Solarized Dark Higher Contrast 配色方案](https://github.com/mbadolato/iTerm2-Color-Schemes/blob/master/schemes/Solarized%20Dark%20Higher%20Contrast.itermcolors)：
将该配色方案文件（Solarized Dark Higher Contrast.itermcolors）复制出来，保存到本地，文件命名为 SolarizedDarkHigherContrast.itermcolors ，然后双击即可安装。安装完后打开 iTerm2 终端，依次选择菜单栏：**iTerm2 --> Preferences --> Profiles --> Colors --> Colors Presets -->  SolarizedDarkHigherContrast**，至此 iTerm2 Solarized Dark Higher Contrast 配色方案已成功安装。
##### 3. 安装  [Monaco for Powerline 字体](https://github.com/supermarin/powerline-fonts)：
将该仓库克隆到本地，然后进入工程目录的 Monaco 目录，双击后缀名为 .otf 的字体文件即可完成该字体的安装。安装该字体的原因主要是为了和 Oh-My-Zsh 的 powerlevel9k 主题相兼容，如果不安装该字体，那么后面安装 powerlevel9kn 主题后会出现乱码。
```
# 该仓库中包含好几种优化后的字体
git clone https://github.com/supermarin/powerline-fonts.git
```
##### 4. 安装配置 zsh：
zsh 一般 Mac 已经自带了，无需额外安装。可以用 cat /etc/shells 查看 zsh 是否安装，如果列出了 /bin/zsh 则表明 zsh 已经安装了。
接下来修改 iTerm2 终端的默认 Shell，可以用 echo $SHELL 查看当前 Shell 是什么，如果不是 /bin/zsh 则用如下命令修改 iTerm2 的默认 Shell 为 zsh：
```
chsh -s /bin/zsh
```
##### 5. 安装  [Oh-My-Zsh](http://ohmyz.sh/)：
```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
##### 6. 安装配置 [Oh-My-Zsh powerlevel9k 主题](https://github.com/bhilburn/powerlevel9k)：
- 克隆该仓库到 oh-my-zsh 用户自定义主题目录
```
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```
- 修改 ~/.zshrc 配置文件，配置该主题
```
ZSH_THEME="powerlevel9k/powerlevel9k"
```
- 修改命令提示符
默认的命令提示符为 *user@userdemackbookPro*，这样的提示符配合 powerlevel9k 主题太过冗长，因此我选择将该冗长的提示符去掉，在 ~/.zshrc 配置文件后面追加如下内容：
```
# 注意：DEFAULT_USER 的值必须要是系统用户名才能生效
DEFAULT_USER="user"
```
- 简单定制下 powerlevel9k 主题
powerlevel9k 主题的好处就是可以做很多自定义，只需要简单修改 ~/.zshrc 配置即可生效。更多关于该主题的定制请看 [customizing-prompt-segments](https://github.com/bhilburn/powerlevel9k#customizing-prompt-segments)；
默认的 powerlevel9k 主题最右侧显示的元素为：*每条命令的执行状态，历史命令条数，当前时间*，这样也比较冗余，我在这里将 *历史命令条数* 这一元素去掉，这样看起来比较简洁。这需要修改 ~/.zshrc 配置文件，在后面追加如下内容，定制该主题的显示元素：
```
# 设置 oh-my-zsh powerlevel9k 主题左边元素显示
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(context dir rbenv vcs)
# 设置 oh-my-zsh powerlevel9k 主题右边元素显示
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status root_indicator background_jobs time)
```
##### 7. 配置 zsh 命令语法高亮
[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) 插件可以使你终端输入的命令有语法高亮效果，安装方法如下（oh-my-zsh 插件管理的方式安装）：
1.Clone this repository in oh-my-zsh's plugins directory:
```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
2.Activate the plugin in ~/.zshrc:
```
# 注意：zsh-syntax-highlighting 必须放在最后面（官方推荐）
plugins=( [plugins...] zsh-syntax-highlighting)
```
3.Source ~/.zshrc to take changes into account:
```
source ~/.zshrc
```
##### 8. 安装一些比较实用的 oh-my-zsh 插件
关于 oh-my-zsh 插件的管理是很简单的，有两个插件目录：

- **/Users/user/.oh-my-zsh/plugins**: oh-my-zsh 官方插件目录，该目录已经预装了很多实用的插件，只不过没激活而已；
- **/Users/user/.oh-my-zsh/custom/plugins**: oh-my-zsh 第三方插件目录；

需要安装哪个插件，只需要把插件下载到上面任何一个目录即可，然后在 ~/.zshrc 配置文件中的 plugins 变量中添加对应插件的名称即可，以下是我挑选的几个比较好用的插件（都是官方自带的，无需另外下载），~/.zshrc 配置文件如下：
```
# Add wisely, as too many plugins slow down shell startup.
plugins=(
   git
   extract
   z
   zsh-syntax-highlighting
 )
```
- **git**：oh-my-zsh 默认开启的，没什么好说的；
- **extract**：通用的解压缩插件，可以解压缩任何后缀的压缩文件，使用方法很简单：*x 文件名*；
- **z**：很智能的目录跳转插件，能记录之前 cd 过哪些目录，然后模糊匹配跳转，不需要输入全路径即可跳转，使用方法：*z dir_pattern*
