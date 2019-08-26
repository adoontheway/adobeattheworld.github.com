---
title: Leetcode Weekly Contest 96 - 上
author: ado
date: 2019-08-19 17:46:05
description: 编码累了，干点别的
tags: [go, leetcode]

---

# Preface

编码累了，于是去Leetcode上找乐子，于是看到 Contest了，就试试



# Content

## 第一题：3D投射面积



###  Code



```go
func projectionArea(grid [][]int) int {
   result := 0
	len1 := len(grid)
	len2 := len(grid[0])
	yz := make([]int,len1)
	xz := make([]int,len2)
	for i := 0; i < len1; i++ {
		len2=len(grid[i])
		for j := 0; j < len2; j++{
			if grid[i][j] != 0 {
				result++ //xy dimension
				if grid[i][j] > yz[j] {
					yz[j] = grid[i][j] //yz dimension
				}
				if grid[i][j] > xz[i] {
					xz[i] = grid[i][j] //xz dimension
				}
			}
		}
	}

	for i := 0; i < len1; i++ {
		result += yz[i]
	}
	for i := 0; i < len2; i++ {
		result += xz[i]
	}
	return result
}
```

### Result

> | **90 / 90** test cases passed.            | Status: Accepted             |
> | ----------------------------------------- | ---------------------------- |
> | Runtime: **4 ms**Memory Usage: **3.7 MB** | Submitted: **0 minutes ago** |

## 第二题: 用船救人的问题

开始我是审错题了，以为是船不限制人数的，后来发现一船只能坐两人。

### 代码

```go

func numRescueBoats(people []int, limit int) int {
	leng := len(people)
	start1 := time.Now()
	//冒泡排序：耗时3.2343s
	/*
	for i := 0; i<leng-1; i++ {
		for j := 0; j < leng-1-i;j++{
			if people[j] > people[j+1] {
				people[j+1],people[j] = people[j], people[j+1]
			}
		}
	}
	*/
	sort.Ints(people)
	fmt.Println(people)
	end1 := time.Now()
	fmt.Println("Sort Cost:",end1.Sub(start1))
	firstIdx := 0
	i := leng - 1
	//求解耗时0s
	for ; i >= firstIdx; i-- {
		if people[i] >= limit {
			continue
		}else {
			if i != firstIdx && people[i] + people[firstIdx] <= limit {
				firstIdx++
				if firstIdx >= i {
					break
				}
			}
		}
	}
	end2 := time.Now()
	fmt.Println("Solve Cost:",end2.Sub(end1))
	return leng - firstIdx
}
```

提交冒泡排序代码的时候，跑到第71/77的时候，也就是上万条数据的时候，直接就超过解决方案的时间限制了。

自己本地测试了一下5w个数据的情况下，冒泡排序跑了3秒多，debug状态跑了7秒多。

本来写了个快速排序进行优化的，但是无意之间想到，java有自带的数组排序，就连as，js都有默认的数组排序，go应该也有吧，于是找到了`sort.Ints()`

### Result

> Runtime: 136 ms, faster than 25.81% of Go online submissions for Boats to Save People.
>
> Memory Usage: 7.5 MB, less than 100.00% of Go online submissions for Boats to Save People.

耗费时间136ms，只超过了25.81%的人，内存占用7.5M打败了所有人还好，但是总体结果不是很满意。



# Conclude

今天先告一段落，明天继续挑战剩下的题目