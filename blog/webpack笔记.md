---
  category: 专业知识
  tags:
    - Webpack
  date: 2020-11-12
  draft: true
  title: Webpack笔记
  vssue-title: Webpack笔记
---
## Why Webpack
1. 应用规模增大，维护难--手动维护js之间的**依赖**、**作用域**污染、网页内多script的**网络开销**--需要模块化开发
2. Webpack优势
   1. 默认支持多种模块标准（AMD、CommonJS、ES6模块）
   2. 代码分割
   3. 处理各种类型资源
   4. 社区庞大 
   
## Webpack使用
总流程： 1.安装(webpack/webpack-cli/webpack-dev-server) 2.配置npm script 3.配置webpack.config.js 4.运行

#### CommonJS vs ES6 Module
##### CommonJS
1. 动态结构
   1. 依赖关系建立在代码运行阶段
   2. 代码执行前无法确定依赖--难以进行分析优化
   3. 
2. 值拷贝
   1. 导出时导出的是值的拷贝，可以理解为调用函数时传入的变量和函数的形参的关系
3. 循环依赖
   1. module.loaded记录是否被加载过，如果被加载过直接导出数据，不会再执行模块代码
   2. todo

##### ES6 Module
1. 静态结构
   1. 依赖关系建立在代码编译阶段
   2. 导入导出都是声明式的，路径不允许是表达式，必须顶层作用域，不允许if判断--易于分析
      1. 死代码检查
      2. 模块变量类型检查
      3. 编译器优化，直接导入变量，减小引用层级
2. 值动态映射
   1. 导入的值是只读的
   2. 导出的值的修改会直接影响导入的值
3. 循环依赖
   1. todo

##### 打包原理
每个模块都放在各自的函数作用域中，再配上各自模块的唯一标识，组装得到key：function的模块对象，function接受module,exports和webpack自定义的require参数，用于模拟commonjs的三个主要参数
从入口模块开始执行require，就可以以深度优先遍历的方式递归遍历整个模块对象

