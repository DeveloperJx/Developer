---
layout: post
title: "在Xcode中实机调试Apple Watch遇到的问题及解决方案"
excerpt: "解决no symbols for paired Apple Watch"
tags: [Apple Watch, Xcode]
author: Jx
---

1. Xcode提示 no symbols for paired Apple Watch

* Xcode 7.3 beta2 (7D129n)
* iPhone 5 iOS 9.3 beta(13E5191d)
* Watch OS 2.2(13V5108c)

#### 问题描述:

出问题的环境如上所示，表现为将配对了watch的iPhone接入后Xcode提示 no symbols for paired Apple Watch，无论重新插拔iPhone，重新配对，升级Xcode，重启两个终端都无效。

#### 解决方案:

* 退出Xcode，并断开iPhone
* 将[watchOS DeviceSupport](http://pan.baidu.com/s/1o7iqUy2 "title")放入路径~/Users/[user]/Library/Developer/Xcode
* 解压watchOS DeviceSupport.zip
* 更改watchOS DeviceSupport中的文件名(如:“2.0 (13S5325c)”,与手表的版本信息相同)
* 打开Xcode，打开工程
* 连接iPhone

#### 其他解决方案：


手表版本为2.0以上的均可尝试上述方法，在2.0以下的请下载下面链接中的dmg进行尝试

* http://stackoverflow.com/questions/31051908/xcode-7-0-beta-ios9-watchos-2-no-symbols-for-paired-apple-watch

