## SEO 在 SPA 站点中的实践

### 背景

![](http://with.muyunyun.cn/c03d8772da6d57e47c55044aee364103.jpg)

观察基于 [create-react-doc](https://github.com/MuYunyun/create-react-doc) 搭建的[文档站点](http://muyunyun.cn/create-react-doc/), 发现网页代码光秃秃的一片(见下图)。这显然是单页应用 (SPA) 站点的通病 —— 不利于文档被搜索引擎搜索 (SEO)。

![](http://with.muyunyun.cn/0ba88a7544efe0e1978e6c8d8b7775a6.jpg)

难道基于 React、Vue 等现代化前端框架开发的站点就无法进行 SEO 了么, 那 [Gatsby](https://github.com/gatsbyjs/gatsby)、[nuxt](https://github.com/nuxt/nuxt.js) 等框架又为何能作为不少博主搭建博客的首选方案, 此类框架赋能 SEO 的技术原理是什么呢? 在好奇心的驱动下, 笔者尝试给 [creat-react-doc](https://github.com/MuYunyun/create-react-doc) 赋能 SEO 之旅。

### 搜索引擎优化

在实践之前, 先从理论上分析为何单页应用不能被搜索引擎搜索到。核心在于 `爬虫蜘蛛在执行爬取的过程中, 不会去执行网页中的 JS 逻辑`, 所以`隐藏在 JS 中的跳转逻辑也不会被执行`。

查看当前 SPA 站点打包后的代码, 除了一个根目录 index.html 外, 其它都是注入的 JS 逻辑, 浏览器自然不会对其进行 SEO。

![](http://with.muyunyun.cn/0d15d4e3a62516da7c301e8f1c9228d6.jpg)

此外, 关于搜索引擎详优化是一门相对复杂的学科。如果你对 SEO 优化比较陌生, 建议阅读[搜索引擎优化 (SEO) 新手指南](https://developers.google.com/search/docs/beginner/seo-starter-guide) 一文。 该文给出了全面的 **17 个最佳做法**, 以及 **33 个应避免的做法*, 这也是笔者近期在实践的部分。

### SEO 在 SPA 站点中的实践案例

在轻文档站点的背景前提下, 我们暂不考虑 SSR 方案。

在调研市面上文档站点的 SEO 方案后, 笔者总结为如下四类:

* 静态模板渲染方案
* 404 重定向方案
* SSG 方案
* 预渲染方案

#### 静态模板渲染方案

静态模板渲染方案以 [hexo](https://github.com/hexojs/hexo) 最为典型, 此类框架需要指定特定的模板语言(比如 [pug](https://github.com/pugjs/pug))来开发主题, 从而达到网页内容直出的目的。

#### 404 重定向方案

404 重定向方案的原理主要是利用 GitHub Pages 的 404 机制进行重定向。比较典型的案例有 [spa-github-pages](https://github.com/rafgraph/spa-github-pages)、[sghpa](https://github.com/csuwildcat/sghpa)。

但是遗憾的是 [2019 年 Google 调整了爬虫算法](https://github.com/rafgraph/spa-github-pages#seo), 因此此类重定向方案在当前是无利于 SEO 的。spa-github-pages 作者表示如果需要 SEO 的话, 使用 SSG 方案或者付费方案 [Netlify](https://www.netlify.com/blog/2020/04/07/creating-better-more-predictable-redirect-rules-for-spas/)。

![](http://with.muyunyun.cn/bbb5ed8bce1e0c08dae94df98ff33262.jpg)

#### SSG 方案

SSG 方案全称为 static site generator, 中文可译为`路由静态化方案`。社区上 [nuxt](https://github.com/nuxt/nuxt.js)、[Gatsby](https://github.com/gatsbyjs/gatsby) 等框架赋能 SEO 的技术无一例外可以归类此类 SSG 方案。

以 nuxt 框架为例, 在`约定式路由`的基础上, 其通过执行 nuxt generate 命令将 vue 文件静态化为静态网页。

例子:

```bash
-| pages/
---| about.vue/
---| index.vue/
```

静态化后变成:

```bash
-| dist/
---| about/
-----| index.html
---| index.html
```

经过静态化后的文档目录结构此时可以托管于任何一个静态站点服务商。

#### 预渲染方案

经过上文 SSG 方案的分析, 此时 SPA 站点的优化关键已经跃然纸上 —— `静态化路由`。相对于 nuxt、Gatsby 等框架存在约定式路由的限制, [create-react-doc](https://github.com/MuYunyun/create-react-doc) 的理念是一个`文件即站点`的文档框架, 它在目录结构上更加灵活, 存量 markdown 文档的迁移十分便捷。

以 [blog](https://github.com/MuYunyun/blog) 项目结构为例:

```bash
-| BasicSkill/
---| basic/
-----| DOM.md
-----| HTML5.md
```

静态化后应该变成:

```bash
-| BasicSkill/
---| basic/
-----| DOM
-------| index.html
-----| HTML5
-------| index.html
```

经过调研, 该构思与 [prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin) 预渲染方案一拍即合。prerender-spa-plugin 插件的原理可以见如下图:

![](http://with.muyunyun.cn/f3eb18c162a31fb155fd9f4a364f7fb9.jpg)

至此技术选型定下为使用预渲染方案实现 SSG。

### 预渲染方案实践

create-react-doc 在预渲染方案实践的步骤简单概况如下, 完整改动可见 [mr](https://github.com/MuYunyun/create-react-doc/pull/95/files)。

* 改造 hash 路由为 history 路由。因为 history 路由结构与文档静态化目录结构天然匹配。

```diff
export default function RouterRoot() {
  return (
-    <HashRouter>
+    <BrowserRouter>
      <RoutersContainer />
-    </HashRouter>
+    </BrowserRouter>
  )
}
```

* 在开发环境、生成环境的基础上新增`预渲染环境`, 同时对路由进行环境匹配。其主要解决了`资源文件`与`主域名下的子路径`的对应关系。过程比较曲折, 感兴趣的同学可以见 [issue](https://github.com/chrisvfritz/prerender-spa-plugin/issues/215#issuecomment-415942268)。

```diff
const ifProd = env === 'prod'
+ const ifPrerender = window.__PRERENDER_INJECTED && window.__PRERENDER_INJECTED.prerender
+ const ifAddPrefix = ifProd && !ifPrerender

<Route
  key={item.path}
  exact
-  path={item.path}
+  path={ifAddPrefix ? `/${repo}${item.path}` : item.path}
  render={() => { ... }}
/>
```

* 兼容 prerender-spa-plugin 在 webpack 5 的使用。

官方版本当前未支持 webpack 5, 详见 [issue](https://github.com/chrisvfritz/prerender-spa-plugin/issues/414), 同时笔者存在对预渲染后执行回调的需求。因此当前 fork 了一份[版本](https://github.com/create-react-doc/prerender-spa-plugin) 出来, 解决了以上问题。

### SEO 优化附加 buff, 站点秒开?

SEO 优化至此, 来看下站点优化前后 FP、FCP、LCP 等指标数据的变化。

以 [blog](https://muyunyun.cn/blog) 站点为例, 优化前后的指标数据如下(数据指标统计来自未使用梯子访问 gh-pages):

优化前:

![](http://with.muyunyun.cn/23d56cc42fd778c23d8ed80331334343.jpg)

优化后:

![](http://with.muyunyun.cn/9c551d29943c3d76700782374d86c37b.jpg)

对比优化前后:

* 接入预渲染方案前, 首次绘制(FP、FCP) 的时间节点在 `8s` 左右, LCP 在 17s 左右。
* 接入预渲染方案后, 首次绘制时间节点在 `1s` 之内开始, LCP 在 1.5s 之内。

首屏绘制速度提升了 `8` 倍, 最大内容绘制速度提升 `11` 倍。本想优化 SEO, 结果站点性能优化的方式又 get 了一个。

### 生成站点地图 Sitemap

在完成预渲染实现站点路由静态化后, 距离 SEO 的目标又近了一步。暂且抛开 [SEO 优化细节](https://developers.google.com/search/docs/beginner/seo-starter-guide), 单刀直入 SEO 核心腹地 [站点地图](https://developers.google.com/search/docs/advanced/sitemaps/overview)。

站点地图 Sitemap 格式与各字段含义简单说明如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<urlset>
  <!-- 必填标签, 这是具体某一个链接的定义入口，每一条数据都要用 <url> 和 </url> 包含在里面, 这是必须的 -->
  <url>
    <!-- 必填, URL 链接地址,长度不得超过 256 字节 -->
    <loc>http://www.yoursite.com/yoursite.html</loc>
    <!-- 可以不提交该标签, 用来指定该链接的最后更新时间 -->
    <lastmod>2021-03-06</lastmod>
    <!-- 可以不提交该标签, 用这个标签告诉此链接可能会出现的更新频率 -->
    <changefreq>daily</changefreq>
    <!-- 可以不提交该标签, 用来指定此链接相对于其他链接的优先权比值，此值定于 0.0-1.0 之间 -->
    <priority>0.8</priority>
  </url>
</urlset>
```

> 上述 sitemap 中, lastmod、changefreq、priority 字段对 SEO 没那么重要, 可以见 [how-to-create-a-sitemap](https://ahrefs.com/blog/zh/how-to-create-a-sitemap/)

根据上述结构, 笔者开发了 create-react-doc 的站点地图生成包 [crd-generator-sitemap](https://github.com/MuYunyun/create-react-doc/tree/main/packages/crd-generator-sitemap), 其逻辑就是将预渲染的路由路径拼接成上述格式。

使用方只需在站点根目录的 `config.yml` 添加如下参数便可以在自动化发版过程中自动生成 [sitemap](http://muyunyun.cn/create-react-doc/sitemap.xml)。

```bash
seo:
  google: true
```

将生成的站点地图往 [Google Search Console](https://search.google.com/search-console/sitemaps) 中提交试试吧,

![](http://with.muyunyun.cn/97c21838a1e3310b3c1259e30ab85f3b.jpg)

最后验证下 Google 搜索[站点](https://www.google.com/search?q=site%3Amuyunyun.cn%2Fcreate-react-doc&ie=UTF-8)优化前后效果。

优化前: 只有一条数据。

![](http://with.muyunyun.cn/aea3401e5a31587deb8d93a14f32b011.jpg)

优化后: 含有 sitemap 中指定的 loc 数据。

![](http://with.muyunyun.cn/6df1536366c7d45e0f6418af03a7d948.jpg)

至此使用 SSG 优化 SPA 站点实现 SEO 的完整流程完整实现了一遍。后续便剩下参照 [搜索引擎优化 (SEO) 新手指南](https://developers.google.com/search/docs/beginner/seo-starter-guide) 做一些 SEO 细节方面的优化以及支持更多搜索引擎了。

### 小结

本文从 SPA 站点实现 SEO 作为切入点, 先后介绍了 SEO 的基本原理, SEO 在 SPA 站点中的 4 种实践案例, 并结合 [create-react-doc](https://github.com/MuYunyun/create-react-doc) SPA 框架进行完整的 SEO 实践。

如果本文对您有所帮助, 欢迎 [star](https://github.com/MuYunyun/create-react-doc)、[反馈](https://github.com/MuYunyun/create-react-doc/issues/new)。

### 相关链接

- [google:how-to-create-a-sitemap](https://ahrefs.com/blog/zh/how-to-create-a-sitemap/)