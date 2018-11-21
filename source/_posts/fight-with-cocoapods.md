---
title: 与Cocoapods抗战的一天
date: 2018-05-22 22:43:54
tag:
- iOS
---

> 在升级Mac系统到macOS High Sierra（10.13.4）的时候，出现升级失败导致开不了机，所有的数据全没了。
> 🌶🐔🍎💊
> 之前的系统环境都要全部重新配置，在配置cocoapods的时候遇到了一些问题，特此记录。

## 安装cocoapods依赖的环境

这步网上有很多[教程](https://www.google.com.hk/search?q=%E5%AE%89%E8%A3%85cocoapods&oq=%E5%AE%89%E8%A3%85cocoapods&aqs=chrome..69i57j69i61j0.3420j0j4&sourceid=chrome&ie=UTF-8)，就不再赘述。
然而安装完后，在执行 `pod install` 或 `pod update` 时，就会出现如下错误：

```
ArgumentError - Malformed version number string
/Library/Ruby/Site/2.3.0/rubygems/version.rb:209:in `initialize'
/Library/Ruby/Site/2.3.0/rubygems/version.rb:200:in `new'
/Library/Ruby/Site/2.3.0/rubygems/version.rb:200:in `new'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/generator/xcconfig/aggregate_xcconfig.rb:123:in `embedded_content_settings'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/generator/xcconfig/aggregate_xcconfig.rb:68:in `generate'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/generator/xcconfig/aggregate_xcconfig.rb:39:in `save_as'
...
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/installer.rb:183:in `generate_pods_project'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/installer.rb:119:in `install!'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/command/install.rb:41:in `run'
/Library/Ruby/Gems/2.3.0/gems/claide-1.0.2/lib/claide/command.rb:334:in `run'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/lib/cocoapods/command.rb:52:in `run'
/Library/Ruby/Gems/2.3.0/gems/cocoapods-1.5.2/bin/pod:55:in `<top (required)>'
/usr/bin/pod:23:in `load'
/usr/bin/pod:23:in `<main>'
```

（一脸懵逼...）

Google搜了一堆也没找到解决办法，后来在Cocoapods的issues里找到[一个解决方案](https://github.com/CocoaPods/CocoaPods/issues/7756):

> jcharr1: I came across this issue today, too, when trying to run "pod install." What version of Rubygems are you running? For me, I had updated with `gem update --system` which brought me up to v2.7.7. Downgrading to 2.7.6 with `gem update --system 2.7.6` fixed this for me.

这位 `jcharr1` 说gem版本太高了导致的，他降级到2.7.6就可以了。于是我也尝试着降级了一下，果然解决了！不过其中缘由不得而知，有时间再研究一下。

## Pod install

现在可以正常 `pod install` 和 `pod update` 了吗？并不。
当我按下回车键之后，又出现了一个小问题：

```
Analyzing dependencies
Pre-downloading: `XXXX` from `http://xxx.com/xxx/xxx.git`, tag `xxx_6.9.5`
Username for 'http://xxx.com': aaa
Password for 'http://aaa@xxx.com':
```

其实这个输入账号密码就能解决问题了，但是我比较好奇的是我已经有添加了 `SSH Key`，为什么还会要我输入账号密码呢？原来，当我们的pod组件的地址使用的是 `http/https` 地址的时候，就会使用账号密码验证，当使用ssh地址的时候，就会用 `SSH Key` 验证。
所以当我们不想输入账号密码的时候，可以在 `~/.gitconfig` 中加入如下代码：

```
[url "git@github.com:"]
  insteadOf = https://github.com/
[url "git@github.com:"]
  pushInsteadOf = "git://github.com/"
[url "git@github.com:"]
  pushInsteadOf = "https://github.com/"
```

相关地址可以根据具体情况具体修改，修改完之后就会强制使用 `ssh` 验证了。

然后 `pod install`，可以了吗？不！它又出错了。

```
[!] Error installing MTCameraAR
[!] Failed to download 'MTCameraAR'
```

（一脸懵逼again...）

后来尝试卸载重装cocoapods，都不行。正当无可奈何之际，突然想起来，pod有 `--verbose` 这个参数啊，说不定会有帮助。
于是 `pod install --verbose`

![](http://7viin1.com1.z0.glb.clouddn.com/pic.png)

关键的错误就出现了， `git-lfs: command not found`
由于这个库的文件都很大，没有使用git-lfs的话会超时，无法成功拉取。所以只要[安装git-lfs](https://github.com/git-lfs/git-lfs/wiki/Installation)就好了。

bingo！问题都解决了。




