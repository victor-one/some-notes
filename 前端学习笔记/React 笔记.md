# React 笔记

## 核心概念

### JSX

#### 1. 什么是 JSX

```jsx
const element = <h1>Hello, world!</h1>;
```

JSX 是一个JavaScript 的语法扩展。JSX 可以很好地描述 UI 应该呈现出它应有交互的本质形式。

#### 2. 为什么使用 JSX

这是因为 React 认为渲染逻辑本质上与其他 UI 逻辑内在耦合。React 并没有采用将标记与逻辑分离到不同文件这种人为的分离方式，而是通过将二者共同存放在称之为“组件”的松散耦合单元之中，从而实现关注点分离。

#### 3. 在 JSX 中嵌入表达式

在 JSX 语法中，可以在大括号内放置任何有效的 JS 表达式。

```jsx
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);
```

为了便于阅读，会将 jsx 拆分为多行。同时建议将内容包裹在括号中，这样可以避免遇到自动插入分号陷阱。

#### 4. JSX 也是一个表达式

在编译后，JSX 表达式会被转为普通 JS 函数调用，并且对其取值后得到 JS 对象。

可以在 `if` 语句和 `for` 循环的代码块中使用 JSX ，将 JSX 赋值给变量，把 JSX 当作参数传入，以及从函数中返回 JSX 。

```jsx
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

#### 5. JSX 中指定属性

可以通过使用引号，来将属性值指定为字符串字面量：

```jsx
const element = <a href="https://www.reactjs.org"> link </a>;
```

也可以使用大括号来在属性值中插入一个 JS 表达式。

```jsx
const element = <img src={user.avatarUrl}></img>;
```

在属性中嵌入 JS 表达式的时候，不要在大括号外面加上引号。应该仅使用引号（对于字符串值）或大括号（对于表达式）中的一个，对于同一属性不能同时使用这两种符号。

React DOM 使用小驼峰命名来定义属性的名称，而不使用 HTML 属性名称的命名约定。JSX 里的 `class` 变成了 `className`。

#### 6. 使用 JSX 指定子元素

假如一个标签里面没有内容，可以使用 `/>` 来闭合标签。

```jsx
const element = <img src={user.avatarUrl} />;
```

JSX 标签里能够包含很多子元素。

#### 7. JSX 防止注入攻击

React DOM 在渲染所有输入内容之前，默认会进行转义。可以确保在应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 XSS 攻击。

#### 8. JSX 表示对象

Babel 会把 JSX 转译成一个名为 `React.createElement()` 函数调用。

以下两种示例代码完全等效。

```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```jsx
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

`React.createElement()` 会预先执行一些检查，以帮助编写无错代码，但实际上它创建了一个这样的对象：

```jsx
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

这些对象被称为“React 元素”，描述了希望在屏幕上看到的内容。

### 元素渲染

#### 1. 将一个元素渲染为 DOM

根DOM节点：

```html
<div id="root"></div>
```

使用 `ReactDOM.createRoot()` 将一个 React 元素渲染到根 DOM 节点中。

```jsx
const root = ReactDOM.createRoot(
  document.getElementById('root')
);
const element = <h1>Hello, world</h1>;
root.render(element);
```

#### 2. 更新已渲染的元素

React 元素是不可变对象。一旦被创建，就无法更改它的子元素或者属性。

更新 UI 唯一的方式就是创建一个全新的元素，并将其传入 `root.render()`。

以计时器为例：

```jsx
const root = ReactDOM.createRoot(
  document.getElementById('root')
);

function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  root.render(element);
}

setInterval(tick, 1000);
```

注意：在实践中，大多数 React 应用只会调用一次 `root.render()`。

#### 3. React 只更新它需要更新的部分

React DOM 会将元素和它的子元素与它们之前的状态进行比较，并且只会进行必要的更新来使 DOM 达到预期的状态。

### 组件 & Props

组件从概念上类似于 JS 函数。接受任意的入参（“props”），并返回用于描述页面展示内容的 React 元素。

#### 1. 函数组件与 class 组件

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

这类组件被称为“函数组件”，因为本质上就是一个 JS 函数。

也可以使用 ES6 的 `class` 来定义组件：

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

上述两个组件在 React 中是等效的。

#### 2. 渲染组件

React 元素也可以是用户自定义的组件

```jsx
const element = <Welcome name="Sara" />;
```

当 React 元素为用户自定义组件时，它会将 JSX 所接收的属性（attributes）以及子组件（children）转换为单个对象传递给组件，这个对象被称之为“props”。

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
const element = <Welcome name="Sara" />;
root.render(element);
```

注意：组件名称必须以大写字母开头。React 会将以小写字母开头的组件视为原生 DOM 标签。