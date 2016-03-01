---
layout: post
title: "Swift 2.2下使用Keychain保存用户机密数据"
excerpt: "使用Apple官方Demo中的KeychainItemWrapper保存用户机密数据."
tags: [Swift 2.2, Keychain, 数据安全, iOS安全]
---

#### 初衷:

本文改编自[http://www.raywenderlich.com/92667/securing-ios-data-keychain-touch-id-1password](http://www.raywenderlich.com/92667/securing-ios-data-keychain-touch-id-1password "title")

因Apple官方Demo中的KeychainItemWrapper类实现与上文中描述有所区别，加之现有利用官方Demo完成Keychain操作的技术资料较少（到处都在用第三方库），在此着重纪录Keychain存储实现过程中不同的地方供读者参考。本文仅描述上文中Keychain的区别部分，对其余安全技术不予置评。

#### 环境:

* Xcode 7.3 beta2 (7D129n)
* iPhone 5 iOS 9.3 beta(13E5191d)

#### 注意事项:

* 真机调试一定要有证书，可以是开发者Free证书，但一定要有，没有无法完成编译

#### 实现步骤:

1. 请[在此下载](https://github.com/DeveloperJx/developerjx.github.io/raw/master/_data/TouchMeInRev.starter.zip "title")该实现步骤中所要用到的原始工程文件。
2. 解压后打开工程，由于Swift 2.2与之前版本写法有所改变，xcode提示转换一些代码满足以Swift 2.2，你将看到如下提示![Alt text](/img/img.jpg "Optional title")
点击红框中的covert按钮，然后一路next直到出现代码对比框，这里可以看到Xcode自动帮我们转换了一些Swift的语法，最后点击红框的save就可以了。
3. 编译一下发现编译无法通过，可见以下图中的问题![Alt text](/path/to/img.jpg "Optional title")
工程中的警告和错误主要是如下的部分：
第一项警告是工程设置警告，可忽略。

| DetailViewController.swift | 错误原因 | 解决方法 |
|:--------|:-------:|--------:|
| 54 行   | 语法改变   | 点击代码左端黄色三角自动修复  |
| 61 行  | 语法改变  | 删掉save参数中的nil即可，并放入try块中即可，如下所示。
`do {
        try ManagedObjectContext?.save()
    }catch(let error as NSError){
        print(error)
    }`   |
|----
| cell 1   | cell 2   | cell 3   |
| cell 4   | cell 5   | cell 6   |
|=====
| Foot 1   | Foot 2   | Foot 3   |

| AppDelegate.Swift | 错误原因 | 解决方法 |
|:--------|:-------:|--------:|
| 34 行   | 语法改变   | 点击代码左端的小红圈自动修复  |
| 105 行  | optional类型的解包语法问题  | dict后面加个" as Dictionary"，如下所示:
`error = NSError(domain: "YOUR_ERROR_DOMAIN", code: 9999, userInfo: dict as Dictionary)` |
|=====

| MasterViewController.Swift | 错误原因 | 解决方法 |
|:--------|:-------:|--------:|
| 104-110 行   | Xcode没有检测到变量被调用 | 可忽略  |
| 119 行  | 语法改变  | 点击代码左端的小红圈自动修复 |
| 186 行  | Xcode没有检测到变量被调用  | 可忽略 |
| 215 行  | 方法名重复 | 给方法换名字 |
| 226 行  | 枚举被完全遍历 | 删除default |
|=====


4. 解决上述表中的问题后，工程编译通过，请[在此下载](https://developer.apple.com/library/ios/samplecode/GenericKeychain/GenericKeychain.zip "title")来自于Apple的GenericKeychain，官方编程指导可见[Keychain Services Programming Guide](https://developer.apple.com/library/ios/documentation/Security/Conceptual/keychainServConcepts/01introduction/introduction.html "title")解压并将GenericKeychain中的KeychainItemWrapper.h 与 KeychainItemWrapper.m拖入我们的工程中，当Xcode提示是否建立桥头文件时，点击Yes来建立头文件连接Objective-C与Swift。（原文在此用的是KeychainWrapper.h 和 KeychainWrapper.m，文件不同，实现也不同，需要注意。）
5. 打开TouchMeIn-Bridging-Header.h文件，在文件开头导入Keychain 封装包：`＃import "KeychainItemWrapper.h"`
6. 

#### 建议:
为了进一步提高安全性，可在将数据存储入Keychain之前对数据做加密操作。