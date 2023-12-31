---
title: scss配置与运行
category: 专业知识
date: 2019-05-12 10:51:35
tags: 
 - SCSS
 - CSS
 - 工具
---


## 1.`SCSS`和`Sass`

> `Sass` 和 `SCSS` 其实是同一种东西，我们平时都称之为 `Sass`。**他们都是用`Ruby`开发 Css 预处理器，`boostrap4`已经将`less`换成了`sass`。**

不同之处：

*   文件拓展名：分别是`sass`和`scss`
*   缩进：`sass`严格缩进（类似 python 和 ruby），`scss`是 css 的缩进样式
*   是否兼容 css 语法：显然，由于缩进的不同，`scss`是兼容原生的 css 写法。

总的来说，`scss`是`sass`升级版，兼容 css 语法，并且有着自己的独立语法。

## 2.环境配置

1.  安装 ruby：windows 注意添加注册表路径
2.  安装 sass：利用 ruby 的包管理器`gem`安装，命令行运行:`gem install sass`
3.  升级和删除 sass：`gem update/uninstall sass`

**如果国外源过慢？**

```gem sources --remove https://rubygems.org/ 
gem sources -a https://ruby.taobao.org/
gem sources -l #查看是不是淘宝源
```

## 3.编译

> 编译指的是：将 scss 文件编译为 css 文件的过程。

### 3.1 源文件编译

**单文件编译**

```# 格式：sass 待编译的Sass文件名:编译后CSS文件名
scss scss.scss:css.css
```

**实时自动编译**

使用`--watch`参数即可，scss 会在源文件改动时候，自动重新编译。

```scss --watch scss.scss:css.css # 方便
```

### 3.2 输出文件风格

> 命令行编译时候，使用`--style`参数。

一段 scss 代码：

```body {
  h1 {
    color : red;
  }
}
```

**默认：嵌套输出方式 nested**

```
body h1 {
  color: red; }
```

**展开输出方式 expanded**

```
body h1 {
  color: red;
}
```

**紧凑输出方式 compact**

```
body h1 { color: red; }
```

**压缩输出方式 compressed**

```
body h1{color:red}
```

## 4.注意

> 最新的 scss 开启了`sourcemap`功能，`--sourcemap`参数默认添加。
