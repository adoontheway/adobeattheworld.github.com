---
title: 每天进步一点点009 - LeetCode 数组中间值
author: ado
date: 2020-01-02 20:08:12
description: 最近心很方，需要专注起来做点东西充实一下自己
tags: [leetcode, go]
---
# Preface
没有
# Submission
## 来源
[2个已排序数组的中间值](https://leetcode.com/problems/median-of-two-sorted-arrays/submissions/)
## 2个排序数组的中间值
两个长度分别为 **m** 和 **n** 的数组 **nums1** 和 **nums2**  
查找他们的数组中间值  
时间复杂度不得超过 *O(log (m+n))*  
**m** 和 **n** 不会同时为 **0**
## Examples
```go
nums1 = [1, 3]
nums2 = [2]

//中间值： 2.0

nums1 = [1, 2]
nums2 = [3, 4]

//中间值： (2 + 3)/2 = 2.5
```

## 提交方案
```go
package rock

func FindMedianSortedArrays(nums1 []int, nums2 []int) float64 {
	len1 := len(nums1)
	len2 := len(nums2)
	len := len1 + len2
	var index1 int = len1/2
	var index2 int = len2/2
	need0,need1 := -1,-1
	var result float64 = 0.0
	need0 = (len - 1)/2
	if len%2 == 0 {
		need1 = need0 + 1
	}

	if len1 == 0 {
		if need1 == -1 {
			return float64(nums2[need0])
		}
		return  float64(nums2[need0]+nums2[need1])*0.5
	}else if len2 == 0{
		if need1 == -1 {
			return float64(nums1[need0])
		}
		return  float64(nums1[need0]+nums1[need1])*0.5
	}else if nums1[index1] <= nums2[index2] {
		for i := index1 + 1; i< len1; i++ {
			index1 = i
			if nums1[index1] >= nums2[index2] || index1 > need0 + 1 {
				break
			}
		}
	}else if nums1[index1] > nums2[index2] {
		for i := index2 + 1; i < len2; i++ {
			index2 = i
			if nums1[index1] <= nums2[index2] || index2 > need0 + 1 {
				break
			}
		}
	}else {
		return float64(nums1[index1])
	}


	len = index2+index1+1

	for i := len; i >= 0; i-- {
		if index1 >= 0 && index2 >= 0 {
			if nums1[index1] > nums2[index2] {
				if i == need0 || i == need1 {
					result += float64(nums1[index1])
					if i == need0 {
						break
					}
				}
				index1--
			} else {
				if i == need0 || i == need1 {
					result += float64(nums2[index2])
					if i == need0 {
						break
					}
				}
				index2--
			}
		}else if index1 >= 0 {
			if need0 <= i {
				result += float64(nums1[need0])
			}
			if need1 <= i && need1 != -1 {
				result += float64(nums1[need1])
			}
			break
		}else {
			if need0 <= i {
				result += float64(nums2[need0])
			}
			if need1 <= i && need1 != -1 {
				result += float64(nums2[need1])
			}
			break
		}
	}
	if need1 != -1 {
		return result*0.5
	}
	return result
}
```
## 原理
根据2个数组的长度，计算需要获取总数据里面需要的目标索引  
对比两个数组的中间值，迭代小的那一个，直到找到大于等于大的那一个的中间值或者大于等于目标索引值的长度

## 结果
[submissions/detail/290502554/](https://leetcode.com/submissions/detail/290502554/)
### 耗时
![耗时](./1.png)
### 内存
![内存](./2.png)

# 结果
不满意，耗时输给了66%的人，内存还输给了20%的人，稍后参考下别人的做法。