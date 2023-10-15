---
title: "Maximum subarray problem in time O(n) by divide-and-conquer algorithm"
author: "Hsushuai"
date: 2023-10-15T15:00:59+08:00
summary: 一个 O(n) 时间复杂度的分治算法解决最大子数组问题
tags: ["Algorithm"]
showToc: true
draft: false
---

## Promblem on [Leecode](https://leetcode.com/problems/maximum-subarray/)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组** 是数组中的一个连续部分。

**示例 1：**

> **输入：** nums = [-2,1,-3,4,-1,2,1,-5,4]
> 
> **输出：** 6
> 
> **解释：** 连续最大和子数组为 [4,-1,2,1]，和为 6

**示例 2：**

> **输入：** nums = [1]
> 
> **输出：** 1

**示例 3：**

> **输入：** nums = [5,4,-1,7,8]
> 
> **输出：** 23


在 [CLRS 3th](https://pd.daffodilvarsity.edu.bd/course/material/book-430/pdf_content) (Introduction to Algorithms, Third Edition) 4.1 节，也给出了相似的问题，只是多了一点，我们需要同时返回最大子数组的左右位置索引。书中给出的 Solution 是一个 $T(n) = O(nlogn)$ 的分治算法，后面会简单介绍。

CLRS 的 [Exercise 4.1-5](https://atekihcan.github.io/CLRS/04/E04.01-05/) 中再次提到了最大子数组问题。他让我们尝试实现一个线性的算法来解决。从左到右遍历数组，跟踪记录当前最大的子数组。已知数组 $A[1..j]$ 的最大子数组，若拓展到数组 $A[1..j+1]$ 的最大子数组，有以下两种可能：

1.  $A[1..j]$ 的最大子数组也是 $A[1..j+1]$ 的最大子数组
2.  $A[1..j+1]$ 的最大子数组是 $A[i..j+1]$，其中 $1 \leq i \leq j+1$

这种思想就是判断 $A[j+1]$ 是不是 $[1..j+1]$ 数组的一部分。如果是，则更新当前最大子数组为 $[i..j+1]$。否则我们需要维护当前包括 $A[j+1]$ 的一个数组和，以防后面遇到元素和他加起来大于现在的和。

换而言之就是，维护两个数组：一个是以当前元素结尾的数组，另一个是迄今的最大子数组。

Python 代码实现：

```python
import math

def FindMaxSubarrayLinear(A, low, high):
    best = -math.inf
    current = 0
    left = 0
    right = 0
    current_left = 0
    for i in range(low, high):
        current += A[i]
        if current > best:
            # Update best sum
            best = current
            left = current_left
            right = i
        
        if current < 0:
            # Reset current sum
            current = 0
            current_left = i + 1
    return (left, right, best)

# Test with exact same array as in the example give in book
# Expected answer is A[8..11] = 43

arr = [13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7]
ans = FindMaxSubarrayLinear(arr, 0, len(arr))

print(f"Max subarray = A[{ans[0] + 1}..{ans[1] + 1}] = {ans[2]}")
```

显然，算法时间复杂度为 $O(n)$，因为它之遍历了一遍数组。

> 这个算法实际上也叫做 [Kadane's aglorithm](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm)。你可以在 [GeeksForGeeks](https://www.geeksforgeeks.org/largest-sum-contiguous-subarray/) 上找到更多关于此的讨论。

## Divide-and-conquer in $O(n)$

尽管 Kadane 算法已经是线性的，但是或许是出于对分治法的喜爱，亦或是对递归的着迷，又或是只是 *简单的* 灵光乍现。
不管怎么样，他们都是真正的算法高手，科研之星。而我只是苦困于自己的 HW 🥲。

> Recall the FINDMAXIMUMSUBARRAY algorithm introduced in Section 4.1 in the textbook CLRS. Modify it to obtain an O(n) time divide-and-conquer algorithm for the maximum subarray problem. Your algorithm should not leverage the dynamic programming technique.In particular, your algorithm should not look like an answer for Exercise 4.1-5 in CLRS. (Hint: (a) The recurrence for the runtime of the modified algorithm should be T(n) = 2T(n/2) + O(1). (b) In the “conquer” step, instead of just computing the maximum subarray in A[low, · · · , high], compute and return some more information so that the “combine” step is faster. In particular, finding “maximum crossing subarray” should be fast in the combine step.)

在此之前，我们还是先回想一下 $O(nlogn)$ 的分治法是如何实现的。它将输入分成两半，那么最大子数组只可能来源于 3 个地方：

1. 左半部分
2. 右半部分
3. “中间部分”（左半部分的右侧和右半部分的左侧组成）

1 和 2 在递归中很容易实现。对于 3，我们需要找到左半部分右侧的最大前缀，以及右半部分左侧的最大后缀。

{{< img src="example-1.png" width="100%" caption="分治法求最大子数组实例" >}}


图片引用自：[Study Notes](https://snowan.gitbook.io/study-notes/Leetcode/English%20Solution/53.maximum-sum-subarray-en)

这个算法的时间复杂度递推公式为：$T(n)=2T(n/2)+O(n)$，即 $T(n)=O(nlogn)$

改进方法参考 Quora 上的一个回答（[Eugene Yarovoi](https://qr.ae/pKNGEb)）。如果对于每个递归情况，我们 **不仅返回最优子数组的位置及其和，还返回数组和、最大前缀和后缀的总和** ，则每个递归情况只需要做 $O(1)$ 的工作：

`full.maxPrefix = max (left.maxPrefix, left.sum + right.maxPrefix)`

`full.maxSuffix = max(right.maxSuffix, right.sum + left.maxSuffix)`

`full.sum = left.sum + right.sum`

`full.maxInner = max(left.maxInner, right.maxInner, left.maxSuffix + right.maxPrefix)`

由于单个案例的所有计算都在 $O(1)$ 中运行，因此递归现在为 $T(n) = 2T(n/2) + O(1)$，即 $O(n)$。

python 实现如下：

```python
def FindMaxSubarray(A, low, high):
    if low == high:
        return A[low], A[low], A[low], A[low]
    else:
        mid = (low+high) // 2
        left_maxInner, left_maxPrefix, left_maxSuffix, left_sum = FindMaxSubarray(A,low,mid)
        right_maxInner, right_maxPrefix, right_maxSuffix, right_sum = FindMaxSubarray(A,mid+1,high)
        sum = left_sum+right_sum
        maxPrefix = max(left_maxPrefix, left_sum+right_maxPrefix)
        maxSuffix = max(right_maxSuffix, right_sum+left_maxSuffix)
        maxInner = max(left_maxInner, right_maxInner,left_maxSuffix+right_maxPrefix)
        return maxInner, maxPrefix, maxSuffix, sum
        
        
A=[-3,1,3,8,-2,6,-8,5]
maxInner,maxPrefix,maxSuffix,sum=FindMaxSubarray(A,0,7)
print(maxInner,maxPrefix,maxSuffix,sum)
```

如果还需返回数组位置，只需要代码中添加 “一点点” 参数。

```python
def FindMaxSubarray(A, low, high):
    if low == high:
        return A[low], A[low], A[low], A[low], low, high, high, low
    else:
        mid = (low+high) // 2
        left_maxInner, left_maxPrefix, left_maxSuffix, left_sum, \
            left_maxInner_low, left_maxInner_high, left_maxPrefix_high, left_maxSuffix_low = \
            FindMaxSubarray(A, low, mid)
        right_maxInner, right_maxPrefix, right_maxSuffix, right_sum, \
            right_maxInner_low, right_maxInner_high, right_maxPrefix_high, right_maxSuffix_low =  \
            FindMaxSubarray(A, mid+1, high)
        sum = left_sum+right_sum
        if left_maxPrefix > left_sum+right_maxPrefix:
            maxPrefix = left_maxPrefix
            maxPrefix_high = left_maxPrefix_high
        else:
            maxPrefix = left_sum+right_maxPrefix
            maxPrefix_high = right_maxPrefix_high
        if right_maxSuffix > right_sum+left_maxSuffix:
            maxSuffix = right_maxSuffix
            maxSuffix_low = right_maxSuffix_low
        else:
            maxSuffix = right_sum+left_maxSuffix
            maxSuffix_low = left_maxSuffix_low
        if left_maxInner > right_maxInner and left_maxInner > left_maxSuffix+right_maxPrefix:
            maxInner = left_maxInner
            maxInner_low = left_maxInner_low
            maxInner_high = left_maxInner_high
        elif right_maxInner > left_maxInner and right_maxInner > left_maxSuffix+right_maxPrefix:
            maxInner = right_maxInner
            maxInner_low = right_maxInner_low
            maxInner_high = right_maxInner_high
        else:
            maxInner = left_maxSuffix+right_maxPrefix
            maxInner_low = left_maxSuffix_low
            maxInner_high = right_maxPrefix_high
        return maxInner, maxPrefix, maxSuffix, sum, maxInner_low, maxInner_high, maxPrefix_high, maxSuffix_low


A = [13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7]
maxInner, maxPrefix, maxSuffix, sum, maxInner_low, maxInner_high, maxPrefix_high, maxSuffix_low = \
    FindMaxSubarray(A, 0, 15)
print("maxInner: ", maxInner, "\nmaxPrefix: ", maxPrefix, "\nmaxSuffix: ", maxSuffix, "\nsum: ",
      sum, "\nmaxInner_low: ", maxInner_low, "\nmaxInner_high: ",  maxInner_high, "\nmaxPrefix_high: ",
      maxPrefix_high, "\nmaxSuffix_low: ",  maxSuffix_low)
```

改进后的最大子数组问题求解实例。[查看原图](images/example-2.svg)

{{< svg src="example-2.svg">}}