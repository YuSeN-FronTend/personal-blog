---
title: 算法刷题记录
date: 2023-4-12 13:33
categories: 面试
---

# 二叉树

## 树的子结构

输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)，B是A的子结构， 即 A中有出现和B相同的结构和节点值。

**示例**：

- 输入：A = [1,2,3], B = [3,1]  输出：false
- 输入：A = [3,4,5,1,2], B = [4,1]  输出：true

**思路**：

本题使用递归思想，比较两参数根节点是否一样，一样则进入新的递归函数，逐次比较。如果根节点不一样，则比较A的左子树与B是否一致。如果再不一样则比较A的右子树与B是否一直，直至结束。

```js
var isSubStructure = function(A, B) {
    let result = false;
    
    if(A && B) {
        if(A.val === B.val) result = handleTree(A,B);
        if(!result) result = isSubStructure(A.left, B)
        if(!result) result = isSubStructure(A.right, B)
    }

    return result;
};

const handleTree = function(R, B) {
    if(!B) return true;
    if(!R) return false;

    if(R.val !== B.val) return false;

    return handleTree(R.left, B.left) && handleTree(R.right, B.right)
}
```

## 二叉树镜像

请完成一个函数，输入一个二叉树，该函数输出它的镜像。(交换左右子树)

**示例**：

- 输入：root = [4,2,7,1,3,6,9]  输出：[4,7,2,9,6,3,1]

**思路**：

运用递归思想交换左右子树

```js
var mirrorTree = function(root) {
    if(!root) return root;
    if(root.left && !root.right) root.right = null;
    if(!root.left && root.right) root.left = null;
    if((root.left || root.left === null) && (root.right || root.right === null)) {
        let left = root.left;
        let right = root.right;
        root.left = right;
        root.right = left;
        mirrorTree(root.left);
        mirrorTree(root.right)
    }
    return root
};
```

## 对称二叉树

请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

**示例**：

- 输入：root = [1,2,2,3,4,4,3]  输出：true
- 输入：root = [1,2,2,null,3,null,3]  输出：false

**思路**：

递归遍历左子树的最左边和右子树的最右边，判断有无返回值。

- 无返回值即代表全部相等递归结束，即可返回true
- 如果有其中一个节点没有返回值，即为不对称，返回false
- 如果进来的两个节点不相等即为不对称，返回false

直至递归完成

```js
var isSymmetric = function(root) {
    return copyTree(root,root)
}

const copyTree = function(r1,r2) {
    if(!r1 && !r2) return true;
    if(!r1 || !r2) return false;
    if(r1.val !== r2.val) return false;
    return copyTree(r1.left, r2.right) && copyTree(r1.right, r2.left);
}
```

## 顺时针打印矩阵

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

**示例**：

- 输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]  输出：[1,2,3,6,9,8,7,4,5]
- 输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]  输出：[1,2,3,4,8,12,11,10,9,5,6,7]

**思路**：

从矩阵第一行循环推入数组，然后最右侧一列，之后最下面一行，在之后最左边一行。完成之后设置一个变量circle让他自增一，表示已经进入到下一个圈层，如此往复循环直到最终数组与原数组摊平后长度相等

```js
var spiralOrder = function(matrix) {
    let h = matrix.length;
    let w = h > 0 ? matrix[0].length : 0;
    // 结果数组
    let res = [];
    // 列指针
    let i = 0;
    // 行指针
    let j = 0;
    // 判别是第几层
    let circle = 0;
    // 最终数组长度
    let l = w * h;
    while(res.length < l) {
        // 首先循环矩阵的第一行
        while(i < w - circle && res.length < l){
            res.push(matrix[j][i]);
            i++;
        }
        // i超出一个需要减去
        i--;
        // 向下走一行
        j++;
        // 循环右侧竖着的列
        while(j < h - circle && res.length < l) {
            res.push(matrix[j][i]);
            j++;
        }
        // 行超出需要减小
        j--;
        // 向左走一列
        i--;
        while(i >= circle && res.length < l) {
            res.push(matrix[j][i]);
            i--
        };
        // 列少一个需要增加
        i++;
        // 向上走一行
        j--;
        while(j > circle && res.length < l) {
            res.push(matrix[j][i]);
            j--;
        }
        // 循环结束第一层，需要向下移一行
        j++;
        // 向右移一列当作初始值
        i++;
        //意为到第二圈
        circle++;
    }
    return res;
};
```

## 包含min函数的栈

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

**示例**：

- `MinStack minStack = new MinStack();`
  `minStack.push(-2);`
  `minStack.push(0);`
  `minStack.push(-3);`
  `minStack.min();   --> 返回 -3.`
  `minStack.pop();`
  `minStack.top();      --> 返回 0.`
  `minStack.min();   --> 返回 -2.`

**思路**：

建立两个栈，一个正常栈，一个最小值栈，每次进来新的数据要和栈顶的指作比较，最小的放在栈顶

```js
var MinStack = function() {
    this._stack = [];
    this._minStack = [];
};
MinStack.prototype.push = function(x) {
    this._stack.push(x);
    if(this._minStack.length === 0) {
        this._minStack.push(x);
    } else {
        let min = this.min();
        this._minStack.push(Math.min(min, x));
    }
};
MinStack.prototype.pop = function() {
    this._stack.pop();
    this._minStack.pop();
};
MinStack.prototype.top = function() {
    return this._stack.length > 0 ? this._stack[this._stack.length-1] : -1;
};
MinStack.prototype.min = function() {
    return this._minStack.length > 0 ? this._minStack[this._minStack.length-1] : -1;
};
```

