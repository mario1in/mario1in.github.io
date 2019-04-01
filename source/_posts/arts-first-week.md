---
title: ARTS挑战-第一周
date: 2019-03-25 16:25:19
tags:
- ARTS
---

# 什么是ARTS？

> 1.Algorithm：每周至少做一个 leetcode 的算法题
> 2.Review：阅读并点评至少一篇英文技术文章
> 3.Tip：学习至少一个技术技巧
> 4.Share：分享一篇有观点和思考的技术文章


# Algorithm

**Q:**

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

**A:**

```swift
class Solution {
    func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
        for (i, firstItem) in nums.enumerated() {
            for j in i+1..<nums.count {
                let secondItem = nums[j]
                let sum = firstItem + secondItem
                if sum == target {
                    return [i, j]
                }
            }
        }
        
        return []
    }
}
```

# Review

这周分享的一篇技术文章是[Designing a workflow with functions that are capable of failing](https://medium.com/genesis-media/designing-a-workflow-with-functions-that-are-capable-of-failing-c1fb796aa084)，主要介绍了更好的错误处理工作流，包括Swift里标准库新加入的Result<T>，以及适配重试网络请求等逻辑。
（这两天找个时间翻译一下）

# Tip

这周在写Swift的时候，由于我在一个变量的didSet中加入了一些处理的方式，在某些情况下，需要触发变量的didSet方法，于是就会有这样的代码：`self.model = self.model`，这个时候Xcode会报警：`assign a property to itself`。所以在Swift里，是不能把自己赋给自己的。

怎么解决呢？

```swift
let temp = self.model
self.model = temp
```

这样子就简单解决了。可是这样子其他想 assign a property to itself 的地方就得也这么写一遍，有点繁琐。于是就在思考有没有更好的办法呢？

**可以通过自定义operator**

```swift
prefix operator <<
prefix func << <T: Any>(t: T) -> T {
    let temp = t
    return temp
}
```

这样在想要把自己赋给自己的地方这么写就行了

```swift
self.model <<= self.model
```


# Share

这周重温了一下objc的 [行为驱动开发](https://objccn.io/issue-15-1/)

> 有时候改变自己的思维方式真不是一件容易的事，但是当真的能接受新的思考方式并转变的话，一定会有更大的突破的。

