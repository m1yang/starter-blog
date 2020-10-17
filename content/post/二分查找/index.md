---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "二分查找"
subtitle: "借着二分这一算法基础知识，简单的聊聊边界值"
summary: "二分法其实是一种逼近思想，通过不断和中间数比较，找到最接近的那个数。"
authors: [admin]
tags: []
categories: [算法]
date: 2020-10-16T06:25:14+08:00
lastmod: 2020-10-16T06:25:14+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
[二分查找](https://leetcode-cn.com/tag/binary-search/) 是较为基础的一种算法，网上讲解的文章很多，这里调几个重点讲讲。
![](秘技左右横跳.gif)

# 注意事项
一、 线性结构中点的取值：
(low+high)/2 可以写成 (high-low)/2 + low 位运算 (high-low)>>1+low
避免溢出和性能优化

二、 注意查找退出的条件，及两个边界点low和high的更新
当最后剩两个数的时候,low和high仍然要计算中间点，直至只剩一个数的时候，此时low和high是相等的，所以满足继续查询条件需要是low<=high

而当 low和high相等，却没有查询结果时，如果low或high直接被赋值为中间点，那么程序就会进入死循环

# 二分法模板
```JavaScript
var bSearch = function(x, n) {
    let [low, high] = [0, x.length -1]
    while (low <= high) { // 结束条件，当首尾都指向一个数的时候
        mid = low + ((high - low) >> 1)
        if ( x[mid] >= n) { //核心，一直向左找，直到没有比n大的数
            high = mid - 1
        } else {
            low = mid + 1
        }
    }
    return low // 返回第一个大于等于n的坐标
};
```

# 边界值
二分法其实是一种逼近思想，通过不断和中间数比较，找到最接近的那个数。

最后一次循环时low=high，此时如果找到指定值，则high-1，low不变，返回low。如果没有找到指定值，则low+1，返回low。综上，最终返回的是第一个大于等于指定值的数。
如果要返回第一小于指定值的怎么办？
那么需要一直向右找，直到没有比指定值小的数。此时判断条件就变为 x[mid] <=n , low = mid +1。

二分法里，能很明显的看到程序编写过程中对边界判断的普遍。但边界值找起来也并不容易，所以边界值也是测试最常关注的一个点。

我认为，找到边界值和递归思想有很多相似的地方。
首先，都不适合用人脑去分析程序的每个步骤。其次，通晓终止条件都很重要。确定边界的时候，一定要能把终止条件明确的写下来。再根据当前程序判断，达到终止条件后，如果再往下执行一步，会发生什么，这时候程序肯定就不能执行了。再基于终止条件往回走一步，程序还能执行么。有点废话了，就是在你设置的边界，左右反复横跳，看程序是不是处于一个行与不行的中间点。