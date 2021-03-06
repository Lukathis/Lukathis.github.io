---
layout: post
title:  "Binary Tree"
author: Xu Chi
toc: true
tags: [Tech, Algo]
---

二叉树典型题目及小结。

# 二叉树的遍历

遍历分为前序，中序和后序遍历。取决于什么时候访问根节点。

| 遍历方式 | 顺序 |
| --- | --- |
| 前序 | 根左右 |
| 中序 | 左根右 |
| 后序 | 左右根 |

遍历有递归和迭代两种写法，回溯的写法较为简单。

数据结构定义如下：
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
```

## 递归实现

```java

// Preorder
class Solution {
    
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        preorderTraverse(root, result);
        return result;
    }

    private void preorderTraverse(TreeNode node, List<Integer> result) {
        if (node == null) return;
        result.add(node.val);
        preorderTraverse(node.left, result);
        preorderTraverse(node.right, result);
    }
}

// Inorder
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        inorderTraverse(root, result);
        return result;
    }

    private void inorderTraverse(TreeNode node, List<Integer> result) {
        if (node == null) return;
        inorderTraverse(node.left, result);
        result.add(node.val);
        inorderTraverse(node.right, result);
    }
}

// Postorder
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        postorderTraverse(root, result);
        return result;
    }

    private void postorderTraverse(TreeNode node, List<Integer> result) {
        if (node == null) return;
        postorderTraverse(node.left, result);
        postorderTraverse(node.right, result);
        result.add(node.val);
    }
}
```

## 迭代实现

递归实现本质上是隐式地使用了函数的栈，而如果使用迭代则可以自己维护一套栈来存储中间结果。

主要的难点是如何维护这个栈。
我们观察如下的三个迭代实现，在迭代过程中，如果当前节点不为空，或者栈中尚存未处理的节点，则需要不断循环。
在每次循环中，总是需要不断地先把当前节点推进栈中进行暂存，然后把腾出来的当前指针指向左节点，从而达到不断向左推进但是仍旧记录下遍历下来的路径。
* 对于先序遍历来说，在把当前节点指针传递给左节点之前，需要在结果中添加当前根节点的值
* 对于中序遍历来说，因为向栈中推进节点的顺序符合先根节点后左节点的原则，因此只需要每次弹出时向结果集添加该节点的值即可。添加完毕后把当前节点的指针传递给右子树即可。
* 对于后序遍历来说，情形相对复杂，解释如下

对于先序和中序来说，因为根不在最后一个，因此退出当前循环的时候，只需要把当前右子树的节点推进栈中即可。
对于后序遍历来说，需要先处理两棵子树。因此每次大循环开始时和先序、中序类似，先不断地向左进行循环，找到需要最先处理的左子树。
此后，从栈中拿到当前需要处理的节点时，
* 如果该节点右子树为空或*已经被处理*过，则此时可以将当前节点数值添加进结果集。判断为空很简单，而判断*已经被处理*则需要额外维护一个指针`prev`来记录上一次被处理的节点。因为后序是*左右根*，如果当前节点有右子树且被处理过，则其必是前一次处理的节点。
* 如果该节点右子树不为空或者没有被处理过，那么则简单地把当前节点推进栈中，并且把当前节点的指针指向右节点


```java
// preorder
class Solution {
    
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Deque<TreeNode> stack = new LinkedList<>();
        TreeNode node = root;
        while(node != null || !stack.isEmpty()) {
            while (node != null) {
                result.add(node.val);
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            node = node.right;
        }
        return result;
    }
}

// inorder
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Deque<TreeNode> stack = new LinkedList<>();
        TreeNode node = root;
        while (node != null || !stack.isEmpty()) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            result.add(node.val);
            node = node.right;
        }

        return result;
    }
}

// postorder
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Deque<TreeNode> stack = new LinkedList<>();
        TreeNode node = root;
        TreeNode prev = null;
        while (node != null || !stack.isEmpty()) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }

            node = stack.pop();

            if (node.right == null || node.right == prev) {
                result.add(node.val);
                prev = node;
                node = null;
            } else {
                stack.push(node);
                node = node.right;
            }
        }
        return result;
    }
}

```

# 二叉树的层序遍历

所谓层序遍历，其实就是二叉树的宽度优先遍历。宽度优先遍历则需要一个队列来配合搜索。

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();

        if (root == null) return result;

        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);

        while (!q.isEmpty()) {
            List<Integer> layer = new ArrayList<>();
            int size = q.size();

            for (int i = 0; i < size; i++) {
                TreeNode node = q.poll();
                layer.add(node.val);

                if (node.left != null) {
                    q.offer(node.left);
                }

                if (node.right != null) {
                    q.offer(node.right);
                }
            }
            result.add(layer);
        }
        return result;
    }
}
```

# 典型例题

## 104. 二叉树的最大深度


> 给定一个二叉树，找出其最大深度。
>
> 二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。
>
> 说明: 叶子节点是指没有子节点的节点。
>
> 示例：
>
> 给定二叉树 [3,9,20,null,null,15,7]，
>
>        3
>       / \
>      9  20
>        /  \
>      15   7
>
>返回它的最大深度 3 。


利用递归可以简单得到一棵树的最大深度，基本上有两个思路：
* 自顶向下
```java
public class Solution {
    int result = 0;
    public int maxDepth(TreeNode root) {
        maxDepth(root, 0);
        return result;
    }

    public void maxDepth(TreeNode node, int depth) {
        if (node == null) return;

        if (node.left == null && node.right == null) {
            result = Math.max(result, depth + 1);
        }

        maxDepth(node.left, depth + 1);
        maxDepth(node.right, depth + 1);
    }
}
```

* 自底向上
```java
public class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null){
            return 0;
        }
        
        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        return Math.max(left, right) + 1;
    }
}
```


## 101. 对称二叉树

对该棵树同时对称地进行遍历，判断两者遍历的节点是否相同。

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        return isSymmetric(root, root);
    }

    public boolean isSymmetric(TreeNode n1, TreeNode n2) {
      if (n1 == null && n2 == null) return true;
      if (n1 == null || n2 == null) return false;
      return n1.val == n2.val && isSymmetric(n1.left, n2.right) && isSymmetric(n1.right, n2.left);
    }
}
```

## 112. 路径总和


> 给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。
>
> 说明: 叶子节点是指没有子节点的节点。
>
> 示例: 
>
> 给定如下二叉树，以及目标和 sum = 22，
>
>              5
>             / \
>            4   8
>           /   / \
>          11  13  4
>         /  \      \
>        7    2      1
>
> 返回 true, 因为存在目标和为 22 的根节点到叶子节点的路径 5->4->11->2。

递归地解决问题。

```java
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if (root == null) return false;

        if (root.left == null && root.right == null) {
            return root.val == sum;
        }

        return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
    }
}
```

## 106. 从中序与后序遍历序列构造二叉树

> 根据一棵树的中序遍历与后序遍历构造二叉树。
> 
> 注意: 你可以假设树中没有重复的元素。
>
> 例如，给出
>
> 中序遍历 inorder = [9,3,15,20,7]
> 
> 后序遍历 postorder = [9,15,7,20,3]
>
> 返回如下的二叉树：
>
>        3
>       / \
>      9  20
>        /  \
>       15   7


思路：这是一道很典型的题目。
如果我们徒手来复原该树的话，我们可以观察到，在后序遍历中，根永远在最后一个。
取得此根后，可以在中序遍历中，将该树的左右子树分开。
对于左右两棵子树，仍然采用上述方法确定子树的根节点，递归地解决该问题。

直接解如下，在数量较大时表现较差。需要做一定的优化。
```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {

        if (inorder.length == 0) {
            return null;
        }

        int currentVal = postorder[postorder.length - 1];
        TreeNode currentNode = new TreeNode(currentVal);

        if (inorder.length == 1) {
            return currentNode;
        }
   
        int currentInorderIdx = findIdx(inorder, currentVal);

        int[] leftInorder = Arrays.copyOfRange(inorder, 0, currentInorderIdx);
        int[] rightInorder = Arrays.copyOfRange(inorder, currentInorderIdx + 1, inorder.length);

        int[] leftPostorder = constructPostorder(leftInorder, postorder);
        int[] rightPostorder = constructPostorder(rightInorder, postorder);

        TreeNode leftNode = buildTree(leftInorder, leftPostorder);
        TreeNode rightNode = buildTree(rightInorder, rightPostorder);

        currentNode.left = leftNode;
        currentNode.right = rightNode;

        return currentNode;
    }

    private int findIdx(int[] array, int target) {
        for (int i = 0; i < array.length; i++) {
            if (array[i] == target) return i;
        }
        return -1;
    }

    private int[] constructPostorder(int[] inorder, int[] postorder) {
        List<Integer> result = new ArrayList<>();
        
        for (int v : postorder) {
            if (Arrays.stream(inorder).anyMatch(i -> i == v)) {
                result.add(v);
            }
        }

        return result.stream().mapToInt(i->i).toArray();
    }

}
```



# LeetCode题目

- [x] [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

- [x] [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

- [x] [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

- [x] [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

- [x] [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

- [x] [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

- [x] [112. 路径总和](https://leetcode-cn.com/problems/path-sum/)

- [x] [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal)


# TODO

Morris遍历