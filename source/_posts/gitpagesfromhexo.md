---
title: 利用Hexo管理Github page
date: 2019-05-02 14:45:37
tags: [hexo,git]
---

Github Page从原先乱七八糟的方式整理成Hexo规范，且加入了Next主题。

## ISSUES

### 1. Next版本太老

这个是比较老的，有一年多没有更新了，最新版本是5.1.4
[iissnan/hexo-theme-next](https://github.com/iissnan/hexo-theme-next)
这个比较新，版本是7.1.1
[theme-next/hexo-theme-next](https://github.com/theme-next/hexo-theme-next)

### 2. 安装Next的Fancybox插件
[theme-next-fancybox3](https://github.com/theme-next/theme-next-fancybox3)

### 3. blocked:mixed-content
添加**Google Adsense**依赖加载 *adsbygoogle.js* 的时候，在**gitpage**上出现 *blocked:mixed-content* 的错误。
据查是因为在*https*页面中引入了*http*资源所致。
**google adsense**中嵌入*adsbygoogle.js*的脚本是这样子的
```html
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
```
而我却以为是漏掉了*http*误加了个*http:*在地址前面。
话说*http*什么时候变成这个样子了的。

### 4. 修改menu不生效 [update on 2019-5-10]

突然发现是在next中修改\_config.xml 才生效，想起之前似乎设置了一个覆盖属性为true



### 5. 页面统计

在**next:_config.xml**中，修改 **busuanzi_count** 中的*enable* 属性为 *true* 即可。

### 6. error: RPC failed; curl 56 OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054

这个是 git 的 http 推送的缓冲区太小的问题。

```sh
git config --global http.postBuffer 157286400
```



### Reference:

[Github Pages部署个人博客（Hexo篇）](https://juejin.im/post/5acf02086fb9a028b92d8652#heading-8)