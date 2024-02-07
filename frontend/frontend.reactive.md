
# 响应式布局
参考文档：

- [https://www.queness.com/post/13093/responsive-mobile-navigation-menumethods-and-solutions](https://www.queness.com/post/13093/responsive-mobile-navigation-menumethods-and-solutions)

HTML5 引入了一种方法，使 Web 设计者可以通过 `<meta>` 标签来控制视口。<br />您应该在所有网页中包含以下 `<meta>` 视口元素：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```
它为浏览器提供了关于如何控制页面尺寸和缩放比例的指令。<br />`width=device-width` 部分将页面的宽度设置为跟随设备的屏幕宽度（视设备而定）。<br />当浏览器首次加载页面时，`initial-scale=1.0` 部分设置初始缩放级别。


经典的设备分辨率断点。
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
