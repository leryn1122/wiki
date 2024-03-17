---
id: frontend.reactive
tags:
- frontend
title: "\u54CD\u5E94\u5F0F\u5E03\u5C40"

---


# 响应式布局
参考文档：

- [Responsive Mobile Navigation Menu - Methods and Solutions](https://www.queness.com/post/13093/responsive-mobile-navigation-menumethods-and-solutions)


## 导航布局
首先可以通过监听原生 Window 的视宽，根据视宽的大小来判断是否是 PC 端或移动端，不同设备采用不同的布局：

- PC 端可以使用侧边、顶部导航或者混合导航
- 移动端可以使用打开画出侧边目录导航栏
- 对于平板设备，可以视为较宽手机端、也可以视为平板端再额外维护一套布局

以 Vue 3 举例：
```vue
<script lang="ts" setup>
import { computed, defineComponent, onBeforeMount, onMounted, ref } from 'vue';
import { LayoutType, MOBILE_DEVICE_WIDTH_THRESHOLD } from './layout';

import AsideDrawerLayout from './aside-drawer-layout.vue';
import TopMixedLayout from './top-mixed-layout.vue';

const screenWidth = ref<number>(window.innerWidth);

const captureScreenWidth = () => {
  screenWidth.value = window.innerWidth;
};

onMounted(() => {
  window.addEventListener('resize', captureScreenWidth);
});

onBeforeMount(() => {
  window.removeEventListener('resize', captureScreenWidth);
});

// Determine whether it is PC or mobile device depending on the window screen width.
const layout = computed<ReturnType<typeof defineComponent>>(() => {
  let layoutType = ((width: number) => {
    return width < MOBILE_DEVICE_WIDTH_THRESHOLD ? LayoutType.ASIDE_DRAW : LayoutType.TOP_MIXED;
  })(screenWidth.value);

  switch (layoutType) {
    case LayoutType.TOP_MIXED:
      return TopMixedLayout;
    case LayoutType.ASIDE_DRAW:
      return AsideDrawerLayout;
  }
});
</script>

<template>
  <component :is="layout" v-bind="layout">
    <template #[item]="data" v-for="item in Object.keys($slots)" :key="item">
      <slot :name="item" v-bind="data || {}" />
    </template>
  </component>
</template>
```


## CSS
HTML5 引入了一种方法，使 Web 设计者可以通过 `<meta>` 标签来控制视口。
您应该在所有网页中包含以下 `<meta>` 视口元素：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```
它为浏览器提供了关于如何控制页面尺寸和缩放比例的指令。`width=device-width` 部分将页面的宽度设置为跟随设备的屏幕宽度（视设备而定）。当浏览器首次加载页面时，`initial-scale=1.0` 部分设置初始缩放级别。
经典的设备分辨率断点：
```css
/* 超小型设备（电话，600px 及以下） */
@media only screen and (max-width: 600px) {...} 

/* 小型设备（纵向平板电脑和大型手机，600 像素及以上） */
@media only screen and (min-width: 600px) {...} 

/* 中型设备（横向平板电脑，768 像素及以上） */
@media only screen and (min-width: 768px) {...} 

/* 大型设备（笔记本电脑/台式机，992px 及以上） */
@media only screen and (min-width: 992px) {...} 

/* 超大型设备（大型笔记本电脑和台式机，1200px 及以上） */
@media only screen and (min-width: 1200px) {...}
```
