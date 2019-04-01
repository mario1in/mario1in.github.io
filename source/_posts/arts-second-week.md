---
title: ARTS挑战-第二周
date: 2019-03-29 16:16:23
tags:
- ARTS
---

# 什么是ARTS？

> 1.Algorithm：每周至少做一个 leetcode 的算法题
> 2.Review：阅读并点评至少一篇英文技术文章
> 3.Tip：学习至少一个技术技巧
> 4.Share：分享一篇有观点和思考的技术文章

# Algorithm

> 一开始以为这道题目挺难的，后来其实慢慢的研究下去发现还好，就是2数相加，需要考虑进位的问题。代码提交后看了一下Solution，发现思路大致也差不多。一开始还怀疑自己，没想到能抓到老鼠的都是好猫。

**Q:**

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

**A:**

```swift
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init(_ val: Int) {
 *         self.val = val
 *         self.next = nil
 *     }
 * }
 */
class Solution {
    /// 两个数字相加，并返回十位和个位
    func addNum(_ num1: Int, _ num2: Int) -> (high: Int, low: Int) {
        var resultH = 0
        var resultL = 0
        
        let sum = num1 + num2
        
        if sum >= 10 {
            resultH = 1
            resultL = sum - 10
        } else {
            resultL = sum
        }
        return (resultH, resultL)
    }
    
    func addTwoNumbers(_ l1: ListNode?, _ l2: ListNode?) -> ListNode? {
        if let l1 = l1, let l2 = l2 {
            var node1: ListNode? = l1
            var node2: ListNode? = l2
            var cursor: ListNode?
            
            var waitToAdd: Int = 0
            var result: ListNode? = ListNode(-1)

            while true {
                // 当两个node节点均为空了停止循环
                if node1 == nil, node2 == nil {
                    break
                }
                
                // 构建空节点
                if node1 == nil {
                    node1 = ListNode(0)
                }
                
                if node2 == nil {
                    node2 = ListNode(0)
                }
                
                let r = addNum(node1!.val, node2!.val)
                let t = addNum(r.low, waitToAdd)
                
                if t.high == 1 {
                    waitToAdd = t.high
                } else {
                    waitToAdd = r.high
                }
                
                if result?.val == -1 {
                    result?.val = t.low
                    cursor = result
                } else {
                    let node = ListNode(t.low)
                    cursor?.next = node
                    cursor = node
                }
                
                node1 = node1!.next
                node2 = node2!.next
            }
            
            if waitToAdd != 0 {
                cursor?.next = ListNode(1)
            }
            
            return result
        }
        
        if l1 == nil {
            return l2
        }
        
        if l2 == nil {
            return l1
        }
        
        return nil
    }
}
```

# Review

这一篇 [Pro Pattern-Matching in Swift](https://www.bignerdranch.com/blog/pro-pattern-matching-in-swift/) 把 Swift 中的 Pattern[ˈpatərn]-Matching 讲得比较详细了。 Pattern-Matching 是 Swift 中一个能把代码写得整洁又安全的方式，必须掌握。
 
# Tip

Swift 中部分高阶函数对比（以操作数组为例）：
`map` vs `flatMap` vs `compactMap` vs `filter` vs `reduce`

### map

将数组中的每个元素进行一次操作

### flatMap

同map，但是不会返回nil，且会对数组中的 `optional` 进行解包

### compactMap

如果 `flatMap` 的回调中可能返回 `optional`，则需要使用 `compactMap`

> 'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value

### filter

顾名思义，可以按照 `closure` 中的标准，把数组中的一些元素过滤掉

### reduce

`reduce` 就是可以把数组拍平，比如一个 `[String]`，用 `reduce` 可以让它返回成 `String`

# Share

[Notes to Myself on Software Engineering](https://shixiong.name/2019/03/29/note-to-myself-on-software-engineering/)，这是 Keras 的开发者 François Chollet 整理的笔记，里面有很多思考值得学习。由于写得太好了，我把它翻译了一下。



