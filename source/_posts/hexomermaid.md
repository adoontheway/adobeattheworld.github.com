---
title: Hexo添加Mermaid支持
date: 2019-05-04 15:06:07
tags: [hexo,mermaid]
author: who else?
description: 对Hexo添加Mermaid支持
---



## Preface

**Mermaid** 是一个支持**Markdown**的流行的js图标库。

让你可以直接在**Markdown**中画流程图，支持流程图，甘特图，序列图等等。

之前的笔记中已经有写了一个流程图，但是一直没有渲染出来。

搁置今日去解决了一下

## Steps

### 1. 安装Mermaid插件

在当前文件夹下面运行如下命令：

```bash
npm i hexo-filter-mermaid-diagrams
```

### 2. 修改 \_config.yml

*theme/hexo-theme-next-7.1.1*下面有一个 *\_config.yml* ，**hexo**支持的样式和配置都在这里找的。

找到*mermaid*相关:

```ini
# Mermaid tag
mermaid:
  enable: true
  # Available themes: default | dark | forest | neutral
  theme: forest
  cdn: //cdn.jsdelivr.net/npm/mermaid@8/dist/mermaid.min.js
  #cdn: //cdnjs.cloudflare.com/ajax/libs/mermaid/8.0.0/mermaid.min.js
```

复制到项目项目的 *\_config.yml* 中去，然后修改为如下：

```ini
mermaid:
  enable: true
  # Available themes: default | dark | forest | neutral
  version: "7.1.2"
  options:
    theme: forest
    startOnLoad: true
  # cdn: //cdn.jsdelivr.net/npm/mermaid@8/dist/mermaid.min.js
  #cdn: //cdnjs.cloudflare.com/ajax/libs/mermaid/8.0.0/mermaid.min.js
```

### 3. 添加Mermaid页面嵌入

进入 *theme/hexo-theme-next-7.1.1/layout/* 中，打开*footer.swig*，在最后面添加如下代码：

```ejs
{% if mermaid.enable %}
  <script src='https://unpkg.com/mermaid@{{ mermaid.version }}/dist/mermaid.min.js'></script>
  <script>
    if (window.mermaid) {
      mermaid.initialize({{ JSON.stringify( mermaid.options) }});
    }
  </script>
{% endif %}
```

### 4. 重启

运行

```bash
hexo g
```

此时，可以在页面上看到图表正常渲染出来了。



## Tips

遇到问题可以查看**Terminal**和浏览器的**console**看是否有异常。



### 其他

Hexo多tags格式是

```markdown
tags: [tag1,tag2]
```

已测

或者

```markdown
tags:
 - tag1
 - tag2
```



未测

## Reference

[Mermaid](https://mermaidjs.github.io/)

[Hexo Mermaid插件](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)