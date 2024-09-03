---
id: frontend.vite
tags:
- frontend
- vite
title: Vite

---
# Vite
参考文档：

+ [Vite 参数配置](https://my.oschina.net/rc6688/blog/5178428)

Vite 是尤雨溪主导开发的一个构建工具，底层基于 SWC 和 Rollup。

## Vite 打包优化
Vite 打包会提示包过大，这会导致网页加载过慢。以 Ant-Design 为例，全局导入后 js 约 1.3M，css 约 600K，需要 90 秒才能完全加载。

> (!) Some chunks are larger than 500 KiB after minification. Consider:
>
> + Using dynamic import() to code-split the application
> + Use build.rollupOptions.output.manualChunks to improve chunking: [https://rollupjs.org/guide/en/#outputmanualchunks](https://rollupjs.org/guide/en/#outputmanualchunks)
> + Adjust chunk size limit for this warning via build.chunkSizeWarningLimit.
>

### 调整 Rollup Options
查看 [Rollup Options 文档](https://rollupjs.org/guide/en/#outputmanualchunks)，这一条对 Webpack 等打包工具也适用。

```typescript
// 仅供参考
export default defineConfig({
  // 略
  build: {
    minify: true,
    manifest: true,
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            // pnpm因为目录结构形式有变化, 导致文件名的裁剪方式也与传统npm不同
            // cnpm和yarn的用这个分割方案
            // return id.toString().split('node_modules/')[1].split('/')[0].toString();
            let segment = id.toString().split('node_modules/')[2].split('/');
            if (['ant-design-vue'].includes(segment[0])) {
              return segment[0] + '+' + segment[1];
            } else {
              return segment[0];
            }
          }
          return 'vendor';
        },
      },
    },
  },
});
```

### nginx 开启 gzip
许多浏览器都支持 gzip，建议 nginx 开启 gzip 模式传输静态文件，可以提高页面加载速度，减小网络传输。docker 镜像中带上开启 gzip 的 nginx 配置文件即可。

目前从实操下来，文件传输比最高接近 20%，大部分情况在 35%~50%。

### Dockerfile
这项优化与打包的静态文件大小无关，但可以显著减小前端镜像的大小。

Dockerfile 用底镜像选择小巧 `node:slim` 的镜像，在其中编译后，将编译好的静态文件和 nginx 配置中 copy 到 nginx 镜像中。可以显著减小镜像体积，最终大约之比 nginx 镜像大约十几到几十M。

