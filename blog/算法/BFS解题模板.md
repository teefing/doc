---
category: 专业知识
top: true
tags:
  - BFS
  - 算法
date: 2021-03-06
title: BFS解题模板
vssue-title: BFS解题模板
---

# BFS

## 模板

```js
// BFS 找到的路径一定是最短的，但代价就是空间复杂度比 DFS 大很多

// 计算从起点 start 到终点 target 的最近距离
function bfs(start, target) {
  const queue = []; // 核心数据结构
  const visited = new Set(); // 避免走回头路, 如果是对树处理就可以不需要visited了

  queue.push(start); // 将起点加入队列
  visited.add(start); // 记录扩散的步数
  let level = 0;

  while (queue.length) {
    const len = queue.length;
    /* 将当前队列中的所有节点向四周扩散 */
    for (let i = 0; i < len; i++) {
      const cur = queue.shift();
      /* 划重点：这里判断是否到达终点 */
      if (cur === target) {
        return level;
      }
      /* 将 cur 的相邻节点加入队列 */
      for (let x of cur.neighbors) {
        if (visited.has(x) === false) {
          queue.push(x);
          visited.add(x);
        }
      }
    }
    /* 划重点：更新步数在这里 */
    level++;
  }
}
```

## 真题

### 二叉树的最小深度

```js
/*
 * @lc app=leetcode.cn id=111 lang=javascript
 *
 * [111] 二叉树的最小深度
 */

// @lc code=start
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
var minDepth = function(root) {
  if (!root) return 0;
  const queue = [root];
  let depth = 1,
    cur;
  while (queue.length) {
    const len = queue.length;
    for (let i = 0; i < len; i++) {
      cur = queue.shift();
      if (!cur.left && !cur.right) return depth;
      cur.left && queue.push(cur.left);
      cur.right && queue.push(cur.right);
    }
    depth++;
  }
  return depth;
};

// @lc code=end
```

### 二叉树的层次遍历 II

```js
/*
 * @lc app=leetcode.cn id=107 lang=javascript
 *
 * [107] 二叉树的层次遍历 II
 */
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number[][]}
 */
var levelOrderBottom = function(root) {
  if (!root) return [];
  let queue = [];
  let levelArr = [];
  let res = [];
  queue.push(root);
  queue.push("#");
  while (queue.length) {
    let p = queue.shift();
    if (p === "#") {
      res.push(levelArr);
      levelArr = [];
      queue.length && queue.push("#");
    } else {
      levelArr.push(p.val);
      p.left && queue.push(p.left);
      p.right && queue.push(p.right);
    }
  }
  return res.reverse();
};
```

### 打开转盘锁

```js
/*
 * @lc app=leetcode.cn id=752 lang=javascript
 *
 * [752] 打开转盘锁
 */

// @lc code=start
/**
 * @param {string[]} deadends
 * @param {string} target
 * @return {number}
 */
var openLock = function(deadends, target) {
  const deads = new Set(deadends);
  const queue = [];
  const visited = new Set();
  queue.push("0000");
  visited.add("0000");
  let step = 0,
    cur,
    len,
    i,
    j;

  while (queue.length) {
    len = queue.length;
    for (i = 0; i < len; i++) {
      cur = queue.shift();
      if (cur === target) return step;
      if (deads.has(cur)) continue;
      for (j = 0; j < 4; j++) {
        const up = addOne(cur, j);
        if (!visited.has(up)) {
          queue.push(up);
          visited.add(up);
        }

        const down = minusOne(cur, j);
        if (!visited.has(down)) {
          queue.push(down);
          visited.add(down);
        }
      }
    }
    step++;
  }
  return -1;
};

function addOne(str, position) {
  str = str.split("");
  let cur = +str[position];
  cur = (cur + 1) % 10;
  str[position] = cur;
  return str.join("");
}
function minusOne(str, position) {
  str = str.split("");
  let cur = +str[position];
  cur = (cur - 1 + 10) % 10;
  str[position] = cur;
  return str.join("");
}
// @lc code=end
```

### 单词接龙

```js
/*
 * @lc app=leetcode.cn id=127 lang=javascript
 *
 * [127] 单词接龙
 */

// @lc code=start
/**
 * @param {string} beginWord
 * @param {string} endWord
 * @param {string[]} wordList
 * @return {number}
 */
var ladderLength = function(beginWord, endWord, wordList) {
  const wordSize = beginWord.length;
  const allow = new Set(wordList);
  if (!allow.has(endWord)) return 0;
  const queue = [beginWord];
  const visited = new Set(queue);
  let step = 1,
    len,
    i,
    j,
    k,
    newWord,
    cur;
  while (queue.length) {
    len = queue.length;
    for (i = 0; i < len; i++) {
      cur = queue.shift();
      if (cur === endWord) return step;
      for (j = 0; j < wordSize; j++) {
        for (k = 0; k < 26; k++) {
          newWord = getNewWord(cur, j, k);
          if (newWord !== cur && allow.has(newWord) && !visited.has(newWord)) {
            queue.push(newWord);
            visited.add(newWord);
          }
        }
      }
    }
    step++;
  }
  return 0;
};

function getNewWord(str, pos, letter) {
  str = str.split("");
  str[pos] = String.fromCharCode(letter + 97);
  return str.join("");
}
// @lc code=end
```

### 滑动谜题

```js
/*
 * @lc app=leetcode.cn id=773 lang=javascript
 *
 * [773] 滑动谜题
 */

// @lc code=start
/**
 * @param {number[][]} board
 * @return {number}
 */
var slidingPuzzle = function(board) {
  let start = board[0].concat(board[1]).join("");
  const target = "123450";
  const neighborMap = [
    [1, 3],
    [0, 2, 4],
    [1, 5],
    [0, 4],
    [1, 3, 5],
    [2, 4],
  ];
  let queue = [start];
  let visited = new Set(queue);
  let step = 0;

  while (queue.length) {
    let len = queue.length;
    for (let i = 0; i < len; i++) {
      let curBoard = queue.shift();
      if (curBoard === target) {
        return step;
      }
      let zeroIndex = curBoard.indexOf("0");

      let neighbor = neighborMap[zeroIndex];
      neighbor.forEach((neighborPos) => {
        let newBoard = swap(curBoard, zeroIndex, neighborPos);

        if (!visited.has(newBoard)) {
          queue.push(newBoard);
          visited.add(newBoard);
        }
      });
    }
    step++;
  }
  return -1;
};

function swap(str, i, j) {
  str = str.split("");
  [str[i], str[j]] = [str[j], str[i]];
  return str.join("");
}
// @lc code=end
```
