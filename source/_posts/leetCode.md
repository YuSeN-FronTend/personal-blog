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

## 从上到下打印二叉树(层序遍历)

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

**示例**：

- 给定二叉树: `[3,9,20,null,null,15,7]`   返回：`[3,9,20,15,7]`

**思路**：

定义一个结果空数组，再用一个queue数组将二叉树包裹起来，用queue的长度进行循环，弹出二叉树，将第一项添加到结果数组中，开始判断二叉树是否存在左子树和右子树，如果存在则添加到queue，让下一次循环时遍历下一层存在的子树，以此类推直至循环结束

```js
var levelOrder = function(root) {
    let res = [];
    if(!root) return res;

    let queue = [root];

    while(queue.length) {
        let node = queue.shift();
        res.push(node.val);
        if(node.left) {
            queue.push(node.left)
        }
        if(node.right) {
            queue.push(node.right)
        }
    }
    return res;
};
```

## 从上到下打印二叉树Ⅱ(层序遍历扩展)

从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

**示例**：

- 给定二叉树: `[3,9,20,null,null,15,7]`,  返回：`[ [3], [9,20], [15,7] ]`

**思路**：

也是层序遍历的思路，只不过需要新建两个数组，一个存放每一层的值，方便push到结果数组，一个存放每一层的值方便以后遍历

```js
var levelOrder = function(root) {
    let res = [];
    if(!root) return res;
    let queue = [root];
    while(queue.length) {
        let resChild = [];
        let level = [];
        for(let i = 0; i < queue.length; i++) {
            level.push(queue[i].val);
            if(queue[i].left) {
                resChild.push(queue[i].left);
            }
            if(queue[i].right) {
                resChild.push(queue[i].right);
            }
        }
        queue = resChild;
        res.push(level);
    }
    return res;
};
```

## 从上到下打印二叉树Ⅲ(层序遍历扩展)

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

**示例**：

- 给定二叉树: `[3,9,20,null,null,15,7]`,  返回：`[ [3], [9,20], [15,7] ]`

**思路**：

和上一个类似，做一个奇偶数识别，然后确定是unshift还是push

```js
var levelOrder = function(root) {
    let res = [];
    if(!root) return res;
    let queue = [root];
    let p = 1;
    while(queue.length) {
        let queueChild = [];
        let level = [];
        for(let i = 0; i < queue.length; i++) {
            if(p % 2 !== 0) {
                level.push(queue[i].val);
            } else {
                level.unshift(queue[i].val);
            }
            if(queue[i].left){
                queueChild.push(queue[i].left);
            }
            if(queue[i].right) {
                queueChild.push(queue[i].right)
            }
        }
        p++;
        queue = queueChild;
        res.push(level)
    }
    return res;
};
```

## 二叉搜索树的后序遍历序列

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 `true`，否则返回 `false`。假设输入的数组的任意两个数字都互不相同。

**示例**：

- 输入: [1,6,3,2,5]  输出: false
- 输入: [1,3,2,6,5]  输出: true

**思路**：

后序遍历的顺序是，左子树 --> 右子树 --> 根。二叉搜索树是，左子树节点值 < 根节点值 < 右子树节点值。所以只需要分离出左子树和右子树，然后递归遍历，不满足条件直接返回false即可。

```js
var verifyPostorder = function(postorder) {
    if(postorder.length <= 2) return true;
    let root = postorder.pop();
    let i = 0;
    while(postorder[i] < root) {
        i++;
    }

    let rightTree = postorder.slice(i);

    let resultArr = rightTree.every((item) => item > root);

    return resultArr && verifyPostorder(postorder.slice(0, i)) && verifyPostorder(rightTree)
};
```

## 二叉树中和为某一值的路径

给你二叉树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。叶子节点 是指没有子节点的节点。

**示例**：

- 输入：root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22  输出：[[5,4,11,2],[5,8,4,5]]
- 输入：root = [1,2,3], targetSum = 5 输出：[]
- 输入：root = [1,2], targetSum = 0 输出：[]

**思路**：

通过递归依此遍历出每条路径，然后去比较每条路径上的值相加之和和目标值是否相等

```js
var pathSum = function(root, target) {
    if(!root) return [];
    let res = [];
    let path = [root.val];
    let handle = (node) => {
        if(node.left) {
            path.push(node.left.val);
            handle(node.left);
            path.pop();
        }
        if(node.right) {
            path.push(node.right.val);
            handle(node.right);
            path.pop();
        }
        if(!node.left && !node.right) {
            let add = 0;
            for(let i = 0; i < path.length; i++) {
                add += path[i];
            }
            if(add === target) {
                res.push(path.slice());
            }
        }
    }
    handle(root);
    return res;
};
```



# 顺时针打印矩阵

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

# 栈

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

## 栈的压入、弹出序列

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

**示例**：

- 输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]  输出：true
- 输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]  输出：false

**思路**：

创建一个栈，将pushed的数字依此推入栈中，用栈中的尾项和popped数组中的首项作比较，如若相同，栈中弹出一项，popped的比较项后移，直至循环结束，去检查栈中的长度是否为0

```js
var validateStackSequences = function(pushed, popped) {
    let stack = [];
    let j = 0;
    for(let i = 0; i < pushed.length; i++) {
        stack.push(pushed[i]);
        while(stack.length && stack[stack.length - 1] === popped[j]) {
            stack.pop();
            j++;
        }
    }
    return !stack.length;
};
```

# 链表

## 复杂链表的复制

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

**示例**：

- 输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]  输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
- 输入：head = [[1,1],[2,1]]  输出：[[1,1],[2,1]]
- 输入：head = [[3,null],[3,0],[3,null]]   输出：[[3,null],[3,0],[3,null]]

**思路**：

新的链表节点用map存起来，通过引用地址找到random的节点

```js
var copyRandomList = function(head) {
    if(!head) return null;
    let cur = head;
    let preHead = new Node();
    let temp = preHead;
    let map = new Map();
    while(cur){
        temp.val = cur.val;
        temp.next = cur.next ? new Node() : null;
        map.set(cur, temp);
        temp = temp.next;
        cur = cur.next;
    }
    temp = preHead;
    while(head) {
        temp.random = head.random ? map.get(head.random) : null;
        temp = temp.next;
        head = head.next;
    }
    return preHead;
};
```

## 二叉搜索树与双向链表

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

**示例**：

- 输入：[4,2,5,1,3]   输出：[1,2,3,4,5]

**思路**：

使用中序遍历思路，根据左中右顺序递归，最后在结果出来之后，让头节点和尾节点相互指向

```js
var treeToDoublyList = function(root) {
    if(!root) return null;
    let prev = null;
    let head = null;

    let dfsHelper = function(root) {
        if(!root) return null;

        dfsHelper(root.left);

        if(!head) {
            head = new Node(root.val);
            prev = head;
        } else {
            let newHead = new Node(root.val);
            prev.right = newHead;
            newHead.left = prev;
            prev = newHead;
        }
        dfsHelper(root.right);
    }
    dfsHelper(root);
    head.left = prev;
    prev.right = head;
    return head;
};
```
