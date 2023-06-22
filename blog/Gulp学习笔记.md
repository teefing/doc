---
category: 专业知识
tags:
  - Gulp
  - 前端工程化
date: 2021-02-23
title: Gulp学习笔记
vssue-title: Gulp学习笔记
---

# Grunt 学习笔记

## 起步

1. 安装 grunt `yarn add gulp`
2. 添加 gulpfile.js 文件

## 开始

gulpfile.js

```js
exports.foo = () => {
  console.log("foo task working~");
};
```

在 gulpfile.js 中导出的函数都会作为 gulp 任务

### 任务异步

gulp 的任务函数都是异步的  
可以通过调用回调函数标识任务完成

```js
exports.foo = (done) => {
  console.log("foo task working~");
  done(); // 标识任务执行完成
};
```

### 默认任务

```js
// default 是默认任务
// 在运行时可以省略任务名参数 yarn gulp
exports.default = (done) => {
  console.log("default task working~");
  done();
};
```

### v4.0 前

```js
// v4.0 之前需要通过 gulp.task() 方法注册任务
const gulp = require("gulp");

gulp.task("bar", (done) => {
  console.log("bar task working~");
  done();
});
```

## 组合任务

使用 gulp 提供的 series 和 parallel 能实现串行任务和并行任务

```js
const { series, parallel } = require("gulp");

const task1 = (done) => {
  setTimeout(() => {
    console.log("task1 working~");
    done();
  }, 1000);
};

const task2 = (done) => {
  setTimeout(() => {
    console.log("task2 working~");
    done();
  }, 1000);
};

const task3 = (done) => {
  setTimeout(() => {
    console.log("task3 working~");
    done();
  }, 1000);
};

// 让多个任务按照顺序依次执行
exports.foo = series(task1, task2, task3);

// 让多个任务同时执行
exports.bar = parallel(task1, task2, task3);
```

## 异步

gulp 任务的编写形式十分丰富，有通过形参 done 的，有 promise，也有流

### 基础 done

最基础的就是通过一个形参 done，在任务完成时手动调用

```js
exports.callback = (done) => {
  console.log("callback task");
  done();
};
```

### 基础任务失败

如果需要通知 gulp 任务失败，可以为 done 方法传入一个 error

```js
exports.callback_error = (done) => {
  console.log("callback task");
  done(new Error("task failed"));
};
```

### promise 形式

也可以返回一个 resolve 状态的 promise 告知 gulp 任务成功

```js
exports.promise = () => {
  console.log("promise task");
  return Promise.resolve();
};
```

### promise 形式失败

返回一个 reject 状态的 promise 告知 gulp 任务失败

```js
exports.promise_error = () => {
  console.log("promise task");
  return Promise.reject(new Error("task failed"));
};
```

### async/await 形式

使用 async/await 语法糖可以简化代码编写

```js
const timeout = (time) => {
  return new Promise((resolve) => {
    setTimeout(resolve, time);
  });
};

exports.async = async () => {
  await timeout(1000);
  console.log("async task");
};
```

### stream 形式

gulp 中最常用的一种形式--流，因为在 gulp 中大多数情况都是进行读取文件，处理文件，再写入文件的操作

```js
exports.stream = () => {
  const read = fs.createReadStream("yarn.lock");
  const write = fs.createWriteStream("a.txt");
  read.pipe(write);
  // 结束时机是read stream end时
  return read;
};
```

流的方式可以用如下代码理解

```js
exports.stream = (done) => {
  const read = fs.createReadStream("yarn.lock");
  const write = fs.createWriteStream("a.txt");
  read.pipe(write);
  read.on("end", () => {
    done();
  });
};
```

### 任务失败对其他任务影响

当 gulp 的合成任务中，有一个任务失败时，后续任务也将不再执行

```js
exports.error_series = series(this.callback, this.callback_error, this.promise);
exports.error_parallel = parallel(
  this.callback,
  this.callback_error,
  this.promise
);
```

## gulp 流处理

gulp 充分利用了 node 中的流
下面代码演示了 gulp 中最常见的处理方式

1. 创建文件读取流读取文件
2. 创建转换流转换文件内容
3. 创建文件写入流写入文件

```js
const fs = require("fs");
const { Transform } = require("stream");

exports.default = () => {
  // 文件读取流
  const readStream = fs.createReadStream("normalize.css");

  // 文件写入流
  const writeStream = fs.createWriteStream("normalize.min.css");

  // 文件转换流
  const transformStream = new Transform({
    // 核心转换过程
    transform: (chunk, encoding, callback) => {
      const input = chunk.toString();
      // 将css文件中的空格和注释删除
      const output = input.replace(/\s+/g, "").replace(/\/\*.+?\*\//g, "");
      callback(null, output);
    },
  });

  return readStream
    .pipe(transformStream) // 转换
    .pipe(writeStream); // 写入
};
```

### gulp 文件操作 api

```js
const { src, dest } = require("gulp");
const cleanCSS = require("gulp-clean-css");
const rename = require("gulp-rename");

exports.default = () => {
  return src("src/*.css")
    .pipe(cleanCSS())
    .pipe(rename({ extname: ".min.css" }))
    .pipe(dest("dist"));
};
```

gulp 提供了 src 和 dest 方法简化文件读取流和写入流的创建，在社区中的各种插件在执行后都是返回一个转换流，这样通过流的 pipe 方法就能很方便的进行文件处理

## 实战

下面是文件目录树
.
├── LICENSE
├── README.md
├── gulpfile.js
├── package.json
├── public
│ └── favicon.ico
├── src
│ ├── about.html
│ ├── assets
│ │ ├── fonts
│ │ │ ├── pages.eot
│ │ │ ├── pages.svg
│ │ │ ├── pages.ttf
│ │ │ └── pages.woff
│ │ ├── images
│ │ │ ├── brands.svg
│ │ │ └── logo.png
│ │ ├── scripts
│ │ │ └── main.js
│ │ └── styles
│ │ ├── \_icons.scss
│ │ ├── \_variables.scss
│ │ ├── demo.scss
│ │ └── main.scss
│ ├── features.html
│ ├── index.html
│ ├── layouts
│ │ └── basic.html
│ └── partials
│ ├── footer.html
│ ├── header.html
│ └── tags.html
└── yarn.lock

gulpfile.js

```js
const { src, dest, parallel, series, watch } = require("gulp");

// 第三方模块用于删除文件
const del = require("del");
// 用于创建开发服务器，能实现文件监听并自动刷新
const browserSync = require("browser-sync");
// 简化gulp的模块引入的插件
const loadPlugins = require("gulp-load-plugins");

const plugins = loadPlugins();
// 创建开发服务器实例
const bs = browserSync.create();

// swag模板页面中需要注入的数据
const data = {
  menus: [
    {
      name: "Home",
      icon: "aperture",
      link: "index.html",
    },
    {
      name: "Features",
      link: "features.html",
    },
    {
      name: "About",
      link: "about.html",
    },
    {
      name: "Contact",
      link: "#",
      children: [
        {
          name: "Twitter",
          link: "https://twitter.com/w_zce",
        },
        {
          name: "About",
          link: "https://weibo.com/zceme",
        },
        {
          name: "divider",
        },
        {
          name: "About",
          link: "https://github.com/zce",
        },
      ],
    },
  ],
  pkg: require("./package.json"),
  date: new Date(),
};

// 任务：删除dist和temp文件夹
const clean = () => {
  return del(["dist", "temp"]);
};

// 任务：匹配src/assets/styles目录下的scss文件，调用gulp-sass插件，从而将scss文件转换为css文件，存放于temp目录下的assets/styles文件夹中
const style = () => {
  // 设置base，从而src后面的路径会在写入时被保留，css文件最终生成到temp/assets/styles目录下
  return (
    src("src/assets/styles/*.scss", { base: "src" })
      // 将scss文件处理为css文件，outputStyle: "expanded"用于将右括号换行
      .pipe(plugins.sass({ outputStyle: "expanded" }))
      // 生成到temp目录下
      .pipe(dest("temp"))
      // 任务结束后服务器reload刷新页面，并采用流的方式写入文件
      .pipe(bs.reload({ stream: true }))
  );
};

// 任务：匹配src/assets/scripts目录下的js文件，调用gulp-babel插件，从而将高版本js语法转为低版本js语法，存放于temp目录下的assets/scripts文件夹中
const script = () => {
  // 设置base，从而src后面的路径会在写入时被保留，js文件最终生成到temp/assets/scripts目录下
  return (
    src("src/assets/scripts/*.js", { base: "src" })
      // 将高版本js语法转为低版本js语法，使用@babel/preset-env这个转换规则
      // 对于babel的配置也可以放在.babelrc中
      .pipe(plugins.babel({ presets: ["@babel/preset-env"] }))
      // 生成到temp目录下
      .pipe(dest("temp"))
      // 任务结束后服务器reload刷新页面，并采用流的方式写入文件
      .pipe(bs.reload({ stream: true }))
  );
};

// 任务：匹配src目录下的html文件，调用gulp-swig插件，从而向swig模板中注入数据生成html，存放于temp目录下的temp文件夹中
const page = () => {
  // 设置base，从而src后面的路径会在写入时被保留，html文件最终生成到temp目录下
  return (
    src("src/*.html", { base: "src" })
      //向swig模板中注入数据data生成html
      .pipe(plugins.swig({ data, defaults: { cache: false } })) // 防止模板缓存导致页面不能及时更新
      // 生成到temp目录下
      .pipe(dest("temp"))
      // 任务结束后服务器reload刷新页面，并采用流的方式写入文件
      .pipe(bs.reload({ stream: true }))
  );
};

// 任务：匹配src/assets/images目录下的所有文件，调用gulp-imagemin插件，从而将图片进行压缩，存放于dist目录下的assets/images文件夹中
const image = () => {
  return src("src/assets/images/**", { base: "src" })
    .pipe(plugins.imagemin())
    .pipe(dest("dist"));
};

// 任务：匹配src/assets/fonts目录下的所有文件，调用gulp-imagemin插件，从而将图片进行压缩，存放于dist目录下的assets/fonts文件夹中
// 正常情况下直接拷贝就好了，但是考虑到字体中存在svg，使用imagemin插件处理一下
const font = () => {
  return src("src/assets/fonts/**", { base: "src" })
    .pipe(plugins.imagemin())
    .pipe(dest("dist"));
};

// 任务：匹配public目录下的所有文件，复制，存放于dist目录下
const extra = () => {
  return src("public/**", { base: "public" }).pipe(dest("dist"));
};

// 任务：启动开发服务器，并监视html,js,scss,图片，字体文件，当它们发生变动后重新执行对应的任务
const serve = () => {
  watch("src/assets/styles/*.scss", style);
  watch("src/assets/scripts/*.js", script);
  watch("src/*.html", page);
  // watch('src/assets/images/**', image)
  // watch('src/assets/fonts/**', font)
  // watch('public/**', extra)
  // watch可以监视一个数组
  watch(
    ["src/assets/images/**", "src/assets/fonts/**", "public/**"],
    bs.reload
  );

  bs.init({
    notify: false,
    port: 2080,
    // open: false,
    // 当dist目录下文件被修改后，自动重启bs
    // files: 'dist/**',
    server: {
      // 配置备用baseDir，当temp目录下没找到需要的文件，就会去src，public目录下依次查找
      baseDir: ["temp", "src", "public"],
      // 优先于baseDir
      routes: {
        // 创建一个路由，将/node_modules的请求映射到node_modules文件夹
        "/node_modules": "node_modules",
      },
    },
  });
};

// 任务：处理文件中的路径引用，并且压缩html js css文件
/**
 * 在html中添加如下注释，可以用于告诉useref如何处理引用并压缩文件
  <!-- build:css assets/styles/vendor.css -->
  <link rel="stylesheet" href="/node_modules/bootstrap/dist/css/bootstrap.css">
  <!-- endbuild -->
  <!-- build:css assets/styles/main.css -->
  <link rel="stylesheet" href="assets/styles/main.css">
  <!-- endbuild -->
 */
const useref = () => {
  return (
    src("temp/*.html", { base: "temp" })
      .pipe(plugins.useref({ searchPath: ["temp", "."] }))
      // html js css
      .pipe(plugins.if(/\.js$/, plugins.uglify()))
      .pipe(plugins.if(/\.css$/, plugins.cleanCss()))
      .pipe(
        plugins.if(
          /\.html$/,
          plugins.htmlmin({
            // 移除多余空格
            collapseWhitespace: true,
            // 压缩html中的css
            minifyCSS: true,
            // 压缩html中的js
            minifyJS: true,
          })
        )
      )
      .pipe(dest("dist"))
  );
};

// 任务：并行运行style,script和page任务
const compile = parallel(style, script, page);

// 上线之前执行的任务
// 任务：打包
const build = series(
  clean,
  parallel(series(compile, useref), image, font, extra)
);

// 任务：编译并启动服务器
const develop = series(compile, serve);

// 只对外导出三个任务
module.exports = {
  clean,
  build,
  develop,
};
```

## 脚手架化

1. 创建一个新的包
package.json中添加
```js
"main": "lib/index.js",
"bin": "bin/cli.js",
```

2. index.js
在之前代码的基础上，从外部文件中读取配置
```js
const { src, dest, parallel, series, watch } = require('gulp')

const del = require('del')
const browserSync = require('browser-sync')

const loadPlugins = require('gulp-load-plugins')

const plugins = loadPlugins()
const bs = browserSync.create()
const cwd = process.cwd()
let config = {
  // default config
  build: {
    src: 'src',
    dist: 'dist',
    temp: 'temp',
    public: 'public',
    paths: {
      styles: 'assets/styles/*.scss',
      scripts: 'assets/scripts/*.js',
      pages: '*.html',
      images: 'assets/images/**',
      fonts: 'assets/fonts/**'
    }
  }
}

try {
  const loadConfig = require(`${cwd}/pages.config.js`)
  config = Object.assign({}, config, loadConfig)
} catch (e) {}

const clean = () => {
  return del([config.build.dist, config.build.temp])
}

const style = () => {
  // 配置cwd
  return src(config.build.paths.styles, { base: config.build.src, cwd: config.build.src })
    .pipe(plugins.sass({ outputStyle: 'expanded' }))
    .pipe(dest(config.build.temp))
    .pipe(bs.reload({ stream: true }))
}

const script = () => {
  return src(config.build.paths.scripts, { base: config.build.src, cwd: config.build.src })
  // 使用require方式，以同步的方式加载代码，防止在项目中打包时出现找不到@babel/preset-env这个包的问题
    .pipe(plugins.babel({ presets: [require('@babel/preset-env')] }))
    .pipe(dest(config.build.temp))
    .pipe(bs.reload({ stream: true }))
}

const page = () => {
  return src(config.build.paths.pages, { base: config.build.src, cwd: config.build.src })
    .pipe(plugins.swig({ data: config.data, defaults: { cache: false } }))
    .pipe(dest(config.build.temp))
    .pipe(bs.reload({ stream: true }))
}

const image = () => {
  return src(config.build.paths.images, { base: config.build.src, cwd: config.build.src })
    .pipe(plugins.imagemin())
    .pipe(dest(config.build.dist))
}

const font = () => {
  return src(config.build.paths.fonts, { base: config.build.src, cwd: config.build.src })
    .pipe(plugins.imagemin())
    .pipe(dest(config.build.dist))
}

const extra = () => {
  return src('**', { base: config.build.public, cwd: config.build.public })
    .pipe(dest(config.build.dist))
}

const serve = () => {
  watch(config.build.paths.styles, { cwd: config.build.src }, style)
  watch(config.build.paths.scripts, { cwd: config.build.src }, script)
  watch(config.build.paths.pages, { cwd: config.build.src }, page)
  // watch('src/assets/images/**', image)
  // watch('src/assets/fonts/**', font)
  // watch('public/**', extra)
  watch([
    config.build.paths.images,
    config.build.paths.fonts
  ], { cwd: config.build.src }, bs.reload)

  watch('**', { cwd: config.build.public }, bs.reload)

  bs.init({
    notify: false,
    port: 2080,
    // open: false,
    // files: 'dist/**',
    server: {
      baseDir: [config.build.temp, config.build.dist, config.build.public],
      routes: {
        '/node_modules': 'node_modules'
      }
    }
  })
}

const useref = () => {
  return src(config.build.paths.pages, { base: config.build.temp, cwd: config.build.temp })
    .pipe(plugins.useref({ searchPath: [config.build.temp, '.'] }))
    // html js css
    .pipe(plugins.if(/\.js$/, plugins.uglify()))
    .pipe(plugins.if(/\.css$/, plugins.cleanCss()))
    .pipe(plugins.if(/\.html$/, plugins.htmlmin({
      collapseWhitespace: true,
      minifyCSS: true,
      minifyJS: true
    })))
    .pipe(dest(config.build.dist))
}

const compile = parallel(style, script, page)

// 上线之前执行的任务
const build =  series(
  clean,
  parallel(
    series(compile, useref),
    image,
    font,
    extra
  )
)

const develop = series(compile, serve)

module.exports = {
  clean,
  build,
  develop
}


```

3. bin/cli.js
```javascript
#!/usr/bin/env node

// gulp 运行时可以指定gulpfile文件目录
process.argv.push('--cwd')
process.argv.push(process.cwd())
process.argv.push('--gulpfile')
process.argv.push(require.resolve('..'))

require('gulp/bin/gulp')
```

4. 测试
命令行`yarn link`，从而创建包的软链  
在需要的使用该cli的项目中，配置pages.config.js  
```js
module.exports = {
  build: {
    src: 'src',
    dist: 'release',
    temp: '.tmp',
    public: 'public',
    paths: {
      styles: 'assets/styles/*.scss',
      scripts: 'assets/scripts/*.js',
      pages: '*.html',
      images: 'assets/images/**',
      fonts: 'assets/fonts/**'
    }
  },
  data: {
    menus: [
      {
        name: 'Home',
        icon: 'aperture',
        link: 'index.html'
      },
      {
        name: 'Features',
        link: 'features.html'
      },
      {
        name: 'About',
        link: 'about.html'
      },
      {
        name: 'Contact',
        link: '#',
        children: [
          {
            name: 'Twitter',
            link: 'https://twitter.com/w_zce'
          },
          {
            name: 'About',
            link: 'https://weibo.com/zceme'
          },
          {
            name: 'divider'
          },
          {
            name: 'About',
            link: 'https://github.com/zce'
          }
        ]
      }
    ],
    pkg: require('./package.json'),
    date: new Date()
  }
}

```
配置package.json
```js
"scripts": {
    "clean": "你的cli包名 clean",
    "build": "你的cli包名 build",
    "develop": "你的cli包名 develop"
  },
```
命令行`yarn link 你的cli包名`  
命令行`yarn build`运行