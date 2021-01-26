---
layout: post
title: "让CI里的Swift Package Manager HTTP鉴权也能666的避坑指南"
excerpt: "Make swift package manager working well in continuous integration workflows"
tags: [CI, Swift Package Manager, Git]
author: Jason Jiang
---

# 前述
本文是一次探【Mo】索【Yu】过程的记述，为了节约您的宝贵时间，需要执行步骤的，请直接查阅步骤总结。

# 遇到问题   
众所周知，Swift Package Manager是Apple官方出品的一套包管理工具，内置于Xcode，其具有较好的离散型，编译时再拉取依赖的特性使得宿主工程更轻便，相较传统cocoapod有亲爹支持，又轻量的优点。我司在整体切换到xcframework后也开始转投Swift Package Manager的怀抱。

然而使用过程中我们发现CI在拉取依赖时会发生鉴权问题，这在Xcode健全的GUI界面下，本不应该是个棘手的问题，然而，在根据[Apple官方文档](https://developer.apple.com/documentation/swift_packages/building_swift_packages_or_apps_that_use_them_in_continuous_integration_workflows "Apple官方文档")中的描述我们发现，SPM的CI鉴权方式，Apple只提供了SSH和Xcode Server两条路子，（Xcode Server用的人太少，不在本文讨论范围）。也就是说如果我们使用Jenkins进行自动化构建，此时又使用了需要鉴权的SPM，若不巧又碰上因特殊原因SSH不能被CI机器访问，那么你就跟我一样遇到了一个尴尬的无法让Jenkins通过鉴权拉取SPM依赖的问题，如下图所示

![Alt text](/img/SwiftPM/auth_failed.png "Optional title")

# 作出假设   
在遇到该问题时，我们首先考虑的是事先将密码扔给Git，因为我们知道理论上，需要鉴权的Git仓库，在我们的每一次操作时都需要进行用户名和密码的鉴权，是Git工具缓存了我们的鉴权凭证，使得在我们的日常使用中，只需要在首次clone Git仓库的时候需要进行鉴权。也就是说，如果提前将需要鉴权的Git仓库的凭据设置到Git工具，让Git在每次拉取的时候自动把鉴权的用户名和密码填上，就能解决HTTP链接下Git仓库的鉴权问题，这TM不就是半小时的事情，Nice😏【事实上 这是个Flag】

# 疯狂踩坑

1. 远程连接没有命令回显

使用SSH远程连接CI机器时，在终端执行```git credential-osxkeychain```相关指令是没有回显的，但实际上它已经执行成功了，所以千万不要在这种状态下执行```git credential-osxkeychain get```来查看保存的凭证，否则你会怀疑人生。

正确的做法是使用GUI工具如VNC或者Mac自带的屏幕共享工具连接CI机器，再到机器上打开终端执行指令。这样就能看到正确的回显以使得工作得以正常进行。

2. 设置了凭据也验证设置成功了，Git命令行依然需要输入凭据

设置的凭据需要保证host路径、protocol协议与Git仓库高度一致，打错了即使能保存进去也是用不了的。

特别的，如果Git仓库路径上带端口，那么host需要写全域名和端口号，如

 ``` host=xxx.xxx.xxx:1234 ``` 

3. 钥匙串访问设置账号密码无效

直接在钥匙串访问上Git工具也是无法用来鉴权的，还是需要通过执行```git credential-osxkeychain store```来保存

4. 把SPM放到工程仓库本地企图更改SPM cache路径

在Fastlane中可以使用```cloned_source_packages_path```参数将依赖放到工程中，然后再自定义SPM以来包路径，理论上可以实现使用工程文件夹下的SPM依赖包来进行编译，但实际上经过多次后尝试发现，它会报出一个invalid错误，如下图所示

![Alt text](/img/SwiftPM/cache_invalid.png "Optional title")

并且这样的操作把依赖拉到了工程目录下，违背了我们使用SPM减少cocoapods对项目工程的侵入这一初衷，所以不建议大家尝试，如果读者知道为何这个自定义的本地SPM路径不能正常编译欢迎评论或给我留言。

5. 密码已成功配置，但Xcode自带的SPM获取不到鉴权信息

经过正确的设置后，我们已经可以直接用```git clone```指令拉取SPM依赖，而不需要输入鉴权信息了。但在xcode中使用其自带的包管理工具添加SPM依赖时，还是会提示输入鉴权信息，这是因为用xcode打开工程时，它默认使用了Xcode自带的Git来进行操作，读取的是它自己维护的鉴权信息，而我们保存的凭据是保存在了系统的Git工具上，Xcode GUI无法读取到，因为它是两个不同的Git，使用两套不同的鉴权信息。

# 成效初显

从前面的五个大坑里爬出来后，终于在命令行下使用```git clone```指令可以直接把Http的依赖库拉取到本地，而不需要再输入用户名和密码了。🌈🌈🌈

![Alt text](/img/SwiftPM/clone_success.png "Optional title")

# 完美交差

经过一顿猛如虎的操作，终于在Jenkins的控台上出现了拉取SPM依赖成功的Log🫂

![Alt text](/img/SwiftPM/success.png "Optional title")

# 步骤总结

* **检查Mac启用的鉴权协助工具**
  
使用命令```git config  -l | grep credential.helper```查看，Mac是否使用了```osxkeychain```，如果没有请使用度娘将这里配置为```osxkeychain```

* **设置xcodebuild/Fastlane使用CI机器的Git**

由于xcode自带Git，默认```xcodebuild```命令使用的是xcode自带的Git，然而它并不会从系统的```osxkeychain```中获取鉴权信息。因此这里需要在```xcodebuild```命令后加上```-scmProvider system```，指定xcode使用CI机器的Git而不是xcode自带的。

如果CI使用Fastlane，则需要在gym里加上```use_system_scm: true```如：

```
gym(
      toolchain: "xxx",
      scheme: "#{SCHEME}",
      export_method: "#{EXPORT_METHOD}",
      configuration: option[:configuration],
      output_directory: "#{OUTPUT_DIRECTORY}",
      include_symbols: true,
      include_bitcode: false,
      xcargs: 'DEBUG_INFORMATION_FORMAT="dwarf-with-dsym"',
      output_name: "#{IPA_NAME}",
      export_xcargs: "-allowProvisioningUpdates",
      use_system_scm: true
    )
```

* **将Git凭据设置到CI机器**

这里可以使用交互式命令```git credential-osxkeychain store```将Git凭据保存到CI机器，如：

```
$git credential-osxkeychain store
host=xxx.xxx.xxx:xxx
protocol=http
username=xxxx
password=xxxx

$git credential-osxkeychain get
host=xxx.xxx.xxx:xxx

>password=xxxx
>username=xxxx
```
##### 注意：这里store的时候```password=xxxx```后面需要按两次回车，以确认前面的输入。同理，get的时候host=xxx.xxx.xxx:xxx后面也是需要两个回车，当看到命令行输出了正确的password和username即为保存成功。另外，如果这里Git仓库路径有端口号，请在host中输入带端口号、带全域名的url，不然会掉入前文所述的坑中，切记！

**第一次操作``` credential-osxkeychain store ```时，可能会弹出Git读取Keychain的许可对话框，建议选择永远允许，并输入CI机器的管理员密码，之后执行所有Git操作时，需要读取Keychain中保存的凭据便不会再弹出对话框**


如果这里想用CI命令保存用户名和密码，那么可以将下方信息保存到一个文件中，如```Password```

```
host=xxx.xxx.xxx:xxx
protocol=http
username=xxxx
password=xxxx
```
然后在对应路径执行```cat Password | git credential-osxkeychain store```即可


如果在使【Da】用【Gong】过程中发现除了上述坑点以外的大坑，或依照本指南无法完成CI和SPM的配置，欢迎与[我](href "mailto:jiangx.jason@gmail.com")取得联系，我将给予一定的帮助，并把遗漏坑点补充到这篇指南中。再次感谢各位大佬对我【Da Gong Ren】的支持，期望本指南能为您节省宝贵的时间。

# 参考文献

1. https://developer.apple.com/documentation/swift_packages/building_swift_packages_or_apps_that_use_them_in_continuous_integration_workflows
2. https://blog.miniasp.com/post/2018/05/28/Git-Credential-Howto
3. https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8
4. https://uptech.team/blog/swift-package-manager
5. https://docs.github.com/cn/github/using-git/caching-your-github-credentials-in-git
