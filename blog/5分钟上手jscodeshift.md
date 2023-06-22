---
  category: 专业知识
  tags:
    - jscodeshift
  date: 2021-03-08
  title: 5分钟上手jscodeshift
  vssue-title: 5分钟上手jscodeshift
---

# 5分钟上手jscodeshift

## 小白鼠代码
```js
app.say = function(test) {
  console.log(test);
}

app.get('/api/config/save', checkConfigHighRiskPermission, function() {
  console.log('cool')
});

app.say('123')
```

## 手术刀代码
```js
const j = require("jscodeshift");
const fs = require("fs");
const path = require("path");

const jsContent = fs
  .readFileSync(path.join(__dirname, "./test.js"))
  .toString("utf-8");

// 将原代码转为ast
let rootAst = j(jsContent);

// 查
// 获取我们想要的部分ast
ast = rootAst.find(j.CallExpression, {
  callee: {
    object: {
      name: "app",
    },
    property: {
      name: "get",
    },
  },
});

// 改：ast节点替换
// 将get方法替换为post
ast.find(j.Identifier, { name: "get" }).forEach((path) => {
  j(path).replaceWith(j.identifier("post"));
});

// 将function回调函数替换为箭头函数
ast.find(j.FunctionExpression).forEach((path) => {
  j(path).replaceWith(
    j.arrowFunctionExpression(path.value.params, path.value.body, false)
  );
});

// 增：增加ast节点
// 在上面的箭头函数后面再加一个箭头函数
ast.find(j.ArrowFunctionExpression).forEach((path) => {
  j(path).insertAfter(
    j.arrowFunctionExpression(
      path.value.params, // 方法参数
      path.value.body, // 方法体
      false // expression
    )
  );
});

// 删
// 删除箭头函数
ast.find(j.ArrowFunctionExpression).forEach((path) => {
  j(path).replaceWith()
})

// 将给定的代码快速转ast加入ast
ast = rootAst.find(j.ExpressionStatement).forEach((path) => {
  j(path).insertAfter(`console.log(111)`)
})

// 所有的操作都会修改rootAst
// 导出 将ast导出为代码
console.log(rootAst.toSource());

```

## 处理后代码
```js
app.say = function(test) {
  console.log(test);
  console.log(111)
}

console.log(111)

app.post('/api/config/save', checkConfigHighRiskPermission, , );

console.log(111)

app.say('123')
console.log(111)
```