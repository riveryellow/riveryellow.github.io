---
title: Markdown基本用法
date: 2017-09-05 20:57:49
cdn: "header-off"
header-img: "./img/markdown.jpeg"
tags:
	- Markdown
	- Hexo
---

>  Markdown是一种轻量级的**标记语言**，相对于复杂的HTML标记语言来说，常用的标记符号不超过十个。
>  可以很方便地导出为PDF、HTML格式文件，使得使用者可以专心于码字，而无需花大量精力在排版上。

# 常用标签

## 段落
一个段落是由一个以上的连接的行句组成，而一个以上的空行则会划分出不同的段落（空行的定义是显示上看起来像是空行，就被视为空行，例如有一行**只有空白和 tab**，那该行也会被**视为空行**），**一般的段落不需要用空白或换行缩进**。

## Header — 标题
Markdown 支持两种标题的语法，Setext 和 atx 形式。
* Setext 形式是用底线的形式，利用 = （最高阶标题）和 - （第二阶标题）
* Atx 形式在行首插入 1 到 6 个 # ，对应到标题 1 到 6 阶。

### 1. markdown header写法
``` markdown
一级标题 or # 一级标题
=======
二级标题 or ## 二级标题
-------
### 三级标题
#### 四级标题 ...
```
### 2. HTML输出代码
``` html
<h1>一级标题</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>
```

Blockquote — 块引用
---- 
### 1. 段落内容展示
> This is a blockquote.
> ps.段落中也可以增加标题，标题级数不相对继承
> #### H4 in blockquote

### 2. markdown blockquote写法
``` markdown
> This is a blockquote.
>
> ps.段落中也可以增加标题，标题级数不相对继承
>#### H4 in blockquote
```
### 3. HTML输出代码
``` html
<blockquote>
  <p>This is a blockquote.<br>ps.段落中也可以增加标题，标题级数不相对继承</p>
  <h4>H4 in blockquote</h4>
</blockquote>
```

Highlight — 代码高亮
---- 
### 1. markdown highlight写法
```
```+语言名（比如java）
//代码内容
``` 
```
### 2. highlight展示
``` java
public static void main(String[] args){
    System.out.println("abc")
}
```
支持语言参考：http://highlightjs.readthedocs.io/en/latest/css-classes-reference.html


修辞和强调
---- 
Markdown 使用星号和底线来标记需要强调的区段。
### 1. 斜体&加粗
Some of these words *are emphasized*.
Some of these words _are emphasized also_.
Use two asterisks for **strong emphasis**.
Or, if you prefer, __use two underscores instead__.
### 2. markdown写法
``` markdown
Some of these words *are emphasized*.
Some of these words _are emphasized also_.
Use two asterisks for **strong emphasis**.
Or, if you prefer, __use two underscores instead__.
```
### 3. HTML输出代码
``` html
<p>Some of these words <em>are emphasized</em>.
Some of these words <em>are emphasized also</em>.</p>
<p>Use two asterisks for <strong>strong emphasis</strong>.
Or, if you prefer, <strong>use two underscores instead</strong>.</p>
```

List Item — 列表项
---- 
无序列表使用星号、加号和减号来做为列表的项目标记，这些符号是都可以使用的(效果相同)，例如：
* Candy.
* Gum.
* Booze.

### 1. markdown列表写法
使用星号：
```
* Candy.
* Gum.
* Booze.
```
加号：
```
+ Candy.
+ Gum.
+ Booze.
```
减号：
```
- Candy.
- Gum.
- Booze.
```
### 2. HTML输出代码
``` html
<ul>
  <li>Candy.</li>
  <li>Gum.</li>
  <li>Booze.</li>
</ul>
```
### 3. 切记！
列表元素下一行如果要跟标题(header)，\_\_一定要空行\_\_，不然标题会被包在`<li>`标签里面，页面展示会产生大块空白。

Link — 链接
---- 
### 行内链接
This is an [example link][1].

#### 1. markdown 行内链接写法
``` markdown
This is an [example link](http://example.com/).
```

#### 2. HTML输出代码
``` html
<p>This is an <a href="http://example.com/" title="With a Title">
example link</a>.</p>
```

### 参考链接
参考形式的链接让你可以为链接定一个名称，之后你可以在文件的其他地方定义该链接的内容：
I get 10 times more traffic from [Google][2] than from
[Yahoo][3] or [MSN][4].

#### 1. markdown 参考链接写法
``` markdown
I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"
```

#### 2. HTML输出代码
``` html
<p>I get 10 times more traffic from <a href="http://google.com/"
title="Google">Google</a> than from <a href="http://search.yahoo.com/"
title="Yahoo Search">Yahoo</a> or <a href="http://search.msn.com/"
title="MSN Search">MSN</a>.</p>
```

Image — 图片
---- 
### 行内形式
``` markdown
![alt text](/path/to/img.jpg "Title")
//title非必填项
```

### 参考形式
``` markdown
![alt text][id]

[id]: /path/to/img.jpg "Title"
```

以上两种形式的HTML输出代码均为：
``` html
<img src="/path/to/img.jpg" alt="alt text" title="Title" />
```

插入HTML标签
---- 
在一般的段落文字中，可以使用反引号 \` 来标记代码区段，区段内的 &、\< 和 \> 都会被自动的转换成 HTML 实体，这项特性让你可以很容易的在代码区段内插入 HTML 码：


I strongly recommend against using any `<blink>` tags.

I wish SmartyPants used named entities like `&mdash;`
instead of decimal-encoded entites like `&#8212;`.

[1]:	http://example.com/
[2]:	http://google.com/ "Google"
[3]:	http://search.yahoo.com/ "Yahoo Search"
[4]:	http://search.msn.com/ "MSN Search"

# Markdown编辑器
## Mou
![](img/mou.png)
  由于macdown暂时不支持10.12.2 Sierra版本，于是放弃了。

## Macdown
![](img/macdown.png)
![](img/macdowndetail.png)
  说起来 Macdown 和 Mou 还是有一段鲜为人知的虐恋史的，有兴趣可以baidu一下。
  Macdown的操作界面分为左右两部分，分别是markdown和渲染后的预览，实时更新，体验非常友好。

## Ulysses
![](img/ulysses.png)
  功能很强大，不过由于Macdown就已经能满足我平时写写blog的需求，就没做太多研究。