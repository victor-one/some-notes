# HTML 教程

---

## HTML 教程

**注意**：对于中文网页需要使用**\<meta charset="utf-8">**声明编码，否则会出现乱码。有些浏览器会设置**GBK**为默认编码，则需要设置为**\<meta charset="gbk">**。

HTML文档的后缀名为：.html、.htm，以上两种后缀名没有区别。

---

## HTML 简介

![完整HTML页面](https://www.runoob.com/wp-content/uploads/2013/06/02A7DD95-22B4-4FB9-B994-DDB5393F7F03.jpg "完整HTML页面")

- **\<!DOCTYPE html>** 声明为 HTML5 文档
- **\<html>** 元素是 HTML 页面的根元素
- **\<head>** 元素包含了文档的元（meta）数据，如 **`<meta charset="utf-8">`**定义网页编码格式为 **utf-8**。
- **\<title>** 元素描述了文档的标题
- **\<body>** 元素包含了可见的页面内容
- **\<h1>** 元素定义一个大标题
- **\<p>** 元素定义一个段落

---

### 什么是HTML？

HTML 是用来描述网页的一种语言。

- HTML 指的是超文本标记语言: **H**yper**T**ext **M**arkup **L**anguage
- HTML 不是一种编程语言，而是一种**标记**语言
- 标记语言是一套**标记标签** (markup tag)
- HTML 使用标记标签来**描述**网页
- HTML 文档包含了HTML **标签**及**文本**内容
- HTML文档也叫做 **web 页面**

---

### HTML 标签

HTML 标记标签通常被称为 HTML 标签 (HTML tag)。

- HTML 标签是由*尖括号*包围的关键词，比如 **\<html>**
- HTML 标签通常是*成对出现*的，比如 **\<b>** 和 **\</b>**
- 标签对中的第一个标签是*开始标签*，第二个标签是*结束标签*
- 开始和结束标签也被称为*开放标签*和*闭合标签*

---

### <!DOCTYPE>声明

**\<!DOCTYPE>**声明有助于浏览器中正确显示网页。

doctype声明是不区分大小写的。

---

## HTML 基础

### HTML 标题

HTML标题（Heading）是通过\<h1>-\<h6>标签来定义的。

---

### HTML 段落

HTML段落是通过标签\<p>来定义的。

---

### HTML 链接

HTML链接是通过标签\<a>来定义的。

---

### HTML 图像

HTML图像是通过标签\<img>来定义的。

---

## HTML 元素

### HTML 元素

| 开始标签               | 元素内容     | 结束标签 |
| ---------------------- | ------------ | -------- |
| \<p>                    | 这是一个段落 | \</p>     |
| \<a href="default.htm"> | 这是一个链接 | \</a>     |
| \<br>                   | 换行         |          |

---

### HTML 元素语法

* HTML元素以**开始标签**起始
* HTML元素以**结束标签**终止
* **元素的内容**是开始标签与结束标签之间的内容
* 某些HTML元素具有**空内容（empty content）**
* 空元素**在开始标签中进行关闭**（以开始标签的结束而结束）
* 大多数HTML元素可拥有**属性**

---

### 嵌套的HTML元素

大多数HTML元素可以嵌套（HTML元素可以包含其他HTML元素）。

---

### HTML 提示：使用小写标签

HTML标签对大小写不敏感

---

## HTML 属性

### HTML 属性

+ HTML可以设置**属性**
+ 属性可以在元素中添加**附加信息**
+ 属性一般描述于**开始标签**
+ 属性总是以名称/值对的形式出现，**比如：name="value"**

---

### HTML 属性常引用属性值

属性值应该始终被包括在引号内。
**注意：**在某些个别情况下，比如属性值本身就包含有双引号，那么必须使用单引号。

---

### HTML 提示：使用小写属性

---

### HTML 属性参考手册

查看完整的HTML属性列表：[HTML标签参考手册](https://www.runoob.com/tags/html-reference.html)。
|属性|描述|
|---|---|
|class|为html元素定义一个或多个类名(classname)(类名从样式文件引入)|
|id|定义元素的唯一id|
|style|规定元素的行内样式(inline style)|
|title|描述了元素的额外信息(作为工具条使用)|

---

## HTML 标题

### HTML 标题

标题(Heading)是通过**\<h1>-\<h6>**标签进行定义的。
**\<h1>**定义最大的标题。**\<h6>**定义最小的标题。

---

### HTML 水平线

**\<hr>**标签在HTML页面中创建水平线。
hr元素可用于分隔内容。

---

### HTML 注释

```HTML
<!-- 这是一个注释 -->
```

---

## HTML 段落

### HTML 段落

段落是通过**\<p>**标签定义的。

---

### 不要忘记结束标签

---

### HTML 折行

如果希望在不产生一个新段落的情况下进行换行，使用\<br>标签。
\<br>元素没有结束标签。

---

### HTML 输出-使用提醒

当显示页面时，浏览器会移除源代码中多余的空格和空行。所有连续的空格或空行都会被算作一个空格。需要注意的是，HTML代码中的所有连续的空行（换行）也被显示为一个空格。

---

## HTML 文本格式化

### HTML 格式化标签

HTML使用标签**\<b>**("bold")与**\<i>**("italic")对输出的文本进行格式，如：**粗体**or*斜体* 
这些HTML标签被称为格式化标签。

---

## HTML 链接

### HTML 超链接（链接）

HTML使用标签\<a>来设置超文本链接。
在标签\<a>中使用了href属性来描述链接的地址。
**注意：**如果为这些超链接设置了CSS样式，展示样式会根据CSS的设定而显示。

---

### HTML 链接语法

`<a herf="url">链接文本</a>`
href属性描述了链接的目标。
**注意：**“链接文本”不必一定是文本。图片或者其他HTML元素都可以成为链接。

---

### HTML 链接-target属性

使用target属性，你可以定义被链接的文档在何处显示。

---

### HTML 链接-id属性

id属性可用于创建在一个HTML文档书签标记。
**注意：**书签是不以任何特殊的方式显示，在HTML文档中是不显示的，所以对于读者来说是隐藏的。
**实例：**
```HTML
<a id="tips">有用的提示部分</a><!--在HTML文档中插入ID-->
<a href="#tips">访问有用的提示部分</a><!--在HTML文档中创建一个链接到"有用的提示部分(id="tips"）"-->
```

---

## HTML \<head>

### HTML \<head>元素

\<head>元素包含了所有的头部标签元素。在\<head>元素中你可以插入脚本(scripts)，样式文件(CSS)，及各种meta信息。
可以添加在头部区域的元素标签为：\<title>，\<style>，\<meta>，\<link>，\<script>，\<noscript>和\<base>。

---

### HTML \<title>元素

\<title>标签定义了不同文档的标题。
\<title>在HTML/XHTML文档中是必须的。
\<title>元素：
+ 定义了浏览器工具栏的标题
+ 当网页添加到收藏夹时，显示在收藏夹中的标题
+ 显示在搜索引擎结果页面的标题

---

### HTML \<base>元素

\<base>标签描述了基本的链接地址/链接目标，该标签作为HTML文档中所有的链接标签的默认链接。

---

### HTML \<link>元素

\<link>标签定义了文档与外部资源之间的关系。
\<link>标签通常用于链接到样式表。

---

### HTML \<style>元素
\<style>标签定义了HTML文档的样式文件引用地址
在\<style>元素中可以直接添加样式来渲染HTML文档

---

### HTML \<meta>元素

meta标签描述了一些基本的元数据。
\<meta>标签提供了元数据，元数据不显示在页面上，但会被浏览器解析。
meta元素通常用于指定网页的描述，关键词，文件的最后修改时间，作者，和其他元数据。
元数据可以使用于浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他web服务。

---

### HTML \<script>元素

\<script>标签用于加载脚本文件。

---

## HTML 样式-CSS

CSS (Cascading Style Sheets) 用于渲染HTML元素标签的样式。

### 如何使用CSS

CSS是在HTML4开始使用的，是为了更好的渲染HTML元素而引入的。
CSS可以通过以下方式添加到HTML中：
* 内联样式-在HTML元素中使用“style”**属性**
* 内部样式表-在HTML文档头部\<head>区域使用\<style>**元素**来包含CSS
* 外部引用-使用外部CSS**文件**

**最好的方式是通过外部引用CSS文件。**

---

### 内联样式

当特殊的样式需要应用到个别元素是，就可以使用内联样式。使用内联样式的方法是在相关的标签中使用样式属性。样式属性可以包含任何CSS属性。
实例：
``` HTML 
<p style="color:blue;margin-left:20px;">这是一个段落。</p>
```

---

### 内部样式表

当单个文件需要特别样式时，就可以使用内部样式表。你可以在\<head>部分通过\<style>标签定义内部样式表：
> ```HTML
> <head>
> <style type="text/css">
> body {background-color:yellow;}
> p{color:blue;}
> </style>
> </head>
> ```

---

### 外部样式表

当样式需要被应用到很多页面的时候，外部样式表将是理想的选择。使用外部样式表，你就可以通过更改一个文件来改变整个站点的外观。

> ```HTML
> <head>
> <link rel="stylesheet" type="text/css" href="mystyle.css">
> </head>
> ```

---

## HTML 图像

### HTML 图像-图像标签（\<img>）和源属性（Src）

在HTML中，图像由\<img>标签定义。
\<img>是空标签，意识是说，它只包含属性，并且没有闭合标签。
要在页面上显示图像，你需要使用源属性（src）。src指“source”。源属性的值是图像的URL地址。
**定义图像的语法是：**
```HTML
<img src="url" alt="some_text">
```
浏览器将图像显示在文档中图像标签出现的地方。如果将图像标签置于两个段落之间，那么浏览器会首先显示第一个段落，然后显示图片，最后显示第二段。

---

### HTML 图像-Alt属性

alt属性用来为图像定义一串预备的可替换的文本。
```HTML
<img src="boat.gif" alt="Big Boat">
```
在浏览器无法载入图像时，替换文本属性告诉读者失去的信息。

---

### HTML 图像-设置图像的高度与宽度

height（高度）与width（宽度）属性用于设置图像的高度与宽度。
属性值默认单位为像素：
```HTML
<img src="pulpit.jpg" alt="Pulpit rock" width="304" height="228">
```
---

## HTML 表格

### HTML 表格

表格由\<table>标签来定义。每个表格均有若干行（由\<tr>标签定义），每行被分割为若干单元格（由\<td>标签定义）。字母td指表格数据（table data），即数据单元格的内容。数据单元格可以包含文本、图片、列表、段落、表单、水平线、表格等等。

---

### HTML 表格和边框属性

如果不定义边框属性，表格将不显示边框。
使用边框属性来显示一个带有边框的表格：
```HTML
<table border="1">
	<tr>
		<td>Row 1, cell 1</td>
		<td>Row 1, cell 2</td>
	</tr>
</table>
```

---

### HTML 表格表头

表格的表头使用\<th>标签进行定义。
大多数浏览器会把表头显示为粗体居中的文本：
```HTML
<table border="1">
    <tr>
        <th>Header 1</th>
        <th>Header 2</th>
    </tr>
    <tr>
        <td>row 1, cell 1</td>
        <td>row 1, cell 2</td>
    </tr>
    <tr>
        <td>row 2, cell 1</td>
        <td>row 2, cell 2</td>
    </tr>
</table>
```

---

## HTML 列表

HTML支持有序、无序和定义列表。

---

### HTML 无序列表

无序列表是一个项目的列表，此列项目使用粗体圆点进行标记。
无序列表使用\<ul>标签
```HTML
<ul>
<li>Coffee</li>
<li>Milk</li>
</ul>
```

---

### HTML 有序列表

列表项目使用数字进行标记。有序列表始于\<ol>标签。每个列表始于\<li>标签。
列表项使用数字来标记。
```HTML
<ol>
<li>Coffee</li>
<li>Milk</li>
</ol>
```

---

### HTML 自定义列表

自定义列表不仅仅是一列项目，而是项目及其注释的组合。
自定义列表以\<dl>标签开始。每个自定义列表项以\<dt>开始。每个自定义列表项的定义以\<dd>开始。
```HTML
<dl>
<dt>Coffee</dt>
<dd>- black hot drink</dd>
<dt>Milk</dt>
<dd>- white cold drink</dd>
</dl>
```
---

## HTML \<div>和\<span>

HTML可以通过\<div>和\<span>将元素组合起来。

---

### HTML 区块元素

大多数HTML元素被定义为**块级元素**或**内联元素**。
块级元素在浏览器显示时，通常会以新行来开始（和结束）。
实例：\<h1>，\<p>，\<ul>，\<table>

---

### HTML 内联元素

内联元素在显示时通常不会以新行开始。
实例: \<b>, \<td>, \<a>, \<img>

---

### HTML \<div> 元素

HTML \<div> 元素是**块级元素**，它可用于组合其他 HTML 元素的容器。
\<div> 元素没有特定的含义。除此之外，由于它属于块级元素，浏览器会在其前后显示折行。
如果与 CSS 一同使用，\<div> 元素可用于对大的内容块设置样式属性。
\<div> 元素的另一个常见的用途是文档布局。它取代了使用表格定义布局的老式方法。使用 \<table> 元素进行文档布局不是表格的正确用法。\<table> 元素的作用是显示表格化的数据。

---

### HTML \<span> 元素

HTML \<span> 元素是**内联元素**，可用作文本的容器
\<span> 元素也没有特定的含义。
当与 CSS 一同使用时，\<span> 元素可用于为部分文本设置样式属性。

---

## HTML 布局

### HTML 布局 - 使用\<div> 元素

div 元素是用于分组 HTML 元素的块级元素。

---

### HTML 布局 - 有用的提示

**Tip:** 使用 CSS 最大的好处是，如果把 CSS 代码存放到外部样式表中，那么站点会更易于维护。通过编辑单一的文件，就可以改变所有页面的布局。
**Tip:** 由于创建高级的布局非常耗时，使用模板是一个快速的选项。通过搜索引擎可以找到很多免费的网站模板（您可以使用这些预先构建好的网站布局，并优化它们）。

---

## HTML 表单和输入

### HTML 表单

表单是一个包含表单元素的区域。
表单元素是允许用户在表单中输入内容,比如：文本域(textarea)、下拉列表、单选框(radio-buttons)、复选框(checkboxes)等等。
表单使用表单标签 \<form> 来设置:
```HTML
<form>
.
input 元素
.
</form>
```

---

### HTML 表单 - 输入元素
多数情况下被用到的表单标签是输入标签（\<input>）。
输入类型是由类型属性（type）定义的。大多数经常被用到的输入类型如下：

---

#### 文本域（Text Fields）

文本域通过\<input type="text"> 标签来设定，当用户要在表单中键入字母、数字等内容时，就会用到文本域。
```HTML
<form>
First name: <input type="text" name="firstname"><br>
Last name: <input type="text" name="lastname">
</form>
```

**注意:**表单本身并不可见。同时，在大多数浏览器中，文本域的默认宽度是 20 个字符。

---

#### 密码字段

密码字段通过标签\<input type="password"> 来定义:
```HTML
<form>
Password: <input type="password" name="pwd">
</form>
```

**注意:**密码字段字符不会明文显示，而是以星号或圆点替代。

---

#### 单选按钮（Radio Buttons）

\<input type="radio"> 标签定义了表单单选框选项
```HTML
<form>
<input type="radio" name="sex" value="male">Male<br>
<input type="radio" name="sex" value="female">Female
</form>
```

---

#### 复选框（Checkboxes）

\<input type="checkbox"> 定义了复选框. 用户需要从若干给定的选择中选取一个或若干选项。
```HTML
<form>
<input type="checkbox" name="vehicle" value="Bike">I have a bike<br>
<input type="checkbox" name="vehicle" value="Car">I have a car
</form>
```

---

#### 提交按钮(Submit Button)

\<input type="submit"> 定义了提交按钮.
当用户单击确认按钮时，表单的内容会被传送到另一个文件。表单的动作属性定义了目的文件的文件名。由动作属性定义的这个文件通常会对接收到的输入数据进行相关的处理。
```HTML
<form name="input" action="html_form_action.php" method="get">
Username: <input type="text" name="user">
<input type="submit" value="Submit">
</form>
```
假如您在上面的文本框内键入几个字母，然后点击确认按钮，那么输入数据会传送到 "html_form_action.php" 的页面。该页面将显示出输入的结果。

---

## HTML 框架

通过使用框架，你可以在同一个浏览器窗口中显示不止一个页面。

**iframe语法：**
```HTML
<iframe src="URL"></iframe>
```
该URL指向不同的网页。

---

### Iframe - 设置高度与宽度

height 和 width 属性用来定义iframe标签的高度与宽度。
属性默认以像素为单位, 但是你可以指定其按比例显示 (如："80%")。
```HTML
<iframe loading="lazy" src="demo_iframe.htm" width="200" height="200"></iframe>
```

---

### Iframe - 移除边框

frameborder 属性用于定义iframe表示是否显示边框。
设置属性值为 "0" 移除iframe的边框。
```HTML
<iframe src="demo_iframe.htm" frameborder="0"></iframe>
```

---

### 使用iframe来显示目标链接页面

iframe可以显示一个目标链接的页面，目标链接的属性必须使用iframe的属性。
```HTML
<iframe src="demo_iframe.htm" name="iframe_a"></iframe>
<p><a href="http://www.runoob.com" target="iframe_a" rel="noopener">RUNOOB.COM</a></p>
```

---

## HTML 颜色

HTML 颜色由红色、绿色、蓝色混合而成。

---

### 颜色值

HTML 颜色由一个十六进制符号来定义，这个符号由红色、绿色和蓝色的值组成（RGB）。
每种颜色的最小值是0（十六进制：#00）。最大值是255（十六进制：#FF）。

|颜色（Color）|颜色十六进制（Color HEX）|颜色RGB（Color RGB）|
|---|---|---|
|黑色|#000000|rgb(0,0,0)|
|红色|#FF0000|rgb(255,0,0)|
|绿色|#00FF00|rgb(0,255,0)|
|蓝色|#0000FF|rgb(0,0,255)|
|黄色|#FFFF00|rgb(255,255,0)|
|青色|#00FFFF|rgb(0,255,255)|
|紫色|#FF00FF|rgb(255,0,255)|
|灰色|#C0C0C0|rgb(192,192,192)|
|白色|#FFFFFF|rgb(255,255,255)|

---

### 1600万种不同颜色

三种颜色 红，绿，蓝的组合从0到255，一共有1600万种不同颜色(256 x 256 x 256)。

---

## HTML 颜色名

查表：
[HTML 颜色名](https://www.runoob.com/html/html-colornames.html)

---

## HTML 颜色值

[通过十六进制（Hex）的颜色值排序](https://www.runoob.com/html/html-colorvalues.html)

---

## HTML 脚本

JavaScript 使 HTML 页面具有更强的动态和交互性。

---

### HTML \<script> 标签

\<script> 标签用于定义客户端脚本，比如 JavaScript。
\<script> 元素既可包含脚本语句，也可通过 src 属性指向外部脚本文件。
JavaScript 最常用于图片操作、表单验证以及内容动态更新。

---

### HTML\<noscript> 标签

\<noscript> 标签提供无法使用脚本时的替代内容，比方在浏览器禁用脚本时，或浏览器不支持客户端脚本时。
\<noscript>元素可包含普通 HTML 页面的 body 元素中能够找到的所有元素。
只有在浏览器不支持脚本或者禁用脚本时，才会显示 \<noscript> 元素中的内容。

---

## HTML 字符实体

HTML 中的预留字符必须被替换为字符实体。
一些在键盘上找不到的字符也可以使用字符实体来替换。

---

### HTML 实体

在 HTML 中，某些字符是预留的。
在 HTML 中不能使用小于号（<）和大于号（>），这是因为浏览器会误认为它们是标签。
如果希望正确地显示预留字符，我们必须在 HTML 源代码中使用字符实体（character entities）。字符实体类似这样：
```HTML
&entity_name;
<!--或-->
&#entity_number;
```

**注意：**浏览器也许不支持所有的实体名称（对实体数字的支持却很好）。

---

### 不间断空格(Non-breaking Space)

HTML 中的常用字符实体是**不间断空格(&nbsp;)**。

---

### 结合音标符

发音符号是加到字母上的一个"glyph(字形)"。
变音符号可以与字母、数字字符的组合来使用。
实例如下：
|音标符|字符|Construct|输出结果|
|----|----|----|----|
|&#768;|a|a\&#768;|a&#768;|
|&#769;|a|a\&#769;|a&#769;|
|&#770;|a|a\&#770;|a&#770;|
|&#771;|a|a\&#771;|a&#771;|
|&#768;|O|O\&#768;|O&#768;|
|&#769;|O|O\&#769;|O&#769;|
|&#770;|O|O\&#770;|O&#770;|
|&#771;|O|O\&#771;|O&#771;|

---

### HTML字符实体

[HTML 实体参考手册](https://www.runoob.com/tags/ref-entities.html)

---

## HTML 统一资源定位器(Uniform Resource Locators)

URL 是一个网页地址。
URL可以由字母组成，如"runoob.com"，或互联网协议（IP）地址： 192.68.20.50。

---

### URL - 统一资源定位器

一个统一资源定位器(URL) 用于定位万维网上的文档。
实例:[]( http://www.runoob.com/html/html-tutorial.htm)
语法规则：
```HTML
scheme://host.domain:port/path/filename
```
说明：
* scheme - 定义因特网服务的类型。最常见的类型是 http
* host - 定义域主机（http 的默认主机是 www）
* domain - 定义因特网域名，比如 runoob.com
* port - 定义主机上的端口号（http 的默认端口号是 80）
* path - 定义服务器上的路径（如果省略，则文档必须位于网站的根目录中）。
* filename - 定义文档/资源的名称

---

### 常见的 URL Scheme

|Scheme|访问|用于...|
|----|----|----|
|http|超文本传输协议|以http://开头的普通网页。不加密。|
|https|安全超文本传输协议|安全网页，加密所有信息交换。|
|ftp|文件传输协议|用于将文件下载或上传至网站。|
|file||您计算机上的文件。|

---

### URL 字符编码

URL 只能使用 ASCII 字符集来通过因特网进行发送。由于 URL 常常会包含 ASCII 集合之外的字符，URL 必须转换为有效的 ASCII 格式。
URL 编码使用 "%" 其后跟随两位的十六进制数来替换非 ASCII 字符。
URL 不能包含空格。URL 编码通常使用 + 来替换空格。

---

### URL 编码参考手册

[URL参考手册](https://www.runoob.com/tags/html-urlencode.html)

---

## HTML 速查列表

### HTML 基本文档

```HTML
<!DOCTYPE html>
<html>
<head>
<title>文档标题</title>
</head>
<body>
可见文本...
</body>
</html>
```

---

### 基本标签（Basic Tags）

```HTML
<h1>最大的标题</h1>
<h2> . . . </h2>
<h3> . . . </h3>
<h4> . . . </h4>
<h5> . . . </h5>
<h6>最小的标题</h6>
 
<p>这是一个段落。</p>
<br> （换行）
<hr> （水平线）
<!-- 这是注释 -->
```

---

### 文本格式化（Formatting）

```HTML
<b>粗体文本</b>
<code>计算机代码</code>
<em>强调文本</em>
<i>斜体文本</i>
<kbd>键盘输入</kbd> 
<pre>预格式化文本</pre>
<small>更小的文本</small>
<strong>重要的文本</strong>
 
<abbr> （缩写）
<address> （联系信息）
<bdo> （文字方向）
<blockquote> （从另一个源引用的部分）
<cite> （工作的名称）
<del> （删除的文本）
<ins> （插入的文本）
<sub> （下标文本）
<sup> （上标文本）
```

---

### 链接（Links）

```HTML
普通的链接：<a href="http://www.example.com/">链接文本</a>
图像链接： <a href="http://www.example.com/"><img src="URL" alt="替换文本"></a>
邮件链接： <a href="mailto:webmaster@example.com">发送e-mail</a>
书签：
<a id="tips">提示部分</a>
<a href="#tips">跳到提示部分</a>
```

---

### 图片（Images）

```HTML
<img loading="lazy" src="URL" alt="替换文本" height="42" width="42">
```

---

### 样式/区块（Styles/Sections）

```HTML
<style type="text/css">
h1 {color:red;}
p {color:blue;}
</style>
<div>文档中的块级元素</div>
<span>文档中的内联元素</span>
```

---

### 无序列表

```HTML
<ul>
    <li>项目</li>
    <li>项目</li>
</ul>
```

---

### 有序列表

```HTML
<ol>
    <li>第一项</li>
    <li>第二项</li>
</ol>
```

---

### 定义列表

```HTML
<dl>
  <dt>项目 1</dt>
    <dd>描述项目 1</dd>
  <dt>项目 2</dt>
    <dd>描述项目 2</dd>
</dl>
```

---

### 表格（Tables）

```HTML
<table border="1">
  <tr>
    <th>表格标题</th>
    <th>表格标题</th>
  </tr>
  <tr>
    <td>表格数据</td>
    <td>表格数据</td>
  </tr>
</table>
```

---

### 框架（Iframe）

```HTML
<iframe src="demo_iframe.htm"></iframe>
```

---

### 表单（Forms）

```HTML
<form action="demo_form.php" method="post/get">
<input type="text" name="email" size="40" maxlength="50">
<input type="password">
<input type="checkbox" checked="checked">
<input type="radio" checked="checked">
<input type="submit" value="Send">
<input type="reset">
<input type="hidden">
<select>
<option>苹果</option>
<option selected="selected">香蕉</option>
<option>樱桃</option>
</select>
<textarea name="comment" rows="60" cols="20"></textarea>
 
</form>
```

---

### 实体（Entities）

```HTML
&lt; 等同于 <
&gt; 等同于 >
&#169; 等同于 ©
```

---

## HTML 标签简写及全称

[HTML 标签简写及全称](https://www.runoob.com/html/html-tag-name.html)

---

