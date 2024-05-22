---
id: frontend.vue3
tags:
- frontend
- vue
- vue3
title: Vue.js

---


# Vue.js




### 数据代理
数据代理是响应式的基础。
Vue 通过 JS 的 API `Object.defineProperty()`，将属性添加到对象中。这样读取数据的时候，都会通过 getter 和 setter 获得代理后的数据。



### 设置属性
```typescript
defineProperty({
  name: {
    type: String,
    default: "",
  }
})
```
