#  平衡二叉树

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树

**示例**

```go
输入  
	3
   / \
  9  20
    /  \
   15   7
返回: true
输入:
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
返回: false
```



**代码**

```go
func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
func getHeight(root *TreeNode) int {
    if root == nil {
        return 0
    }
    leftHeight := getHeight(root.Left)
    rightHeight := getHeight(root.Right)
    if leftHeight == -1 || rightHeight == -1 {
        return -1
    }
    if abs(leftHeight-rightHeight)<=1 {
        return max(leftHeight, rightHeight)+1
    }
    return -1
}
func isBalanced(root *TreeNode) bool {
    if root == nil {
        return true
    }
    res := getHeight(root)
    return res != -1
}
```

