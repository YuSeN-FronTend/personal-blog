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
