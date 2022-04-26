# Less学习

## CSS预处理器

### 什么是CSS预处理器

CSS预处理器是CSS语言的**超集**，比CSS更丰满。

**用一种专门的编程语言，为CSS增加了一些编程的特性**，将CSS作为目标生成文件，然后开发者就只要使用这种语言进行编码工作。

通俗的说，**CSS预处理器用一种专门的编程语言，进行Web页面样式设计，然后再编译成正常的CSS文件**。

常见的CSS预处理器语言：**Sass（SCSS）、LESS**

## less相关文档

less 是一款比较流行的**预处理 CSS**，支持变量、混合、函数、嵌套、循环等特点。

- [官网](http://lesscss.org/)
- [中文网（less 文档）](http://lesscss.cn/)
- [Bootstrap网站的 less 文档](https://less.bootcss.com/)
- 推荐文章：http://www.w3cplus.com/css/less

## less的语法

### 注释

less 的注释可以有两种。

第一种注释：模板注释

```less
  // 模板注释 这里的注释转换成CSS后将会删除
```

因为 less 要转换为 css才能在浏览器中使用。转换成 css 之后，这种注释会被删除（毕竟 css 不识别这种注释）。

第二种注释：CSS 注释语法

```less
/* CSS 注释语法 转换为CSS后让然保留 */
```

总结：如果在less中写注释，我们推荐写第一种注释。除非是类似于版权等内容，就采用第二种注释。

### 定义变量

我们可以把**重复使用或经常修改的值**定义为变量，在需要使用的地方引用这个变量即可。这样可以避免很多重复的工作量。

（1）在less文件中，定义一个变量的格式：

```less
@变量名: 变量值;        //格式

@bgColor: #f5f5f5;      //格式举例
```

（2）同时，在 less 文件中引用这个变量。

最终，less文件的完整版代码如下：

main.less：

```less
// 定义变量
@bgColor: #f5f5f5;

// 引用变量
body{
    background-color: @bgColor;
}
```

我们将上面的less文件编译为 css 文件后（下一段讲less文件的编译），自动生成的代码如下：

main.css：

```css
body{
    background-color: #f5f5f5;
}
```

### 使用嵌套

在 css 中经常会用到子代选择器，效果可能是这样的：

```css
.container {
  width: 1024px;
}

.container > .row {
  height: 100%;
}

.container > .row a {
  color: #f40;
}

.container > .row a:hover {
  color: #f50;
}
```

上面的代码嵌套了很多层，写起来很繁琐。可如果用 less 的嵌套语法来写这段代码，就比较简洁。

嵌套的举例如下：

main.less:

```less
.container {
  width: @containerWidth;

  > .row {
    height: 100%;
    a {
      color: #f40;

      &:hover {
        color: #f50;
      }

    }
  }

  div {
    width: 100px;

    .hello {
      background-color: #00f;
    }

  }
}
```

将上面的less文件编译为 css 文件后，自动生成的代码如下：

main.css

```css
.container {
    width: 1024px;
}

.container > .row {
    height: 100%;
}

.container > .row a {
    color: #f40;
}

.container > .row a:hover {
    color: #f50;
}

.container div {
    width: 100px;
}

.container div .hello {
    background-color: #00f;
}
```

### Mixin

Mixin 的作用是把**重复的代码**放到一个类当中，每次只要引用类名，就可以引用到里面的代码了，非常方便。

（1）在 less 文件中定义一个类：（将重复的代码放到自定义的类中）

```less
/* 定义一个类 */
.roundedCorners(@radius: 5px) {
  -moz-border-radius: @radius;
  -webkit-border-radius: @radius;
  border-radius: @radius;
}
```

上方代码中，第一行里面，括号里的内容是参数：这个参数是**缺省值**。

（2）在 less 文件中引用上面这个类：

```less
#header {
  .roundedCorners;
}
#footer {
  .roundedCorners(10px);
}
```

上方代码中，header 中的引用没有带参数，表示参数为缺省值； footer 中的引用带了参数，那就用这个参数。

### Import

在开发阶段，我们可以将不同的样式放到多个文件中，最后通过@import 的方式合并。意思就是，当出现多个 less 文件时，怎么引用它们。

这个很好理解， css 文件可以有很多个，less文件也可以有很多个。

（1）定义一个被引用的less文件，名为`_button1.less`：

```less
_button1.less:
.btn{
  line-height: 100px;
  color: @btnColor;
}
```

PS1：被引用的less文件，我们习惯在前面加**下划线**，表示它是**部分文件**。

PS2：`_button1.less`里可以引用`main.css`里的自定义变量。

（2）在 `main.css` 中引用上面的 `_button1.less`：（代码的第二行）

main.css：

```less
@btnColor: red;

@import url('_button1.less:');    //这里的路径写的是相对路径

body{
  width: 1024px;
}
```

将 上面的main.less 编译为 main.css之后，自动生成的代码如下：

```css
.btn {
    line-height: 100px;
    color: red;
}

body {
    width: 1024px;
}
```

### 内置函数

less 里有一些内置的函数，这里讲一下 lighten 和 darken 这两个内置函数。

main.less:

```less
body {
  background-color: lighten(#000, 10%);   // 让黑色变亮 10%
  color: darken(#fff, 10%);               // 让白色变暗 10%
}
```

将 上面的 main.less 编译为 main.css 之后，自动生成的代码如下：

main.css：

```css
body {
  background-color: #1a1a1a;
  color: #e6e6e6;
}
```

## 在 index.html中直接引用 less.js

- 做法一：写完 less文件后，将其编译为 css 文件，然后在代码中引用css文件。
- 做法二：在代码中直接用引用 less 文件。

产品上线后，当然是使用做法一，因为做法二会多出编译的时间。

做法二见[千古壹号文档](https://web.qianguyihao.com/09-%E7%A7%BB%E5%8A%A8Web%E5%BC%80%E5%8F%91/03-Less%E8%AF%A6%E8%A7%A3.html)

注意，我们要在服务器中打开 html 文件，否则，看不到效果。

这里也告诉了我们：

> 不提倡将 less 引入页面，因为 less 需要编译，因此你就需要再引入一个less.js, 多了一个HTTP 请求，同时当浏览器禁用了 js 你的样式就不起作用了，less 编译应该在服务端或使用 grunt 自动编译。

## less 的编译

见[千古壹号文档](https://web.qianguyihao.com/09-%E7%A7%BB%E5%8A%A8Web%E5%BC%80%E5%8F%91/03-Less%E8%AF%A6%E8%A7%A3.html)