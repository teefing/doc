---
category: 专业知识
tags:
  - Vue
date: 2021-03-29
title: 自己动手实现一个vue-router
vssue-title: 自己动手实现一个vue-router
---

```js
let _Vue = null;
export default class VueRouter {
  static install(Vue) {
    // 缓存
    if (VueRouter.install.installed) return;
    VueRouter.install.installed = true;
    _Vue = Vue;

    //
    Vue.mixin({
      beforeCreate() {
        if (this.$options.router) {
          // 因为只有根节点的$options中存在router，所以全局只会执行一次
          Vue.prototype.$router = this.$options.router;
        }
      },
    });
  }

  constructor(options) {
    this.options = options;
    this.routeMap = {};
    this.data = _Vue.observable({
      current: window.location.pathname,
    });
    this.init();
  }

  init() {
    // 创建routeMap
    this.createRouteMap();
    // 定义全局router-link router-view组件
    this.initComponent();
    // 处理popstate事件
    this.initEvent();
  }

  createRouteMap() {
    // 从配置中得到path -> component映射关系
    this.options.routes.forEach((route) => {
      this.routeMap[route.path] = route.component;
    });
  }

  initComponent() {
    _Vue.component("router-link", {
      props: {
        to: String,
      },
      methods: {
        clickHandler(e) {
          // 修改导航栏的路径
          history.pushState({}, "", this.to);
          // 渲染对应的组件（因为vue的响应式，所以修改数据能带动组件渲染）
          this.$router.data.current = this.to;
          // 点击a标签，防止页面自动刷新
          e.preventDefault();
        },
      },
      render(createElement) {
        return createElement(
          "a",
          {
            attrs: {
              href: this.to,
            },
            on: {
              click: this.clickHandler,
            },
          },
          [this.$slots.default]
        );
      },
    });

    const _this = this;
    _Vue.component("router-view", {
      render(createElement) {
        // 得到当前路径对应的组件
        const component = _this.routeMap[_this.data.current];
        // 渲染组件
        return createElement(component);
      },
    });
  }

  initEvent() {
    // 对浏览器前进后退做出响应
    window.addEventListener("popstate", () => {
      this.data.current = window.location.pathname;
    });
  }
}
```
