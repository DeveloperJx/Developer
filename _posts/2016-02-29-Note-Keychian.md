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
* Swift 2.2

#### 注意事项:

* 真机调试一定要有证书，可以是开发者Free证书，但一定要有，没有无法完成编译

#### 实现步骤:

1.请[在此下载](https://github.com/DeveloperJx/developerjx.github.io/raw/master/_data/TouchMeInRev.starter.zip "title")该实现步骤中所要用到的原始工程文件。   
2.解压后打开工程，由于Swift 2.2与之前版本写法有所改变，xcode提示转换一些代码满足以Swift 2.2，你将看到如下提示：![Alt text](/img/keychian/convert_alert.png "Optional title")    
点击红框中的covert按钮，然后一路next直到出现代码对比框如下图所示：![Alt text](/img/keychian/convert_finish.png "Optional title")    这里可以看到Xcode自动帮我们转换了一些Swift的语法，最后点击红框的save就可以了。    
3.编译一下发现编译无法通过，可见以下图中的问题    
![Alt text](/img/keychian/warning_error.png "Optional title")    
工程中的警告和错误主要是如下的部分：
第一项警告是工程设置警告，可忽略。

| DetailViewController.swift | 错误原因 | 解决方法 |
| -------------------------- | ------- | ------ |
|  54 行  |  语法改变 | 点击代码左端黄色三角自动修复  |
|  61 行  |  语法改变 | 删掉save参数中的nil即可，并放入try块中即可，如下方代码块所示: |    

```swift
do {
		try ManagedObjectContext?.save()
	}catch(let error as NSError){
       	print(error)
	}
```

| AppDelegate.Swift | 错误原因 | 解决方法 |
| ----------------- | ------- | ------ |
| 34 行   | 语法改变   | 点击代码左端的小红圈自动修复  |
| 105 行  | optional类型的解包语法问题  | dict后面加个" as Dictionary"，如下方代码所示: |

```
error = NSError(domain: "YOUR_ERROR_DOMAIN", code: 9999, userInfo: dict as Dictionary)
```

| MasterViewController.Swift | 错误原因 | 解决方法 |
| -------------------------- | ------- | ------ |
| 104-110 行   | Xcode没有检测到变量被调用 | 可忽略  |
| 119 行  | 语法改变  | 点击代码左端的小红圈自动修复 |
| 186 行  | Xcode没有检测到变量被调用  | 可忽略 |
| 215 行  | 方法名重复 | 给方法换名字 |
| 226 行  | 枚举被完全遍历 | 删除default |
    
4.解决上述表中的问题后，工程编译通过，请[在此下载](https://developer.apple.com/library/ios/samplecode/GenericKeychain/GenericKeychain.zip "title")来自于Apple的GenericKeychain，官方编程指导可见[Keychain Services Programming Guide](https://developer.apple.com/library/ios/documentation/Security/Conceptual/keychainServConcepts/01introduction/introduction.html "title")解压并将GenericKeychain中的KeychainItemWrapper.h 与 KeychainItemWrapper.m拖入我们的工程中，当Xcode提示是否建立桥头文件时，如下图所示：![Alt text](/img/keychian/bridge_header_alert.png "Optional title")    
点击Yes来建立头文件连接Objective-C与Swift。（原文在此用的是KeychainWrapper.h 和 KeychainWrapper.m，文件不同，实现也不同，需要注意。）    
5.打开TouchMeIn-Bridging-Header.h文件，在文件开头导入Keychain 封装包：    
```
#import "KeychainItemWrapper.h"
```   
6.在LoginViewController.swift第34行加入以下代码块
```
	@IBOutlet weak var loginButton: UIButton!

	let myKeychainItemWrapper = KeychainItemWrapper(identifier: "Account_Number", accessGroup: nil)
    let createLoginButtonTag = 0
    let loginButtonTag = 1
```
其中myKeychainItemWrapper为Keychain的管理对象，loginButton及下方两个tag用于实现button形态的转变。这里accessGroup填nil是因为使用特殊group无法获得权限（正在查找原因，后期将更新原因）    
    
7.打开Main.storyboard建立loginButton的连接关系，如下图所示：![Alt text](/img/keychian/storyboard_function.png "Optional title")    
8.在LoginViewController.swift的viewDidLoad方法中添加如下过程代码：

```
// 1.
let hasLogin = NSUserDefaults.standardUserDefaults().boolForKey("hasLoginKey")
// 2.
if hasLogin {
    loginButton.setTitle("Login", forState: UIControlState.Normal)
    loginButton.tag = loginButtonTag
    createInfoLabel.hidden = true
} else {
    loginButton.setTitle("Create", forState: UIControlState.Normal)
    loginButton.tag = createLoginButtonTag
    createInfoLabel.hidden = false
}
// 3.
usernameTextField.text = NSUserDefaults.standardUserDefaults().valueForKey("username") as? String
```

以上代码主要完成在NSUserDefaults中存取用户名及随是否登陆过应用改变登录按钮文字状态的功能。    
9.将LoginViewController.swift的loginAction方法的实现改为如下代码：

```
// 1.
if (usernameTextField.text == "" || passwordTextField.text == "") {
    let alert = UIAlertController(title: "You must enter both a username and password!", message: nil, preferredStyle: .Alert)
    let action = UIAlertAction(title: "Oops!", style: .Cancel, handler: nil)
    alert.addAction(action)
    presentViewController(alert, animated: true, completion: nil)
}else{
    // 2.
    usernameTextField.resignFirstResponder()
    passwordTextField.resignFirstResponder()
    
    // 3.
    if sender.tag == createLoginButtonTag {
        
        // 4.
        let hasLoginKey = NSUserDefaults.standardUserDefaults().boolForKey("hasLoginKey")
        if hasLoginKey == false {
            NSUserDefaults.standardUserDefaults().setValue(self.usernameTextField.text, forKey: "username")
        }
        
        // 5.
        myKeychainItemWrapper.setObject(passwordTextField.text, forKey: kSecValueData)
        NSUserDefaults.standardUserDefaults().setBool(true, forKey: "hasLoginKey")
        NSUserDefaults.standardUserDefaults().synchronize()
        loginButton.tag = loginButtonTag
        
        performSegueWithIdentifier("dismissLogin", sender: self)
    } else if sender.tag == loginButtonTag {
        // 6.
        if checkLogin(usernameTextField.text!, password: passwordTextField.text!) {
            performSegueWithIdentifier("dismissLogin", sender: self)
        } else {
            // 7.
            let alert = UIAlertController(title: "Login Problem", message: "Wrong username or password.", preferredStyle: .Alert)
            let action = UIAlertAction(title: "Foiled Again!", style: .Cancel, handler: nil)
            alert.addAction(action)
            presentViewController(alert, animated: true, completion: nil)
        }
    }
}
```
    
其中关键的代码是    
`myKeychainItemWrapper.setObject(passwordTextField.text, forKey: kSecValueData)`
    
该句完成了Keychain的存储操作

10.在LoginViewController.swift的loginAction方法的实现下方加入checkLogin方法检查密码正确性：

```
func checkLogin(username: String, password: String ) -> Bool {
        if password == myKeychainItemWrapper.objectForKey(kSecValueData) as? String &&
            username == NSUserDefaults.standardUserDefaults().valueForKey("username") as? String {
            return true
        } else {
            return false
        }
    }
```
其中关键的代码是    
`myKeychainItemWrapper.objectForKey(kSecValueData)`
    
该句完成了Keychain的读取操作    
11.现在进行编译你会发现工程报错，原因是KeychainItemWrapper是基于MRC模式管理内存的，而工程为ARC，因此需要在工程配置中填入Compliter Flags如下图所示：![Alt text](/img/keychian/compliter_flags.png "Optional title")    
到此App已经可以在模拟器上正常运行了，Keychain功能也是正常的，真机调试还需多做以下几步操作（原文中未提及该部分）：

* 打开工程配置中的Capabilites设置，申请keychain功能权限，如下图所示：
![Alt text](/img/keychian/capabilities_setting.png "Optional title")    
如果你遇到下图所示情形，说明你的证书及Bundle Identifier设置依然存在问题，请继续完成后面的步骤
![Alt text](/img/keychian/capabilities_setting_error.png "Optional title")
* 打开工程配置中的General设置，更改Bundle Identifier至唯一，并在Team中选择自己的开发证书中的团队，点击Fix Issue让Xcode自动修复问题即可，具体如下图所示：
![Alt text](/img/keychian/general_set.png "Optional title")

好了完成以上步骤你已经掌握了Keychain的基本操作

#### 建议:
为了进一步提高安全性，可在将数据存储入Keychain之前对数据做加密操作。
