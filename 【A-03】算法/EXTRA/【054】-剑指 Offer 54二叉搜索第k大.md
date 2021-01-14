#  二叉搜索树的第k大节点

给定一棵二叉搜索树，请找出其中第k大的节点。

 **示例**

```go
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4

输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4
```



**代码**

```go
func kthLargest(root *TreeNode, k int) int {
    flag := 0
    if root == nil {
        return 0
    }
    stack := []*TreeNode{}
    for root != nil || len(stack)>0 {
        for root != nil {
            stack = append(stack, root)
            root = root.Right
        }
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        flag++
        if flag == k {
            return node.Val
        }
        root = node.Left
    }
    return 0
}
```

