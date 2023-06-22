---
title: emmet使用
category: 专业知识
date: 2019-05-12 11:18:26
tags:
  - 工具
---

# emmet 使用

Emmet 使用类似于 CSS 选择器的语法描述元素在生成的文档树中的位置及其属性。

## 元素

可以使用元素名（如 div 或者 p）来*生成* HTML 标签。Emmet 没有预定义的有效元素名的集合，可以把任何单词当作标签来生成和使用：`div` → `<div></div>`, `foo` → `<foo></foo>`  等。

## 嵌套运算符

嵌套运算符用于以缩写的方式安排元素在生成文档树中的位置：将其放在内部或成为相邻的元素。

### 子: `>`

可以使用 >  运算符指定嵌套元素在另一个元素内部：

```css
div>ul>li
```

生成的结果为：

```html
<div>
  <ul>
    <li></li>
  </ul>
</div>
```

### 兄弟: `+`

使用  `+`  运算符将相邻的其它元素处理为同级：

```css
div+p+bq
```

生成的结果为：

```html
<div></div>
<p></p>
<blockquote></blockquote>
```

### 上升: `^`

使用  `>`  运算符将会降低所有后续所有元素在生成树中的级别，每一级的兄弟元素也被解析成相同深度的元素：

```css
div+div>p>span+em
```

将生成：

```html
<div></div>
<div>
  <p><span></span><em></em></p>
</div>
```

使用  `^`  运算符，能够提升元素在生成树中的一个级别，并同时影响其后的元素：

```css
div+div>p>span+em^bq
```

将生成：

```html
<div></div>
<div>
  <p><span></span><em></em></p>
  <blockquote></blockquote>
</div>
```

可以连续使用多个  `^`  运算符，每次提高一个级别：

```css
div+div>p>span+em^^^bq
```

将生成：

```html
<div></div>
<div>
  <p><span></span><em></em></p>
</div>
<blockquote></blockquote>
```

### 重复: `*`

使用  `*`  运算符可以定义一组元素：

```css
ul>li*5
```

将生成：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

### 分组: `()`

括号用于在复杂的 Emmet 缩写中处理一组子树：

```css
div>(header>ul>li*2>a)+footer>p
```

将生成：

```html
<div>
  <header>
    <ul>
      <li><a href=""></a></li>
      <li><a href=""></a></li>
    </ul>
  </header>
  <footer>
    <p></p>
  </footer>
</div>
```

如果想与浏览器 DOM 协同工作，可能想要对文档片段分组：每个组包含一个子树，所有的后续元素都插入到与组中第一个元素相同的级别中。

能够在组中嵌套组并且使用  `*`  运算符绑定它们：

```css
(div>dl>(dt+dd)*3)+footer>p
```

将生成：

```html
<div>
  <dl>
    <dt></dt>
    <dd></dd>
    <dt></dt>
    <dd></dd>
    <dt></dt>
    <dd></dd>
  </dl>
</div>
<footer>
  <p></p>
</footer>
```

使用分组，可以使用单个缩写逐个写出整页的标签，不过尽量不要这么做。

## 属性运算符

属性运算符用于编辑所生成的元素的属性，在 HTML 和 XML 中可以快速地为生成元素添加  `class`  属性。

### ID 和 CLASS

在 CSS 中，可以使用  `elem#id`  和  `elem.class`  注解来达到为元素指定  `id`  或  `class 属性的目的。`在 Emmet 中，可以使用几乎相同的语法来为指定的元素*添加*这些属性：element:

```css
div#header+div.page+div#footer.class1.class2.class3
```

生成：

```html
<div></div>
<div></div>
<div></div>
```

### 自定义属性

可以使用  `[attr]`  注解（就像在 CSS 中一样）来为元素添加自定义属性：

```td[title="Hello world!" colspan=3]

```

将生成：

```html
<td title="Hello world!" colspan="3"></td>
```

- 能够在方括号中放置许多属性，
- 可以不为属性指定值： `td[colspan title]`  将生成  `<td colspan="" title="">` ，如果你的编辑器支持，可以使用 tab 来跳到每个空属性中填写。
- 属性可以用单引号或双引号作为定界符。
- 如果属性不包含空格，不需要用定界符括住它：`td[title=hello colspan=3]`  是正确的。

### 编号: `$`

使用  `*`  运算符可以重复生成元素，如果带  `$`  就可以为它们编号。把  `$`  放在元素名、属性名或者属性值中，将为每个元素生成正确的编号：

```css
ul>li.item$*5
```

将生成：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

使用多  `$`  可以填充前导的零：

```css
ul>li.item$$*5
```

将生成：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

#### 改变编号的基数和方向

使用  `@` ，可以改变数字的走向（升序或降序）和基数（例如起始值）。

在  `$ 后添加 @- 来改变数字走向：`

```css
ul>li.item$@-*5
```

将生成：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

在  `$ 后面添加 @N 改变编号的基数：`

```css
ul>li.item$@3*5
```

将生成：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

这些附加的运算符可以同时使用：

```css
ul>li.item$@-3*5
```

将生成：

```html
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

## 文本: `{}`

可以用花括号向元素中添加文本：

```css
a{Click me}
```

将生成：

```html
<a href="">Click me</a>
```

注意，这个  `{text}`  是被当成独立元素解析的（类似于  `div`, `p` ），但当其跟在其它元素后面时则有所不同。例如， `a{click}`  和  `a>{click}`  产生相同的输出，但是  `a{click}+b{here}`  和  `a>{click}+b{here}`  的输出就不同了：

```html
<!-- a{click}+b{here} -->
<a href="">click</a><b>here</b>

<!-- a>{click}+b{here} -->
<a href="">click<b>here</b></a>
```

在第二示例中， `<b>`  元素放在了  `<a>`  元素的里面。差别如下：当  `{text}`  写在元素的后面，它不影响父元素的上下文。下面是展示这种差别的重要性的较复杂的例子：

```css
p>{Click }+a{here}+{ to continue}
```

生成：

```html
<p>Click <a href="">here</a> to continue</p>
```

在这个例子里， 我们用  > 运算符明确的将  `Click here to continue`  下移一级，放在  `<p>`  元素内，但对于  `a`  元素的内容就不需要了，因为  `<a>`  仅有  `here`  这一部分内容，它不改变父元素的上下文。

作为比较，下面是不带有 > 运算符的相同缩写：

```css
p{Click }+a{here}+{ to continue}
```

生成：

```html
<p>Click</p>
<a href="">here</a> to continue
```

## 缩写格式的注意事项

当熟悉了 Emmet 的缩写语法后，可能会想要使用一些格式来生成更可读的缩写。例如，在元素和运算符之间使用空格间隔：

```css
(header > ul.nav > li*5) + footer
```

但是这种写法是错误的，因为空格是 Emmet 停止缩写解析的*标识符*。

请多用户误以为每个缩写都应写在新行上，但是他们错了：可以在文本的任意位置键入和扩展缩写。

这也就是为什么当想要停止解析和扩展时，Emmet 需要一些标志的原因。如果你仍然认为复杂的缩写需要一些格式使其更易读：

- 缩写不是模板语言，它们不需要”易读 “，它们必须” 可快速扩展和移动“。
- 不需要写复杂的缩写。不要认为在 web 编程中” 键入 “是最慢的运算。想快速找出构建单个的复杂缩写比构造和键入一些较短较简单的缩写更慢。
