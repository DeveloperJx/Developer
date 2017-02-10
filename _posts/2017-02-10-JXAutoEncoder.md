---
layout: post
title: "一个基于Swift reflect的归解档简化工具类"
excerpt: "An archiver simplification utility class base on swift reflect."
tags: [归档, 解档, Swift反射]
author: Jx
---

# 初衷
众所周知，在OC时代，MJ大神使用一句宏定义MJCodingImplementation就搞定了归解档中繁琐的编码与解码工作。而在Swift中，没有Runtime大法，在日常编码过程中遇到需要归档的模型，只要写编码和解码方法，就会写到吐血。于是我找到了Swift中的Reflect特性来帮助我们简化这部分的工作。    

# 效果   
在使用JXAutoEncoder这个工具类之前，要完成一个类的归档工作需要编写类似以下的代码

```Swift
class ArchiverModel: NSObject, NSCoding {

    var bool = true
    var int = 1
    var double = M_PI
    var string = ""
    var array = ["123", "456"]
    var dictionary = ["abc": "cba"]
    var data = "hello world".data(using: String.Encoding.utf8)
    var date = Date()


    /// 归档到文件
    func archiveToFile() {
        var modelFile = NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.documentDirectory,
                                                            FileManager.SearchPathDomainMask.userDomainMask,
                                                            true)[0]
        modelFile += "/model.data"
        NSKeyedArchiver.archiveRootObject(self, toFile: modelFile)
    }

    /// 从文件中解档
    ///
    /// - Returns: 解档后的Model
    class func decodedFromFile() throws -> ArchiverModel {
        var modelFile = NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.documentDirectory,
                                                            FileManager.SearchPathDomainMask.userDomainMask,
                                                            true)[0]
        modelFile += "/model.data"

        if FileManager.default.fileExists(atPath: modelFile) {
            if let model = NSKeyedUnarchiver.unarchiveObject(withFile: modelFile) as? ArchiverModel {
                return model
            }else{
                throw NSError(domain: "Unarchive fail", code: 100, userInfo: nil)
            }
        }else{
            throw NSError(domain: "File doesn't exists", code: 101, userInfo: nil)
        }
    }

    required init?(coder aDecoder: NSCoder) {
        super.init()
        if let boolValue = aDecoder.decodeObject(forKey: "bool") as? Bool {
            bool = boolValue
        }
        if let intValue = aDecoder.decodeObject(forKey: "int") as? Int {
            int = intValue
        }
        if let doubleValue = aDecoder.decodeObject(forKey: "double") as? Double {
            double = doubleValue
        }
        if let stringValue = aDecoder.decodeObject(forKey: "string") as? String {
            string = stringValue
        }
        if let arrayValue = aDecoder.decodeObject(forKey: "array") as? [String] {
            array = arrayValue
        }
        if let dictionaryValue = aDecoder.decodeObject(forKey: "dictionary") as? [String: String] {
            dictionary = dictionaryValue
        }
        if let dataValue = aDecoder.decodeObject(forKey: "data") as? Data {
            data = dataValue
        }
        if let dateValue = aDecoder.decodeObject(forKey: "date") as? Date {
            date = dateValue
        }
    }

    func encode(with aCoder: NSCoder) {
        aCoder.encode(bool, forKey: "bool")
        aCoder.encode(int, forKey: "int")
        aCoder.encode(double, forKey: "double")
        aCoder.encode(string, forKey: "string")
        aCoder.encode(string, forKey: "array")
        aCoder.encode(string, forKey: "dictionary")
        aCoder.encode(string, forKey: "data")
        aCoder.encode(string, forKey: "date")
    }
}
```  

密密麻麻的```aCoder.encode```看得我密集恐惧症都犯了，更别说这个类如果要添加新的属性会怎样了，并且当属性变多，encode和decode就很容易漏写，造成应用崩溃。使用工具类之后，这种情况就可以完全避免了，如下所示：

```Swift

import JXAutoEncoder

class ArchiverModel: JXAutoEncoder {

    var bool = true
    var int = 1
    var double = M_PI
    var string = ""
    var array = ["123", "456"]
    var dictionary = ["abc": "cba"]
    var data = "hello world".data(using: String.Encoding.utf8)
    var date = Date()


    /// 归档到文件
    func archiveToFile() {
        var modelFile = NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.documentDirectory,
                                                            FileManager.SearchPathDomainMask.userDomainMask,
                                                            true)[0]
        modelFile += "/model.data"
        NSKeyedArchiver.archiveRootObject(self, toFile: modelFile)
    }

    /// 从文件中解档
    ///
    /// - Returns: 解档后的Model
    class func decodedFromFile() throws -> ArchiverModel {
        var modelFile = NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.documentDirectory,
                                                            FileManager.SearchPathDomainMask.userDomainMask,
                                                            true)[0]
        modelFile += "/model.data"

        if FileManager.default.fileExists(atPath: modelFile) {
            if let model = NSKeyedUnarchiver.unarchiveObject(withFile: modelFile) as? ArchiverModel {
                return model
            }else{
                throw NSError(domain: "Unarchive fail", code: 100, userInfo: nil)
            }
        }else{
            throw NSError(domain: "File doesn't exists", code: 101, userInfo: nil)
        }
    }
}
```
简洁明了，干净透彻，若要增加属性，则直接var...就好了，是不是方便了许多。

# 部署

JXAutoEncoder这个工具类支持pod导入，导入方式```pod 'JXAutoEncoder'``` ,具体可查阅[项目主页](https://github.com/DeveloperJx/JXAutoEncoder "项目主页")。使用中若发现Bug，请提issue，使用顺手请star，谢谢🙏！ 
