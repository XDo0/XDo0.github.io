---
title: Jekyll博客TeXt搭建过程问题记录
tag: [frontend, github pages]
key: 2022-10-01-Jekyll博客TeXt搭建过程问题记录
---

# 背景

想搭建个个人博客，担心买的服务器可能会跑路，没有选择搭载在云服务器上。
再考虑到hexo等需要本地端要安装，感觉自己应该是轻度使用博客，嫌麻烦。最后选择用Github Page+Jekyll来实现。
这样的话，配置好后只需要git push写好的markdown文件即可。
经过一番物色，最后发现有大佬做好的主题很不错，想要的功能都准备好了。
# 搭建
直接看这篇[快速开始](https://tianqi.name/jekyll-TeXt-theme/docs/zh/quick-start)，不是很稳定，需要你懂得上网。
按照中文的文档搭建即可，我是选择fork后更名为github.io，再git clone下来，这样不用安装Jekyll。
看完这个文档再来看下面的。

# 遇到的问题
## git使用
可以参考我的博客[git简要使用](https://xdo0.github.io/2020/06/07/git%E7%AE%80%E8%A6%81%E4%BD%BF%E7%94%A8.html)

## gittalk
配置好会出现`未找到相关issue进行评论`的问题
* 首先，确保已经为你的username.github,io打开了issue
* 然后在你的blog的gittalk上登陆你的github账号
* 这样以后你在发布好博客预览时就会初始化gittalk  

这可能不是最好的解决方法，如果哪位有较好的解决方法请评论里说一下\^_^
也有大佬写了自动初始化gittalk的脚本，百度搜索就有

### gitalk报错403

> Error: Request failed with status code *403* 

解决：把`_data/variables.yml`的gitalk换成最新版本就可以

```yaml
gitalk:
      js: 'https://unpkg.com/gitalk/dist/gitalk.css'
      css: 'https://unpkg.com/gitalk/dist/gitalk.min.js'
```

Ref:

- [Feature request: custom proxy config for Gitalk to solve 403 error · Issue #348 · kitian616/jekyll-TeXt-theme (github.com)](https://github.com/kitian616/jekyll-TeXt-theme/issues/348)
- [Hexo使用Gitalk评论系统偶现403问题解决 YY的闲时小站 (liuyueyang.top)](https://liuyueyang.top/022111.html#其它的问题)

## leancloud统计点击量为0
按文档配置好后发现每个文章的阅读量一直是0，后来把`_config.yml`的`pageview provider:`的`leancloud`加上双引号就成功了。
## 搜索引擎收录博客
写完博客后使用baidu和google都搜索不到，需要自己提交。
提交网址为：
* [Google](https://search.google.com/search-console?hl=zh-CN)
1.选择网址前缀
2.选择html验证，把html文件下载后放到github.io的repo根目录就行
3.验证成功后选择站点地图（TeXt已经把sitemap插件安装好了），添加`https://username.github.io/sitemap.xml`即可。
sitemap文件就是能够让搜索引擎
* [Baidu](https://ziyuan.baidu.com/dashboard/index)
和google类似，不过据说

## 写个自己的about
直接更改根目录的about.md

## yaml
yaml写在自己每篇博客的markdown文件开头，举个例子
```yaml
---
title:Example
tag: example
key :2020-06-14example
---
```
其中key是任意的只需要每篇文章不同即可，用来区分gittalk的issue和leancloud统计的
真正的时间是看md文件名的

## vscode替换图片
这个博客有个比较大的问题是：图片在构建博客后无法按照相对路径寻址，只能上传图床
### 解决
使用vscode的全局文本替换功能
将`.. /imgsrc`替换为`https://xdo0.github.io/imgsrc`

### p.s.
- 之前尝试用GitHub action替换，试了几个github action market的，效果不好；也尝试[自己写](https://juejin.cn/post/6875857705282568200#heading-7)，发现太麻烦了
- 而且图片在`_posts/imgsrc`下时，无法通过url获得，只能放在`.. /imgsrc`下

## 折叠块代码高亮

## 第一种方法（不推荐）

（不推荐，因为背景色会保持博客整体背景）

这个尝试使用[highlight.js](https://highlightjs.org/)来实现：
在`_layouts\page.html`中添加：
```html
<link rel="stylesheet"
      href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/11.6.0/styles/base16/tomorrow.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/11.6.0/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```
其中default.min.css可以换成其他style的名字，来切换style，在[highlightjs/cdn-release](https://github.com/highlightjs/cdn-release)的目录里查看有哪些style
这样就可以在md文件里，因为highlight.js可以匹配`<pre><code></></>`来高亮代码块
```html
<details><summary>Click to show the Code</summary>
<pre><code class='python'>
import os    
</code></pre>
</details>
```
都选择tomorrow的style效果最好

## 第二种方法（推荐）

利用kramdown解析html块的功能

注意不要使用缩进！！！

代码参考：

```html
<details  markdown="1"><summary>Click to show the Code</summary>
~~~ java
    public class Main {
    public static void main(String[] args) {
        C c = new C();       
    }
}
~~~ 
</details>
```
或者
```html
{::options parse_block_html="true" /}
<details><summary markdown="span">Click to show the Code</summary>
~~~ java
    public class Main {
    public static void main(String[] args) {
        C c = new C();       
    }
}
~~~ 
</details>
{::options parse_block_html="false" /}
```



## 飞书文档转换为markdown
可以参考[这里](https://sspai.com/post/73386)，其中飞书应用申请要看最新的[README](https://github.com/Wsine/feishu2md)

其中要配置config：`C:\Users\XXX\AppData\Roaming\feishu2md\config.json`

## 其他问题
- 代码块高亮要用小写如`json`，`Json`和`JSON`都会导致高亮效果不全



欢迎参观我的博客：[https://xdo0.github.io](https://xdo0.github.io)