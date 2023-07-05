---
layout: post
title: Typora 官方 Markdown 教程翻译
date: 2023-05-13 14:44:45 +0800
categories: [markdown, typora]
tags: [typora]
author: 
---



>  转载自：
>
> [Typora 官方 Markdown  教程翻译](https://www.cnblogs.com/shuoliuchina/p/11461099.html)

## 概述

## 块级元素

###  段落和换行符

段落是简单的一行或多行连续的文本。在markdown的源代码中，段落由两个或两个以上的空行分隔开。在Typora中，你只需要一个空行（按一次`回车`键）就可以创建一个新的段落。

按下`Shift` + `回车`创建一个简单换行符。大多数其他的markdown语法分析程序会无视简单换行符。 为了让其他的markdown语法分析程序也能识别你的换行符，你可以在段落末尾留下**两个空格**或者插入`<br/>`。

### 标题

标题通过在段落开头插入1-6个井字符（`#`）实现，不同数量的井字符分别代表1-6级标题。例如：

```markdown
# 这是一级标题

## 这是二级标题

###### 这是六级标题
```

### 引文区块

Markdown使用电子邮件风格的“>”符号来表示引文区块。它们这样表示：

```markdown
> 这是一个包含两个段落的引文区块，这里是第一段。
>
> 这里是第二段。Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.



> 这是另外一个只有一个段落的引文区块。相邻的引文区块之间由三个空行分隔开来。
```

### 列表

输入`* 列表项目 1` 会创建一个无序列表 - `*`符号可以由`+`或者`-`代替。

输入`1. 列表项目 1`会创建一个有序列表 - 它们的markdown源代码如下所示：

```markdown
## 无序列表
*   红
*   绿
*   蓝

## 有序列表
1.  红
2. 	绿
3.	蓝
```

### 任务列表

任务列表由[ ]或[x]表示（未完成或已完成）。例如：

```markdown
- [ ] 一个任务列表项目
- [ ] 需要列表语法
- [ ] 正常的 **有格式的**，@提及的，#1234 引用的
- [ ] 未完成的
- [x] 已完成的
```

你可以通过点击项目前面的复选框来改变它的完成/未完成状态。

### （隔绝的）代码区块

Typora仅支持GitHub风格的Markdown隔绝区块，不支持Markdown中原始的代码区块。

插入隔绝区块非常容易：输入```，然后按下`回车`键即可。在```后面插入支持的编程语言标识符，相应的语法强调就会自动显示出来：

~~~ruby
这有一个例子：

```
function test() {
  console.log("notice the blank line before this function?");
}
```

语法强调：
```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```
~~~

### 数学区块

通过使用**MathJax**，你可以实现*LaTeX*数学表达式。

输入`$$`并按下`回车`键即可创建数学表达式。这将会触发一个可以识别*Tex/LaTex*源代码的输入区。例如：
$$
c^2= a^2 + b^2
$$



在markdown源文件中，数学区块是一个被一对`$$`符号包裹的*LaTeX*表达式：

```java
$$
\mathbf{V}_1 \times \mathbf{V}_2 = 
\begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

$$
\mathbf{V}_1 \times \mathbf{V}_2 = 
\begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$



你可以从[这里](http://support.typora.io/Math/)找到更多细节。

### 表格

输入`| 第一个标题 | 第二个标题 |`然后按下`回车`键，就创建出一个两列的表格。

| 商品 | 价格   |
| ---- | ------ |
| PC   | 1000￥ |

创建表格的完整语法在下面标识出来了。但是没有必要记住这个愈发的细节，因为在Typora中，表格的markdown源代码已经被自动创建好了。

表格的markdown源代码长这个样子：

```markdown
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
```

你也可以加入行内的Markdown标记，例如链接、加粗、斜体和删除线。

最后，通过在标题行中加入冒号（`:`），可以实现相应列的左对齐、右对齐或居中对齐。

```markdown
| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```

把冒号放在最左边表示这一列左对齐；冒号放在最右边表示这一列右对齐；把两边都加上冒号表示这一列居中对齐。

### 脚注

```markdown
你可以像这样创建脚注[^脚注].

[^脚注]: 这里是 **脚注**的*文本*.
```

你可以像这样创建脚注[^脚注].

[^脚注]: 这里是 **脚注**的*文本*.

### 水平线

在空白行中输入 `***` 或`---` 然后按下 `回车` 会自动生成一条水平线。

---

### YAML 前言

Typora 如今支持[YAMLFront Matter](http://jekyllrb.com/docs/frontmatter/)。在文本的最顶端输入`---`并按下`回车`来引入一个元数据块。除此之外，你也可以从Typora的菜单栏中，依次点击`段落`、`YAML Front Matter`插入YAML Front Matter元数据块。

### 目录

输入`[toc]`并按下`回车`键创建目录。目录会提取文档中所有的标题。当你在文档中增加内容时，目录也会自动更新。

## 行内元素

行内元素会在输入完成后立刻被语法分析并相应。把鼠标光标点击到这些行内元素的中部会让它们展开为markdown源代码。下面是每一个行内元素的语法介绍。

### 链接

Markdown支持两种类型的链接：**行内链接**和**引用链接**。

这两种连接的文本都装在[方括号]中。

在连接文本封闭的方括号后面直接使用一个圆括号就可以创建一个行内链接。把你想要指向的链接放到圆括号中。链接可以选择性地附加title属性，title内容用引号标记。例如：

```markdown
这是一个[行内链接](http://example.com/ "Title")的例子。

[这个链接](http://example.net/)没有title属性。
```

运行出来的结果就是：

这是一个[行内链接](http://example.com/ "Title")的例子。

[这个链接](http://example.net/)没有title属性。

#### 行内链接

**你可以把标题设置成为链接**，这样的话当你点击连接之后就可以直接跳到标记位置。例如：

Command（在Windows中：Ctrl）+ 点击[这个链接](#块级元素)就会直接跳到`块级元素`。想知道这是怎么实现的，请把鼠标光标点击到链接上面，展开源代码。

#### 引用链接

引用风格的链接把你选择的标签放到另一个方括号中来识别链接：

```java
这是[引用链接][id]的一个例子。

然后，在文档中的任何一个位置，你可以把你的链接标记成这个样子：

[id]: http://example.com/  "可以在这里输入Title属性"
```

在Typora中，他们先是出来的结果是这个样子的：

这是[引用链接](http://example.com/)的一个例子。

内置的链接名快捷方式允许你省略连接名称。在这种情况下，连接文本本身被用作为链接名。只需要使用一对空的方括号——例如，把词语“谷歌”连接到`google.com`网站，你是需要简单地写下下面的这几行代码：

```markdown
[Google][]
然后定义链接：

[Google]: http://google.com/
```

在Typora中，点击链接会展开为可编辑形式，ctrl + 鼠标左键（Mac中为 command + 鼠标左键）会在你的浏览器中打开超链接。

### URLs

Typora允许你把URL（统一资源定位器，也就是网址）插入为连接形式，只需把链接装到`<`尖括号`>`里面。

`<i@typora.io>`就会显示成i@typora.io>.

Typora也会自动把标准的URL转化为链接，比如：www.google.com。

### 图片

图片的语法格式和链接非常相似，只是它们需要在链接起始的地方加一个`!`字符。插入图片的语法长这样：

```markdown
![Alt text](/path/to/img.jpg)

![Alt text](/path/to/img.jpg "Optional title")
```

你也能通过拖拽的方式把文件或者网页中的图片插入进来。点击图片之后，你就能修改它的markdown源代码。如果你拖拽的图片在你正在编辑的文件的同一级或次一级目录中，将会使用相对路径。

如果你正在使用markdown搭建网站，你需要在最开头的YAML Front Matters中加入`typora-root-url`属性，声明一个在你的本地电脑中预览图片的URL前缀。比如，在YAML Front Matters中输入`typora-root-url:/User/Abner/Website/typora.io/`，那么`![alt](/blog/img/test.png)`在Typora中就会被识别为`![alt](file:///User/Abner/Website/typora.io/blog/img/test.png)`。

你可以在[这里](https://support.typora.io/Images/)找到更多细节。

### 强调

Markdown把星号（`*`）和下划线（`_`）识别为强调的标志。被一对`*` 或`_`包裹的文本会被HTML的`<em>`标签包裹。例如：

```markdown
*single asterisks*

_single underscores_
```

输出：

*single asterisks*

_single underscores_

GFM会忽略文字中的下划线，这种情况在代码和名字中非常常见，就像这样：

>wow_great_stuff
>
>do_this_and_do_that_and_another_thing.

要想在可能会被视为强调定义符的位置生成一个能显示的星号或下划线，你可以用反斜杠来避免：

```java
\*这段文字处在可以显示的星号中\*
```

Typora建议使用`*`符号表示强调。

### 加粗

两个`*`或`_`能让包裹其中的内容包裹在一个HTML `<strong>`标签中。例如：

```markdown
**double asterisks**

__double underscores__
```

输出：

**double asterisks**

__double underscores__

Typora建议使用`**`符号。

### 代码

若想在行内显示代码，直接把它放在一对反引号（`）中。跟预先格式化的代码块不同的是，行内代码会显示在一个普通的段落中。例如：

```java
使用`printf()`函数。
```

结果是这个样子的：

使用`printf()`函数。

### 下划线

Underline由HTML语言实现。

`<u>下划线</u>`就成了<u>下划线</u>.

### Emoji 😄

使用语法`:smile:`输入Emoji小表情。

用户可以通过按`ESA`键触发emoji的自动补全建议，也可以通过设置`偏好设置`来自动触发建议。同时，也可以在菜单栏中的`编辑`->`Emoji & Symbols`直接插入UTF-8字符（仅限于macOS）。

### 行内Math公式

如果想要使用这个特性，请先在`偏好设置`->`markdown`标签中启用它。然后，使用一对`$`包裹一个TeX命令。例如：`$\lim_{x \to \infty} \exp(-x) = 0$`就会被转换成LaTeX命令。

若想在编辑过程中预览行内数学：输入“$”，然后按下`ESC`键，接着输入TeX命令。

你可以在[这里](http://support.typora.io/Math/)找到更多细节。

### 下标

如果想要使用这个特性，请先在`偏好设置`->`markdown`标签中启用它。然后，使用一对`~`包裹一个TeX命令。例如：`H~2~O`, `X~long\ text~`

H~2~O 

 X~long\ text~

### 上标

如果想要使用这个特性，请先在`偏好设置`->`markdown`标签中启用它。然后，使用一对`^`包裹一个TeX命令。例如：`X^2^`.

a^2^ = b^2^ + c^2^

### Highlight

如果想要使用这个特性，请先在`偏好设置`->`markdown`标签中启用它。然后，使用一对`==`包裹一个TeX命令。例如：`==highlight==`.

==highlight==

## HTML

你可以使用一些原版Markdown不支持的HTML样式内容。比如，使用`<span style="color:red">this text is red</span>`设置文本为红色。

<span style="color:red">this text is red</span>

### 嵌套内容

一些网站提供的基于iframe的嵌套代码你也可以直接粘贴到Typora中。例如：

```java
<iframe height='265' scrolling='no' title='Fancy Animated SVG Menu' src='http://codepen.io/jeangontijo/embed/OxVywj/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>
</iframe>
```

<iframe 
        height='265' 
        scrolling='no' 
        title='Fancy Animated SVG Menu'
        src='http://codepen.io/jeangontijo/embed/OxVywj/?height=265&theme-id=0&default-tab=css,result&embed-version=2' 
        frameborder='no' 
        allowtransparency='true' 
        allowfullscreen='true' 
        style='width: 100%;'>
</iframe>

### 视频

你可以用HTML标签`<video>`来嵌入视频。例如：

```markdown
<video src="xxx.mp4" />
```

### 其他的HTML支持

你可以在[这里](http://support.typora.io/HTML/)找到更多细节

