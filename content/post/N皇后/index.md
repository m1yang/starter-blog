---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "N皇后"
subtitle: ""
summary: ""
authors: [miyang]
tags: [算法]
categories: [工作]
date: 2020-10-16T06:25:40+08:00
lastmod: 2020-10-16T06:25:40+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。

首先考虑，用一个什么数据结构来解决问题，棋盘通常是二维数组，用`a[m][n]` 代表行列来表示皇后所在位置。
但是N皇后不能互相攻击使一行只有一个皇后，所以将该行皇后所在那一列作为值存储在一维数组下标中，用下标表示行，值表示列。
例如，二维皇后位置`a[2][3]` ，一维可以表示为`a[2] = 3` 

再来考虑，用什么算法思想来解决这个问题，如果没有看过参考答案的话，基本上考自己小脑袋瓜是很难凭空想像一个'回溯算法'来作为解决方案。
所以这时候来看，这个问题能不能拆解，有点废话的说，n x n 的棋盘看作是每一行放置一个皇后，将关注点放在某一行皇后该怎么放。

皇后的攻击线路是一个'米'字，除了同一行、同一列只能有一个皇后外，左右对角线也只能是这一个皇后。
简单粗暴一点，就是从该皇后落子的位置起，向上判断每一行是否会被攻击(往下的每一行还未落子，所以不需要判断)。假如落子的位置是`a[2]=3` ，那么前两行的值都不能为3。再获取对角线，左对角线每向上一行，列数减1；右对角线每向上一行，列数加1。判断a[前两行皇后位置]!=对角线所在列。例如，左对角线`a[1]=2` ，那么这一行皇后落子的位置就不能在第2列。
写成代码就是

```javascript
let isValid = (row, column) => {
	// 左右对角线
	let [leftup, rightup] = [column - 1, column + 1]
    // 从当前行开始向上判断攻击
    for (var i = row - 1; i >= 0; i -= 1) {
        // 前i行，列数相等，被攻击
        if (board[i] === column) return false;
        // 前i行，左对角线有皇后攻击
        if (leftup >= 0) {
            if (board[i] === leftup) return false;
        }
        // 前i行，右对角线有皇后攻击
        if (rightup < n) {
            if (board[i] === rightup) return false;
        }
        // 延申对角线
        leftup -= 1;
        rightup += 1;
    }
    return true
};
```

知道某一行皇后是否可以落子了，那么遍历第一行到第n行就可以得到我们想要的答案了。于是嵌套两个for循环嘛，但有一个最明显的问题，for循环会直接执行到底，即使前两步下错了，也不能回退重新下，所以最终大概率是得到一个空答案。
for循环遍历不能回退验证所有结果，那什么类型的函数是可以返回到上一个状态的呢？递归！
从前面可以知道，此时已经满足了递归的3要素

1. 可以分解为子问题（从第1行开始到最后一行）
2. 子问题解题思路一致（均为判断是否会被攻击）
3. 存在终止条件（最后一行落子）

写成代码就是

```javascript
let backTrack = (n, row) => {
    // 终止条件
    if (row === n) {
        // TODO:将board数组转换为棋盘
        return result.push([...board]);
    }
    for (let column = 0; column < n; column++) {
        //同一解题思路
        if (isValid(row, column)) {
            board[row] = column;
            // 分解为子问题
            backTrack(n, row + 1);
        }
    }
};
```

通常来说在回溯时，也就是在递归代码`backTrack(n, row + 1) ` 之后会撤销之前做的选择，但是这里因为是从当前行往上判断的，后面的旧数据并不影响判断。

这里通过观察归纳，还能从对角线上得出一个结论，即 对皇后所在对角线有 `行号 +-列号 = 常量` ，且同一对角线常量相等。通过这一结论，可以将解题思路优化

```javascript
let isValid = (row, column) => {
    // i = row -1；逐行检查上一行
    for (var i = row - 1; i >= 0; i -= 1) {
        // 约束，同一列只有一个'Q'
        // 对'Q'所在对角线有 行号 +-列号 = 常量，且同一对角线常量相等
        if (board[i] === column
            || i + board[i] === row + column
            || i - board[i] === row - column) return false;
    }
    return true
};
```

然后在得到答案后，将一维数组转换为二维棋盘的格式去满足题目要求

```javascript
if (row === n) {
    // map方法创建一个新数组来表示'Q'的位置
    let place = board.map((b) => '.'.repeat(b) + 'Q' +  '.'.repeat(n - b - 1))
    return result.push([...place]);
}
```

这样N皇后的问题就一步步的解决了，除了N皇后本身不能相互攻击这一特性的判断代码外。其问题本身最重要的一部分就是通过递归的方式遍历所有可能的情况，这种走不通就往回走，试图找到所有可能解的思路，我们称其为回溯算法。
回顾整篇文章，其实主要就是找到满足递归条件的3要素。

1. 确定要处理哪些数据 
2. 如何将问题拆解 
3. 到哪一步问题才算解决

N皇后问题，重点在于递归那一步，让代码可以遍历所有结果。难点在于判断皇后是否可以落子那一步，不熟悉这套判断，一时间也很难写成恰当的判定条件来。
