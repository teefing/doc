---
category: 专业知识
tags:
  - Grunt
  - 前端工程化
date: 2021-02-20
title: Grunt学习笔记
vssue-title: Grunt学习笔记
---
# Grunt学习笔记
## 起步

1. 安装 grunt `yarn add grunt`
2. 添加 gruntfile.js 文件

## 开始

gruntfile.js

```js
/** @param {import('grunt')} grunt */
module.exports = (grunt) => {
  grunt.registerTask("bar", () => {
    console.log("other task");
  });
};
```

gruntfile.js 是使用 grunt 的入口文件, 该文件中导出一个函数，这个函数接收一个 grunt 对象，该对象中提供了一些创建任务时用到的 API  
使用`grunt.registerTask`来创建一个基本任务，第一个参数是任务名称，第二个参数传入任务处理的逻辑函数  
命令行中通过`yarn grunt taskname`来运行任务

## 任务提示

```js
grunt.registerTask("foo", "a sample task", () => {
  console.log("hello grunt");
});
```

给第二个参数传入字符串就可以给该任务配置任务提示  
使用`yarn grunt --help`获取提示

## 在一个任务中执行其他任务

```js
grunt.registerTask("foo", "a sample task", () => {
  // foo 和 bar 会在当前任务执行完成过后自动依次执行
  grunt.task.run("foo", "bar");
  console.log("current task runing~");
});
```

使用 `grunt.task.run` 来手动运行其他任务

```js
// 第二个参数可以指定此任务的映射任务，
// 这样执行 default 就相当于执行对应的任务
// 这里映射的任务会按顺序依次执行，不会同步执行
grunt.registerTask("default", ["foo", "bar"]);
```

也可以使用第二个参数来映射任务

## 异步任务

```js
// 由于函数体中需要使用 this，所以这里不能使用箭头函数
grunt.registerTask("async-task", function() {
  const done = this.async();
  setTimeout(() => {
    console.log("async task working~");
    done();
  }, 1000);
});
```

使用`this.async()`得到用于通知 grunt 结束任务的函数，在异步任务结束后调用该函数通知 grunt 任务已结束

## 任务失败

## 同步任务失败

```js
grunt.registerTask("bad", () => {
  console.log("bad working~");
  return false;
});
```

任务函数执行过程中如果返回 false ,则意味着此任务执行失败

## 异步任务失败

```js
grunt.registerTask("bad-async", function() {
  const done = this.async();
  setTimeout(() => {
    console.log("async task working~");
    done(false);
  }, 1000);
});
```

异步函数中标记当前任务执行失败的方式是为回调函数指定一个 false 的实参

## 任务失败对任务流影响

```js
grunt.registerTask("default", ["foo", "bad", "bar"]);
```

如果一个任务列表中的某个任务执行失败,则**后续任务默认不会运行**,除非 grunt 运行时指定 --force 参数强制执行 `yarn grunt --force`

## 任务配置选项

```js
module.exports = (grunt) => {
  // grunt.initConfig() 用于为任务添加一些配置选项
  grunt.initConfig({
    // 键一般对应任务的名称
    // 值可以是任意类型的数据
    foo: {
      bar: "baz",
    },
  });

  grunt.registerTask("foo", () => {
    // 任务中可以使用 grunt.config() 获取配置
    console.log(grunt.config("foo"));
    // 如果属性值是对象的话，config 中可以使用点的方式定位对象中属性的值
    console.log(grunt.config("foo.bar"));
  });
};
```

## 多目标模式
```js
module.exports = grunt => {
  //多目标模式，可以让任务根据配置形成多个子任务

  grunt.initConfig({
    build: {
      foo: 100,
      bar: '456'
    }
  })
  
  // yarn grunt build 运行build任务下所有子任务
  // yarn grunt build:foo 可以运行指定目标任务
  grunt.registerMultiTask('build', function () {
    console.log(`task: build, target: ${this.target}, data: ${this.data}`)
  })

  
}

```

## 多目标模式config
```js
grunt.initConfig({
    build: {
      options: {
        msg: 'task options',
        name: 'test'
      },
      foo: {
        options: {
          msg: 'foo target options'
        }
      },
      bar: '456'
    }
  })
// 使用this.options()能获取到配置的options数据
// bar任务中的options会继承build任务中的options
// foo任务因为自定义了options，会覆盖部分build中的数据，类似于merge操作
  grunt.registerMultiTask('build', function () {
    console.log(this.options())
  })
```

## 使用插件
```js
module.exports = grunt => {
  grunt.initConfig({
    clean: {
      temp: 'temp/**'
    }
  })
  
  grunt.loadNpmTasks('grunt-contrib-clean')
}
```
使用 `grunt.loadNpmTasks` 能在配置好了任务参数后直接使用npm上的任务插件
`yarn grunt clean`命令实现将temp目录及其目录下所有文件清除

## 插件使用简化
```js
const sass = require('sass')
const loadGruntTasks = require('load-grunt-tasks')

module.exports = grunt => {
  grunt.initConfig({
    sass: {
      options: {
        sourceMap: true,
        implementation: sass
      },
      main: {
        files: {
          'dist/css/main.css': 'src/scss/main.scss'
        }
      }
    },
    babel: {
      options: {
        sourceMap: true,
        presets: ['@babel/preset-env']
      },
      main: {
        files: {
          'dist/js/app.js': 'src/js/app.js'
        }
      }
    },
    watch: {
      js: {
        files: ['src/js/*.js'],
        tasks: ['babel']
      },
      css: {
        files: ['src/scss/*.scss'],
        tasks: ['sass']
      }
    }
  })

  // grunt.loadNpmTasks('grunt-sass')
  loadGruntTasks(grunt) // 自动加载所有的 grunt 插件中的任务

  // grunt.registerTask('default', ['sass'])
  grunt.registerTask('default', ['sass', 'babel', 'watch'])
}
```
使用`load-grunt-tasks`模块一次性加载所有配置了的任务