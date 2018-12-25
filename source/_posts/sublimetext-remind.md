---
title: Sublime Text 使用备忘
date: 2018-12-24 21:56:15
tags: sublime text
---

### 快捷键备忘

* 【control+tab】：在tab之间切换
* 【command+j】：合并两行
* 【command+l】：选择当前行
* 【command+enter】：下一行开辟新行
* 【shift+command+enter】：上一行开辟新行
* 【option+鼠标左键】：块选择并进入多点编辑模式
* 【option+左右键】：移动一个单词（+shift 同时进行选择）
* 【command+左右键】：行首行位切换
* 【shift+command+p】：在命令中直接输入文档类型（如css等可直接切换）
* 【shift+command+p】：reindent lines 代码风格缩进


### 配置文件

那么如何绑定快捷键呢，首先 `shift+command+p` 打开命令面板，输入`keybindings`，点击 `Key Bindings - User`，在配置文件中加入 `[{"key": ["shift+tab"], "command": "reindent", "agrs": {"single_line": false }},]`。

那么如何知道具体命令是什么呢？
可以通过 
```
control+`
``` 
打开工作台，在工作台中输入 `sublime.log_commands(True)`，然后再通过`shift+command+p`打开命令面板，输入 `reindent lines`, 就可以在下边的工作台中看到具体命令名及参数了。

**那么所有的配置都保存到哪了呢？**

可以点击左上角的 `Sublime Text - Preferences - browse Packages...`
这个时候应该会打开到这个目录下 `~/Library/Application Support/Sublime Text 3/Packages`，具体的根据环境不同路径会不同。

在该目录下，用户的所有配置项会存在`User`文件夹中，并且这些配置文件都是纯文本的，所以可以支持版本控制。

除了这个之外，还有一些自定义项也可以通过 `shift+command+p` 打开命令面板，输入 `settings` 打开 `Preferences: Settings - User` 便可找到配置自定义配置项的配置文件了。

### 使用Packages Control安装扩展包

安装 `Packages Control` 的方法可以参考这个 [Installation - Package Control](https://packagecontrol.io/installation)

官网上也有很多库可供选择。安装完`Packages Control`之后，包都会存放在 `~/Library/Application Support/Sublime Text 3/Packages` 里边的 `Installed Packages` 里。并且会在 `Packages/User` 里新加一个 `Packge Control.sublime-settings` 配置项。

推荐的一些扩展包

> `AdvancedNewFile`
`Emmet`
`Git`
`Sass`
`SublimeERB`
`SyncedSideBar`

### 快速查找

正常情况下，可以通过 `command+p` 直接输入模糊搜索：

> eg: 模糊搜索名+冒号+行号，如：`demo:10`，光标会直接停在第10行
> eg: 搜索名@函数名会跳转到函数名，如：`demo@new`

可以通过 `command+f` 在本文件中进行查找：

> `enter`：搜索下一处；`shift+enter`：搜索上一处；`esc`：停在当前单词
> `command+d` n次，可以同时编辑，并进入多点编辑

要在文件夹下查找的话，需要在文件夹上右键，点击 `find in finder`，会把相应的内容记在一个文本里，按`F4`可以跳转到第一个查找项，或者也可以直接在文本里用鼠标双击打开。

跳走了可以用 `control+-` 直接跳回去。
如果跳多了，可以 `control+shift+-` 可以再进行回到上一级的操作。

### Emmet

牛逼！（todo）

### 代码片段
（todo）

### 自动补齐
（todo）

/user下的 `damemon.sublime-completions`

sublimeCodeIntel

### build system
（todo）

