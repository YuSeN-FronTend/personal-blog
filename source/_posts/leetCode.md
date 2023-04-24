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

## 二叉搜索树的第k大节点(中序遍历)

给定一棵二叉搜索树，请找出其中第 `k` 大的节点的值。

**示例**：

- 输入: `root = [3,1,4,null,2], k = 1`   输出: 4
- 输入: `root = [5,3,6,2,4,null,null,1], k = 3`   输出: 4

**思路**：

利用中序遍历处理二叉树，再反转数组，取得第k大的值

```js
var kthLargest = function(root, k) {
    let roots = []
    function middle(root) {
        if(root == null) return;
        middle(root.left);
        roots.push(root.val);
        middle(root.right)
    }
    middle(root);
    roots.reverse();
    return roots[--k];
};
```

## 二叉树的深度

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

**示例**：

- 给定二叉树 `[3,9,20,null,null,15,7]`，  返回它的最大深度 3 。

**思路**：

一个是层序遍历，遍历一层使结果数字加一，返回即可。一个是深度优先搜索，循环遍历直至不存在左子树或右子树返回即可

```js
// 层序遍历
var maxDepth = function(root) {
    let num = 0;
    let queue = [root]
    while(queue.length) {
        let resChild = [];
        for(let i = 0; i < queue.length; i++){
            if(queue[i].left) {
                resChild.push(queue[i].left);
            }
            if(queue[i].right) {
                resChild.push(queue[i].right)
            }
        }
        queue = resChild;
        num++;
    }
    return num;
};
```

```js
// DFS深度优先搜索
var maxDepth = function(root) {
    if(!root) return 0;
    return Math.max(maxDepth(root.left), maxDepth(root.right))+1
};
```

## 平衡二叉树

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

**示例**：

- 给定二叉树 `[3,9,20,null,null,15,7] `  返回 `true` 。
- 给定二叉树 `[1,2,2,3,3,null,null,4,4]`   返回 `false` 。

**思路**：

先用写一个dfs算法方法，然后遍历二叉树的左右节点并算出同一节点左右子树的深度差

```js
var isBalanced = function(root) {
    function max(root) {
        if(!root) return 0;
        return Math.max(max(root.left),max(root.right))+1;
    }
    if(!root) return true;
    return isBalanced(root.left) && isBalanced(root.right) && Math.abs(max(root.left) - max(root.right))<=1;
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

## 序列化二叉树

请实现两个函数，分别用来序列化和反序列化二叉树。

你需要设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

**示例**：

- 输入：root = [1,2,3,null,null,4,5]  输出：[1,2,3,null,null,4,5]

**思路**：

先把二叉树转化为字符串，再将字符串转换为二叉树

```js
var serialize = function(root) {
    if(!root) return [];
    let res = [];
    let queue = [root];
    while(queue.length) {
        let node = queue.shift();
        if(!node){
            res.push(node);
            continue;
        }
        res.push(node.val);
        queue.push(node.left);
        queue.push(node.right);
    }
    return res;
};

/**
 * Decodes your encoded data to tree.
 *
 * @param {string} data
 * @return {TreeNode}
 */
var deserialize = function(data) {
    if(!data || !data.length) {
        return null;
    }
    let root = new TreeNode(data[0]);
    let queue = [root];
    let i = 1;
    while(i < data.length) {
        let node = queue.shift();

        if(i < data.length) {
            if(data[i] !==null) {
                node.left = new TreeNode(data[i]);
                queue.push(node.left);
            }
            i++;
        }
        if(i < data.length) {
            if(data[i] !== null) {
                node.right = new TreeNode(data[i]);
                queue.push(node.right);
            }
            i++;
        }
    }
    return root;
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

# 队列

## 队列的最大值

请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的均摊时间复杂度都是O(1)。若队列为空，pop_front 和 max_value 需要返回 -1

**示例**：

- 输入: ["MaxQueue","push_back","push_back","max_value","pop_front","max_value"]  [[],[1],[2],[],[],[]]   输出: [null,null,null,2,1,2]
- 输入: ["MaxQueue","pop_front","max_value"]   [[],[],[]]   输出: [null,-1,-1]

**思路**：

队列先进先出，所以出队列要shift，定义stack数组当作队列，遍历数组取得最大值返回，并且和弹队列都要识别是否队列为空返回-1的情况

```js
var MaxQueue = function() {
    this.stack = [];
};
MaxQueue.prototype.max_value = function() {
    let sum = 0;
     if(!this.stack.length) {
         return -1;
     } else {
         this.stack.forEach((item) => {
             sum = sum >= item ? sum : item;
         })
     }
     return sum;
};
MaxQueue.prototype.push_back = function(value) {
    this.stack.push(value)
};
MaxQueue.prototype.pop_front = function() {
    let num = 0;
    if(!this.stack.length) {
        return -1;
    } else {
        num = this.stack.shift();
    }
    return num;
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

## 两个链表的第一个公共节点

输入两个链表，找出它们的第一个公共节点。

**示例**：

- 输入：`intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3`   输出：`Reference of the node with value = 8`
- 输入：`intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1 `  输出：`Reference of the node with value = 2`
- 输入：`intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2`   输出：`null`

**思路**：

先计算两数组长度，算出差值让长的链表先移动，直至相同后再一起移动寻找相交链表

```js
var getIntersectionNode = function(headA, headB) {
    let lenA = 0;
    let lenB = 0;
    let crrA = headA;
    let crrB = headB;
    let nextA;
    let nextB;

    while(crrA) {
        lenA++;
        nextA = crrA;
        crrA = crrA.next;
    }

    while(crrB) {
        lenB++;
        nextB = crrB;
        crrB = crrB.next;
    }
    if(nextA !== nextB) {
        return null;
    }
    let len = lenA - lenB;

    for(let i = 0; i < Math.abs(len); i++) {
        if(len > 0){
            headA = headA.next;
        }

        if(len < 0) {
            headB = headB.next;
        }
    }

    while(headA && headB) {
        if(headA === headB) {
            return headA
        }
        headA = headA.next;
        headB = headB.next;
    }
    return null
};
```



# 回溯

## 字符串的排列

输入一个字符串，打印出该字符串中字符的所有排列。你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

**示例**：

- 输入：`s = "abc"`  输出：`["abc","acb","bac","bca","cab","cba"]`

**思路**：

利用回溯法逐层处理给定的字符串递归排列，因为case中有重复字符，需要用set去重

```js
var permutation = function (s) {
    let res = new Set();
    let path = [];
    let visited = [];
    dfsHelper([...s], path, res, visited);
    return Array.from(res)
};

function dfsHelper(arr, path, res, visited) {
    if(arr.length === path.length) {
        res.add(path.join(''));
        return;
    }
    for(let i = 0; i < arr.length; i++) {
        if(visited[i]) {
            continue;
        }
        visited[i] = true;
        path.push(arr[i]);
        dfsHelper(arr, path, res, visited);
        path.pop();
        visited[i] = false;
    }
}
```

## 机器人的运动范围

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

**示例**：

- 输入：m = 2, n = 3, k = 1   输出：3
- 输入：m = 3, n = 1, k = 0   输出：1

**思路**：

创建一个数组visited来存放方格中的点是否已经遍历过，如果遍历过则不重复遍历。思想为顺着一行或一列一直走，如果遇到边界，则停止并回退，直到完全遍历完

```js
var movingCount = function (m, n, k) {
    function dfs(i, j, m, n, k, visited) {
        // 来判断是否移动到方格外或者已经走过当前点
        if(i >= m || i < 0 || j >= n || j < 0 || visited[i][j]){
            return;
        }
        // 没走过就给当前点标记为走过
        visited[i][j] = true;
        // 计算当前点行坐标和列坐标数位之和是否小于k
        let sum = i % 10 + j % 10 + Math.floor(i / 10) + Math.floor(j / 10);
        if(sum > k) {
            return;
        }
        // 满足上述要求 走过的格数加一
        res++;
        // 行循环和列循环齐头并进
        dfs(i + 1, j, m, n, k, visited);
        dfs(i, j + 1, m, n, k, visited);
    }

    let res = 0;
    // 创建标记是否走过的数组
    let visited = new Array(m).fill(0).map(() => new Array(n));

    dfs(0, 0, m, n, k, visited);
    return res;
};
```

# 动态规划

## 连续子数组的最大和

输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。

**示例**：

- 输入: `nums = [-2,1,-3,4,-1,2,1,-5,4]`   输出: 6

**思路**：

让数组的每一项依次增加，如果被加数小于零，则加0以此类推，循环一次数组，即可得到答案

```js
var maxSubArray = function(nums) {
    let max = nums[0];
    for(let i = 1; i < nums.length; i++) {
        nums[i] += Math.max(nums[i - 1], 0);
        max = Math.max(nums[i], max);
    }
    return max;
};
```

## 把数字翻译成字符串

给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

**示例**：

- 输入: 12258   输出: 5

**思路**：

创建dp数组，利用动态规划思想进行叠加

```js
var translateNum = function(num) {
    if(num < 10) {
        return 1;
    }
    let str = num.toString();
    let dp = [1,1];
    for(let i = 1; i < str.length; i++) {
        let tmp = parseInt(str.slice(i-1, i+1)) || 0;
        if(tmp >= 10 && tmp <= 25){
            dp[i+1] = dp[i-1] + dp[i];
        } else {
            dp[i+1] = dp[i];
        }
    }
    return dp[dp.length-1]
};
```

## 礼物的最大价值

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

**示例**：

- 输入: [[1,3,1],[1,5,1],[4,2,1]]   输出: 12

**思路**：

循环并比较当前基点的相邻的两个数大小，并且依次累加，直至循环完成

```js
var maxValue = function (grid) {
    if(grid.length ===0 && grid[0].length===0) return 0;
    let rowLimit = grid.length;
    let colLimit = grid[0].length;
    for(let row = 0; row < rowLimit; row++) {
        for(let col = 0; col < colLimit; col++) {
            let left = col - 1 < 0 ? 0 : grid[row][col - 1];
            let top = row - 1 < 0 ? 0 : grid[row - 1][col];

            grid[row][col] += Math.max(left, top);
        }
    }
    return grid[rowLimit - 1][colLimit - 1];
};
```

## 最长不含重复字符的子字符串

请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

**示例**：

- 输入: `"abcabcbb"`   输出: 3
- 输入: `"bbbbb"`   输出: 1
- 输入: `"pwwkew"`   输出: 3

**思路**：

利用双指针思想，判断当前字符串是否在截取字符串中，如果存在，就获取截取字符串中当前字符串的索引，加上原索引再加1，二指针也要后移。如果

不存在，则当前字符串和原来最长字符串最比较，返回最长的长度。

```js
var lengthOfLongestSubstring = function(s) {
    if(!s.length) return 0;
    let i = 0;
    let j = 1;
    let res = 1;
    while(j < s.length) {
        if(s.slice(i,j).includes(s[j])){
            i += s.slice(i,j).indexOf(s[j]) + 1
            j++;
        } else {
            j++;
            res = Math.max(s.slice(i,j).length, res);
        }
    }
    console.log(i,j)
    return res;
};
```

## 丑数

我们把只包含质因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

**示例**：

- 输入: n = 10   输出: 12

**思路**：

创建一个数组里面只有1，在创建三个指针，我们把此题想象成矩阵，第一行2、3、5，第二行4、6、10，以此类推，这些数都是丑数，我们只需让三个索引代表第一行的分别三列，哪列值最小，就进入到下一行，即可获取答案。

```js
var nthUglyNumber = function(n) {
    let dp = [1];
    let a = 0;
    let b = 0;
    let c = 0;
    for(let i = 1; i < n; i++) {
        let am = dp[a] * 2;
        let bm = dp[b] * 3;
        let cm = dp[c] * 5;
        dp[i] = Math.min(am,bm,cm);
        if(dp[i] === am) a++;
        if(dp[i] === bm) b++;
        if(dp[i] === cm) c++;
    }
    return dp[n-1]
};
```

## n个骰子的点数

把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

**示例**： 

- 输入: 1  输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
- 输入: 2  输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]

**思路**：

创建一个二维数组取得到值，先初始化数组，再遍历得到每个骰子摇出对应和的次数，最后算出比例返回结果

```js
var dicesProbability = function(n) {
    let dp = new Array(n+1).fill().map(() => new Array(n*6 + 1).fill(0));
    let result = [];
    for(let i = 1; i <= 6; i++) {
        dp[1][i] = 1;
    }
    for(let i = 2; i <= n; i++){
        for(let j = i; j <= 6*i; j++) {
            for(let cur = 1; cur <= 6; cur++){
                if(j <= cur) break;
                dp[i][j] += dp[i-1][j-cur]
            }
        }
    }
    let all = Math.pow(6,n);
    for(let i = n; i <= n*6; i++) {
        result.push(parseFloat((dp[n][i] / all).toFixed(5)))
    }
    return result;
};
```



# 哈希表

## 第一个只出现一次的字符

在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

**示例**：

- 输入：s = `"abaccdeff"`    输出：'b'
- 输入：s = ""   输出：' '

**思路**：

利用哈希表存储思路，把出现过的字符都存为对象中的key，再次出现就使value加一，最后判断第一个只出现一次的字符。

```js
var firstUniqChar = function(s) {
    if(!s.length) return " ";
    let obj = {}
    for(let i = 0; i < s.length; i++) {
        if(!obj[s[i]]){
            obj[s[i]] = 1;
        } else {
            obj[s[i]]++;
        }
    }

    for(let key in obj){
        if(obj[key] === 1) {
            return key;
        }
    }
    return " ";
};
```

## 在排序数组中查找数字

统计一个数字在排序数组中出现的次数。

**示例**：

- 输入: `nums = [5,7,7,8,8,10], target = 8`   输出: 2
- 输入: `nums = [5,7,7,8,8,10], target = 6`   输出: 0

**思路**：

利用哈希表存储，把每个出现的数字存储到对象中并且添加次数

```js
var search = function(nums, target) {
    let obj = {};
    for(let i = 0; i < nums.length; i++) {
        if(!obj[nums[i]]){
            obj[nums[i]] = 1
        } else {
            obj[nums[i]]++;
        }
    }
 
   for(let key in obj){
       if(key == target) {
           return obj[key]
       }
   }
   return 0;
};
```



# 归并排序

## 数组中的逆序对

在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

**示例**：

- 输入: [7,5,6,4]   输出: 5

**思路**：

将数组对半拆分，递归拆分成两个长度为1的数组，如果前数组大于后数组，则使结果长度加1并且左侧数组长度加一并添加到新数组，如果不大于则添加到新数组里面，如果左右数组指针大于等于数组长度，则直接使另一侧添加到新数组并且加1，往复循环直至递归完成

```js
var reversePairs = function(nums) {
    let sum = 0;
    mergeSort(nums);
    return sum;

    function mergeSort(nums) {
        if(nums.length < 2) return nums;
        let mid = parseInt(nums.length/2);
        let leftArr = nums.slice(0, mid);
        let rightArr = nums.slice(mid);
        return merge(mergeSort(leftArr), mergeSort(rightArr));
    }

    function merge(left, right) {
        let res = [];
        let leftLen = left.length;
        let rightLen = right.length;
        let len = leftLen + rightLen;

        for(let index = 0, i = 0, j = 0; index < len; index++) {
            if(i >= leftLen) {
                res[index] = right[j++];
            } else if(j >= rightLen) {
                res[index] = left[i++];
            } else if(left[i] <= right[j]) {
                res[index] = left[i++]
            } else {
                res[index] = right[j++];
                sum += leftLen - i;
            }
        } 
        return res;
    }
};
```

## 0~n-1中缺失的数字

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

**示例**：

- 输入: [0,1,3]   输出: 2
- 输入: [0,1,2,3,4,5,6,7,9]   输出: 8

**思路**：

定义头尾指针，检测前半段数组和后半段数组那个符合要求，不符合的就直接移动指针砍掉，循环至找到答案

```js
var missingNumber = function(nums) {
    if(nums[nums.length - 1] === nums.length-1) {
        return nums.length;
    }

    let start = 0, end = nums.length -1, mid;
    while(start <= end) {
        mid = start + Math.floor((end - start)/2);
        if(nums[mid] === mid) {
            start = mid + 1
        } else {
            end = mid - 1;
        }
    }
    return start;
};
```



# 其他

## 数组中出现次数超过一半的数字

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。你可以假设数组是非空的，并且给定的数组总是存在多数元素。

**示例**：

- 输入: `[1, 2, 3, 2, 2, 2, 5, 4, 2]`   输出：2

**思路**：

我想到的思路有两个，一个是遍历数组，将每一项出现的次数包装成一个对象，最后再遍历对象取到相应的值。另一个就是因为目标数字出现的数字肯定大于数组长度的一般，那么排序数组之后的`nums[nums.length - 1]`项就是目标数字

```js
var majorityElement = function(nums) {
    let obj = {};
    let len = (nums.length) / 2;
    for(let num of nums) {
        obj[num] = obj[num] ? obj[num] + 1 : 1;
    }
    for(let key in obj) {
        if(obj[key] > len) {
           return key;
        }
    }
};
```

```js
var majorityElement = function(nums) {
    return nums.sort((a,b) => a - b)[parseInt((nums.length -1)/2)];
};
```

## 最小的k个数

输入整数数组 `arr` ，找出其中最小的 `k` 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。

**示例**：

- 输入：arr = [3,2,1], k = 2   输出：[1,2] 或者 [2,1]
- 输入：arr = [0,1,2,1], k = 1   输出：[0]

**思路**：

直接使用js的sort排序，然后截取前k项即可 

```js
var getLeastNumbers = function(arr, k) {
    arr.sort((a,b) => a-b);
    return arr.slice(0,k);
};
```

## 数据流中的中位数

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

**示例**：

- 输入：`["MedianFinder","addNum","addNum","findMedian","addNum","findMedian"]  [[],[1],[2],[],[3],[]]`

  输出：`[null,null,null,1.50000,null,2.00000]`

- 输入：`["MedianFinder","addNum","findMedian","addNum","findMedian"]   [[],[2],[],[3],[]]`

  输出：`[null,null,2.00000,null,2.50000]`

**思路**：

定义一个数组来存放结果，需要给此数组进行排序，并且判断数组长度在奇偶不同的情况下的中位数求解方法

```js
var MedianFinder = function() {
    this.math = []
};

MedianFinder.prototype.addNum = function(num) {
    if(this.math.length === 0) {
        this.math.push(num);
    } else {
        let i = this.math.length - 1;
        while(i >= 0 && num < this.math[i]) {
            i--;
        }
        this.math.splice(i + 1, 0, num)
    }
};

MedianFinder.prototype.findMedian = function() {
    let len = this.math.length;
    if(this.math.length %2 === 0) {
        return (this.math[len/2] + this.math[(len/2) - 1]) /2
    } else {
        return this.math[(len - 1) /2]
    }
};
```

## 数字序列中某一位数字

数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4，等等。请写一个函数，求任意第n位对应的数字。

**示例**：

- 输入：n = 3   输出：3
- 输入：n = 11   输出：0

**思路**：先算出第n位为几位数，再算出目标数字，最后通过取余求出目标单个数字对应目标数字的第几位

```js
var findNthDigit = function(n) {
    if(n < 10 && n >= 0) return n;
    let start = 9;
    let i = 1;
    while(start < n){
        i++;
        start += Math.pow(10, i - 1) * 9 * i;
    }

    let diff_n = Math.floor((start - n) / i);
    let diff_y = (start - n) % i;
    return `${Math.pow(10, i) - 1 - diff_n}`.charAt(i - 1 -diff_y);
};
```

## 把数组排成最小的数

输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

**示例**：

- 输入: [10,2]   输出: "102"
- 输入: [3,30,34,5,9]   输出: "3033459"

**思路**：

将数组排序，拼接两个数字去比较他们组合的大小直至比较完成

```js
var minNumber = function(nums) {
    return nums.sort((a,b) => {
        if(`${a}${b}` - `${b}${a}` > 0){
            return 1;
        } else {
            return -1;
        }
    }).join('')
};
```

## 数组中数字出现的次数

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

**示例**：

- 输入：`nums = [4,1,4,6]`   输出：`[1,6] 或 [6,1]`
- 输入：`nums = [1,2,10,4,1,4,3,3]`   输出：`[2,10] 或 [10,2]`

**思路**：

本题有两种思路，第一种是排序后将数字两两比较，最后得出答案。第二种是位运算，利用异或来解决问题。

```js
//排序
var singleNumbers = function(nums) {
    let result = [];
    nums.sort((a,b) => {
        return a-b;
    })
    for(let i = 0; i < nums.length; i++) {
        if(nums[i] === nums[i+1]) {
            i++;
        } else {
            result.push(nums[i]);
            if(result.length === 2){
                return result;
            }
        }
    }
};
```

```js
// 位运算
var singleNumbers = function(nums) {
    let res = 0;
    for(let n of nums) {
        res ^= n;
    }
    let div = 1;
    while((div&res) === 0){
        div <<= 1
    }
    let a = 0, b = 0;
    for(let n of nums) {
        if(div & n) {
            a ^= n;
        } else {
            b ^= n;
        }
    }
    return [a,b];
};
```

## 数组中数字出现的次数Ⅱ

在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

**示例**：

- 输入：`nums = [3,4,3,3]`   输出：4
- 输入：`nums = [9,1,7,9,7,9,7]`   输出：1

**思路**：

遍历32位数，与原数组元素作比较。

```js
var singleNumber = function(nums) {
    let res = 0;
    for(let i = 0; i < 32; i++) {
        let bit = 1 << i;
        let cut = 0;
        for(let j = 0; j < nums.length; j++) {
            if(bit & nums[j]) {
                cut++
            }
        }
        if(cut % 3 !== 0){
            res = res | bit;
        }
    }
    return res;
};
```

## 和为s的两个数字

输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

**示例**：

- 输入：`nums = [2,7,11,15], target = 9`   输出：`[2,7] 或者 [7,2]`

- 输入：`nums = [10,26,30,31,47,60], target = 40`   输出：`[10,30] 或者 [30,10]`

**思路**：

定义两个指针，头尾相加和目标值比较移动指针即可。

```js
var twoSum = function(nums, target) {
    if(nums.length !== 2) {
        let n = 0;
        let p = nums.length - 1;
        while(n < p){
            if(nums[n] + nums[p] > target){
                p--;
            } else if(nums[n] + nums[p] < target){
                n++;
            } else {
                return [nums[n], nums[p]];
            }
        }
    } else {
        return nums;
    }
};
```

## 和为s的连续正数序列

输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

**示例**：

- 输入：target = 9   输出：[[2,3,4],[4,5]]
- 输入：target = 15   输出：[[1,2,3,4,5],[4,5,6],[7,8]]

**思路**：

循环目标数字的一半加一次数，声明一个窗口数组用来保存结果，最后用到深拷贝完成添加进结果数组操作

```js
var findContinuousSequence = function(target) {
    let sum = 1;
    let list = [1];
    let result = [];
    for(let i = 2; i <= Math.ceil(target/2); i++) {
        sum += i;
        list.push(i);
        while(sum > target) {
            sum -= list.shift();
        }
        if(target === sum) {
            result.push(list.slice(0))
        }
    }
    return result
};
```

## 翻转单词顺序

输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。

**示例**：

- 输入: "the sky is blue"   输出: "blue is sky the"
- 输入: "  hello world!  "   输出: "world! hello"
- 输入: "a good   example"   输出: "example good a"

**思路**：

按照js语言特性，先用replace方法将单词间多余的空格都变为一个空格，然后用trim方法清除首尾空格，再使用split方法将字符串转换为数组，使用数组的reverse方法反转数组，最后再用join方法拼接成字符串

```js
var reverseWords = function(s) {
    return s.replace(/\s+/g, ' ').trim().split(' ').reverse().join(' ')
};
```

## 左旋转字符串

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

**示例**：

- 输入: `s = "abcdefg", k = 2`   输出: `"cdefgab"`
- 输入: `s = "lrloseumgh", k = 6`   输出: `"umghlrlose"`

**思路**：

截取字符串而后拼接

```js
var reverseLeftWords = function(s, n) {
    return s.slice(n) + s.slice(0,n); 
};
```

## 滑动窗口最大值

给定一个数组 `nums` 和滑动窗口的大小 `k`，请找出所有滑动窗口里的最大值。

**示例**：

- 输入: `nums = [1,3,-1,-3,5,3,6,7], 和 k = 3`   输出: `[3,3,5,5,6,7]` 

**思路**：

使用queue存放队列，向里面添加nums的索引，用两个while分别做超出长度的判断和最大值判断，最后添加至数组中

```js
var maxSlidingWindow = function (nums, k) {
    let res = [];
    let queue = [];

    for(let i = 0; i < nums.length; i++) {
        // 超出长度则弹出元素
        while(queue.length && queue[0] <= i - k) queue.shift();
        // 进来的元素 >= 队尾元素，就要弹出，因为它们永远不是答案
        while(queue.length && nums[queue[queue.length - 1]] <= nums[i]) queue.pop()
        
        queue.push(i);
        if(i >= k - 1) {
            res.push(nums[queue[0]]);
        }
    }
    return res;
};
```

## 扑克牌中的顺子

从若干副扑克牌中随机抽 5 张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

**示例**：

- 输入: [1,2,3,4,5]   输出: true
- 输入: [0,0,1,2,5]   输出: true

**思路**：

先判断数组中是否存在0，不存在则判断数组中前后项插值是否为1，存在零就剔除0元素，给剩余数组排序，判断数组中那两项差不为1向当前位置插入值，最后再判断前后项插值是否为1

```js
var isStraight = function(nums) {
    let p = 0;
    function correctfun(nums) {
        for(let i = 1; i < nums.length; i++) {
            if(nums[i] !== nums[i-1] + 1){
                return false;
            }
        }
        return true;
    }
    if(!nums.includes(0)){
        return correctfun(nums)
    } else {
        while(nums.includes(0)){
            nums.splice(nums.indexOf(0), 1);
        }
        nums.sort((a,b) => a-b)
        while(nums.length < 5) {
            if(nums[p] + 1 !== nums[p+1]){
                nums.splice(p+1,0,nums[p] + 1)
            }
            p++;
        }
        return correctfun(nums);
    }
};
```

## 圆圈中最后剩下的数字

0,1,···,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字（删除后从下一个数字开始计数）。求出这个圆圈里剩下的最后一个数字。

**示例**：

- 输入: n = 5, m = 3  输出: 3
- 输入: n = 10, m = 17  输出: 2

**思路**：

按照数学公式和迭代法完成

```js
var lastRemaining = function(n, m) {
    let f = 0;
    for(let i = 2; i != n+1;i++) {
        f = (m+f)%i;
    }
    return f;
};
```

