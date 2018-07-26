---
title: Hexo 渲染思维导图
date: 2018-07-24 21:45:52
categories:
- 工具
tags:
- 脑图
- kityminder
---
## 简洁版

最终修改看这两个[commit](https://github.com/hargao/hexo-theme-next/commit/0b971cd6bd0f9f99178c65c4483161e26da1a1d1), [commit](https://github.com/hargao/hexo-theme-next/commit/569ec0de7414da1a8b1886c9c41474bbe2a45002)

<!-- more -->

## 背景

引擎: hexo
主题: next

## 动手

### JS

三个 js 放入目录 themes\next\source\js\src
- kity.min.js
- kityminder.core.min.js
- mindmap.js (reference 1 抄的)

编辑 themes\next\layout\_scripts\commons.swig, 把上面三个加进 js_commons

### CSS
css 放在 themes\next\source\css\_custom, 并且在 custom.styl import 一下
- kityminder.core.css 改名 kityminder.core.styl

## Debug

遇到了问题, 填入 XMind 导出的 markdown

```
{% pullquote mindmap %}
# 中心主题
## 分支主题 1
### 子主题 1
### 子主题 2
## 分支主题 2
### 子主题 1
## 分支主题 4
### 子主题 1
### 子主题 2
### 子主题 3
### 子主题 4
## 分支主题 3
{% endpullquote %}
```

出错
```
Uncaught TypeError: Cannot read property 'data' of undefined
```

hexo 会按照 markdown 渲染一遍, 把"#"渲染掉, 导致`$('.mindmap').text()` 得到如下字符串
`"中心主题分支主题 1子主题 1子主题 2分支主题 2子主题 1分支主题 4"`
kityminder 按照 markdown importData, 理所当然的报错了

尝试了`raw` 标签嵌套 `pullquote`. 未解决问题

## 自定义标签

搜了一遍, 没有合适的方案。只能自己写个不渲染又可以加 class name 的标签

进入 /themes/next/scripts/tags, 新建文件, 照着 note.js 和原版tag pullquote.js 抄一个raw-class, 去掉render markdown的部分

```
{% rawclass mindmap %}
# 中心主题
## 分支主题 1
### 子主题 1
## 分支主题 4
## 分支主题 2
### 子主题 1
### 子主题 2
### 子主题 3
## 分支主题 3
{% endrawclass %}
```

测试发现需要增加 `$('.mindmap').text('')` 消除原文字.

## 最终效果

{% rawclass mindmap mindmap-md%}
# 中心主题
## 分支主题 1
### 子主题 1
## 分支主题 4
## 分支主题 2
### 子主题 1
### 子主题 2
### 子主题 3
## 分支主题 3
{% endrawclass %}

对比一下 XMind

{% asset_img XMind.png xmind %}

## 问题
1. 显示不全。 后续还得调整下大小和样式, 或者优化交互， 或者换其他渲染器比如markmap
2. 如果能把思维导图数据作为文件引入会更好
3. 一个页面只能渲染一个思维导图. 能改, 后续调整

## 后续

改完正开心, 发现除了这一篇, 其他页面打开都是空白的...包括首页, 也是空白. 调试发现东西好好的都在, 只是不显示??
果然是mindmap.js出错了, 加一个`$('.mindmap').length`的判断吧

## 后续二

想了想还是改改样式。 kityminder 文档有改[layout](https://github.com/fex-team/kityminder-core/wiki/command#layout)和[theme](https://github.com/fex-team/kityminder-core/wiki/command#theme)的API, 可是试了不生效
不死心查源代码, 发现`useTemplate`, `useLayout` 两个方法, 在浏览器控制台下执行可生效. 可是写在 mindmap.js 又不行
看起来是 svg 加载延迟的问题. 暂时 setTimeout 解决. 目测要改成回调.

后续二[commit](https://github.com/hargao/hexo-theme-next/commit/569ec0de7414da1a8b1886c9c41474bbe2a45002)

-------
References:
[1. Hexo中使用markdown来绘制脑图](https://qsli.github.io/2017/01/01/markdown-mindmap/)
[2. 在 Hexo 中使用思维导图](https://hunterx.xyz/use-mindmap-in-hexo.html)
[3. kityminder.core.js](https://github.com/fex-team/kityminder-core/blob/dev/dist/kityminder.core.js)
