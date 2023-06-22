---
category: 专业知识
tags:
  - typescript
date: 2020-06-09
title: typescript学习笔记
vssue-title: typescript学习笔记
---

## 基础

### 原始数据类型

- boolean
- number
- string
- null
- undefined
- Symbol
- void
- any

### 任意值

- any

### 类型推论

在`定义变量时`如果进行了`赋值`，那么变量的`类型就会被自动设置为值的类型`，反之，如果`没有进行赋值`，则会被设置为`any`

### 联合类型

- `type1 | type2`
- 联合类型的变量在被赋值的时候，会根据类型推论的规则推断出一个类型，比如 string|number 类型的变量，当被赋值为 number 类型时不能使用 xxx.length

### 接口 interface

- 确定属性 赋值的时候，变量的形状必须和接口的形状保持一致(不能多也不能少属性)
- 可选属性 `key?: type`
- 任意属性 `[propName: string]: any`
  - 一旦定义任意属性，确定属性和可选属性的类型必须是它的类型的子集
- 只读属性 `readonly key: type`

### 数组类型

- 「类型 + 方括号」表示法
  - eg: `let arr: number[] = [1, 2, 3];`
- 数组泛型
  - eg: `let arr: Array<number> = [1,2,3]`
- 用接口表示数组
  - ```ts
    interface NumberArray {
      [index: number]: number;
    }
    let fibonacci: NumberArray = [1, 1, 2, 3, 5];
    ```
- 类数组
  - 类数组不是数组类型，有自己的类型
  - ```ts
    function sum() {
      let args: {
        [index: number]: number;
        length: number;
        callee: Function;
      } = arguments;
    }
    ```
  - 常用的类数组都有自己的接口定义，如 `IArguments`, `NodeList`, `HTMLCollection`

### 函数类型

- 函数声明
  - ```ts
    function sum(x: number, y: number): number {
      return x + y;
    }
    ```
- 函数表达式

  - ```ts
    let mySum: (x: number, y: number) => number = function(
      x: number,
      y: number
    ): number {
      return x + y;
    };

    // 利用类型推断进行简化
    let mySum = function(x: number, y: number): number {
      return x + y;
    };
    ```

- 接口定义

  - ```ts
    interface SearchFunc {
      (source: string, subString: string): boolean;
    }

    let mySearch: SearchFunc;
    mySearch = function(source: string, subString: string) {
      return source.search(subString) !== -1;
    };
    ```

- 可选参数
  - x?: type
- 剩余参数
  - 同 es6
- 函数重载
  - 匹配多种完全不同的入参格式
  - ```ts
    function reverse(x: number): number;
    function reverse(x: string): string;
    function reverse(x: number | string): number | string {
      if (typeof x === "number") {
        return Number(
          x
            .toString()
            .split("")
            .reverse()
            .join("")
        );
      } else if (typeof x === "string") {
        return x
          .split("")
          .reverse()
          .join("");
      }
    }
    ```

### 类型断言

- 语法
  - 值 as 类型
- 用途
  - 将一个联合类型断言为其中一个类型
  - 将一个父类断言为更加具体的子类
  - 将任何一个类型断言为 `any`
  - 将 `any` 断言为一个具体的类型
  - 要使得 A 能够被断言为 B，只需要 A 兼容 B 或 B 兼容 A 即可
- 双重断言（不要使用）
- 关系
  - 与类型转换
  - 与类型声明
  - 与泛型

### 声明文件

- 新语法

  - `declare var` 声明全局变量
  - `declare let` 声明全局变量
  - `declare const` 声明不可变全局变量
  - `declare function` 声明全局方法
  - `declare class` 声明全局类
  - `declare enum` 声明全局枚举类型
  - `declare namespace` 声明（含有子属性的）全局对象
  - `interface` 和 `type` 声明全局类型
  - `export` 导出变量
  - `export namespace` 导出（含有子属性的）对象
  - `export default` ES6 默认导出
  - `export =` commonjs 导出模块
  - `export as namespace` UMD 库声明全局变量
  - `declare global` 扩展全局变量
  - `declare module` 扩展模块
  - `/// <reference />` 三斜线指令

- 声明语句
  - 当引入了第三方代码的时候，ts 无法识别第三方代码的全局变量，例如可以通过`declare var jQuery: (selector: string) => any;`使用 jQuery
- 声明文件
  - 当把声明语句放到一个单独文件中，就是声明文件
  - 声明文件以`.d.ts`为后缀
  - 社区已经有不少声明文件了，使用@types 统一管理第三方库的声明文件`npm install @types/jquery --save-dev`
  - 可以在[这个页面](https://microsoft.github.io/TypeSearch/)搜索
  - 如果没有生效，可以检查下 tsconfig.json 中的 files、include 和 exclude 配置，确保其包含了 xxx.d.ts 文件
  - 只能定义类型，不能在声明语句中定义具体实现
- `declare namespace`

  - ```ts
    declare namespace jQuery {
      function ajax(url: string, settings?: any): void;
      const version: number;
      class Event {
        blur(eventType: EventType): void;
      }
      enum EventType {
        CustomClick,
      }
      <!-- 可以进行深层嵌套 -->
      namespace fn {
        function extend(object: any): void;
      }
    }

    <!-- 如果只有extend方法，也可以这样写 -->
    declare namespace jQuery.fn {
      function extend(object: any): void;
    }
    ```

  - interface 和 type 可以放在 namespace 下防止命名冲突
  - ```ts
    declare namespace jQuery {
      interface AjaxSettings {
        method?: "GET" | "POST";
        data?: any;
      }
      function ajax(url: string, settings?: AjaxSettings): void;
    }
    ```

- 多个声明可以合并，类似于函数的重载
- npm 包
  - 倒入的 npm 包的声明文件位置
    - 与 npm 包绑定在一起。package.json 中的 types 字段或者 index.d.ts 文件
    - 如果 npm 包维护者没有提供声明文件，其他人会把声明文件发布到@types 中
    - 如果上面两种都没找到，那么就要自己写了
  - 自己写声明文件
    - tsconfig.json 同级目录下创建 types 目录，管理声明文件
    - 需要配置 tsconfig.json 中的 paths 和 baseUrl 字段
    - ```json
      {
        "compilerOptions": {
          "module": "commonjs",
          "baseUrl": "./",
          "paths": {
            "*": ["types/*"]
          }
        }
      }
      ```
- 导入导出
  - npm 包中的声明文件中，使用 declare 只会声明一个局部变量
  - 使用 export 导出时，可以不用写 declare
  - 可以使用 declare 声明多个变量，再使用 export 一次性导出
  - commonjs
    - 导出
      - 整体导出 `module.exports = foo`
      - 单个导出 `exports.bar = bar`
    - 导入
      - 方式 1
        - `const foo = require('foo');`
        - `const bar = require('foo').bar;`
      - 方式 2
        - `import * as foo from 'foo';`
        - `import { bar } from 'foo';`
      - 方式 3
        - `import foo = require('foo');`
        - `import bar = foo.bar;`
  - UMD 库
    - export as namespace
  - 在 npm 包或 UMD 库中扩展全局变量
    - declare global
  - 模块插件
    - declare module
- 声明文件中的依赖

  - import
  - 三斜线

    - 场景
      - 当我们在书写一个全局变量的声明文件时
      - 当我们需要依赖一个全局变量的声明文件时
    - ```ts
      // types/jquery-plugin/index.d.ts

      /// <reference types="jquery" />
      /// <reference path="JQuery.d.ts" />

      declare function foo(options: JQuery.AjaxSettings): string;
      ```

- 自动生成声明文件
  - ```json
    {
      "compilerOptions": {
        "module": "commonjs",
        "outDir": "lib",
        "declaration": true
        "declarationDir": "设置生成 .d.ts 文件的目录",
        "declarationMap": "对每个 .d.ts 文件，都生成对应的 .d.ts.map（sourcemap）文件",
        "emitDeclarationOnly": "仅生成 .d.ts 文件，不生成 .js 文件"
      }
    }
    ```
- 寻找声明文件流程

  1. 寻找`package.json`中的`types`或者`typing`字段指定的地址
  2. 第 1 步没有，就会在项目根目录下寻找是否存在 `index.d.ts` 文件
  3. 第 2 步没有，就会寻找`入口文件`(package.json 中的 main 字段指定的入口文件)的目录下的`同名不同后缀`的`.d.ts`文件

- 将声明文件发布到 @types 下
  - [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/)

### 内置对象

- [TypeScript 核心库的定义文件](https://github.com/Microsoft/TypeScript/blob/master/src/lib/README.md)
- ECMAScript 的内置对象
  - Boolean、Error、Date、RegExp 等
- DOM 和 BOM 的内置对象
  - Document、HTMLElement、Event、NodeList 等
- Node.js
  - TypeScript 核心库的定义中不包含 Node.js 部分
  - 使用 ts 写 node 时，要引入第三方声明文件 `npm install @types/node --save-dev`

## 进阶

### 类型别名

- `给一个类型起个新名字`
- `type`
  - 类型可以是联合类型

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
  if (typeof n === "string") {
    return n;
  } else {
    return n();
  }
}
```

### 字符串字面量类型

- `用来约束取值只能是某几个字符串中的一个`
- `type`

```ts
type EventNames = "click" | "scroll" | "mousemove";
function handleEvent(ele: Element, event: EventNames) {
  // do something
}

handleEvent(document.getElementById("hello"), "scroll"); // 没问题
handleEvent(document.getElementById("world"), "dblclick"); // 报错，event 不能为 'dblclick'

// index.ts(7,47): error TS2345: Argument of type '"dblclick"' is not assignable to parameter of type 'EventNames'.
```

### 元组

- 数组合并了相同类型的对象，而元组（Tuple）合并了不同类型的对象
- `let tom: [string, number] = ['Tom', 25];`

### 枚举

- 用于取值被限定在一定范围内的场景
- 简单例子

```ts
enum Days {
  Sun,
  Mon,
  Tue,
  Wed,
  Thu,
  Fri,
  Sat,
}
// 枚举成员会被赋值为从 0 开始递增的数字，同时也会对枚举值到枚举名进行反向映射

console.log(Days["Sun"] === 0); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true

console.log(Days[0] === "Sun"); // true
console.log(Days[1] === "Mon"); // true
console.log(Days[2] === "Tue"); // true
console.log(Days[6] === "Sat"); // true
```

- 手动赋值

```ts
enum Days {
  Sun = 7,
  Mon = 1,
  Tue,
  Wed,
  Thu,
  Fri,
  Sat,
}

console.log(Days["Sun"] === 7); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true

// 其实等价于enum Days {Sun = 7, Mon = 1, Tue=2, Wed=3, Thu=4, Fri=5, Sat=6};
// 所以可能会出现手动赋值和自动赋值重复的情况，因此要不全部手动赋值，要不全部自动赋值
```

- 常数枚举
  - `const enum`

```ts
const enum Directions {
  Up,
  Down,
  Left,
  Right,
}

let directions = [
  Directions.Up,
  Directions.Down,
  Directions.Left,
  Directions.Right,
];
```

### 类

- 基础部分参考 js 的类
- 访问修饰符
  - public 公有
  - private 私有，不能在类的外部被访问
  - protected 受保护的，只能在类和子类中被访问
- readonly
  - 只允许出现在属性声明或索引签名或构造函数中
  - 如果 readonly 和其他访问修饰符同时存在的话，需要写在其后面
- 构造函数参数修饰符
  - 修饰符和 readonly 可以使用在构造函数参数中，等同于类中定义该属性同时给该属性赋值
- 抽象类
  - 不允许被实例化
  - 抽象类中的抽象方法必须被子类实现

### 类与接口

- 类实现接口
  - implements 关键字
  - 一个类可以实现多个接口
- 接口继承接口
  - extends 关键字
- 接口继承类
  - 当声明一个类的时候，相当于同时创建了一个对应的类型
  - 接口继承类时只会继承类的实例属性和实例方法

### 泛型

- todo

### 声明合并

- 函数合并
- 接口合并
  - 对同一个接口进行多次定义，只要内部属性不冲突，就会合并
  - 接口中方法的合并，与函数合并一样
- 类合并
  - 同接口合并

## 工程

### eslint

```
npm install --save-dev eslint
npm install --save-dev typescript @typescript-eslint/parser
npm install --save-dev @typescript-eslint/eslint-plugin
npm install --save-dev prettier
npm install --save-dev eslint-plugin-react

.eslintrc
module.exports = {
    parser: '@typescript-eslint/parser',
    plugins: ['@typescript-eslint'],
    rules: {
        // 禁止使用 var
        'no-var': "error",
        // 优先使用 interface 而不是 type
        '@typescript-eslint/consistent-type-definitions': [
            "error",
            "interface"
        ]
    }
}

vscode setting.json
{
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "eslint.autoFixOnSave": true,
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        {
            "language": "typescript",
            "autoFix": true
        },
        {
            "language": "typescriptreact",
            "autoFix": true
        }
    ],
    "typescript.tsdk": "node_modules/typescript/lib"
}


// .prettierrc.js
module.exports = {
    // 一行最多 100 字符
    printWidth: 100,
    // 使用 4 个空格缩进
    tabWidth: 4,
    // 不使用缩进符，而使用空格
    useTabs: false,
    // 行尾需要有分号
    semi: true,
    // 使用单引号
    singleQuote: true,
    // 对象的 key 仅在必要时用引号
    quoteProps: 'as-needed',
    // jsx 不使用单引号，而使用双引号
    jsxSingleQuote: false,
    // 末尾不需要逗号
    trailingComma: 'none',
    // 大括号内的首尾需要空格
    bracketSpacing: true,
    // jsx 标签的反尖括号需要换行
    jsxBracketSameLine: false,
    // 箭头函数，只有一个参数的时候，也需要括号
    arrowParens: 'always',
    // 每个文件格式化的范围是文件的全部内容
    rangeStart: 0,
    rangeEnd: Infinity,
    // 不需要写文件开头的 @prettier
    requirePragma: false,
    // 不需要自动在文件开头插入 @prettier
    insertPragma: false,
    // 使用默认的折行标准
    proseWrap: 'preserve',
    // 根据显示样式决定 html 要不要折行
    htmlWhitespaceSensitivity: 'css',
    // 换行符使用 lf
    endOfLine: 'lf'
};
```

### [编译选项](https://www.tslang.cn/docs/handbook/compiler-options.html)

## 参考文档
- [TypeScript Playground](https://www.typescriptlang.org/play/index.html)
- [了不起的 TypeScript 入门教程](https://juejin.im/post/5edd8ad8f265da76fc45362c#heading-10)
- [一文读懂 TypeScript 泛型及应用](https://juejin.im/post/5ee00fca51882536846781ee#heading-0)
- [TypeScript 入门教程](https://ts.xcatliu.com/)
- [TypeScript官方中文文档](https://www.tslang.cn/docs/home.html)