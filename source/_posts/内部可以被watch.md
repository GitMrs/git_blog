---
layout: vue2
title: Vue2内部可以被watch
date: 2024-10-28 15:11:33
cover: "/img/11186.jpg"
mp3: "/music/C400002FDEqn0UGW1N.m4a"
---

在 Vue 2.x 中，使用 inject 注入的值默认情况下是不能被 watch 直接监控到的，因为 inject 提供的值不是响应式的。如果想通过 watch 监控 inject 注入的值，需要使用 Vue 的 computed 计算属性来监控。

# 解决方法

- 父组件定义说明

```js
export default {
  data: () => {
    return {
      activeTab: 0,
    };
  },
  provide() {
    return {
      container: this,
    };
  },
  methods: {
    getActiveTab() {
      return this.activeTab;
    },
    changeActiveTab(index) {
      this.activeTab = index;
    },
  },
};
```

- 子组件使用

```js
export default {
  inject: ["container"],
  computed: {
    activeTab() {
      return this.container.getActiveTab();
    },
  },
  watch: {
    activeTab(val) {
      console.log(val);
    },
    "container.activeTab"(val) {
      console.log(val);
    },
  },
};
```
