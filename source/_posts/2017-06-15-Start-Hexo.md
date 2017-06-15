---
layout: blog
title: 开始使用Hexo
date: 2017-06-15 19:11:21
tags:
---

在朋友的推荐下尝试了一些`Hexo` + `Next`，使用后感觉真的很简单，并且`Next`效果很棒，最终切换为`Hexo` + `Next`。

切换过程遇到了一些问题，做一个记录，分享出来。

<!-- more -->

# 安装
安装过程遇到2个问题：
1. 我是在`Mac`上使用刚刚发布的`Node.js V8.0`，结果在安装过程中发现依赖包`fsevents`包V1.1.1不支持`Node.js V8`，好在等了几天 V1.1.2解决了这个问题。然后按照[官方文档](https://hexo.io/zh-cn/docs)进行安装。
2. 我使用了npm 淘宝镜像（参考[常用国内源汇总](/2017/03/19/china-source)), 结果发现部分包按照失败，指定官方源解决：
```npm
npm i hexo-cli -g --registry=https://registry.npmjs.org/

npm i --registry=https://registry.npmjs.org/
```

# 404
将404.html放在`themes/next/source/404.html`，注意，这个配置在本地调整不生效，`deploy`后生效。

# CDN配置
参考[Next进价设定](http://theme-next.iissnan.com/advanced-settings.html)修改国内CDN，以提高网页访问速度。

# SVG问题解决
之前使用`Jekyll`写过一篇"[用SVG实现一个半圆形仪表盘](/2017/03/11/svg-circle)"的文章，其中显示SVG部分直接写`<svg>`标签就可以，更换之后发现svg无法显示。修改如下：
```html
{% raw %}
<svg>
...
</svg>
{% endraw %}
```

# Reveal支持
以前做了一些`Keynote`，并在`Blog`中进行展示，这个我还是希望能支持。没有找到很好的办法，只能修改`Next`源码。

首先，我的`Markdown`文件中的`Front-matter`中增加一个描述，`key`为`iframe`，值为`Keynote`地址。
例如：

```markdown
iframe:     "https://qiaolb.github.io/presentation/brief-history-time.html"
```

然后修改`themes/next/layout/_macro/post.swig`，在`header`最后增加如下内容：
```html
        {% if page.iframe %}
          <iframe src="{{page.iframe}}"></iframe>
        {% endif %}
```

这样就可以在header区域显示出`Keynote`。接下来调整样式，在`themes/next/source/css/_custom/custom.styl`，增加以下内容：

```css
.post-header iframe {
  width: 100%;
  height: 450px;
  border: 0;
}

@media (max-width: 767px) {
  .post-header iframe { height: 200px; }
}
```

一切就绪，所有以前的内容都可以正常显示了。

# Search

我配置了`Local Search`，这个比较简单。首先需要安装模块：

```npm
npm install hexo-generator-search --save
npm install hexo-generator-searchdb --save
```

然后打开`主题设置文件`（`themes/next/_config.yml`），打开`Local Search`功能：
```
local_search:
  enable: true
```



还有很多好玩的配置，以后慢慢研究。
