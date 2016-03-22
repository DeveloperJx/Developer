---
layout: post
title: "一个精确到纳秒级的Swift代码性能测试方法"
excerpt: "An accurate to nanoseconds code execution time detection method."
tags: [性能, 测试]
author: Jx
---

# 原理
这是一个比较底层的代码运行性能测试方法，基于的是CPU的时钟计数（又名时钟节拍、CPU心跳）。计时原理是通过读取代码执行前和执行后的系统内核CPU滴答计数，并计算其差值，乘上每个滴答的时间来获得被测代码段的执行时间。实在不能理解的硬件同学请复习μC/OS时间管理相关章节，软件的同学请自行学习数字电路原理，并尝试理解CPU的工作方式。
绕远了，实际上就是下方这堆代码。    
# 代码
```
func time(time:UInt64) -> Double {
        let startTime = mach_absolute_time()
        
        // TODO: add your code here
        
        let finishTime = mach_absolute_time()
        var timebase = mach_timebase_info_data_t()
        mach_timebase_info(&timebase)
        return (Double(time) * Double(timebase.numer) / Double(timebase.denom) / 1e9)
    }
```  
# 代码解释
这里稍微解释一下，mach_absolute_time()是一个CPU/总线依赖函数，返回一个基于系统启动后的时钟“嘀嗒”数。mach_timebase_info_data_t是一个描述CPU时间基准的结构体，内部包含一个number（分子），一个denom（分母），通过mach_timebase_info函数获得时间基准后将用于后面的将“嘀嗒”数转成时间的过程。在进行Double(time) * Double(timebase.numer) / Double(timebase.denom)运算后“嘀嗒”数被转换成了纳秒，再除10的9次方就是秒了。   

# 参考文献

* [http://iosdeveloperzone.com/2011/05/03/quick-performance-measurements](http://iosdeveloperzone.com/2011/05/03/quick-performance-measurements "") 

* [http://www.cocoachina.com/industry/20130608/6362.html](http://www.cocoachina.com/industry/20130608/6362.html "")
