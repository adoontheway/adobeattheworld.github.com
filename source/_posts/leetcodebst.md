---
title: Leetcode-二叉树
author: ado
date: 2019-08-18 10:59:19
description: 以前的leetcode不知道刷到哪里了，重新刷一下
tags: [leetcode, go]
---

# Preface

最近开始补一补数据结构和算法方式的知识了，想起了很久以前刷过几题的leetcode，于是就试试了。

登陆进去选择了递归主题，递归主题建议先熟悉二叉树和栈。

# Reference

[traverse-a-tree](<https://leetcode.com/explore/learn/card/data-structure-tree/134/traverse-a-tree/992/>)

# Content

 简单的二叉树定义，包括了后面要用到的Preorder,Inorder,Postorder函数：

```go
package data

type BTreeNode struct {
	isRoot bool
	Name string
	Left *BTreeNode
	Right *BTreeNode
}

func (node *BTreeNode) isLeaf() bool {
	return node.Left == nil && node.Right == nil
}

func (node *BTreeNode) Preorder() string {
	var result string = node.Name
	if node.Left != nil {
		result = result + node.Left.Preorder()
	}
	if node.Right != nil {
		result = result + node.Right.Preorder()
	}
	return  result
}

func (node *BTreeNode) Inorder() string {
	var result string = ""
	if node.Left != nil {
		result = result + node.Left.Inorder()
	}
	result = result + node.Name
	if node.Right != nil {
		result = result + node.Right.Inorder()
	}

	return  result
}

func (node *BTreeNode) Postorder() string {
	var result string = ""
	if node.Left != nil {
		result = result + node.Left.Postorder()
	}

	if node.Right != nil {
		result = result + node.Right.Postorder()
	}
	result = result + node.Name
	return  result
}
```

测试类：

```go
package test

import (
	"fmt"
	"leetcode/data"
	"testing"
)

func TestTree(t *testing.T)  {
	root := &data.BTreeNode{
		Name:"F",
		Left:&data.BTreeNode{
			Name:"B",
			Left:&data.BTreeNode{
				Name:"A",
			},
			Right:&data.BTreeNode{
				Name:"D",
				Left:&data.BTreeNode{
					Name:"C",
				},
				Right:&data.BTreeNode{
					Name:"E",
				},
			},
		},
		Right:&data.BTreeNode{
			Right:&data.BTreeNode{
				Name:"G",
				Right:&data.BTreeNode{
					Name:"I",
					Left:&data.BTreeNode{
						Name:"H",
					},
				},
			},
		},
	}
	fmt.Printf("PreOrder result:%s\n",root.Preorder())
	fmt.Printf("InOrder result:%s\n",root.Inorder())
	fmt.Printf("PostOrder result:%s\n",root.Postorder())
}

```

# Result

> === RUN   TestTree
> PreOrder result:FBADCEGIH
> InOrder result:ABCDEFGHI
> PostOrder result:ACEDBHIGF
> --- PASS: TestTree (0.00s)
> PASS

# Solution

[traverse-a-tree](<https://leetcode.com/explore/learn/card/data-structure-tree/134/traverse-a-tree/928/>)

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func preorderTraversal(root *TreeNode) []int {
   result := make([]int,0)
	if root != nil {
		result = append(result, root.Val)
		if root.Left != nil {
			result = append(result, preorderTraversal(root.Left)...)
		}
		if root.Right != nil {
			result = append(result, preorderTraversal(root.Right)...)
		}
	}
	return  result
}

```

>| **68 / 68** test cases passed.            | Status: Accepted             |
>| ----------------------------------------- | ---------------------------- |
>| Runtime: **0 ms**Memory Usage: **2.1 MB** | Submitted: **0 minutes ago** |

# PS

1. 需要注意的是Pre-Order和 In-Order的展示视频其实是有问题的，视频里面演示的其实都是 Post-Order
2. pre-order,in-order,post-order只需要在以上代码中将`result = append(result, root.Val)`的顺序调整即可