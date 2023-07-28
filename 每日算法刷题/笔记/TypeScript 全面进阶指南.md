# TypeScript 全面进阶指南

## 如何用正确地方式学习 TypeScript

TypeScript 由三个部分组成：**类型、语法、工程**。

* 类型指的是**为 JS 代码添加类型与类型检查来确保健壮性**；
* **提前使用新语法或新特性**来简化代码；
* 最终**通过 tsc 编译得到可用的 JS 代码**。

## TS 开发环境搭建

### VS code 配置与搭建

1. **TypeScript Importer**：会收集项目中所有的类型定义，在敲出`:`的时候提供这些类型来进行补全，且能够自动帮助把类型导入进来。
2. **Move TS**：可以通过编辑文件的路径，直接修改项目的目录结构。
3. **Ctrl + Shift + P 打开命令面板**，打开工作区设置，输入 typescript，筛选出所有 Typescript 有关的配置。在搜索处搜索“typescript Inlay Hints”，设置提示相关配置。

### Playground：懒人福音

[Playground](https://link.juejin.cn/?target=https%3A%2F%2Fwww.typescriptlang.org%2Fzh%2Fplay)

### TS 文件的快速执行：ts-node 与 ts-node-dev

1. 全局安装 ts-node 与 typescript；

2. 在项目中创建 TS 的项目配置文件：tsconfig.json；

   ```sh
   npx typescript --init
   // 如果全局安装了 TypeScript，可以这么做
   tsc --init
   ```

3. 之后创建一个 TS 文件，用 ts-node 执行。

ts-node 配置有两种方式：1、在 tsconfig 中新增`'ts-node'`字段；2、在执行 ts-node 的时候作为命令行的参数。

**命令行常见配置方式**：

* `-P,--project`：指定 tsconfig 文件位置。默认情况下 ts-node 会查找项目下的 tsconfig.json，如果配置文件是 `tsconfig.script.json`、`tsconfig.base.json`这种，就需要使用这个参数进行配置。
* `-T,--transpileOnly`：禁用执行过程中的类型检查过程，这能让文件执行速度更快，且不会被类型报错卡住。
* `--swc`：在 transpileOnly 的基础上，还会使用 swc 来进行文件的编译，进一步提升速度。
* `--emit`：将编译产物输出到`.ts-node`文件下（需要同时与`--compilerHost`选项一同使用）

也可以使用 node + require hook 的形式来执行 TS 文件：

```sh
node -r ts-node/register index.ts
```

如果想要传递参数给 ts-node，就需要使用环境变量，比如说传递之前的 transpileOnly 选项：

```sh
TS_NODE_TRANSPILE_ONLY=true node -r ts-node/register index.ts
```

ts-node 本身不支持自动监听文件变更然后重新执行，需要使用 ts-node-dev 库来实现。可以使用简写 `tsnd`。

`--respawn`选项启用了监听重启的能力，`--transpileOnly`提供了更快的编译速度。

### 更方便的类型兼容性检查

在只是想要进行类型比较的时候，没有必要真的去声明两个变量，即涉及了值空间的操作。可以只在类型空间中比较这些类型，只需要**使用 declare 关键字**。

还可以通过工具类型的形式，如使用 tsd 包提供的一系列工具类型。

* `expectType<预期的类型>(表达式或变量等)`：检查预期类型与表达式或变量的类型是否一致；
* `expectNotType`：检查预期类型与表达式或变量的类型是否不同；
* `expectAssignable`：检查表达式或变量的类型能否赋值给预期类型。

### require extension

NodeJs 中的 require 逻辑执行：

* Resolution，基于入参拼接出 require 文件的绝对路径。
* 基于绝对路径去 `require.cache` 这个全局变量中查找此文件是否已经缓存，如果存在直接使用缓存的文件内容。
* Loading，基于绝对路径实例化一个 Module 类实例，基于路径后缀名调用内置的处理函数。
* Wrapping，对于 JS 文件，将文件内容字符串外层包裹一个函数，执行这个函数。对于 JSON  文件，将内容包裹挂载到 `module.exports` 下。
* Evaluating，执行这个文件内容。
* Caching，对于未缓存的文件，将其执行结果缓存起来。

## 原始类型与对象类型

### 原始类型的类型标注

js 中内置的原始类型在 ts 中都有对应的类型注解，需要注意，在 ts 中使用 `bigint`，需要在配置文件中将 target 设置为 ES2020 以上。

**null 与 undefined**

需要注意的是 **null 和 undefined** 在 ts 中的使用与 js 不同。js 中 null 和 undefined 分别表示“**这里有值，但是是个空值**”和“**这里没有值**”。而在 TS 中，null 和 undefined 类型都是**有具体意义的类型**。当没有开启 `strictNullChecks` 检查的情况下，**会被视作其他类型的子类型**。

```typescript
const temp1: null = null;
const temp2: undefined = undefined;
// 仅在关闭 strictNullChecks 的时候成立
const temp3: string = null;
const temp4: string = undefined;
```

**void**

在 js 中，void 操作符会强制将后面的函数声明转化为表达式。但是 ts 中 **void 用于描述一个内部没有 return 语句，或者没有显式 return 一个值的函数的返回值**。

undefined 能够被赋值给 void 类型的变量，但是 null 类型只有在关闭 `strictNullChecks` 配置的情况下才成立。

### 数组的类型标注

在 ts 中有两种方式声明一个**数组**类型：

```typescript
const arr1: string[] = [];
const arr2: Array<string> = [];
```

两种方式完全等价，一般以第一种为主。

**元组（Tuple）**

```typescript
const arr3: [string, string, string] = ['0', '1', '2'];
console.log(arr3[599]);// 报错

// 可精确获得对应位置的类型
const arr4: [string, number, boolean] = ['test', 1, true];
// 支持设置可选
const arr5: [string, number?, boolean?] = ['test', 1];
const arr6: [string, number?, boolean?] = ['test', ,];
const arr7: [string, number?, boolean?] = ['test', true]; // 报错,必须按顺序
const arr8: [string, number?, boolean] = ['test', 1]; // 报错 必选元素不能再可选元素后面
```

对于标记可选的成员，在 `--strictNullChecks` 配置下会被视为一个 `string | undefined` 的类型。此时元组的长度会发生变化。

```typescript
type TupleLength = typeof arr5.length; // 1 | 2 | 3
```

> ​	type：类型别名、typeof：类型查询 以及 联合类型后续再讲

具名元组：

```typescript
const arr9: [name: string, age: number, male: boolean] = ['zhangsan', 14, true];
const arr10: [name: string, age: number, male?: boolean] = ['zhangsan', 14];
```

除了显式的越界访问，还可能存在隐式的越界访问。对于数组，无法检查出是否存在隐式访问，因为在类型层面不知道他到底有多少个元素，但是对于元组会警告。

### 对象的类型标注

**对象类型描述使用 interface**，可以理解为这个对象对外提供的接口结构。

```typescript
interface IDesc {
  name: string;
  age: number;
  male: boolean;
}

const obj1: IDesc = {
  name: 'zhangsan',
  age: 13,
  male: true,
}
```

* 每一个属性必须一一对应到接口的属性类型；
* 不能有多的或者少的属性，包括直接在对象内部声明，或是 `obj1.other = 'xxx'` 这样的属性访问赋值的形式。

**可以对属性进行修饰，包括可选（Optional）与只读（Readonly）这两种**。

**修饰接口属性**

```typescript
interface IDesc {
  name: string;
  age: number;
  // 可选
  male?: boolean;
  other?: string;
}

const obj1: IDesc = {
  name: 'zhangsan',
  age: 13,
  other: 'test',
}
```

可选属性标记不会影响对这个属性进行赋值。

可以将属性标记为**已读**：`readonly`，它的作用是**防止对象的属性被再次赋值**。

```typescript
interface IDesc {
  name: string;
  age: number;
  // 只读
  readonly male: boolean;
  other?: string;
}

const obj1: IDesc = {
  name: 'zhangsan',
  age: 13,
  male: true
}
obj1.male = true; // 报错
```

数组与元组也存在只读的修饰，但与对象类型有两处不同。

* 只能将整个数组/元组标记为只读，而不能像对象一样标记某个属性为只读；
* 一旦被标记为只读，整个只读数组/元组的类型上，将不再有 push、pop 等方法（即会改变原数组的方法）。报错会报不存在属性“push”这种，实现的本质是只读数组与只读元组的类型实际上变成了 ReadonlyArray，而不再是 Array。

**type 与 interface**

type 是类型别名，推荐使用 interface 来描述对象、类的结构，用 type 将一个函数签名、一个联合类型、一个工具类型等等抽离成一个完整独立的类型。

**object、Object 以及 \{\}**

在 ts 中 **Object 包含所有的类型**（对于 undefined、null、void 0，需要关闭 strictNullChecks）。

与 Object 类似的还有 Boolean、Number、String、Symbol，这几个装箱类型（Boxed Types）同样包含一些超出预期的类型。如 String，其同样包括 undefined、null、void，以及代表的 拆箱类型（Unboxed Types）string。

**任何时候都不应该使用装箱类型。**

**object 代表所有非原始类型的类型**，即数组、对象与函数类型这些。

`{}`，可以认为是一个对象字面量类型，也可以认为是一个合法的但内部无属性定义的空对象，表示任何 非 null/undefined 的值。

```typescript
const tmp1: {} = undefined;
const tmp2: {} = null;
const tmp3: {} = void 0;

const tmp4: {} = 'test';
const tmp5: {} = 123;
const tmp6: {} = { name1: 'test' };
const tmp7: {} = () => {};
const tmp8: {} = [];
```

需要注意的是虽然能将其作为变量的类型，但**无法对这个变量进行任何赋值操作**。

```typescript
const tmp6: {} = { name1: 'test' };
tmp6.name1 = 'test2'; // 类型“{}”上不存在属性“age”
```

总结：

* **任何时候都不要使用装箱类型！！！**
* 如果不确定某个变量的具体类型，但能确定不是原始类型，可以使用 object。但是更推荐进一步区分，使用 `Record<string, unknown>` 或 `Record<string, any>`表示对象，`unknown[]`或`any[]`表示数组，`(...args: any[]) => any`表示函数。
* 避免使用`{}`。

### unique symbol

在 ts 中，symbol 类型并不具有唯一性，任何 symbol 类型的对象，它们的 symbol 类型指的都是 ts 中的同一个类型。为了实现独一无二这个特性，ts 支持 unique symbol 这一类型声明，它是 symbol 类型的子类型，每一个类型都是独一无二的。

在 JS 中，可以使用`Symbol.for`方法来复用已创建的 Symbol。在 ts 中如果要引用已创建的 unique symbol 类型，需要使用类型查询操作符 typeof。

```typescript
declare const uniqueSym: unique symbol;

const uniqueSym2: typeof uniqueSym = uniqueSym
```

## 字面量类型与枚举

### 字面量类型与联合类型

当对象的属性来自于一组确定值的集合时，可以使用联合类型加上字面量类型。

```typescript
interface Res {
  code: 100 | 101 | 102;
  status: 'success' | 'failure';
  data: any;
}
```

此时就可以在访问的时候获得精确的类型推导了。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a428d95d0eee4c269302df47bf45e7b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

`declare var res: Res`，可以认为它其实就是快速生成一个符合指定类型，但没有实际值的变量，同时也不存在于运行时中。

### 字面量类型

字面量类型代表着比原始类型更精确的类型，同时也是原始类型的子类型。

字面量类型主要包括**字符串字面量类型、数字字面量类型、布尔字面量类型和对象字面量类型**。

字面量类型要求的是**值级别的字面量一致**。

一般不会单独使用字面量类型，通常和联合类型（`|`）一起使用，表达一组字面量类型。

**联合类型**

代表了**一组类型的可用集合**。

需要注意的是：

* 对于联合类型中的函数类型，需要使用括号`()`包裹起来
* 函数类型并不存在字面量类型
* 可以在联合类型中进一步嵌套联合类型，但这些嵌套的联合类型最终会被展平到第一级中

使用场景之一是通过多个对象类型的联合，来实现手动的互斥属性，即这一属性如果有字段1，那么就没有字段2：

```typescript
interface Tmp {
  user:
    | {
      vip: true;
      expires: string;
    }
    | {
      vip: false;
      promotion: string;
    };
}

declare var tmp: Tmp;

if (tmp.user.vip) {
  console.log(tmp.user.expires);
}
```

也可以通过类型别名来复用一组字面量联合类型。

除了原始类型的字面量类型以外，对象类型也有着对应的字面量类型。

**对象字面量类型**

对象字面量类型就是一个对象类型的值。意味着这个对象的值全都为字面量值。

需要注意的是，**无论是原始类型还是对象类型的字面量类型，它们的本质都是类型而不是值。**在编译时同样会被擦除，同时也是被存储在内存中的类型空间而不是值空间。

### 枚举

```typescript
enum PageUrl {
  Home_Page_Url = 'url1',
  Setting_Page_Url = 'url2',
  Share_page_Url = 'url3',
}

const home = PageUrl.Home_Page_Url;
```

这样做的好处是：1、拥有了更好的类型提示。2、这些常量被真正地**约束在一个命名空间下**。

如果没有声明枚举地值，会默认使用数字枚举，并且从0开始，以1递增。

如果只为某一个成员指定了枚举值，那么之前未赋值地成员仍然会使用从 0 递增地方式，之后地成员则会开始从枚举值递增。

在数字型枚举中，可以使用延迟求值的枚举值，比如函数。

```typescript
const returnNum = () => 100 + 499;

enum Items {
  Foo = returnNum(),
  Bar = 599,
  Baz
}
```

**使用延迟求值的枚举值是有条件的。如果使用了延迟求值，那么没有使用延迟求值的枚举成员必须放在使用常量枚举值声明的成员之后**（如上例），或者放在第一位。

```typescript
const returnNum = () => 100 + 499;

enum Items {
  Baz,
  Foo = returnNum(),
  Bar = 599,
}
```

ts 中也可以同时使用字符串枚举值和数字枚举值。

枚举和对象的重要差异在于，**对象是单向映射的**，我们只能从建映射到键值。而**枚举是双向映射的**，即你可以从枚举成员映射到枚举值，也可以从枚举值映射到枚举成员。

```typescript
enum Items {
  Foo,
  Bar,
  Baz
}

const fooVal = Items.Foo; // 0
const fooKey = Items[0]; // "Foo"
```

上述枚举编译如下：

```javascript
"use strict";
var Items;
(function (Items) {
    Items[Items["Foo"] = 0] = "Foo";
    Items[Items["Bar"] = 1] = "Bar";
    Items[Items["Baz"] = 2] = "Baz";
})(Items || (Items = {}));
```

这里的 `obj[obj[k] = v] = k` 本质上就是进行了 `obj[k] = v` 与 `obj[v] = k` 这样两次赋值。

需要注意的是，仅有值为数字的枚举成员才能够进行双向枚举，**字符串枚举成员仍然只会进行单次映射**。

```typescript
enum Items {
  Foo,
  Bar = 'test of bar',
}

"use strict";
var Items;
(function (Items) {
    Items[Items["Foo"] = 0] = "Foo";
    Items["Bar"] = "test of bar";
})(Items || (Items = {}));
```

**常量枚举**

常量枚举和枚举相似，只是其声明多了一个 const ：

```typescript
const enum Items {
	Foo,
	Bar,
	Baz
}

const fooVal = Items.Foo
```

对于常量枚举，**只能通过枚举成员访问枚举值**（而不能通过值访问成员）。同时，在编译产物中不会存在一个额外的辅助对象，对枚举成员的访问会被**直接内联替换为枚举的值**。

```javascript
// 编译产物
const fooValue = 0 /* Foo */; // 0
```

### 类型控制流分析中的字面量类型

使用 const 声明的变量，其类型会从值推导出最精确的字面量类型。而对象类型则只会推导至符合其属性结构的接口，不会使用字面量类型。

这是因为使用 let 声明的变量是可以再次赋值的，因此对于 let 声明，**只需要推导至这个值从属的类型即可**。而 const 声明的原始类型变量将不再可变，因此**可以一步到位收窄到最精确的字面量类型**，但对象类型变量仍可变。

## 函数与Class中的类型：详解函数重载与面向对象

### 函数

**函数的类型签名**

函数的类型是描述了**函数入参类型与函数返回值类型**，同样使用`:`的语法进行类型标注。

**函数声明**方式：

```typescript
function foo(name: string): number {
  return name.length;
}
```

**函数表达式**方式：

```typescript
const foo = function (name: string): number {
  return name.length;
}
```

也可以像对变量进行类型标注那样，对`foo`这个变量进行类型声明：

```typescript
const foo: (name: string) => number = function (name) {
  return name.length
}
```

这里的`(name: string) => number`是 typescript 中的**函数类型签名**。

箭头函数标注

```typescript
// 方式一
const foo = (name: string): number => {
  return name.length
}
// 方式二
const foo: (name: string) => number = (name) => {
  return name.length
}
```

不推荐使用方式二，因为可读性非常差。要么**直接在函数中进行参数和返回值的类型声明**，要么**使用类型别名将函数声明抽离出来**：

```typescript
type FuncFoo = (name: string) => number

const foo: FuncFoo = (name) => {
  return name.length
}
```

如果只是为了描述这个函数的类型结构，甚至可以使用 interface 来进行函数声明：

```typescript
interface FuncFooStruct {
  (name: string): number
}
```

此时的 interface 被称为 **Callable Interface**。

**void 类型**

在 ts 中，**一个没有返回值的函数，其返回类型应当被标记为 void**。

void 表示没有进行返回操作，undefined 表示进行了返回操作，但没有返回实际的值。

**可选参数与 rest 参数**

在函数类型中使用`?`描述一个可选参数。

```typescript
// 在函数逻辑中注入可选参数默认值
function foo1(name: string, age?: number): number {
  const inputAge = age || 18;// 或使用 age ?? 18
  return name.length + inputAge
}

// 直接为可选参数声明默认值
function foo2(name: string, age: number = 18): number {
  const inputAge = age;
  return name.length + inputAge
}
```

**可选参数必须位于必选参数之后**。

对于 rest 参数的类型标记，由于其**实际上是一个数组，应该使用数组类型进行标注**：

```typescript
function foo(arg1: string, ...rest: any[]) { }
```

**也可以使用元组类型进行标注**：

```typescript
function foo(arg1: string, ...rest: [number, boolean]) { }
```

**重载**

想要实现与入参关联的返回值类型，可以使用 typescript 提供的**函数重载签名**。

```typescript
function func(foo: number, bar: true): string;
function func(foo: number, bar?: false): number;
function func(foo: number, bar?: boolean): string | number {
  if (bar) {
    return String(foo);
  } else {
    return foo * 599;
  }
}

const res1 = func(599);
const res2 = func(599, true);
const res3 = func(599, false);
```

这里我们的三个`function func`其实具有不同的意义：

* `function func(foo: number, bar: true): string`，**重载签名一**，传入 bar 的值为 true 时，函数返回值为 string 类型；
* `function func(foo: number, bar?: false): number`，**重载签名二**，传入 bar 的值为 false 时，函数返回值为 number 类型；
* `function func(foo: number, bar?: boolean): string | number`，**函数的实现签名**，会包含重载签名的所有可能情况；

需要注意的是：拥有多个重载声明的函数在被调用时，是**按照重载的声明顺序往下查找的**。

ts 中的重载是一种伪重载，它**只有一个具体实现，其重载体现在方法调用的签名上而非具体实现上**。而在 C++ 等语言中，重载体现在多个**名称一致但入参不同的函数实现上**，这才是更广义上的函数重载。

**异步函数、Generator 函数等类型签名**

```typescript
async function asyncFunc(): Promise<void> {}

function* genFunc(): Iterable<void> {}

async function* asyncGenFunc(): AsyncIterable<void> {}
```

### Class

**类与类成员的类型签名**

Class 的主要结构只有**构造函数、属性、方法和访问符（Accessor）**。

属性的类型标注类似于变量，而构造函数、方法、存取器的类型标注类似于函数。

需要注意的是，setter 方法**不允许进行返回值的类型标注**。

**修饰符**

在 ts 中沃们能够为 Class 成员添加这些修饰符：`public`/`private`/`protected`/`readonly`。

```typescript
class Foo {
  private prop: string;

  constructor(inputProp: string) {
    this.prop = inputProp;
  }

  protected print(addon: string): void {
    console.log(`${this.prop} and ${addon}`);
  }

  public get propA(): string {
    return `${this.prop}+A`;
  }

  public set propA(value: string) {
    this.propA = `${value}+A`
  }
}
```

* **public**：此类成员在**类、类的实例、子类**中都能被访问；
* **private**：此类成员仅能在**类的内部**被访问；
* **protected**：此类成员仅能在**类与子类中**被访问，示例**不允许再访问受保护的成员**。

为了简单起见，可以再构造函数中对参数应用访问性修饰符：

```typescript
class Foo {
	constructor(public arg1: string, private arg2: boolean) { }
}

new Foo('test', true)
```

此时，参数会直接作为类的成员（即实例的属性），免去后续的手动赋值。

**静态成员**

可以使用 static 关键字来标识一个成员为静态成员。

不同于实例成员，在类的内部静态成员无法通过 this 来访问，需要通过`Foo.staticHandler`这种形式进行访问。

这是因为**静态成员是直接被挂载在函数体上**，而**实例成员是挂载在原型上**的。因此，**静态成员不会被实例继承，它始终只属于当前定义的这个类（以及其子类）**。而原型对象上的实例成员则会**沿着原型链进行传递**，也就是能够被继承。

**继承、实现、抽象类**

与 js 中一样，ts 中也使用 extends 关键字来实现继承。

```typescript
class Base { }

class Derived extends Base { }
```

对于这里的两个类，比较严谨的称呼是 **基类（Base）** 与 **派生类（Derived）**。当然也可以叫父类与子类。主要需要了解**派生类对基类成员的访问与覆盖操作**。

基类中的哪些成员能够被派生类访问，完全是由其修饰符决定的。派生类中可以访问到使用 `public` 或 `protected` 修饰符的基类成员。除了访问以外，基类中的方法也可以在派生类中被覆盖，但我们仍然可以通过 super 访问到基类中的方法。

```typescript
class Base {
	print() { }
}

class Derived extends Base {
	print() {
		super.print()
	}
}
```

由于在派生类中覆盖基类方法时，无法确保派生类的这一方法能覆盖基类方法，因此，ts 4.3 新增了`override`关键字，来确保派生类尝试覆盖的方法一定在基类中存在定义。

```typescript
class Base {
  printWithLove() { }
}

class Derived extends Base {
  override print() {
    
  }
}
```

此时，ts 会报错，因为**尝试覆盖的方法并未在基类中声明**。

除了基类与派生类以外，还有一个比较重要的概念：**抽象类**。抽象类是对类结构与方法的抽象。

**一个抽象类中描述了一个类中应当有哪些成员（属性、方法等），一个抽象方法描述了这一方法在实际实现中的结构**。

抽象类使用 abstract 关键字声明：

```typescript
abstract class AbsFoo {
  abstract absProp: string;
  abstract get absGetter(): string;
  abstract absMethod(name: string): string
}
```

注意：抽象类中的成员也需要使用 abstract 关键字才能被视为抽象类成员。可以实现（implements）一个抽象类。

```typescript
class Foo implements AbsFoo {
  absProp: string = "test"

  get absGetter(): string {
    return 'test'
  }

  absMethod(name: string): string {
    return name
  }
}
```

此时，必须完全实现这个抽象类的每一个抽象成员。需要注意的是，在 ts 中**无法声明静态的抽象成员**。

也可以使用 interface 声明类的结构。

除此之外，还可以使用 Newable Interface 来描述一个类的结构：

```typescript
class Foo { }

interface FooStruct {
	new(): Foo
}

declare const NewableFoo: FooStruct;

const foo = new NewableFoo();
```

### 补充

**私有构造函数**

可以将类的构造函数标记为私有来阻止它被错误地实例化。

**SOLID原则**

* S，单一功能原则，一个类应该仅具有一种职责。
* O，开放封闭原则，一个类应该是可扩展但不可修改的。
* L，里式替换原则，一个派生类可以在程序的任何一处对其基类进行替换。
* I，接口分离原则，类的实现方应只需要实现自己需要的那部分接口。
* D，依赖倒置原则，对功能的实现应该依赖于抽象层，即不同的逻辑通过实现不同的抽象类。

## 内置类型：any、unknown、never 与类型断言

### 内置类型：any、unknown 与 never

**any 类型**

为了能够表示“**任意类型**”，ts 中提供了一个内置类型 any。

在某些情况下，变量/参数也会被隐式地推导为 any。

```typescript
// any
let foo;

// foo、bar 均为 any
function func(foo, bar){}
```

当在 tsconfig 中启用了`noImplicitAny`时会报错，可以显式为这两个参数指定 any 类型。

any 类型地变量可以在声明后再次接受任意类型地值，同时也可以被赋值给任意其他类型的变量。

可以在 any 类型变量上任意地进行操作，包括赋值、访问、方法调用等等，此时可以认为类型推导与检查是被完全禁用的。

any 类型表示一个**无拘无束的“任意类型”，它能兼容所有类型，也能够被所有的类型兼容**。

注意：

* 如果是类型不兼容报错导致使用any，考虑用类型断言替代。
* 如果是类型太复杂导致不想全部声明而使用 any，考虑将这一处的类型去断言为你需要的最简类型。
* 如果想表达一个未知类型，更合理的方式是使用 unknown。

**unknown 类型**

unknown 类型和 any 类型有点类似，一个 unknown 类型的变量可以再次赋值为任意其它类型，但只能赋值给 any 与 unknown 类型的变量。

```typescript
let unknownVar: unknown = "linbudu";

unknownVar = false;
unknownVar = "linbudu";
unknownVar = {
  site: "juejin"
};

unknownVar = () => { }

const val1: string = unknownVar; // Error
const val2: number = unknownVar; // Error
const val3: () => {} = unknownVar; // Error
const val4: {} = unknownVar; // Error

const val5: any = unknownVar;
const val6: unknown = unknownVar;
```

要对 unknown 类型进行属性访问，需要进行类型断言。

**在类型未知的情况下，更推荐使用 unknown 标注**。

**never 类型**

never 是一个“**什么都没有**”的类型，它**甚至不包括空的类型**，严格来说，**never 类型不携带任何的类型信息**，因此会在联合类型中被直接移除。

```typescript
declare let v1: never;
declare let v2: void;

v1 = v2; // X 类型 void 不能赋值给类型 never

v2 = v1;
```

在编程语言的类型系统中，never 类型被称为 **Bottom Type**，是**整个类型系统层级中最底层的类型**。和 null、undefined 一样，它是所有类型的子类型，但只有 never 类型的变量能够赋值给另一个 never 类型变量。

通常不会显式声明 never 类型，但在某些情况下使用 never 确实是符合逻辑的，比如一个只负责抛出错误的函数：

```typescript
function justThrow(): never {
  throw new Error()
}
```

在类型流的分析中，一旦一个返回值类型为 never 的函数被调用，那么下方的代码都会被视为无效的代码（即无法执行到）

```typescript
function justThrow(): never {
  throw new Error()
}

function foo (input:number){
  if(input > 1){
    justThrow();
    // 等同于 return 语句后的代码，即 Dead Code
    const name = "linbudu";
  }
}
```

可以显式利用 never 类型来进行类型检查。

### 类型断言：警告编译器不准报错

类型断言能够显式告知类型检查程序当前这个变量的类型，可以进行类型分析的修正。它其实就是一个将变量的已有类型更改为新指定类型的操作，基本语法是`as NewType`。

* 可以将 any/unknown 类型断言到一个具体的类型；
* 还可以 as 到 any 来跳过所有的类型检查；
* 也可以在联合类型中断言一个具体的分支；
* 但是类型断言的正确使用方式是，在 ts 类型分析不正确或不符合预期时，将其断言为此处的正确类型。

```typescript
interface IFoo {
  name: string;
}

declare const obj: {
  foo: IFoo
}

const {
  foo = {} as IFoo // 这里将 {} 字面量类型断言为了IFoo类型
} = obj
```

当然，更严谨的方式是定义为 `Partial<IFoo>` 类型，即 IFoo 的属性均为可选的。

除了使用 as 语法以外，也可以使用 `<>` 语法，但这种语法在 TSX 中并不能很好地解析出来。

**双重断言**

使用类型断言地时候，当原类型与断言类型之间差异过大地时候，会报类型错误。需要先断言到 unknown 类型再断言到预期类型。

```typescript
const str: string = 'test';
// 直接像下面这样写会报错
// (str as { handler: () => {} }).handler()

(str as unknown as { handler: () => {} }).handler();

// 使用尖括号断言
(<{ handler: () => {} }>(<unknown>str)).handler();
```

**非空断言**

非空断言其实是类型断言地简化，使用`!`语法，即`obj!.func()!.prop`的形式标记前面的一个声明一定是非空的（**实际上就是剔除了 null 和 undefined 类型**）。

```typescript
declare const foo: {
  func?: () => ({
    prop?: number | null;
  })
};

foo.func().prop.toFixed();
```

此时，func 在 foo 中不一定存在，prop 在 func 调用结果中不一定存在，且可能为 null，因此会收获两个类型报错。如果非要坚持调用的话，想要解决掉类型报错就可以使用非空断言：

```typescript
foo.func!().prop!.toFixed();
```

其应用位置类似于**可选链**：

```typescript
foo.func?.().prop?.toFixed();
```

不同的是，**非空断言的运行时仍然会保持调用链，因此在运行时可能会报错。而可选链则会在某个部分收到 undefined 或 null 的时候直接短路掉，不会再发生后面的调用**。

非空断言的常见场景还有 `document.querySelector`、`Array.find`方法等：

```typescript
const element = document.querySelector('#id')!;
const target = [1, 2, 3, 599].find(item => item === 599)!;
```

可以通过`non-nullable-type-assertion-style`规则来检查代码中是否存在类型断言能被简写成非空断言的情况。

类型断言还有一种用法是作为代码提示的辅助工具。

```typescript
interface IStruct {
  foo: string;
  bar: {
    barPropA: string;
    barPropB: number;
    barMethod: () => void;
    baz: {
      handler: () => Promise<void>;
    };
  };
}

// error!
// const obj: IStruct = {}

const obj = <IStruct> {
  bar: {
    baz: {},
  },
};
```

此时类型提示仍然存在。

### 类型层级初探

大致类型层级关系如下（**注意并不完整！！！**）：

* 最顶级的类型，any 与 unknown
* 特殊的 Object，包含了所有的类型，但和 Top Type 比还是差了一层
* String、Boolean、Number 这些装箱类型
* 原始类型与对象类型
* 字面量类型，需要注意的是 null 和 undefined 并不是字面量类型的子类型
* 最底层的never

## TypeScript 类型工具（上）

按照使用方式划分，类型工具可以分为三类：**操作符、关键字与专用语法**。按照使用目的划分，类型工具可以分为**类型创建**与**类型安全保护**两类。

### 类型别名

```typescript
type A = string;
```

通过 `type` 关键字声明了一个类型别名 A，同时它的类型等价于 string 类型。**类型别名的作用主要是对一组类型或一个特定类型结构进行封装，以便于在其他地方复用**。

可以用于**抽离一组联合类型，也可以用于抽离一个函数类型，或者声明一个对象类型**（就像接口一样）。

类型别名**还可以作为工具类型**。工具类同样基于类型别名，只是多了个泛型。

类型别名可以这么声明自己能够接受泛型，一旦接受了泛型，就称之为**工具类型**。

```typescript
type Factory<T> = T | number | string;
```

工具类型的基本功能仍然是创建类型，只不过工具类型能够接受泛型参数，从而实现**更灵活的类型创建功能**。从这个角度看，工具类型就像一个函数一样，泛型是入参，内部逻辑基于入参进行某些操作，再返回一个新的类型。

```typescript
const foo: Factory<boolean> = true;
```

一般不会直接使用工具类型来做类型标注，而是**再度声明一个新的类型别名**：

```typescript
type FactoryWithBool = Factory<boolean>;

const foo: FactoryWithBool = true;
```

同时，泛型参数的名称（上面的 T ）也是不固定的，一般采用**大驼峰命名法**。

```typescript
type Factory<NewType> = NewType | number | string;
```

下面这个工具类型可以接受一个类型，并返回一个包括 null 的联合类型。这样在实际使用的时候就饿可以确保处理了可能为空值的属性读取与方法调用：

```typescript
type MaybeNull<T> = T | null;

function process(input: MaybeNull<{ handler: () => {} }>) {
  input?.handler();
}
```

类似的还有 MaybePromise、 MaybeArray。

```typescript
type MaybeArray<T> = T | T[];

// 函数泛型会在后面了解
function ensureArray<T>(input: MaybeArray<T>): T[] {
  return Array.isArray(input) ? input : [input];
}
```

另外，类型别名中可以接受任意个泛型，以及为泛型指定约束、默认值等。

总之，对于工具类型来说，它的主要意义就是**基于传入的泛型进行各种类型操作**，得到一个新的类型。

### 联合类型和交叉类型

**联合类型** `|`，代表按位或，即只需要符合联合类型中的一个类型。

**交叉类型** `&`，代表按位与，即需要同时满足所有类型。

声明一个交叉类型：

```typescript
interface NameStruct {
  name: string;
}

interface AgeStruct {
  age: number;
}

type ProfileStruct = NameStruct & AgeStruct;

const profile: ProfileStruct = {
  name: 'test',
  age: 13
}
```

如果是原始类型的合并：

```typescript
type StrAndNum = string & number; // never
```

对于对象类型的交叉类型，其**内部的同名属性类型同样会按照交叉类型进行合并**：

```typescript
type Struct1 = {
  primitiveProp: string;
  objectProp: {
    name: string;
  }
}

type Struct2 = {
  primitiveProp: number;
  objectProp: {
    age: number;
  }
}

type Composed = Struct1 & Struct2;
type PrimitivePropType = Composed['primitiveProp']; // never
type ObjectPropType = Composed['objectProp']; // { name: string; age: number; }
```

如果是两个联合类型组成的交叉类型

```typescript
type UnionIntersection1 = (1 | 2 | 3) & (1 | 2); // 1 | 2
type UnionIntersection2 = (string | number | symbol) & string; // string
```

### 索引类型

索引类型指的不是某一个特定的类型工具，它其实包含三个部分：**索引签名类型**、**索引类型查询**与**索引类型访问**。

这三者是独立的类型工具，唯一的共同点是，它们都通过索引的形式来进行类型操作，但**索引签名类型是声明，后两者则是读取**。

**索引签名类型**

索引签名类型主要指的是在接口或类型别名中，通过以下语法来**快速声明一个键值类型一致的类型结构**：

```typescript
interface AllStringTypes {
  [key: string]: string;
}

type AllStringTypes = {
  [key: string]: string;
}
```

此时，即使还没声明具体的属性，对于这些类型结构的属性访问也将全部被视为 string 类型：

```typescript
interface AllStringTypes {
  [key: string]: string;
}

type PropType1 = AllStringTypes['test']; // string
type PropType2 = AllStringTypes['123']; // string
```

在上述例子中声明的键的类型为 string（`[key: string]`），这意味着这个类型结构的变量中只能声明字符串类型的键：

```typescript
interface AllStringTypes {
  [key: string]: string;
}

// 在 JS 中，对于 obj[prop] 形式的访问会将数字索引转化为字符串索引访问
// 因此，obj[123] 和 obj['123'] 的效果是一致的。
// 类似的，symbol 类型也是如此。
const foo: AllStringTypes = {
  'test': '123',
  123: 'test',
  [Symbol('ddd')]: 'symbol',
}
```

索引签名类型也可以和具体的键值对类型声明并存，但这时这些具体的键值类型也需要符合索引签名类型的声明。

```typescript
interface AllStringTypes {
  propA: number; // 类型“number”的属性“propA”不能赋给“string”索引类型“string”。
  [key: string]: string;
}
```

这里的符合即指子类型，因此也包括联合类型：

```typescript
interface StringOrNumberTypes {
  propA: number;
  propB: string;
  [key: string]: string | number;
}
```

索引签名类型的一个常见场景是在重构 JS 代码的时候，为内部属性较多的对象声明一个 any 的索引签名类型，以此来暂时支持对类型未明确属性的访问，并在后续一点点补全类型。

```typescript
interface AnyTypeHere {
  [key: string]: any;
}

const foo: AnyTypeHere['test'] = true;
```

**索引类型查询**

`keyof` 操作符，可以将对象中的所有键转换为对应字面量类型，然后再组合成联合类型。

注意，这里**并不会将数字类型的键名转换为字符串类型字面量，而是仍然保持为数字类型字面量**。

```typescript
interface Foo {
  test: 1,
  123: 2
}

type FooKeys = keyof Foo; // 'test' | 123
```

以下伪代码可以模拟“**从键名到联合类型**”的过程。

```typescript
type FooKeys = Object.keys(Foo).join(" | ");
```

除了应用在已知的对象类型结构上以外，还可以直接 `keyof any` 来生产一个联合类型，它由所有可用作对象键值的类型组成：`string | number | symbol`。

**keyof 的产物必定是一个联合类型**。

**索引类型访问**

在 JS 中可以通过 `obj[expression]` 的方式来动态访问一个对象属性（即计算属性），expression 表达式会先被执行，然后使用返回值来访问属性。在 TS 中也可以通过类似的方式，只不过这里的 expression 要换成类型。

```typescript
interface NumberRecord {
  [key: string]: number;
}

type PropType = NumberRecord[string]; // number
```

比较直观的例子是通过字面量类型来进行索引类型访问：

```typescript
interface Foo {
  propA: number;
  propB: boolean;
}

type PropAType = Foo['propA']; // number
type PropBType = Foo['propB']; // boolean
```

这里的`'propA'` 和 `'propB'` 都是字符串字面量类型，而不是一个 JS 字符串值。索引类型查询的本质其实就是，通过键的字面量类型访问这个键对应的键值类型。

```typescript
interface Foo {
  propA: number;
  propB: boolean;
  propC: string;
}

type PropTypeUnion = Foo[keyof Foo]; // string | number | boolean
```

索引类型查询、索引类型访问通常会和映射类型一起搭配使用，前两者负责访问键，而映射类型在其基础上访问键值类型。

注意，在未声明索引签名类型的情况下，不能使用 `NUmberRecord[string]`这种原始类型的访问方式，只能通过键名的字面量类型来进行访问。

```typescript
interface Foo {
  propA: number;
}

// 类型“Foo”没有匹配的类型“string”的索引签名。
type PropAType = Foo[string];
```

### 映射类型

映射类型的主要作用即是**基于键名映射到键值类型**。

```typescript
// 使用 keyof 获得这个对象类型的键名组成字面量联合类型，
// 然后通过映射类型（即此处的 in 关键字）将这个联合类型的每一个成员映射出来。
// 将其键值类型设置为 string。
type Stringify<T> = {
  [K in keyof T]: string;
};

interface Foo {
  prop1: string;
  prop2: number;
  prop3: boolean;
  prop4: () => void;
}

type StringfiedFoo = Stringify<Foo>;

// 等价于
interface StringifiedFoo {
  prop1: string;
  prop2: string;
  prop3: string;
  prop4: string;
}
```

以伪代码形式说明：

```typescript
const StringifiedFoo = {};
for (const k of Object.keys(Foo)){
	StringifiedFoo[k] = string;
}
```

可以利用映射类型拿到键值类型：

```typescript
type Clone<T> = {
	[K in keyof T]: T[K];
};
```

这里的 `T[K]` 就是上面说到的索引类型访问，使用键的字面量类型访问到了键值的类型，这里就相当于克隆了一个接口。需要注意的是，这里其实只有 `K in` 属于映射类型的语法，`keyof T` 属于 keyof 操作符，`[K in keyof T]` 的 `[]` 属于索引签名类型，`T[K]` 属于索引类型访问。

### 总结

| 类型工具     | 创建新类型的方式                                           | 常见搭配           |
| ------------ | ---------------------------------------------------------- | ------------------ |
| 类型别名     | 将一组类型/类型结构封装，作为一个新的类型                  | 联合类型、映射类型 |
| 工具类型     | 在类型别名的基础上，基于泛型去动态创建新类型               | 基本所有类型工具   |
| 联合类型     | 创建一组类型集合，满足其中一个即满足这个联合类型           | 类型别名、工具类型 |
| 交叉类型     | 创建一组类型集合，满足其中所有类型才满足映射联合类型       | 类型别名、工具类型 |
| 索引签名类型 | 声明一个拥有任意属性，键值类型一致的接口结构               | 映射类型           |
| 索引类型查询 | 从一个接口结构，创建一个由其键名字符串字面量组成的联合类型 | 映射类型           |
| 索引类型访问 | 从一个接口结构，使用键名字符串字面量访问到对应的键值类型   | 类型别名、映射类型 |
| 映射类型     | 从一个联合类型依次映射到其内部的每一个类型                 | 工具类型           |

## TypeScript 类型工具（下）

类型别名、联合类型与交叉类型、索引类型与映射类型的作用是**基于已有的类型去创建出新的类型**。

类型查询操作符和类型守卫的主要作用是**类型的安全保障**。

### 类型查询操作符：typeof

TS 中存在两种功能不同的 typeof 操作符。最常见的一种 typeof 操作符就是 JS 中用于检查变量类型的 typeof。除此之外，TS 还新增了用于类型查询的 typeof，这个 typeof 返回的是一个 TS 类型：

```typescript
const str = 'test';

const obj = { name: 'test' };

const nullVar = null;
const undefinedVar = undefined;

const func = (input: string) => {
  return input.length > 10;
}

type Str = typeof str; // 'test'
type Obj = typeof obj; // { name: string; }
type Null = typeof nullVar; // null
type Undefined = typeof undefinedVar; // undefined
type Func = typeof func; // (input: string) => boolean
```

不仅可以直接在类型标注中使用 typeof，还能在工具类型中使用 typeof。

```typescript
const func = (input: string) => {
  return input.length > 10;
}

const func2: typeof func = (name: string) => {
  return name === 'test'
}
```

ReturnType 这个工具类型会返回一个函数类型中返回值位置的类型：

```typescript
const func = (input: string) => {
  return input.length > 10;
}

// boolean
type FuncReturnType = ReturnType<typeof func>;
```

绝大部分情况下，typeof 返回的类型是**最窄的推导程度（即到字面量类型的级别）**。

为了更好的隔离类型层和逻辑层，类型查询操作符后不允许使用表达式：

```typescript
const isInputValid = (input: string) => {
  return input.length > 10;
}

// 不允许表达式
let isValid: typeof isInputValid("linbudu");
```

### 类型守卫

TS 提供了强大的类型推导能力，会随着代码逻辑不断尝试收窄类型，这一能力称之为**类型的控制流分析**。

类型控制流分析做不到跨函数上下文来进行类型的信息收集：

```typescript
function isString(input: unknown): boolean {
  return typeof input === 'string';
}

function foo(input: string | number) {
  if (isString(input)) {
    // 类型“string | number”上不存在属性“replace”。
    // 类型“number”上不存在属性“replace”。
    (input).replace('test', 'tested')
  }
  if (typeof input === 'number') { }
}
```

为了解决这一类型控制流分析的能力不足，TS 引入了 **is 关键字**来显式地提供类型信息：

```typescript
function isString(input: unknown): input is string {
  return typeof input === 'string';
}

function foo(input: string | number) {
  if (isString(input)) {
    // 正确了
    (input).replace('test', 'tested')
  }
  if (typeof input === 'number') { }
}
```

isString 函数称为类型守卫，在它的返回值中，不再使用 boolean 作为类型标注，而是使用 `input is string`。

**is 关键字 + 预期类型**，即如果这个函数成功返回为 true，那么 is 关键字前这个入参的类型，就会**被这个类型守卫调用方后续的类型控制流分析收集到**。

类型守卫函数并不会对判断逻辑和实际类型的关联进行检查：

```typescript
function isString(input: unknown): input is number {
  return typeof input === 'string';
}

function foo(input: string | number) {
  if (isString(input)) {
    // 报错，在这里变成了 number 类型
    (input).replace('test', 'tested')
  }
  if (typeof input === 'number') { }
}
```

从这个角度看，类型守卫有些类似于类型断言，但类型守卫更宽松，也更信任你一些。指定什么类型，就是什么类型。

**基于 in 与 instanceof 的类型保护**

`in` 操作符在 JS 中就已经存在了，它可以通过  `key in object` 的方式来判断 key 是否存在于 object 或其原型链上（返回 true 说明存在）。

TS 中可以使用它来保护类型：

```typescript
interface Foo {
  foo: string;
  fooOnly: boolean;
  shared: number;
}

interface Bar {
  bar: string;
  barOnly: boolean;
  shared: number;
}

function handle(input: Foo | Bar) {
  if ('foo' in input) {
    input.fooOnly;
  } else {
    input.barOnly;
  }
}
```

如果用 shared 来区分，就会发现在分支代码块中 input 仍然是初始的联合类型：

```typescript
function handle(input: Foo | Bar) {
  if ('shared' in input) {
    // 类型“Foo | Bar”上不存在属性“fooOnly”。
    // 类型“Bar”上不存在属性“fooOnly”。
    input.fooOnly;
  } else {
    // 类型“never”上不存在属性“barOnly”。
    input.barOnly;
  }
}
```

因为 Foo 和 Bar 都满足 `'shared' in input` 这个条件，因此在 if 分支中，Foo 和 Bar 都会被保留，那在 else 分支中就只剩下 never 类型。

由于 foo/bar 和 fooOnly/barOnly 是各个类型独有的属性，因此可以作为**可辨识属性**。Foo 与 Bar 可以称为**可辨识联合类型**。

这个可辨识属性也可以是结构层面的，甚至可以是共同属性的字面量类型差异：

```typescript
// 可辨识属性为结构层面
function ensureArray(input: number | number[]): number[] {
  if (Array.isArray(input)) {
    return input;
  } else {
    return [input];
  }
}

// 可辨识属性是共同属性的字面量类型差异
interface Foo {
  kind: 'foo';
  diffType: string;
  fooOnly: boolean;
  shared: number;
}

interface Bar {
  kind: 'bar';
  diffType: number;
  barOnly: boolean;
  shared: number;
}

function handle1(input: Foo | Bar) {
  if (input.kind === 'foo') {
    input.fooOnly;
  } else {
    input.barOnly;
  }
}
```

对于同名但不同类型的属性，需要使用字面量类型的区分，并不能使用简单的 typeof。

```typescript
function handle2(input: Foo | Bar) {
  // 报错，并没有起到区分的作用，在两个代码块中都是 Foo | Bar
  if (typeof input.diffType === 'string') {
    input.fooOnly;
  } else {
    input.barOnly;
  }
}
```

也可以使用 instanceof 来进行类型保护：

```typescript
class FooBase {}

class BarBase {}

class Foo extends FooBase {
  fooOnly() {}
}

class Bar extends BarBase {
  barOnly() {}
}

function handle(input: Foo | Bar) {
  if (input instanceof FooBase) {
    input.fooOnly();
  } else {
    input.barOnly();
  }
}
```

除了使用 is 关键字的类型守卫外，还存在使用 asserts 关键字的类型断言守卫。

**类型断言守卫**

断言守卫和类型守卫最大的不同点在于，在判断条件不通过时，断言守卫需要抛出一个错误，类型守卫只需要剔除掉预期的类型。

```typescript
function assert(condition: any, msg?: string): asserts condition {
  if(!condition) {
    throw new Error(msg);
  }
}
```

这里使用的是 `asserts condition`，而 condition 来自于实际逻辑，这也意味着**将 condition 这一逻辑层面的代码，作为了类型层面的判断依据**，相当于在返回值类型中使用一个逻辑表达式进行了类型标注。

对于 `assert(typeof name === 'number');` 这么一个断言，如果函数成功返回，就说明其后续的代码中 condition 均成立，也就是 name 变成了一个 number 类型。

还可以结合使用 is 关键字来提供进一步的类型守卫能力：

```typescript
let name1: any = 'test';

function assertIsNumber(val: any): asserts val is number {
  if (typeof val !== 'number') {
    throw new Error('Not a number!');
  }
}

assertIsNumber(name1);

// number 类型
name1.toFixed();
```

### 扩展

**接口的合并**

接口还能够使用继承进行合并，在继承时子接口可以声明同名属性，但不能覆盖掉父接口中的此属性。**子接口中的属性类型需要能够兼容（extends）父接口中的属性类型。**

```typescript
interface Struct1 {
  primitiveProp: string;
  objectProp: {
    name: string;
  };
  unionProp: string | number;
}

// 接口“Struct2”错误扩展接口“Struct1”。
interface Struct2 extends Struct1 {
  // “primitiveProp”的类型不兼容。不能将类型“number”分配给类型“string”。
  primitiveProp: number;
  // 属性“objectProp”的类型不兼容。
  objectProp: {
    age: number;
  };
  // 属性“unionProp”的类型不兼容。
  // 不能将类型“boolean”分配给类型“string | number”。
  unionProp: boolean;
}
```

类似，如果直接声明多个同名接口，虽然接口会进行合并，但这些同名属性的类型仍然需要兼容，此时的表现和显式扩展接口基本一致：

```typescript
interface Struct1 {
  primitiveProp: string;
}

interface Struct1 {
  // 后续属性声明必须属于同一类型。
  // 属性“primitiveProp”的类型必须为“string”，但此处却为类型“number”。
  primitiveProp: number;
}
```

这也是接口和类型别名的重要差异之一。

接口和类型别名之间的合并，规则一致，如接口继承类型别名，和类型别名使用交叉类型合并接口：

```typescript
type Base = {
  name: string;
};

interface IDerived extends Base {
  // 报错，就像继承接口一样需要类型兼容
  name: number;
  age: number;
}

interface IBase {
  name: string;
}

// 合并后的 name 同样是 never 类型
type Drived = IBase & {
  name: number;
};
```

**更强大的可辨识联合类型分析**

类型控制流分析是在不断增强的，这里说的增强包括了**对可辨识联合类型的分析能力**。如下这个例子在 4.6 版本之前会报错。

```typescript
type Args = ['a', number] | ['b', string];

type Func = (...args: ['a', number] | ['b', string]) => void;

const f1: Func = (kind, payload) => {
  if (kind === 'a') {
    // 仍然是 string | number
    payload.toFixed();
  }
  if (kind === 'b') {
    // 仍然是 string | number
    payload.toUpperCase();
  }
};
```

在 4.6 版本则对这一情况下的 联合类型辨识（即元组） 做了支持。

## 类型编程基石：泛型

### 类型别名中的泛型

类型别名如果声明了泛型坑位，就等价于一个接受参数的函数：

```typescript
type Factory<T> = T | number | string;
```

上面这个类型别名的本质就是一个函数，T 就是它的变量，返回值则是一个包含 T 的联合类型，伪代码如下：

```typescript
function Factory(typeArg){
	return [typeArg, number, string]
}
```

类型别名中的泛型大多是用来进行工具类型封装。

```typescript
type Stringify<T> = {
  [K in keyof T]: string;
};

type Clone<T> = {
  [K in keyof T]: T[K];
};
```

Stringify 会将一个对象类型的所有属性类型置为 string，而 Clone 则会进行类型的完全复制。

```typescript
type Partial<T> = {
	[P in keyof T]?: T[P];
};
```

这样就获得了一个属性均为可选的山寨版：

```typescript
interface IFoo {
	prop1: string;
	prop2: number;
	prop3: boolean;
	prop4: () => void;
}

type PartialIFoo = Partial<IFoo>;

// 等价于
interface PartialIFoo {
	prop1?: string;
	prop2?: number;
	prop3?: boolean;
	prop4?: () => void;
}
```

类型别名与泛型的结合中，除了映射类型、索引类型等类型工具以外，还有一个非常重要的工具：条件类型。

```typescript
type IsEqual<T> = T extends true ? 1 : 2;

type A = IsEqual<true>; // 1
type B = IsEqual<false>; // 2
type C = IsEqual<'test'>; // 2
```

在条件类型参与的情况下，通常泛型会被作为条件类型中的判断条件（`T extends Condition` 或者 `Type extends T`）以及返回值（即`:`两端的值）。

**泛型约束与默认值**

泛型可以设置默认值：

```typescript
type Factory<T = boolean> = T | number | string;

const foo: Factory = false;
```

除了声明默认值以外，泛型还可以进行**泛型约束**。

```typescript
function add(source: number, add: number){
 if(typeof source !== 'number' || typeof add !== 'number') {
  throw new Error('Invalid arguments!')
 }

 return source + add;
}
```

在泛型中可以使用 `extends` 关键字来约束传入的泛型参数必须符合要求。`A extends B` 意味着 **A 是 B 的子类型**。

* 更精确，如字面量类型是对应原始类型的子类型，联合类型子集均为联合类型的子类型。
* 更复杂，如 `{ name: string }` 是 `{}` 的子类型因为在 `{}` 的基础上增加了额外的类型，基类与派生类同理。

可以通过 extends 来标明其类型约束。

```typescript
type ResStatus<ResCode extends number> = ResCode extends 10000 | 10001 | 10002
  ? 'success'
  : 'failure';

type Res1 = ResStatus<10000>; // 'success'
type Res2 = ResStatus<20000>; // 'failure'

type Res3 = ResStatus<'10000'>; // 类型“string”不满足约束“number”。
```

可以结合默认值使用：

```typescript
type ResStatus<ResCode extends number = 10000> = ResCode extends 10000 | 10001 | 10002
  ? 'success'
  : 'failure';

type Res4 = ResStatus; // 'success'
```

在 TS 中，泛型参数存在默认约束（下面的函数泛型、Class 泛型中也是）。这个默认约束值在 TS 3.9 版本以前是 any，而在 3.9 版本以后则为 unknown。

**多泛型关联**

不仅可以同时传入多个泛型参数，还可以让这几个泛型参数之间也存在联系。

```typescript
type Conditional<Type, Condition, TruthyResult, FalsyResult> = 
  Type extends Condition ? TruthyResult : FalsyResult;

type Result1 = Conditional<'test', string, 'passed!', 'rejected!'>; // 'passed!'

type Result2 = Conditional<'test', boolean, 'passed!', 'rejected!'>; // 'rejected!'
```

**多泛型参数其实就像接受更多参数的函数，其内部的运行逻辑（类型操作）会更加抽象，表现在参数（泛型参数）需要进行的逻辑计算（类型操作）会更加复杂。**

### 对象类型中的泛型

如下所示，是一个响应类型结构的泛型处理：

```typescript
interface IRes<TData = unknown> {
  code: number;
  error?: string;
  data: TData;
}
```

这个接口描述了一个通用的相应类型结构，预留出了实际响应数据的泛型坑位，然后在请求函数中就可以传入特定的响应类型了。

```typescript
interface IUserProfileRes {
  name: string;
  homepage: string;
  avatar: string;
}

function fetchUserProfile(): Promise<IRes<IUserProfileRes>> {}

type StatusSucceed = boolean;
function handleOperation(): Promise<IRes<StatusSucceed>> {}
```

泛型嵌套的场景也非常常用，比如对存在分页结构的数据，我们可以将其分页的响应结构抽离出来：

```typescript
interface IPaginationRes<TItem = unknown> {
  data: TItem[];
  page: number;
  totalCount: number;
  hasNextPage: boolean;
}

function fetchUserProfileList(): Promise<IRes<IPaginationRes<IUserProfileRes>>> {}
```

这些结构看起来很复杂，但其实就是简单的泛型参数填充而已。

### 函数中的泛型

假设有一个函数可以接受多个类型的参数并进行对应处理，此时应如何对函数进行类型声明？

首先肯定不能用 any，那么如果使用联合类型呢？

```typescript
function handle(input: string | number | {}): string | number | {} {}
```

但在调用之后发现调用结果的类型仍然是一个宽泛的联合类型 `string | number | {}`。

此时就应该使用泛型了。

```typescript
function handle<T>(input: T): T {}
```

这样，在这个函数接收到参数的时候，T 会自动地被填充为这个参数的类型。这也就意味着不再需要预先确定参数的可能类型了，而在返回值与参数类型关联的情况下，也可以通过泛型参数来进行运算。

在基于参数类型进行填充泛型时，其类型信息会被推断到尽可能精确的程度。

```typescript
function handle<T>(input: T): T {}

const author = 'test'; // 使用 const 声明，被推导为 "test"

let authorAge = 18; // 使用 let 声明，被推导为 number

handle(author); // 填充为字面量类型 "test"
handle(authorAge); // 填充为基础类型 number
```

再看一个例子

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/251765b69571411eb607680aff6f7c5a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

函数中的泛型同样存在约束与默认值。

```typescript
function handle<T extends string | number>(input: T): T {}
```

```typescript
function swap<T extends number, U extends number>([start, end]: [T, U]): [U, T] {
	return [end, start];
}
```

多泛型关联也是如此，以 lodash 的 pick 函数为例：

```typescript
const object = { 'a': 1, 'b': '2', 'c': 3 };

_.pick(object, ['a', 'c']);
// => { 'a': 1, 'c': 3 }
```

这个函数很明显需要在泛型层面声明关联，即数组中的元素只能来自于对象的属性名（组成的字面量联合类型！）。

```typescript
pick<T extends object, U extends keyof T>(object: T, ...props: Array<U>): Pick<T, U>;
```

这里 T 声明约束为对象类型，而 U 声明约束为 `keyof T`。同时对应的，其返回值类型中使用了 `Pick<T, U>` 这一工具类型，它与 pick 函数的作用一致，对一个对象结构进行裁剪。

函数的泛型参数也会被内部的逻辑消费。

```typescript
function handle<T>(payload: T): Promise<[T]> {
  return new Promise<[T]>((res, rej) => {
    res([payload]);
  });
}
```

对于箭头函数的泛型，其书写方式是这样的：

```typescript
const handle = <T>(input: T): T => {}
```

需要注意的是在 tsx 文件中泛型的尖括号可能会报错，编译器无法识别这是一个组件还是一个泛型，此时可以让它长得更像泛型一点：

```typescript
const handle = <T extends any>(input: T): T => {}
```

函数的泛型是日常使用较多的一部分，更明显的体现了**泛型在调用时被填充这一特性**。

通常使用类型别名来对已经确定的类型结构进行类型操作。

需要注意的是，不要为了使用泛型而用泛型：

```typescript
function handle<T>(arg: T): void {
	console.log(arg);
};
```

在这个函数中，泛型参数 T 没有被返回值消费，也没有被内部的逻辑消费，这种情况下及时随着调用填充了泛型参数，也是没有意义的。因此，这里完全可以使用 any 来进行类型标注。

### Class 中的泛型

Class 中的泛型和函数中的泛型非常类似，只不过函数中泛型参数的消费方是参数和返回值类型，Class 中的泛型消费方则是属性、方法、乃至装饰器等。同时 Class 内的方法还可以再声明自己独有的泛型参数。

```typescript
class Queue<TElementType> {
  private _list: TElementType[];

  constructor(initial: TElementType[]) {
    this._list = initial;
  }

  // 入队一个队列泛型子类型的元素
  enqueue<TType extends TElementType>(ele: TType): TElementType[] {
    this._list.push(ele);
    return this._list;
  }

  // 入队一个任意类型元素（无需为队列泛型子类型）
  enqueueWithUnknownType<TType>(element: TType): (TElementType | TType)[] {
    return [...this._list, element];
  }

  // 出队
  dequeue(): TElementType[] {
    this._list.shift();
    return this._list;
  }
}
```

其中，enqueue 方法的入参类型 TType 被约束为队列类型的子类型，而 enqueueWithUnknownType 方法中的 TType 类型参数则不会受此约束，它会在其被调用时再对应地填充，同时也会在返回值类型中被使用。

**内置方法中的泛型**

TS 中为非常多的内置对象都预留了泛型坑位，如 Promise 中

```typescript
function p() {
  return new Promise<boolean>((resolve, reject) => {
    resolve(true);
  });
}
```

在填充 Promise 的泛型以后，其内部的 resolve 方法也自动填充了泛型，而在 TS 内部的 Promise 类型声明中同样是通过泛型实现：

```typescript
interface PromiseConstructor {
	resolve<T>(value: T | PromiseLike<T>): Promise<T>;
}

declare var Promise: PromiseConstructor;
```

还有数组 `Array<T>` 当中，其泛型参数代表数组的元素类型，几乎贯穿所有的数组方法：

```typescript
const arr: Array<number> = [1, 2, 3];

// 类型“string”的参数不能赋给类型“number”的参数。
arr.push('test');
// 类型“string”的参数不能赋给类型“number”的参数。
arr.includes('test');

// number | undefined
arr.find(() => false);

// 第一种 reduce
arr.reduce((prev, curr, idx, arr) => {
  return prev;
}, 1);

// 第二种 reduce
// 报错：不能将 number 类型的值赋值给 never 类型
arr.reduce((prev, curr, idx, arr) => {
  return [...prev, curr]
}, []);
```

reduce 方法是相对特殊的一个，它的类型声明存在几种不同的重载：

* 当你不传入初始值时，泛型参数会从数组的元素类型中进行填充。
* 当你传入初始值时，如果初始值的类型与数组元素类型一致，则使用数组的元素类型进行填充。即这里第一个 reduce 调用。
* 当你传入一个非数组元素类型的初始值，比如这里的第二个 reduce 调用，reduce 的泛型参数会默认从这个初始值推导出的类型进行填充，如这里是 `never[]`。

其中第三种情况也就意味着**信息不足，无法推导出正确的类型**，可以手动传入泛型参数来解决：

```typescript
arr.reduce<number[]>((prev, curr, idx, arr) => {
	return prev;
}, []);
```

在 React 中，同样可以找到无处不在的泛型坑位：

```typescript
const [state, setState] = useState<number[]>([]);
// 不传入默认值，则类型为 number[] | undefined
const [state, setState] = useState<number[]>();

// 体现在 ref.current 上
const ref = useRef<number>();

const context = createContext<ContextType>({});
```

## 结构化类型系统

TS 的类型系统特性：结构化类型系统。

### 结构化类型系统

TS 比较两个类型并非通过类型的名称，而是比较这两个类型上实际拥有的属性与方法。如果一个类型完全实现了另外一个类型并添加了额外的方法或属性，则可以认为这个类型是另一类型的子类。

对于结构中的函数类型（即方法）进行比较的时候，同样存在类型的兼容性比较。

严格来说，鸭子类型系统和结构化类型系统并不完全一致，结构化类型系统意味着基于完全的类型结构来判断类型兼容性，而鸭子类型则只基于运行时访问的部分来决定。

需要注意，TS 本身并不是在运行时进行类型检查，同时官方文档认为这两个概念是一致的。因此，可以认为鸭子类型与结构化类型是同一概念。

除了基于类型结构进行兼容性判断的结构化类型系统以外，还有一种基于类型名进行兼容性判断的类型系统，标称类型系统。

### 标称类型系统

标称类型系统要求两个可兼容的类型，其名称必须是完全一致的。

对于标称类型系统，父子类型关系只能通过显式的继承来实现，称为标称子类型。

### 在 TS 中模拟标称类型系统

在 TS 中实现需要为类型额外附加元数据。

可以通过交叉类型的方式来实现信息的附加。

```typescript
export declare class TagProtector<T extends string> {
  protected __tag__: T;
}

export type Nominal<T, U extends string> = T & TagProtector<U>;
```

在这里使用 TagProtector 声明了一个具有 `protected` 属性的类，使用它来携带额外的信息，并和原本的类型合并到一起，就得到了 Nominal 工具类型。

```typescript
export type CNY = Nominal<number, 'CNY'>;

export type USD = Nominal<number, 'USD'>;

const CNYCount = 100 as CNY;

const USDCount = 100 as USD;

function addCNY(source: CNY, input: CNY) {
  return (source + input) as CNY;
}

addCNY(CNYCount, CNYCount);

// 报错了！
addCNY(CNYCount, USDCount);
```

这一实现方式本质上只在类型层面做了数据的处理，在运行时无法进行进一步的限制。我们还可以从逻辑层面入手进一步确保安全性：

```typescript
class CNY {
  private __tag!: void;
  constructor(public value: number) {}
}

class USD {
  private __tag!: void;
  constructor(public value: number) {}
}
```

相应的，现在使用方式也要进行变化：

```typescript
const CNYCount = new CNY(100);
const USDCount = new USD(100);

function addCNY(source: CNY, input: CNY) {
  return (source.value + input.value);
}

addCNY(CNYCount, CNYCount);
// 报错了
addCNY(CNYCount, USDCount);
```

这两种方式的本质都是通过非公开（即 `private`/ `protected`）的额外属性实现了类型信息的附加，从而使得结构化类型系统将结构一致的两个类型也视为不兼容的。另外，在 type-fest 中也通过 [Opaque Type](https://codemix.com/opaque-types-in-javascript/) 支持了类似的功能，其实现如下：

```typescript
declare const tag: unique symbol;

declare type Tagged<Token> = {
	readonly [tag]: Token;
};

export type Opaque<Type, Token = unknown> = Type & Tagged<Token>;
```

总结：在 TS 中我们可以通过类型或者逻辑的方式来模拟标称类型，这两种方式其实没有明显的优劣之分，基于类型实现更加轻量，代码逻辑不会受到影响，但难以进行额外的逻辑检查工作。而使用逻辑实现稍显繁琐，但能够更进一步或更细致的约束。

### 扩展

**类型、类型系统与类型检查**

对于类型、类型系统、类型检查，可以认为它们是不同的概念。

* 类型：限制了数据的可用操作、意义、允许的值的集合，总的来说就是**访问限制**与**赋值限制**。
* 类型系统：一组为变量、函数等结构分配、实施类型的规则，通过显式地指定或类型推导来分配类型。同时类型系统也定义了如何判断类型之间的兼容性：在 TS 中即是结构化类型系统。
* 类型检查：确保**类型遵循类型系统下的类型兼容性**，对于静态类型语言，在**编译时**进行，而对于动态语言，则在**运行时**进行。TS 在编译时进行类型检查。

需要注意的是，静态类型与动态类型指的是类型检查发生的时机，并不等于这门语言的类型能力。比如 JS 实际上是动态类型语言，它的类型检查发生在运行时。

另一个静态类型与动态类型的重要区别体现在变量赋值时，如在 TS 中无法给一个声明为 number 的变量使用字符串赋值，因为这个变量在声明时的类型就已经确定了。而在 JS 中则没有这样的限制，可以随时切换一个变量的类型。

另外，在编程语言中还有强类型、弱类型的概念，它们体现在对变量类型检查的程度，如在 JS 中可以实现 `'1' - 1` 这种操作（通过隐式转换），这其实就是弱类型语言的显著特点之一。

## 类型系统层级

### 判断类型兼容性的方式

```typescript
type Result = 'test' extends string ? 1 : 2;
```

如果返回 1，则说明 `'test'` 为 string 的子类型。否则，说明不成立。需要注意，不成立并不意味着 string 就是 `test` 的子类型了。

还有一种备选的，通过赋值来进行兼容性检查的方式：

```typescript
declare let source: string;

declare let anyType: any;
declare let neverType: never;

anyType = source;

// 不能将类型“string”分配给类型“never”。
neverType = source;
```

对于变量a = 变量b，如果成立，意味着`<变量 b 的类型> extends <变量 a 的类型>` 成立，即 b 类型是 a 类型的子类型。

在需要判断多个类型的层级时，条件类型更为直观，而如果只是两个类型之间的兼容性判断时，使用类型声明则更好。

### 从原始类型开始

首先从原始类型、对象类型（统称为基础类型）和它们对应的字面量类型开始。

```typescript
type Result1 = 'test' extends string ? 1 : 2; // 1
type Result2 = 1 extends number ? 1 : 2; // 1
type Result3 = true extends boolean ? 1 : 2; // 1
type Result4 = { name: string } extends object ? 1 : 2; // 1
type Result5 = { name: 'test' } extends object ? 1 : 2; // 1
type Result6 = [] extends object ? 1 : 2; // 1
```

结论：**字面量类型 < 对应的原始类型**

### 向上探索，直到穹顶之上

**联合类型**

```typescript
type Result7 = 1 extends 1 | 2 | 3 ? 1 : 2; // 1
type Result8 = 'test' extends 'test' | 'test1' | 'test2' ? 1 : 2; // 1
type Result9 = true extends true | false ? 1 : 2; // 1
type Result10 = string extends string | false | number ? 1 : 2; // 1
```

结论：**字面量类型 < 包含此字面量类型的联合类型，原始类型 < 包含此原始类型的联合类型**。

如果一个联合类型都由同一个基础类型的类型字面量组成，此时情况就不一样了。

```typescript
type Result11 = 'string1' | 'string2' | 'string3' extends string ? 1 : 2; // 1
type Result12 = {} | (() => void) | [] extends object ? 1 : 2; // 1
```

结论：**同一基础类型的字面量联合类型 < 此基础类型**。

合并结论：**字面量类型 < 包含此字面量类型的联合类型（同一基础类型）< 对应的原始类型** 。

```typescript
// 2
type Result13 = 'string1' extends 'string1' | 'string2'
  ? 'string1' | 'string2' extends string
    ? 2
    : 1
  : 0;
```

**装箱类型**

```typescript
type Result14 = string extends String ? 1 : 2; // 1
type Result15 = String extends {} ? 1 : 2; // 1
type Result16 = {} extends object ? 1 : 2; // 1
type Result17 = object extends Object ? 1 : 2; // 1
```

在结构化类型系统的比较下，String 会被认为是 `{}` 的子类型。这里从 `string < {} < object` 看起来构建了一个类型链，但实际上 `string extends object` 并不成立：

```typescript
type Tmp = string extends object ? 1 : 2; // 2
```

由于结构化类型系统这一特性的存在，我们能得到一些看起来矛盾的结论：

```typescript
type Result16 = {} extends object ? 1 : 2; // 1
type Result18 = object extends {} ? 1 : 2; // 1

type Result17 = object extends Object ? 1 : 2; // 1
type Result20 = Object extends object ? 1 : 2; // 1

type Result19 = Object extends {} ? 1 : 2; // 1
type Result21 = {} extends Object ? 1 : 2; // 1
```

这里的 `{} extends` 和 `extends {}` 实际上是两种完全不同的比较方式。`{} extends object` 和 `{} extends Object` 意味着，`{}` 是 object 和 Object 的字面量类型，是从**类型信息的层面**出发的，即**字面量类型在基础类型之上提供了更详细的类型信息**。`object extends {}` 和 `Object extends {}` 则是从**结构化类型系统的比较**出发的，即 `{}` 作为一个一无所有的空对象，几乎可以被视作是所有类型的基类。

而 `object extends Object` 和 `Object extends object` 这两者的情况就要特殊一些，它们是因为“系统设定”的问题，Object 包含了所有除 Top Type 以外的类型（基础类型、函数类型等），object 包含了所有非原始类型的类型，即数组、对象与函数类型。

结论：**原始类型 < 原始类型对应的装箱类型 < Object 类型**。

**Top Type**

这里只有 any 和 unknown 。

```typescript
type Result22 = Object extends any ? 1 : 2; // 1
type Result23 = Object extends unknown ? 1 : 2; // 1

type Result24 = any extends Object ? 1 : 2; // 1 | 2
type Result25 = unknown extends Object ? 1 : 2; // 2

type Result26 = any extends 'test' ? 1 : 2; // 1 | 2
type Result27 = any extends string ? 1 : 2; // 1 | 2
type Result28 = any extends {} ? 1 : 2; // 1 | 2
type Result29 = any extends never ? 1 : 2; // 1 | 2
```

any 代表了任何可能的类型，当使用 `any extends` 时，它包含了“让条件成立的一部分”，以及“让条件不成立的一部分”。而从实现上说，在 TS 内部代码的条件类型处理中，如果接受判断的是 any，那么会直接**返回条件类型结果组成的联合类型**。

因此 `any extends string` 并不能简单地认为等价于以下条件类型：

```typescript
type Result30 = ("I'm string!" | {}) extends string ? 1 : 2; // 2
```

这种情况下，由于联合类型的成员并非均是字符串字面量类型，条件显然不成立。

**any 可以表达为任何类型**。

```typescript
type Result31 = any extends unknown ? 1 : 2; // 1
type Result32 = unknown extends any ? 1 : 2; // 1
```

结论：**Object < any / unknown**。

### 向下探索，直到万物虚无

首先能确认一定有个 never 类型，因为它代表了“虚无”的类型，一个根本不存在的类型。它是任何类型的子类型。

```typescript
type Result33 = never extends 'test' ? 1 : 2; // 1
```

可能又想到了一些特别的部分，比如 null、undefined、void。

```typescript
type Result34 = undefined extends 'test' ? 1 : 2; // 2
type Result35 = null extends 'test' ? 1 : 2; // 2
type Result36 = void extends 'test' ? 1 : 2; // 2
```

在 TS 中，void、undefined、null 都是切实存在、有实际意义的类型。

在关闭 `--strictNullCheckes` 的情况下，null 会被视为 string 等类型的子类型。

结论：**never < 字面量类型**。

### 类型层级链

```typescript
type TypeChain = never extends 'string'
  ? 'string' extends 'string' | '123'
  ? 'string' | '123' extends string
  ? string extends String
  ? String extends Object
  ? Object extends any
  ? any extends unknown
  ? unknown extends any
  ? 8
  : 7
  : 6
  : 5
  : 4
  : 3
  : 2
  : 1
  : 0
```

其返回结果为 8，也就意味着所有条件均成立。

结合上面的结构化类型系统与类型系统设定，可以构造出一条更长的类型层级链：

```typescript
type VerboseTypeChain = never extends 'linbudu'
  ? 'linbudu' extends 'linbudu' | 'budulin'
  ? 'linbudu' | 'budulin' extends string
  ? string extends {}
  ? string extends String
  ? String extends {}
  ? {} extends object
  ? object extends {}
  ? {} extends Object
  ? Object extends {}
  ? object extends Object
  ? Object extends object
  ? Object extends any
  ? Object extends unknown
  ? any extends unknown
  ? unknown extends any
  ? 8
  : 7
  : 6
  : 5
  : 4
  : 3
  : 2
  : 1
  : 0
  : -1
  : -2
  : -3
  : -4
  : -5
  : -6
  : -7
  : -8
```

结果仍然为 8 。

### 其他比较场景

* 对于基类和派生类，通常情况下**派生类会完全保留基类的结构**，而只是自己新增新的属性与方法。在结构化类型的比较下，其类型自然会存在子类型关系。更不用说派生类本身就是 extends 基类得到的。

* 联合类型判断如果是多个成员：

  ```typescript
  type Result36 = 1 | 2 | 3 extends 1 | 2 | 3 | 4 ? 1 : 2; // 1
  type Result37 = 2 | 4 extends 1 | 2 | 3 | 4 ? 1 : 2; // 1
  type Result38 = 1 | 2 | 5 extends 1 | 2 | 3 | 4 ? 1 : 2; // 2
  type Result39 = 1 | 5 extends 1 | 2 | 3 | 4 ? 1 : 2; // 2
  ```

  对于联合类型的类型层级比较，我们只需要比较**一个联合类型是否可被视为另一个联合类型的子集**，即**这个联合类型中所有成员在另一个联合类型中都能找到**。

* 数组和元组

  ```typescript
  type Result40 = [number, number] extends number[] ? 1 : 2; // 1
  type Result41 = [number, string] extends number[] ? 1 : 2; // 2
  type Result42 = [number, string] extends (number | string)[] ? 1 : 2; // 1
  type Result43 = [] extends number[] ? 1 : 2; // 1
  type Result44 = [] extends unknown[] ? 1 : 2; // 1
  type Result45 = number[] extends (number | string)[] ? 1 : 2; // 1
  type Result46 = any[] extends number[] ? 1 : 2; // 1
  type Result47 = unknown[] extends number[] ? 1 : 2; // 2
  type Result48 = never[] extends number[] ? 1 : 2; // 1
  ```

### 总结

基础的类型层级可以用以下这张图表示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8459e958e581479faa284390e3c6a09c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 类型里的逻辑运算：条件类型与 infer

### 条件类型基础

条件类型的语法类似于我们平时常用的三元表达式，基本语法如下（伪代码）：

```typescript
ValueA === ValueB ? Result1 : Result2;
TypeA extends TypeB ? Result1 : Result2;
```

需要注意的是，条件类型中使用 extends 判断类型的兼容性，而非判断类型的全等性。这是因为在类型层面中，对于能够进行赋值操作的两个变量，我们**并不需要它们的类型完全相等，只需要具有兼容性**，而两个完全相同的类型，其 extends 自然也是成立的。

条件类型绝大部分场景下会和泛型一起使用。

```typescript
type LiteralType<T> = T extends string ? 'string' : 'other';

type Res1 = LiteralType<'test'>; // "string"
type Res2 = LiteralType<123>; // "other"
```

同三元表达式可以嵌套一样，条件类型中也常见多层嵌套。

```typescript
export type LiteralType<T> = T extends string
        ? 'string'
        : T extends number
        ? 'number'
        : T extends boolean
        ? 'boolean'
        : T extends undefined
        ? 'undefined'
        : never;

type Res1 = LiteralType<'test'>; // "string"
type Res2 = LiteralType<123>; // "number"
type Res3 = LiteralType<true>; // "boolean"
```

思考一下如何标注这个函数的返回值类型：

```typescript
function universalAdd<T extends number | bigint | string>(x: T, y: T) {
  return x + (y as any);
}
```

当调用这个函数的时候，由于两个参数都引用了泛型参数 T ，因此泛型会被填充为一个联合类型：

```typescript
universalAdd(123, 1); // T 填充为 123 | 1
universalAdd('123', '1'); // T 填充为 "123" | "1"
```

那么此时的返回值类型就需要从这个字面量联合类型中推导回原本的基础类型。因此，可以使用嵌套的条件类型来进行字面量类型到基础类型的提取：

```typescript
function universalAdd<T extends number | bigint | string>(
        x: T,
        y: T
): LiteralType<T> {
        return x + (y as any);
}

export type LiteralType<T> = T extends string
        ? string
        : T extends number
        ? number
        : T extends bigint
        ? bigint
        : never;

universalAdd('test', '123'); // string
universalAdd(123, 123); // number
universalAdd(10n, 11n); // bigint
```

条件类型还可以用来对更复杂的类型进行比较，比如函数类型：

```typescript
type Func = (...args: any[]) => any;

type FunctionConditionType<T extends Func> = T extends (
  ...args: any[]
) => string
  ? 'A string return func!'
  : 'A non-string return func!';

// "A string return func!"
type StringResult = FunctionConditionType<() => string>;
// "A non-string return func!"
type NonStringResult1 = FunctionConditionType<() => boolean>;
// "A non-string return func!"
type NonStringResult2 = FunctionConditionType<() => number>;
```

在这里，我们的条件类型用于判断两个函数类型是否具有兼容性，而条件中并不限制参数类型，仅比较二者的返回值。

与此同时，存在泛型约束和条件类型两个 extends，它们产生作用的时机完全不同，泛型约束要求你传入符合结构的类型参数，相当于**参数校验**。而条件类型使用类型参数进行条件判断（就像 if else），相当于**实际内部逻辑**。

上面讲到的这些条件类型，本质上就是在泛型基于调用填充类型信息的基础上，新增了**基于类型信息的条件判断**。

### infer 关键字

如果希望拿到填充的类型信息的一部分，而不是只是用它来做条件呢？

TS 中支持通过 infer 关键字来**在条件类型中提取类型的某一部分信息**，比如上面我们要提取函数返回值类型的话，可以这么放：

```typescript
type Func = (...args: any[]) => any;

type FunctionConditionType<T extends Func> = T extends (
  ...args: any[]
) => infer R
  ? R
  : never;

type StringResult = FunctionConditionType<() => string>; // string
type NumberResult = FunctionConditionType<() => number>; // number
type BooleanResult = FunctionConditionType<() => boolean>; // boolean
```

`infer` 是 `inference` 的缩写，意为推断，如 `infer R` 中 `R` 就表示**待推断的类型**。`infer` 只能在条件类型中使用，因为我们实际上仍然需要**类型结构是一致的**。

这里的类型结构不局限于函数类型结构，还可以是数组：

```typescript
type Swap<T extends any[]> = T extends [infer A, infer B] ? [B, A] : T;

type SwapResult1 = Swap<[1, 2]>; // 符合元组结构，首尾元素替换[2, 1]
type SwapResult2 = Swap<[1, 2, 3]>; // 不符合结构，没有发生替换，仍是 [1, 2, 3]
```

但可以使用 rest 操作符来处理任意长度的情况：

```typescript
// 提取首尾两个
type ExtractStartAndEnd<T extends any[]> = T extends [
  infer Start,
  ...any[],
  infer End
] ? [Start, End]
  : T;

// 调换首尾两个
type SwapStartAndEnd<T extends any[]> = T extends [
  infer Start,
  ...infer Left,
  infer End
] ? [End, ...Left, Start]
  : T;

// 调换前两个
type SwapFirstTwo<T extends any[]> = T extends [
  infer Start1,
  infer Start2,
  ...infer Left
]
  ? [Start2, Start1, ...Left]
  : T;
```

也可以进行结构层面的转换。如数组到联合类型：

```typescript
type ArrayItemType<T> = T extends Array<infer ElementType> ? ElementType : never;

type ArrayItemTypeResult1 = ArrayItemType<[]>; // never
type ArrayItemTypeResult2 = ArrayItemType<string[]>; // string
type ArrayItemTypeResult3 = ArrayItemType<[string, number]>; // string | number
```

这里的 `[string, number]` 实际上等价于 `(string | number)[]`。

除了数组，infer 结构也可以是接口：

```typescript
// 提取对象的属性类型
type PropType<T, K extends keyof T> = T extends { [Key in K]: infer R }
  ? R 
  : never;

type PropTypeResult1 = PropType<{ name: string }, 'name'>; // string
type PropTypeResult2 = PropType<{ name: string; age: number }, 'name' | 'age'>; // string | number

// 反转键名与键值
type ReverseKeyValue<T extends Record<string, unknown>> = T extends Record<infer K, infer V>
  ? Record<V & string, K> : never;

type ReverseKeyValueResult1 = ReverseKeyValue<{ 'key': 'value' }>; // { 'value': 'key' }
```

在这里，结合了索引类型与映射类型，以及使用 `& string` 来确保属性名为 string 类型的小技巧。

如果不使用就会出现下面的问题：

```typescript
// 类型“V”不满足约束“string | number | symbol”。
type ReverseKeyValue<T extends Record<string, string>> = T extends Record<
  infer K, 
  infer V
>
  ? Record<V, K> 
  : never;
```

这是因为泛型参数 V 的来源是从键值类型推导出来的，TS 中这样对键值类型进行 infer 推导，将导致类型信息丢失，而不满足索引签名类型只允许 `string | number | symbol` 的要求。使用 `V & string` 这一形式，就确保了最终符合条件的类型参数 V 一定会满足 `string | never` 这个类型，因此可以被视为合法的索引签名类型。

infer 结构还可以是 Promise 结构。

```typescript
type PromiseValue<T> = T extends Promise<infer V> ? V : T;

type PromiseValueResult1 = PromiseValue<Promise<number>>; // number
type PromiseValueResult2 = PromiseValue<number>; // number，但并没有发生提取
```

就像条件类型可以嵌套，infer 关键字也经常被使用在嵌套的场景中，包括对类型结构深层信息的提取，以及对提取到类型信息的筛选等。

```typescript
type PromiseValueResult3 = PromiseValue<Promise<Promise<boolean>>>; // Promise<boolean>，只提取了一层
```

此时需要进行嵌套地提取：

```typescript
type PromiseValue<T> = T extends Promise<infer V> 
  ? V extends Promise<infer N> 
    ? N 
    : V 
  : T;
```

当然，在此时应该使用递归来处理任意嵌套深度：

```typescript
type PromiseValue<T> = T extends Promise<infer V> ? PromiseValue<V> : T;
```

条件类型在泛型地基础上支持了基于类型信息的动态条件判断，但无法直接消费填充类型信息，而 infer 关键字则为它补上了这一部分的能力。

### 分布式条件类型

**分布式条件类型，也称条件类型的分布式特性**，只不过是条件类型在满足一定情况下会执行的逻辑而已。

```typescript
type Condition<T> = T extends 1 | 2 | 3 ? T : never;

// 1 | 2 | 3
type Res1 = Condition<1 | 2 | 3 | 4 | 5>;

// never
type Res2 = 1 | 2 | 3 | 4 | 5 extends 1 | 2 | 3 ? 1 | 2 | 3 | 4 | 5 : never;
```

```typescript
type Naked<T> = T extends boolean ? 'Y' : 'N';
type Wrapped<T> = [T] extends [boolean] ? 'Y' : 'N';

// "N" | "Y"
type Res3 = Naked<number | boolean>;

// "N"
type Res4 = Wrapped<number | boolean>;
```

**对于属于裸类型参数的检查类型，条件类型会在实例化时期自动分发到联合类型上。**这里的裸类型参数其实指的是泛型参数是否完全裸露。

除了上面使用数组包裹泛型参数，还可以这样做：

```typescript
export type NoDistribute<T> = T & {};

type Wrapped<T> = NoDistribute<T> extends boolean ? 'Y' : 'N';

type Res1 = Wrapped<number | boolean>; // "N"
type Res2 = Wrapped<true | false>; // "Y"
type Res3 = Wrapped<true | false | 599>; // "N"
```

某些情况下需要包裹泛型参数来禁用掉分布式特性。最常见的场景还是联合类型的判断，即我们不希望进行联合类型成员的分布判断，而是希望直接判断这两个联合类型的兼容性判断。

```typescript
type CompareUnion<T, U> = [T] extends [U] ? true : false;

type CompareRes1 = CompareUnion<1 | 2, 1 | 2 | 3>; // true
type CompareRes2 = CompareUnion<1 | 2, 1>; // false
```

通过将参数与条件都包裹起来的方式，我们对联合类型的比较就变成了数组成员类型的比较，在此时就会严格遵守类型层级中联合类型的类型判断了（子集为其子类型）。

另外一种情况则是，当我们想判断一个类型是否为 never 时，也可以通过类似的手段：

**IsNever**

```typescript
type IsNever<T> = [T] extends [never] ? true : false;

type IsNeverRes1 = IsNever<never>; // true
type IsNeverRes2 = IsNever<"test">; // false
```

```typescript
// 直接使用，返回联合类型
type Tmp1 = any extends string ? 1 : 2; // 1 | 2

type Tmp2<T> = T extends string ? 1 : 2;
// 通过泛型参数传入，同样返回联合类型
type Tmp2Res = Tmp2<any>; // 1 | 2

// 如果判断条件是 any，那么仍然会进行判断
type Special1 = any extends any ? 1 : 2; // 1
type Special2<T> = T extends any ? 1 : 2;
type Special2Res = Special2<any>; // 1
```

never 仅在作为泛型参数时才会产生：

```typescript
// 直接使用，仍然会进行判断
type Tmp3 = never extends string ? 1 : 2; // 1

type Tmp4<T> = T extends string ? 1 : 2;
// 通过泛型参数传入，会跳过判断
type Tmp4Res = Tmp4<never>; // never

// 如果判断条件是 never，还是仅在作为泛型参数时才跳过判断
type Special3 = never extends never ? 1 : 2; // 1
type Special4<T> = T extends never ? 1 : 2;
type Special4Res = Special4<never>; // never
```

通过分布式条件类型可以轻易地进行集合之间地运算，比如交集：

```typescript
type Intersection<A, B> = A extends B ? A : never;

type IntersectionRes = Intersection<1 | 2 | 3, 2 | 3 | 4>; // 2 | 3
```

进一步的，当联合类型的组成是一个对象的属性名（`keyof IObject`），此时对这样的两个类型集合进行处理，得到属性名的交集，那我们就可以在此基础上获得两个对象类型结构的交集。

### 扩展：IsAny 与 IsUnknown

首先是 **IsAny**，不能通过 `any extends Type` 这样的形式来判断一个类型是否是 any。而是要利用 any 的另一个特性：身化万千。

```typescript
type IsAny<T> = 0 extends 1 & T ? true : false;

type Res1 = IsAny<any>; // true
type Res2 = IsAny<boolean>; // false
type Res3 = IsAny<'test'>; // false
```

`0 extends 1` 必然不成立，而交叉类型 `1 & T` 正常来说只有传入其本身、对应的原始类型、包含其本身的联合类型，才能得到一个有意义的值，并且这个值一定只可能是它本身。**但是当交叉类型的其中一个成员是 any，那么此时最终类型必然是 any**。

```typescript
type Tmp1 = 1 & any; // any
```

利用 `unknown extends T` 时仅有 T 为 any 或 unknown 时成立这一点，可以直接将类型收窄到 any 与 unknown，然后在去掉 any 类型时，仍然利用上面的身化万千的特性。

```typescript
type IsUnknown<T> = unknown extends T
  ? IsAny<T> extends true 
    ? false 
    : true
  : false;

type Res4 = IsUnknown<any>; // false
type Res5 = IsUnknown<unknown>; // true
```

## 内置工具类型基础

工具类型和类型编程并不完全等价。类型编程的复杂度体现在函数的重载与泛型约束方面。

### 工具类型的分类

内置的工具类型按照类型操作的不同，可以划分为这么几类：

* 对属性的修饰，包括对象属性和数组元素的可选/必选、只读/可写。我们将这一类统称为**属性修饰工具类型**。
* 对既有类型的裁剪、拼接、转换等，比如使用对一个对象类型裁剪得到一个新的对象类型，将联合类型结构转换到交叉类型结构。我们将这一类统称为**结构工具类型**。
* 对集合（即联合类型）的处理，即交集、并集、差集、补集。我们将这一类统称为**集合工具类型**。
* 基于 infer 的模式匹配，即对一个既有类型特定位置类型的提取，比如提取函数类型签名中的返回值类型。我们将其统称为**模式匹配工具类型**。
* 模板字符串专属的工具类型，比如神奇地将一个对象类型中的所有属性名转换为大驼峰的形式。这一类统称为**模板字符串工具类型**。

### 属性修饰工具类型

这一部分的工具类型主要使用**属性修饰、映射类型**与**索引类型**相关。

在内置工具类型中，访问性修饰工具类型包括以下三种：

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

其中，Partial 与 Required 可以认为是一对工具类型，它们的功能是相反的，而在实现上，它们的唯一差异是在索引类型签名处的可选修饰符，Partial 是 `?` ，即标记属性为可选，而 Require 则是 `-?`，相当于在原本属性上如果有 `?` 这个标记，则移除它。

Partial 也可以使用 `+?` 来显式的表示添加可选标记：

```typescript
type Partial<T> = {
  [P in keyof T]+?: T[P];
};
```

需要注意的是，可选标记不等于修改此属性类型为 `原类型 | undefined`，如以下的接口结构：

```typescript
interface Foo {
  optional: string | undefined;
  required: string;
}
```

如果声明一个对象去实现这个接口，它仍然会要求你提供 optional 属性：

```typescript
interface Foo {
  optional: string | undefined;
  required: string;
}

// 类型 "{ required: string; }" 中缺少属性 "optional"，但类型 "Foo" 中需要该属性。
const foo1: Foo = {
  required: '1',
};

const foo2: Foo = {
  required: '1',
  optional: undefined
};
```

对于结构声明来说，一个属性是否必须提供仅取决于其是否携带可选标记。即使使用 never 也无法标记这个属性为可选：

```typescript
interface Foo {
  optional: never;
  required: string;
}

const foo: Foo = {
  required: '1',
  // 不能将类型“string”分配给类型“never”。
  optional: '',
};
```

而类似 `+?`，Readonly 中也可以使用 `+readonly`：

```typescript
type Readonly<T> = {
	+readonly [P in keyof T]: T[P];
};
```

参考 Required，可以实现这样一个工具类型 Mutable 来将属性中的 readonly 修饰移除：

```typescript
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
}
```

**思考**

* 现在的属性修饰是浅层的，如果想将**嵌套在里面的对象类型**也进行修饰，需要怎么改进？
* 现在的属性修饰是全量的，如果只想**修饰部分属性**呢？这里的部分属性，可能是**基于传入已知的键名**来确定（比如属性a、b），也可能是**基于属性类型**来确定（比如所有函数类型的值）。

### 结构工具类型

这一部分的工具类型主要使用**条件类型**以及**映射类型**、**索引类型**。

结构工具类型又可以分为两类，**结构声明**和**结构处理**。

结构声明工具类型即快速声明一个结构，比如内置类型中的 Record：

```typescript
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

其中，`K extends keyof any` 即为键的类型，这里使用 `extends keyof any` 标明，传入的 K 可以是单个类型，也可以是联合类型，而 T 即为属性的类型。

```typescript
// 键名均为字符串，键值类型未知
type Record1 = Record<string, unknown>;
// 键名均为字符串，键值类型任意
type Record2 = Record<string, any>;
// 键名为字符串或数字，键值类型任意
type Record3 = Record<string | number, any>;
```

其中，`Record<string, unknown>` 和 `Record<string, any>` 是日常使用较多的形式，通常使用这两者来代替 object 。

在一些工具类库源码中其实还存在类似的结构声明工具类型，如：

```typescript
type Dictionary<T> = {
  [index: string]: T;
};

type NumbericDictionary<T> = {
  [index: number]: T;
};
```

Dictionary （字典）结构只需要一个作为属性类型的泛型参数即可。

对于结构处理工具类型，在 TS 中主要是 Pick、Omit 两位选手：

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

Pick 接受两个泛型参数，T 即是我们会进行结构处理的原类型（一般是对象类型），而 K 则被约束为 T 类型的键名联合类型。由于泛型约束是立即填充推导的，即你为一个泛型参数传入 Foo 类型以后，K 的约束条件会立刻被填充。

```typescript
type JobUnionType = 'teacher' | 'student' | 'worker';

interface Foo {
  name: string;
  age: number;
  job: JobUnionType;
}

type PickedFoo = Pick<Foo, "name" | "age">;
```

Pick 会将传入的联合类型作为需要保留的属性，使用这一联合类型配合映射类型，即上面的例子等价于：

```typescript
type Pick<T> = {
	[P in "name" | "age"]: T[P];
};
```

对于 Omit 类型，它是 Pick 的反向实现：**Pick 是保留这些传入的键，而 Omit 则是移除这些传入的键**。

但它的实现看起来有些奇怪：

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

Omit 是基于 Pick 实现的，这也是 TS 中成对工具类型的另一种实现方式。上面的 Partial 与 Required 使用类似的结构，在**关键位置使用一个相反操作来实现反向**，而这里的 Omit 类型则是基于 Pick 类型实现，也就是**反向工具类型基于正向工具类型实现**。

Exclude 这一工具类型属于工具类型，可以暂时理解为 `Exclude<A, B>` 的结果就是联合类型 A 中不存在于 B 中的部分：

```typescript
type Tmp1 = Exclude<1, 2>; // 1
type Tmp2 = Exclude<1 | 2, 2>; // 1
type Tmp3 = Exclude<1 | 2 | 3, 2 | 3>; // 1
type Tmp4 = Exclude<1 | 2 | 3, 2 | 4>; // 1 | 3
```

因此，在这里 `Exclude<keyof T, K>` 其实就是 T 的键名联合类型中剔除了 K 的部分，将其作为 Pick 的键名，就实现了剔除一部分类型的效果。

**思考**

* Pick 和 Omit 是基于键名的，如果需要**基于键值类型**呢？
* 除了将一个对象结构拆分为多个子结构外，对这些子结构的**互斥处理**也是结构工具类型需要解决的问题之一。互斥处理指的是，假设你的对象存在三个属性 A、B、C，其中 A 与 C 互斥，即 A 存在时不允许 C 存在。而 A 与 B 绑定，即 A 存在时 B 也必须存在，A 不存在时 B 也不允许存在。此时应该如何实现。

另外，可能发现 Pick 会约束第二个参数的联合类型来自于对象属性，而 Omit 并不这么要求。这是由于官方团队的考量是，可能存在这么一种情况：

```typescript
type Omit1<T, K> = Pick<T, Exclude<keyof T, K>>;
type Omit2<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

// 这里就不能使用严格 Omit 了
declare function combineSpread<T1, T2>(obj: T1, otherObj: T2, rest: Omit1<T1, keyof T2>): void; 
  
type Point3d = { x: number, y: number, z: number };

declare const p1: Point3d;

// 能够检测出错误，rest 中缺少了 y
combineSpread(p1, { x: 10 }, { z: 2 });
```

### 集合工具类型

对于两个集合来说，通常存在交集、并集、差集、补集这几种情况。

内置工具类型中提供了交集与差集的实现：

```typescript
// 交集
type Extract<T, U> = T extends U ? T : never;
// 差集
type Exclude<T, U> = T extends U ? never : T;
```

这里的具体实现其实就是条件类型的分布式特性，即当 T、U 都是联合类型（视为一个集合）时，T 的成员会依次被拿出来进行 `extends U ? T1 : T2` 的计算，然后将最终的结果再合并成联合类型。

需要注意差集 Exclude 存在相对概念，即 A 相对于 B 的差集与 B 相对于 A 的差集并不一定相同，而交集则一定相同。

除了差集和交集，我们可以很容易实现并集与补集，为了更好地建立印象，这里使用集合相关的命名：

```typescript
// 并集
export type Concurrence<A, B> = A | B;

// 交集
export type Intersection<A, B> = A extends B ? A : never;

// 差集
export type Difference<A, B> = A extends B ? never : A;

// 补集
export type Complement<A, B extends A> = Difference<A, B>;
```

补集基于差集实现，只需要约束**集合 B 为集合 A 的子集**即可。

内置工具类型中还有一个场景比较明确的集合工具类型：

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type _NonNullable<T> = Difference<T, null | undefined>
```

本质上它就是集合 T 相对于 `null | undefined` 的差集。

在基于分布式条件类型的工具类型中，其实也存在着正反工具类型，但**并不都是简单地替换条件类型结果的两端**，如交集与补集就只是简单调换了结果，但二者的作用却完全不同。

联合类型会自动合并相同的元素，因此我们可以默认这里指的类型集合全部都是类似 Set 那样的结构，不存在重复元素。

**思考**

* 如果要处理**对象类型结构的集合运算**呢？
* 在处理对象类型结构运算时，可能存在不同的需求，比如合并时，可能希望**保留原属性或替换原属性**，可能希望**替换原属性的同时并不追加新的属性**进来。

### 模式匹配工具类型

这一部分的工具类型主要使用**条件类型**与 **infer 关键字**。

严格地说 infer 其实代表了一种模式匹配的思路，如正则表达式、Glob 中等都体现了这一概念。

首先是对函数类型签名的模式匹配：

```typescript
type FunctionType = (...args: any) => any;

type Parameters<T extends FunctionType> = T extends (...args: infer P) => any ? P : never;

type ReturnType<T extends FunctionType> = T extends (...args: any) => infer R ? R : any;
```

根据 infer 的位置不同，我们就能够获取到不同位置的类型，在函数这里则是参数类型与返回值类型。

我们还可以更进一步，比如只匹配第一个参数类型：

```typescript
type FunctionType = (...args: any) => any;

type FirstParameter<T extends FunctionType> = T extends (
  arg: infer P,
  ...args: any
) => any
  ? P 
  : never;

type FuncFoo = (arg: number) => void;
type FuncBar = (...args: string[]) => void;

type FooFirstParameter = FirstParameter<FuncFoo>; // number

type BarFirstParameter = FirstParameter<FuncBar>; // string
```

除了对函数类型进行模式匹配，内置工具类型中还有一组对 Class 进行模式匹配的工具类型：

```typescript
type ClassType = abstract new (...args: any) => any;

type ConstructorParameters<T extends ClassType> = T extends abstract new (
  ...args: infer P
) => any
  ? P 
  : never;

type InstanceType<T extends ClassType> = T extends abstract new (
  ...args: any
) => infer R 
  ? R 
  : any;
```

Class 的通用类型签名可能看起来比较奇怪，但实际上它就是声明了可实例化（new）与可抽象（abstract）罢了。也可以使用接口进行声明：

```typescript
export interface ClassType<TInstanceType = any> {
  new (...args: any[]): TInstanceType;
}
```

对 Class 的模式匹配思路类似于函数，或者说这是一个通用的思路，即基于放置位置的匹配。放在参数部分，那就是构造函数的参数类型，放在返回值部分，那当然就是 Class 的实例类型了。

**思考**

* infer 和条件类型的搭配看起来会有奇效，比如在那些场景？比如随着条件类型的嵌套每个分支会提取不同位置的 infer ？
* infer 在某些特殊位置下应该如何处理？比如上面我们写了第一个参数类型，不妨试着来写**最后一个参数类型**？

### 扩展：infer 约束

某些时候可能对 infer 提取的类型值有些要求，比如只想要数组第一个为字符串的成员，如果第一个成员不是字符串，就不要。

先写一个提取数组第一个成员的工具类型：

```typescript
type FirstArrayItemType<T extends any[]> = T extends [infer P, ...any[]] 
	? P 
	: never;
```

加上对提取字符串的条件类型：

```typescript
type FirstArrayItemType<T extends any[]> = T extends [infer P, ...any[]]
	? P extends string 
		? P 
		: never
	: never;
```

试用一下：

```typescript
type Tmp1 = FirstArrayItemType<['test', 123]>; // "test"
type Tmp2 = FirstArrayItemType<[123, 'test']>; // never
type Tmp3 = FirstArrayItemType<['test']>; // "test"
```

泛型可以声明约束，只允许传入特定的类型，那 infer 中能否也添加约束，只提取特定的类型？

TS 4.7 就支持了 infer 约束功能来实现对特定类型的提取。

```typescript
type FirstArrayItemType<T extends any[]> = T extends [infer P extends string, ...any[]] 
  ? P 
  : never;
```

实际上，infer + 约束的场景是非常常见的，尤其是在某些连续嵌套的情况下，一层层的 infer 提取再筛选会严重地影响代码地可读性，而 infer 约束这一功能无疑带来了更简洁直观地类型编程代码。

## 反方向类型推导：用好上下文相关类型

### 无处不在的上下文类型

首先举一个最常见的例子：

```typescript
window.onerror = (event, source, col, err) => {};
```

在这个例子中，虽然没有为 onerror 的各个参数声明类型，但它们也已经获得了正确的类型。

这是因为 onerror 的类型声明已经内置了：

```typescript
interface Handler {
	// 简化
	onerror: OnErrorEventHandlerNonNull;
}

interface OnErrorEventHandlerNonNull {
	(event: Event | string, source?: string, lineno?: number, colno?: number, error?: Error): any;
}
```

我们自己实现一个函数签名，其实也是一样的效果：

```typescript
type CustomHandler = (name: string, age: number) => boolean;

// 也推导除了参数类型
const handler: CustomHandler = (arg1, arg2) => true;
```

除了参数类型，返回值类型同样会纳入管控：

```typescript
declare const struct: {
	handler: CustomHandler;
};

// 不能将类型“void”分配给类型“boolean”。
struct.handler = (name, age) => {};
```

当然，不仅是箭头函数，函数表达式也是一样的效果，这里就不做展开了。

在这里，参数的类型基于其上下文类型中的参数类型位置来进行匹配，arg1 对应到 name，所以是 string 类型，arg2 对应到 age, 所以是 number 类型。这就是上下文类型的核心理念：**基于位置的类型推导**。同时，相对于我们上面提到的基于开发者输入进行的类型推导，上下文类型更像是**反方向的类型推导**，也就是**基于已定义的类型来规范开发者的使用**。

在上下文类型中，我们实现的表达式可以只使用更少的参数，而不能使用更多，这是因为上下文类型基于位置的匹配，一旦参数个数超过定义的数量，那就没法进行匹配了。

```typescript
// 正常
window.onerror = (event) => {};
// 报错
window.onerror = (event, source, line, col, err, extra) => {};
```

上下文类型也可以进行“嵌套”情况下的类型推导，如以下这个例子：

```typescript
declare let func: (raw: number) => (input: string) => any;

// raw -> number
func = (raw) => {
  // input -> string
  return (input) => {};
};
```

在某些情况下，上下文类型的推导能力也会失效，比如这里我们使用一个由函数类型组成的联合类型：

```typescript
class Foo {
  foo!: number;
}

class Bar extends Foo {
  bar!: number;
}

let f1: { (input: Foo): void } | { (input: Bar): void };
// 参数“input”隐式具有“any”类型。
f1 = (input) => {}; // y: any
```

我们预期的结果是 input 被推导为 `Foo | Bar` 类型，也就是所有符合结构的函数类型的参数，但却失败了。这是因为 TS 中的上下文类型目前暂时不支持这一判断方式（而不是这不属于上下文类型的能力范畴）。

可以直接使用一个联合类型参数的函数签名：

```typescript
let f2: { (input: Foo | Bar): void };
// Foo | Bar
f2 = (input) => {}; // y : any
```

而如果联合类型中将这两个类型再嵌套一层，此时上下文类型反而正常了：

```typescript
let f3: 
  | { (raw: number): (input: Foo) => void } 
  | { (raw: number): (input: Bar) => void };

// raw -> number
f3 = (raw) => {
  // input -> Bar
  return (input) => {};
};
```

这里被推导为 Bar 的原因，其实还和我们此前了解的协变、逆变有关。任何接收 Foo 类型参数的地方，都可以接收一个 Bar 类型参数，因此推导到 Bar 类型要更加安全。

### void 返回值类型下的特殊情况

上下文类型同样会推导并约束函数的返回值类型，但存在特殊情况，当内置函数类型的返回值类型为 void 时：

```typescript
type CustomHandler = (name: string, age: number) => void;

const handler1: CustomHandler = (name, age) => true;
const handler2: CustomHandler = (name, age) => 'test';
const handler3: CustomHandler = (name, age) => null;
const handler4: CustomHandler = (name, age) => undefined;
```

这是一条世界底层的规则，**上下文类型对于 void 返回值类型的函数，并不会真的要求它什么都不能返回**。然而，虽然这些函数实现可以返回任意类型的值，但**对于调用结果的类型，仍然是 void**。

```typescript
const result1 = handler1('test', 123); // void
const result2 = handler2('test', 123); // void
const result3 = handler3('test', 123); // void
const result4 = handler4('test', 123); // void
```

看起来这是一种很奇怪、错误的行为，但实际上，我们日常开发中的很多代码都需要这一“不正确的”行为才不会报错。

```typescript
const arr: number[] = [];
const list: number[] = [1, 2, 3];

list.forEach((item) => arr.push(item));
```

这是我们常用的简写方式，然而，push 方法的返回值是一个 number 类型（push 后数组的长度），而 forEach 的上下文类型声明中要求返回值是 void 类型。如果此时 void 类型真的不允许任何返回值，此时需要多套一个代码块才能确保类型符合。

还可以用另一种方式来描述这个概念：你可以**将返回值非 void 类型的函数（`() => list.push()`）作为返回值类型为 void 类型（`arr.forEach`）的函数类型参数**。

### 扩展：将更少参数的函数赋值给具有更多参数的函数类型

在 forEach 的函数中，我们会消费 list 的每一个成员。但有时也会遇到并不实际消费数组成员的情况：

```typescript
list.forEach(() => arr.push(otherFactory));
```

这个时候，我们实际上就是在将更少参数的函数赋值给具有更多参数的函数类型。

再看一个更明显的例子：

```typescript
function handler(arg: string) {
  console.log(arg);
}

function useHandler(callback: (arg1: string, arg2: number) => void) {
  callback('test', 123);
}

useHandler(handler);
```

handler 函数的类型签名很明显与 useHandler 函数的 callback 类型签名并不一致，但这里却没有报错。这实际上在 JS 中也是我们经常使用的方式：**即使用更少入参的函数来作为一个预期更多入参函数参数的实现**。

## 函数类型：协变与逆变的比较

### 如何比较函数的签名类型

给出三个具有层级关系的类，分别代表动物、狗、柯基。

```typescript
class Animal {
  asPet() {}
}

class Dog extends Animal {
  bark() {}
}

class Corgi extends Dog {
  cute() {}
}
```

对于一个接受 Dog 类型并返回 Dog 类型的函数，可以这样表示：

```typescript
type DogFactory = (arg: Dog) => Dog;
```

将其简化为：`Dog -> Dog` 的表达式。

对于函数类型比较，实际上我们要比较的即是参数类型与返回值类型。对于 Animal、Dog、Corgi 这三个类，如果将它们分别可重复的放置在参数类型与返回值类型处，就可以得到以下这些函数签名类型：

* `Animal -> Animal`
* `Animal -> Dog`
* `Animal -> Corgi`
* `Dog -> Dog`
* `Dog -> Animal`
* `Dog -> Corgi`
* `Corgi -> Animal`
* `Corgi -> Dog`
* `Corgi -> Corgi`

直接比较完整的函数类型并不符合我们的思维直觉，因此需要引入一个辅助函数：它接收一个 `Dog -> Dog` 类型的参数：

```typescript
function transformDogAndBark(dogFactory: DogFactory) {
  const dog = dogFactory(new Dog());
  dog.bark();
}
```

**如果一个值能够被赋值给某个类型的变量，那么可以认为这个值的类型为此变量类型的子类型。**

如一个简单接受 Dog 类型参数的函数：

```typescript
function makeDogBark(dog: Dog) {
  dog.bark();
}
```

它在调用时只可能接受 Dog 类型或 Dog 类型的子类型，而不能接受 Dog 类型的父类型：

```typescript
makeDogBark(new Corgi()); // 没问题
makeDogBark(new Animal()); // 不行
```

**里氏替换原则：子类可以扩展父类的功能，但不能改变父类原有的功能，子类型必须能够替换掉他们的基类型。**

对于这两条约束依次进行检查：

* 对于 `Animal/Dog/Corgi -> Animal` 类型，无论它的参数类型是什么，返回值类型都是不满足条件的。因为返回的不一定是合法的狗狗，即它不是 `Dog -> Dog` 的子类型。
* 对于 `Corgi -> Corgi` 与 `Corgi -> Dog` ，其返回值满足了条件，但是参数类型又不满足了。这两个类型需要接受 Corgi 类型，但没说一定会传入柯基。
* 对于 `Dog -> Corgi`、`Animal -> Corgi`、`Animal -> Dog`，首先它们的参数类型正确的满足了约束，能接受一只狗狗。其次，它们的返回值类型也一定会能汪汪汪。

观察以上排除方式的结论：

* 参数类型**允许**为 Dog 的父类型，**不允许**为 Dog 的子类型。
* 返回值类型**允许**为 Dog 的子类型，**不允许**为 Dog 的父类型。

**最终函数类型的关系直接取决于类型的父子关系**。

### 协变与逆变

上述函数中的参数类型与返回值类型情况的子类型层级关系变化就是 TS 中的**协变与逆变**在函数签名类型中的表现形式。这两个单词最初来自于几何学领域中：**随着某一个量的变化，随之变化一致的即称为协变，而变化相反的即称为逆变**。

而在这里的实例中，**变化即指从单个类型到函数类型的包装过程**，可以使用工具类型来实现独立的包装类型（独立指对参数类型与返回值类型）：

```typescript
type AsFuncArgType<T> = (arg: T) => void;
type AsFuncReturnType<T> = (arg: unknown) => T;
```

```typescript
// 1 成立
type CheckReturnType = AsFuncReturnType<Corgi> extends AsFuncReturnType<Dog>
  ? 1 
  : 2;

// 2 不成立
type CheckArgType = AsFuncArgType<Dog> extends AsFuncArgType<Animal> ? 1 : 2;
```

总结：**函数类型的参数类型使用子类型逆变的方式确定是否成立，而返回值类型使用子类型协变的方式确定**。

但实际上，基于协变逆变的检查并不是始终启用的（毕竟 TS 在严格检查全关与全开的情况下，简直像是两门语言），需要通过配置来开启。

### TSConfig 中的 StrictFunctionTypes

strictFunctionTypes 这一项配置在文档中的描述是：**在比较两个函数类型是否兼容时，将对函数参数进行更严格的检查**，而实际上，这里的更严格指的**即是对函数参数类型启用逆变检查**。

```typescript
function fn(dog: Dog) {
  dog.bark();
}

type CorgiFunc = (input: Corgi) => void;
type AnimalFunc = (input: Animal) => void;
```

通过赋值的方式来实现对函数类型的比较：

```typescript
const func1: CorgiFunc = fn;
const func2: AnimalFunc = fn;
```

根据上面所学，可以发现第二种应当是不成立的，但在禁用了 `strictFuncionTypes` 的情况下，TS 并不会抛出错误。这是因为，在默认情况下，对函数参数的检查采用**双变，即逆变与协变都被认为是可接受的**。

在 typescript ESLint 中，有一条规则：[method-signature-style](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftypescript-eslint%2Ftypescript-eslint%2Fblob%2Fmain%2Fpackages%2Feslint-plugin%2Fdocs%2Frules%2Fmethod-signature-style.md)，它的意图是约束在接口中声明方法时，需要使用 **property** 而非 **method** 形式：

```typescript
// method 声明
interface T1 {
  func(arg: string): number;
}

// property 声明
interface T2 {
  func: (arg: string) => number;
}
```

进行如此约束的原因即，对于 property 声明，才能在开启严格函数类型检查的情况下享受到**基于逆变的参数类型检查**。

对于 method 声明（以及构造函数声明），其无法享受到这一更严格的检查的原因则是对于如 Array 这样的内置定义，我们希望它的函数方法就是以协变的方式进行检查。举个例子，`Dog[] ≤ Animal[]` 是否成立？

* 我们并不能简单的比较 Dog 与 Animal，而是要将它们视为两个完整的类型比较，即 `Dog[]` 的每一个成员（属性、方法）是否都能对应的赋值给 `Animal[]`？
* `Dog[].push ≤ Animal[].push` 是否成立？
* 由 push 方法的类型签名进一步推导，`Dog -> void ≤ Animal -> void` 是否成立？
* `Dog -> void ≤ Animal -> void` 在逆变的情况下意味着 `Animal ≤ Dog`，而这很明显是不对的。
* 简单来说，`Dog -> void ≤ Animal -> void` 是否成立本身就为 `Dog[] ≤ Animal[]` 提供了一个前提答案。

因此，如果 TS 在此时仍然强制使用参数逆变的规则进行检查，那么 `Dog[] ≤ Animal[]` 就无法成立。所以在大部分情况下，我们希望方法参数类型的检查是**双变**的。这就是为什么它们的声明中类型结构使用 method 方式来声明：

```typescript
interface Array<T> {
  push(...items: T[]): number;
}
```

### 总结

**如何对两个函数类型进行兼容性比较**：比较它们的参数类型是否是反向的父子类型关系，返回值是否是正向的父子类型关系。

用本章的知识来说，其实就是判断**参数类型是否遵循类型逆变，返回值类型是否遵循类型协变**。

可以通过 typescript ESLint 的规则以及 `strictFunctionTypes` 配置，来为 interface 内的函数声明启用严格的检查模式。

除了对自定义函数类型的比较，对于部分 TS 内置的方法，会通过显式的 method 声明方式来确保在调用时，对参数类型检查采用**双变**而非**逆变**。

### 扩展

**联合类型与兄弟类型下的比较**

上面只关注了显式的父子类型关系，实际上在类型层级中还有隐式的父子类型关系（联合类型）以及兄弟类型（同一基类的两个派生类）。对于隐式的父子类型其可以仍然沿用显式的父子类型协变与逆变判断，但对于兄弟类型，比如 Dog 和 Cat，需要注意的是它们根本就**不满足逆变与协变的发生条件（父子类型）**，因此无论在严格检查与默认情况下均不成立。

**非函数签名包装类型的变换**

如果要考虑类似数组这种包装类型呢？比如一个简单的笼子 Cage？

先不考虑 Cage 内部的实现，只知道它同时只能放一个物种的动物，`Cage<Dog>` 能被作为 `Cage<Animal>` 的子类型吗？对于这一类型的比较，我们可以直接用实际场景来代入：

* 假设需要一笼动物，但不会对它们进行除了读以外的操作，那么给一笼狗是没有问题的，但不能给一笼植物。也就意味着，此时 List 是 readonly 的，而 `Cage<Dog> ≤ Cage<Animal>` 成立。**即在不可变的 Wrapper 中，我们允许其遵循协变**。
* 假设需要一笼动物，并且会在其中新增其他物种，如兔子等，这个时候给一笼兔子就不行了，因为这个笼子只能放狗。此时 List 是 writeable 的，而 `Cage<Dog>` `Cage<Rabit>` `Cage<Turtle>` 彼此之间是互斥的，我们称为**不变**，用来放狗的笼子绝不能用来放兔子，即无法进行分配。
* 如果再修改规则，现在一个笼子可以放任意物种的动物，此时任意的笼子都可以放任意的物种，即可以互相分配，我们称之为**双变**。

也就是说，包装类型的表现与我们实际需要的效果是紧密关联的。

## 了解类型编程与类型体操的意义，找到平衡点

### 从类型编程到类型体操

类型编程时对实际开发中真的有帮助的类型操作。类型体操指诸如基于 TS 类型实现的四则运算、斐波那契数列、象棋、Lisp 编译器这一类令人叹为观止的操作，这些就属于类型体操。

### Type Challenges

[Type Challenge ](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftype-challenges%2Ftype-challenges) 是 antfu 的作品，其中搜集了许多类型编程的题目。

## 内置工具类型进阶：类型编程进阶

### 属性修饰进阶

在内置工具类型一节中，对属性修饰工具类型的进阶主要分为这么几个方向：

* 深层的属性修饰；
* 基于已知属性的部分修饰，以及基于属性类型的部分修饰。

首先是深层属性修饰。

```typescript
type PrimiseValue<T> = T extends Promise<infer V> ? PrimiseValue<V> : T;
```

可以看到，我们只是在条件类型成立时，再次调用了这个工具类型而已。在某一次递归到条件类型不成立时，会直接返回这个类型值。那么对于 Partial、Required，其实也可以进行这样地处理：

```typescript
export type DeepPartial<T extends object> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
```

简单起见，直接使用了 object 作为泛型约束与条件，这意味着也有可能传入函数、数组等类型。但毕竟对这个类型知根知底，就可以假设只会传入对象结构，因此也只需要对对象类型进行处理了。

使用 tsd 这一工具类型单元测试库来进行验证：

```typescript
import { expectType } from "tsd";

type DeepPartialStruct = DeepPartial<{
  foo: string;
  nested: {
    nestedFoo: string;
    nestedBar: {
      nestedBarFoo: string;
    };
  };
}>;

expectType<DeepPartialStruct>({
  foo: 'bar',
  nested: {},
});

expectType<DeepPartialStruct>({
  nested: {
    nestedBar: {},
  },
});

expectType<DeepPartialStruct>({
  nested: {
    nestedBar: {
      nestedBarFoo: undefined,
    },
  },
});
```

在 expectType 的泛型坑位中传入一个类型，然后再传入一个值，就可以验证这个值是否符合泛型类型了。

类似的，还可以实现其他进行递归属性修饰的工具类型

```typescript
export type DeepPartial<T extends object> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

export type DeepRequired<T extends object> = {
  [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};

export type DeepReadonly<T extends object> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

export type DeepMutable<T extends object> = {
  -readonly [K in keyof T]: T[K] extends object ? DeepMutable<T[K]> : T[K];
};
```

另外，在内置工具类型一节的结构工具类型中，存在一个从联合类型中剔除 `null | undefined` 的工具类型 NonNullable：

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
```

在对象结构中也常声明类型为 `string | null` 的形式，代表了“**这里有值，但可能是空值**”。此时，我们可以将其等价为一种属性修饰（Nullable 属性，前面则是 Optional / Readonly 属性）。因此，我们也可以像访问性修饰工具类型那样，实现一个 DeepNonNullable 来递归剔除所有属性的 null 与 undefined：

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

export type DeepNonNullable<T extends object> = {
  [K in keyof T]: T[K] extends object 
    ? DeepNonNullable<T[K]> 
    : NonNullable<T[K]>;
};
```

当然，就像 Partial 与 Required 的关系一样，DeepNonNullable 也有一个自己的另一半： DeepNullable

```typescript
export type Nullable<T> = T | null;

export type DeepNullable<T extends object> = {
  [K in keyof T]: T[K] extends object ? DeepNullable<T[K]> : Nullable<T[K]>;
};
```

接下来是基于已知属性进行部分修饰了。如果要让一个对象的三个已知属性为可选的，那只要把这个对象拆成 A、B 两个对象结构，分别由三个属性和其他属性组成。然后让对象 A 的属性全部变成可选的，和另外一个对象 B 组合起来。

* 拆分对象结构，就是内置工具类型中的**结构工具类型**，即 Pick 与 Omit ；
* 三个属性的对象全部变为可选，即为属性修饰，可以直接用上面刚学到的**递归属性修饰**；
* 组合两个对象类型，也就意味着得到一个同时符合这两个对象类型的新结构，即为**交叉类型**。

这也是使用最广泛的一种类型编程思想：**将复杂的工具类型，拆解为由基础工具类型、类型工具的组合**。

直接来看基于已知属性的部分修饰，MarkPropsAsOptional 会将一个对象的部分属性标记可选：

```typescript
export type MarkPropsAsOptional<
  T extends object, 
  K extends keyof T = keyof T
> = Partial<Pick<T, K>> & Omit<T, K>;
```

T 为需要处理的对象类型，而 K 为需要标记为可选的属性。由于此时 K 必须为 T 内部的属性，因此我们将其约束为 keyof T ，即对象属性组成的字面量联合类型。同时为了让它能够直接代替掉 Partial，为其默认值也为 keyof T ，这样在不传入第二个泛型参数时，它的表现就和 Partial 一致，即全量的属性可选。

而其组成中，`Partial<Pick<T, K>>` 为需要标记为可选的属性组成的对象子结构，`Omit<T, K>` 则为不需要处理的部分，使用交叉类型将其组成即可。

验证效果如下：

```typescript
type MarkPropsAsOptionalStruct = MarkPropsAsOptional<
  {
    foo: string;
    bar: number;
    baz: boolean;
  },
  'bar'
>;
```

为了看出具体效果，引入一个辅助的工具类型，称其为 Flatten ，对于这种交叉类型的结构，Flatten 能够将它展平为单层的对象结构。

```typescript
export type Flatten<T> = { [K in keyof T]: T[K] };

export type MarkPropsAsOptional<
  T extends object,
  K extends keyof T = keyof T
> = Flatten<Partial<Pick<T, K>> & Omit<T, K>>;
```

在这里其实也可以使用 `DeepPartial<Pick<T, K>>`，来把这些属性标记为深层的可选状态。

实现其他类型的部分修饰：

```typescript
export type MarkPropsAsRequired<
  T extends object,
  K extends keyof T = keyof T
> = Flatten<Omit<T, K> & Required<Pick<T, K>>>;

export type MarkPropsAsReadonly<
  T extends object,
  K extends keyof T = keyof T
> = Flatten<Readonly<Pick<T, K>> & Omit<T, K>>;

export type MarkPropsAsMutable<
  T extends object,
  K extends keyof T = keyof T
> = Flatten<Mutable<Pick<T, K>> & Omit<T, K>>;

export type MarkPropsAsNullable<
  T extends object,
  K extends keyof T = keyof T
> = Flatten<Nullable<Pick<T, K>> & Omit<T, K>>;

export type MarkPropsAsNonNullable<
  T extends object,
  K extends keyof T = keyof T
> = Flatten<NonNullable<Pick<T, K>> & Omit<T, K>>;
```

而对于按照值类型的部分修饰，比如标记所有函数类型属性为可选，其实和这里是一样的思路：**拆分-处理-组合**，只不过之前使用基于键名裁剪的 Pick、Omit，现在需要基于键值类型裁剪的 PickByValueType、OmitByValueType 了。

在结构工具类型进阶中，会了解到如何基于键值类型去裁剪结构。

### 结构工具类型进阶

首先是**基于键值类型的 Pick 与 Omit**，称之为 PickByValueType 。实现方式还是类似部分属性修饰中那样，将对象拆分为两个部分，处理完毕再组装。只不过现在无法预先确定要拆分的属性了，而是需要**基于期望的类型去拿到所有此类型的属性名**。

```typescript
type FuncStruct = (...args: any[]) => any;

type FunctionKeys<T extends object> = {
  [K in keyof T]: T[K] extends FuncStruct ? K : never;
}[keyof T];
```

`{}[keyof T]` 这个写法是第一次见，但可以拆开来看，先看前面的 `{ [K in keyof T]: T[K] extends FuncStruct ? K : never; }` 部分，为何在条件类型成立时它返回了键名 K，而非索引类型查询 `T[K]` ?

```typescript
type Tmp<T extends object> = {
  [K in keyof T]: T[K] extends FuncStruct ? K : never;
};

type Res = Tmp<{
  foo: () => void;
  bar: () => number;
  baz: number;
}>;

type ResEqual = {
  foo: 'foo';
  bar: 'bar';
  baz: never;
};
```

在 Res 中，获得了一个**属性名-属性名字面量类型**的结构，对于非函数类型的属性，其值为 never。然后，加上 `[keyof T]` 这一索引类型查询 + keyof 操作符的组合：

```typescript
type WhatWillWeGet = Res[keyof Res]; // "foo" | "bar"
```

由此可见，获得了所有函数类型的属性名，当索引类型查询中使用了一个联合类型时，它会使用类似分布式条件类型的方式，将这个联合类型的成员依次进行访问，然后再最终组合起来。上面的例子可以这么简化：

```typescript
type WhatWillWeGetEqual1 = Res["foo" | "bar" | "baz"];
type WhatWillWeGetEqual2 = Res["foo"] | Res["bar"] | Res["baz"];
type WhatWillWeGetEqual3 = "foo" | "bar" | never;
```

通过这一方式，就可以获取到符合预期类型的属性名了。如果希望抽象“基于键值类型查找属性名”这么个逻辑，就需要对 FunctionKeys 的逻辑进行封装，即**将预期类型也作为泛型参数**，由外部传入：

```typescript
type ExpectedPropKeys<T extends object, ValueType> = {
  [Key in keyof T]-?: T[Key] extends ValueType ? Key : never;
}[keyof T];

type FunctionKeys<T extends object> = ExpectedPropKeys<T, FuncStruct>;

expectType<
  FunctionKeys<{
    foo: () => void;
    bar: () => number;
    baz: number;
  }>
>('foo');

expectType<
  FunctionKeys<{
    foo: () => void;
    bar: () => number;
    baz: number;
  }>
  // 报错，因为 baz 不是函数类型属性
>('baz');
```

注意，为了避免可选属性对条件类型语句造成干扰，这里使用 `-?` 移除了所有可选标记。

既然现在可以拿到对应类型的属性名，那么把这些属性交给 Pick，不就可以得到由这些属性组成的子结构了。

```typescript
export type PickByValueType<T extends object, ValueType> = Pick<
  T,
  ExpectedPropKeys<T, ValueType>
>;

expectType<PickByValueType<{ foo: string; bar: number }, string>>({
  foo: 'test',
});

expectType<
  PickByValueType<{ foo: string; bar: number; baz: boolean }, string | number>
>({
  foo: 'test',
  bar: 599,
});
```

OmitByValueType 也是类似的，只需要一个和 ExpectedPropKeys 作用相反的工具类型即可，比如来个 FilteredPropKeys，只需要调换条件类型语句结果的两端即可：

```typescript
type FilteredPropKeys<T extends object, ValueType> = {
  [Key in keyof T]-?: T[Key] extends ValueType ? never : Key;
}[keyof T];

export type OmitByValueType<T extends object, ValueType> = Pick<
  T, 
  FilteredPropKeys<T, ValueType>
>;

expectType<OmitByValueType<{ foo: string; bar: number }, string>>({
  bar: 123,
});

expectType<OmitByValueType<{ foo: string; bar: number; baz: boolean }, string | number>>({
  baz: true,
});
```

或者，如果想把 ExpectedPropKeys 和 FilteredPropKeys 合并在一起，可以引入第三个泛型参数来控制返回结果：

```typescript
type Conditional<Value, Condition, Resolved, Rejected> = Value extends Condition 
  ? Resolved 
  : Rejected;

export type ValueTypeFilter<
  T extends object, 
  ValueType,
  Positive extends boolean
> = {
  [Key in keyof T]-?: T[Key] extends ValueType 
    ? Conditional<Positive, true, Key, never> 
    : Conditional<Positive, true, never, Key>;
}[keyof T];

export type PickByValueType<T extends object, ValueType> = Pick<
  T, 
  ValueTypeFilter<T, ValueType, true>
>;

export type OmitByValueType<T extends object, ValueType> = Omit<
  T,
  ValueTypeFilter<T, ValueType, false>
>;
```

在联合类型的情况下，`1 | 2 extends 1 | 2 | 3` （通过泛型参数传入）会被视为是合法的，这是由于分布式条件类型的存在。如何禁用分布式条件类型，让它不满足裸类型参数这一条即可：

```typescript
type Wrapped<T> = [T] extends [boolean] ? "Y" : "N";
```

只需要简单进行改动即可：

```typescript
type StrictConditional<Value, Condition, Resolved, Rejected> = [value] extends [
	Condition
]
	? Resolved 
	: Rejected;
```

但还不够完美，比如说下面这种情况：

```typescript
type Res1 = StrictConditional<1 | 2, 1 | 2 | 3, true, false>; // true
```

当条件不再是一个简单的单体类型，而是一个联合类型时，使用数组的方式就产生问题了。因为 `Array<1 | 2> extends Array<1 | 2 | 3>` 就是合法的，**第一个数组中的可能元素类型均被第二个数组的元素类型包含了，无论如何都是其子类型**。

只要再加一个反方向的比较即可：

```typescript
type StrictConditional<A, B, Resolved, Rejected, Fallback = never> = [
  A
] extends [B] 
  ? [B] extends [A] 
    ? Resolved 
    : Rejected
  : Fallback;
```

在这种情况下 Value 和 Condition 的界限就比较模糊了，只是在比较两个类型是否严格相等，并没有值和表达式的概念了，因此就使用 A、B 来简称。

此时结果就符合预期了，需要联合类型完全一致：

```typescript
type Res1 = StrictConditional<1 | 2, 1 | 2 | 3, true, false>; // false
type Res2 = StrictConditional<1 | 2 | 3, 1 | 2, true, false, false>; // false
type Res3 = StrictConditional<1 | 2, 1 | 2, true, false>; // true
```

应用到 TypeFilter 中：

```typescript
export type StrictValueTypeFilter<
  T extends object,
  ValueType,
  Positive extends boolean = true
> = {
  [Key in keyof T]-?: StrictConditional<
    ValueType,
    T[Key],
    // 为了避免嵌套太多工具类型，这里就不使用 Conditional 了
    Positive extends true ? Key : never,
    Positive extends true ? never : Key,
    Positive extends true ? never : Key
  >;
}[keyof T];

export type StrictPickByValueType<T extends object, ValueType> = Pick<
  T, 
  StrictValueTypeFilter<T, ValueType>
>;

expectType<
  StrictPickByValueType<{ foo: 1; bar: 1 | 2; baz: 1 | 2 | 3 }, 1 | 2>
>({
  bar: 1,
});

export type StrictOmitByValueType<T extends object, ValueType> = Pick<
  T, 
  StrictValueTypeFilter<T, ValueType, false>
>;

expectType<
  StrictOmitByValueType<{ foo: 1; bar: 1 | 2; baz: 1 | 2 | 3 }, 1 | 2>
>({
  foo: 1,
  baz: 3,
});
```

需要注意的是，由于 StrictOmitByValueType 需要的是不符合类型的属性，因此这里 StrictConditional 的 Fallback 泛型参数也需要传入 Key （即第五个参数中的 `Positive extends true ? never : Key`），同时整体应当基于 Pick 来实现。

接下来是基于结构的互斥工具类型。假设有一个用于描述用户信息的对象结构，除了共有的一些基础结构以外，VIP 用户和普通用户、游客这三种类型的用户各自拥有一些独特的字段，如 vipExpires 代表 VIP 过期时间，仅属于 VIP 用户，promotionUsed 代表已领取过体验券，属于普通用户，而 refererType 代表跳转来源，属于游客。

如果使用联合类型并不能约束“不能同时拥有”这个条件：

```typescript
interface VIP {
  vipExpires: number;
}

interface CommonUser {
  promotionUsed: boolean;
}

type User = VIP | CommonUser;

// 并不能约束“不能同时拥有”这个条件
const user1: User = {
  vipExpires: 123,
  promotionUsed: true,
};
```

为了表示不能同时拥有，实际上应该使用 never 类型来标记一个属性。

```typescript
export type Without<T, U> = { [P in Exclude<keyof T, keyof U>]?: never };

export type XOR<T, U> = (Without<T, U> & U) | (Without<U, T> & T);

type XORUser = XOR<VIP, CommonUser>;

expectType<XORUser>({
  vipExpires: 0,
});

expectType<XORUser>({
  promotionUsed: true,
});

// 报错，至少需要一个
expectType<XORUser>({

});

// 报错，不允许同时拥有
expectType<XORUser>({
  promotionUsed: false,
  vipExpires: 0,
});
```

对 Without 做进一步展开可以看到，它其实就是将声明了一个不变的原属性 + 为 never 的其他属性的接口：

```typescript
// {
// 		vipExpires?: never;
// }
type Tmp1 = Flatten<Without<VIP, CommonUser>>;
// {
// 		vipExpires?: never;
// 		promotionUsed: boolean;
// }
type Tmp2 = Flatten<Tmp1 & CommonUser>;
```

再通过联合类型的合并，这样一来 XORUser 就满足了”至少实现 VIP / CommonUser 这两个接口中的一个“，”不能同时实现 VIP / CommonUser“ 这两个条件。如果加上游客类型实现三个互斥属性，只需要额外嵌套一层：

```typescript
interface Visitor {
  refererType: RefererType;
}

// 联合类型会自动合并并重复的部分
type XORUser = XOR<VIP, XOR<CommonUser, Visitor>>;
```

还可以使用互斥类型实现绑定效果，即要么同时拥有 A、B 属性，要么一个属性都没有：

```typescript
type XORStruct = XOR<
  {}, 
  {
    foo: string;
    bar: number;
  }
>;

// 没有 foo、bar
expectType<XORStruct>({});

// 同时拥有 foo、 bar
expectType<XORStruct>({
  foo: 'test',
  bar: 123,
});
```

### 集合工具类型进阶

对于对象类型的交并补差集，我们仍然沿用”降级“的处理思路，把它简化为可以用基础工具类型处理的问题即可。

复习一下前面的一维集合：

```typescript
// 并集
export type Concurrence<A, B> = A | B;

// 交集
export type Intersection<A, B> = A extends B ? A : never;

// 差集
export type Difference<A, B> = A extends B ? never : A;

// 补集
export type Complement<A, B extends A> = Difference<A, B>;
```

对应地实现对象属性名的版本：

```typescript
// 使用更精确的对象类型描述结构
export type PlainObjectType = Record<string, any>;

// 属性名并集
export type ObjectKeysConcurrence<
  T extends PlainObjectType,
  U extends PlainObjectType
> = keyof T | keyof U;

// 属性名交集
export type ObjectKeysIntersection<
  T extends PlainObjectType,
  U extends PlainObjectType
> = Intersection<keyof T, keyof U>;

// 属性名差集
export type ObjectKeysDifference<
  T extends PlainObjectType, 
  U extends PlainObjectType
> = Difference<keyof T, keyof U>;

// 属性名补集
export type ObjectKeysComplement<
  T extends U,
  U extends PlainObjectType
> = Difference<keyof T, keyof U>;
```

对于交集、补集、差集，可以直接使用属性名的集合来实现对象层面的版本：

```typescript
// 交集
export type ObjectIntersection<
  T extends PlainObjectType, 
  U extends PlainObjectType
> = Pick<T, ObjectKeysIntersection<T, U>>;

// 差集
export type ObjectDifference<
  T extends PlainObjectType,
  U extends PlainObjectType
> = Pick<T, ObjectKeysDifference<T, U>>;

// 补集
export type ObjectComplement<
  T extends U, 
  U extends PlainObjectType
> = Pick<T, ObjectKeysComplement<T, U>>;
```

需要注意的是在 ObjectKeysComplement 与 ObjectComplement 中，`T extends U` 意味着 T 是 U 的子类型，但在属性组成的集合类型中却相反， **U 的属性联合类型是 T 的属性联合类型的子类型**，因为既然 T 是 U 的子类型，很显然 T 所拥有的属性更多。

对于并集，就不能简单使用属性名并集版本了，因为使用联合类型实现，不能控制**同名属性的优先级**。

对于 T、U 两个对象，假设以 U 的同名属性类型优先，思路如下：

* T 比 U 多的部分：T 相对于 U 的差集，`ObjectDifference<T, U>`
* U 比 T 多的部分：U 相对于 T 的差集，`ObjectDifference<U, T>`
* T 与 U 的交集，由于 U 的优先级更多，在交集处理中将 U 作为原集合，T 作为后传入的集合，`ObjectIntersection<U, T>`

这样就得到了 Merge：

```typescript
type Merge<
  T extends PlainObjectType,
  U extends PlainObjectType
> = ObjectDifference<T, U> & ObjectDifference<U, T> & ObjectIntersection<U, T>;
```

如果要保证原对象优先级更高，只需要在交集处理中将 T 视为原集合，U 作为后传入的集合：

```typescript
type Assign<
  T extends PlainObjectType,
  U extends PlainObjectType
> = ObjectDifference<T, U> & ObjectDifference<U, T> & ObjectIntersection<T, U>;
```

除了简单粗暴的完全合并外，还可以实现不完全的并集，即使用对象 U 的属性类型覆盖对象 T 中的同名属性类型，但不会将 U 独特的部分合并过来：

```typescript
type Override<
  T extends PlainObjectType,
  U extends PlainObjectType
  // T 比 U 多的部分，加上 T 与 U 交集的部分（类型不同则以 U 的优先级更高（逆并集））
> = ObjectDifference<T, U> & ObjectIntersection<U, T>;
```

### 模式匹配工具类型进阶

在内置工具类型一节中，对模式匹配工具类型的进阶方向其实只有深层嵌套这么一种，特殊位置的 infer 处理大部分时候也是通过深层嵌套实现，比如实现提取函数的首个参数类型：

```typescript
type FunctionType = (...args: any) => any;

type FirstParameter<T extends FunctionType> = T extends (
  arg: infer P, 
  ...args: any
) => any 
  ? P 
  : never;
```

要提取最后一个参数类型则可以这样：

```typescript
type LastParameter<T extends FunctionType> = T extends (arg: infer P) => any 
  ? P 
  : T extends (...args: infer R) => any 
    ? R extends [...any, infer Q] 
      ? Q 
      : never
    : never;

type FuncFoo = (arg: number) => void;
type FuncBar = (...args: string[]) => void;
type FuncBaz = (arg1: string, arg2: boolean) => void;

type FooLastParameter = LastParameter<FuncFoo>; // number
type BarLastParameter = LastParameter<FuncBar>; // string
type BazLastParameter = LastParameter<FuncBaz>; // boolean
```

这也是模式匹配中的常用的一种方法，通过 infer 提取到某一个结构，然后再对这个结构进行 infer 提取。

此前曾经讲过一个提取 Promise 内部值类型的工具类型 PromiseValue， TS 内置工具类型中也存在这么一个作用的工具类型，并且它的实现要更为严谨：

```typescript
type Awaited<T> = T extends null | undefined 
  ? T 
  : T extends object & { then(onfulfilled: infer F): any } 
    ? F extends (value: infer V, ...args: any) => any \
      ? Awaited<V> 
      : never 
    : T;
```

在这里 Awaited 并非通过 `Promise<infer V>` 来提取函数类型，而是通过 `Promise.then` 方法提取，首先提取到 then 方法中的函数类型，再通过这个函数类型的首个参数来提取出实际的值。

更严谨地来说，PromiseValue 和 Awaited 并不应该放在一起比较，前者就只想提取 `Promise<void>` 这样结构的内部类型，后者则像在类型的层面执行了 `await Promise.then()` 之后的返回值类型。同样，这里也用到了 infer 伴随结构转化的例子。

### 总结

主要思路：

* 对一个对象结构拆分为多个子结构再分别处理；
* 将复杂类型降维到基础类型再逐个击破；
* 在嵌套的条件类型中基于 infer 多次修改类型结构来提取最终需要的类型。

### 扩展

**RequiredKeys 、 OptionalKeys**

如果要获取一个接口中所有可选或必选的属性呢？

首先是 RequiredKeys：

```typescript
type Tmp1 = {} extends { prop: number } ? "Y" : "N"; // "N"
type Tmp2 = {} extends { prop?: number } ? "Y" : "N"; // "Y"
```

此时 TS 会使用基于结构化类型的比较，也就意味着由于 `{ prop: number }` 可以视为继承自 `{}`，`{} extends { prop: number }` 是不满足条件的。如果这里的 prop 是可选的，那就不一样了，由于 `{ prop?: number }` 也可以是一个空的接口结构，那么 `{} extends { prop?: number }` 就可以认为是满足的。

因此，可以这么实现：

```typescript
export type RequiredKeys<T> = {
	[K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];
```

OptionalKeys 也是类似：

```typescript
export type OptionalKeys<T> = {
	[K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];
```

**MutableKeys 、 ImmutableKeys**

MutableKeys 和 ImmutableKeys 则要更加复杂一些，因为 readonly 修饰符无法简单地通过结构化类型比较，需要一个能对只读这一特性进行判断地辅助工具类型。

```typescript
type Equal<X, Y, A = X, B = never> = (<T>() => T extends X ? 1 : 2) extends <
  T
>() => T extends Y ? 1 : 2 
  ? A 
  : B;
```

在这里，`<T>() => T extends X ? 1 : 2` 和 `<T>() => T extends Y ? 1 : 2` 这两个函数结构实际上起辅助作用，内部的条件类型并不会真的进行运算。实际上是借助这一辅助结构判断类型 X 与 Y 的全等性，这一全等性就包括了 readonly 修饰符与可选性等。

基于其实现 MutableKeys 和 ImmutableKeys：

```typescript
export type MutableKeys<T extends object> = {
  [P in keyof T]-?: Equal<
    { [Q in P]: T[P] },
    { -readonly [Q in P]: T[P] }, 
    P, 
    never
  >;
}[keyof T];

expectType<MutableKeys<{ a: string; readonly b: string }>>('a');
expectNotType<MutableKeys<{ a: string; readonly b: string }>>('b');

export type ImmutableKeys<T extends object> = {
  [P in keyof T]-?: Equal<
    { [Q in P]: T[P] },
    { -readonly [Q in P]: T[P] },
    never,
    P
  >;
}[keyof T];

expectType<ImmutableKeys<{ a: string; readonly b: string }>>('b');
expectNotType<ImmutableKeys<{ a: string; readonly b: string }>>('a');
```

## 基础类型新成员：模板字符串类型入门

### 模板字符串类型的基础使用

一个最简单的使用例子：

```typescript
type World = 'World';

// "Hello World"
type Greeting = `Hello ${World}`;
```

这里的 Greeting 就是一个模板字符串类型，它内部通过与 JS 中模板字符串相同的语法（`${}`），使用了另一个类型别名 World，其最终的类型就是**将两个字符串类型值组装在一起返回**。

除了使用确定的类型别名外，模板字符串类型也支持通过泛型参数传入。需要注意的是，并不是所有值都能被作为模板插槽：

```typescript
type Greet<T extends string | number | boolean | null | undefined | bigint> = `Hello ${T}`;

type Greet1 = Greet<'test'>; // "Hello test"
type Greet2 = Greet<123>; // "Hello 123"
type Greet3 = Greet<true>; // "Hello true"
type Greet4 = Greet<null>; // "Hello null"
type Greet5 = Greet<undefined>; // "Hello undefined"
type Greet6 = Greet<0x1fffffffffffff>; // "Hello 9007199254740991"
```

目前有效的类型只有 `string | number | boolean | null | undefined | bigint` 这几个。这些类型在最终的字符串结果中都会被转换为字符串字面量类型，即使是 null 与 undefined。

也可以直接为插槽传入一个类型而非类型别名：

```typescript
type Greeting = `Hello ${string}`;
```

在这种情况下，Greeting 类型并不会变成 `Hello string`，而是保持原样。这意味着它并没有实际意义，此时就是一个无法改变的模板字符串类型，但所有 `Hello` 开头的字面量类型都会被视为 `Hello ${string}` 的子类型，如 `Hello TypeScript`。

模板字符串类型的主要目的即是增强字符串字面量类型的灵活性，进一步增强类型和逻辑代码的关联。通过模板字符串可以这样声明版本号：

```typescript
type Version = `${number}.${number}.${number}`;

const v1: Version = '1.1.0';

// 报错：类型 "1.0" 不能赋值给类型 `${number}.${number}.${number}`
const v2: Version = '1.0';
```

而在需要声明大量存在关联的字符串字面量类型时，模板字符串类型也能在减少代码的同时获得更好的类型保障。如当需要声明以下字符串类型时：

```typescript
type SKU = 
  | 'iphone-16G-official'
  | 'xiaomi-16G-official'
  | 'honor-16G-official'
  | 'iphone-16G-second-hand'
  | 'xiaomi-16G-second-hand'
  | 'honor-16G-second-hand'
  | 'iphone-64G-official'
  | 'xiaomi-64G-official'
  | 'honor-64G-official'
  | 'iphone-64G-second-hand'
  | 'xiaomi-64G-second-hand'
  | 'honor-64G-second-hand';
```

随着商品、内存数、货品类型的增加，可能需要成几何倍数的新增。但如果使用模板字符串类型，可以利用其**自动分发的特性**来实现简便又严谨的声明：

```typescript
type Brand = 'iphone' | 'xiaomi' | 'honor';
type Memory = '16G' | '64G';
type ItemType = 'official' | 'second-hand';

type SKU = `${Brand}-${Memory}-${ItemType}`;
```

在插槽中传入联合类型，然后就会发现，所有的联合类型排列组合都已经自动组合完毕了。

如果某一种组合并不存在，例如`iphone-32G`系列，可以使用差集解决这里的问题，比如可以只是**剔除数个确定商品集合**，也可以**再利用模板字符串类型的排列组合能力生成要剔除的集合**。

通过这种方式，不仅不需要再手动声明一大堆工具类型，同时也获得了逻辑层面的保障：它会忠实地将**所有插槽中的联合类型与剩余的字符串部分进行依次的排列组合**。

除了直接在插槽中传递联合类型，通过泛型传入联合类型时同样会有分发过程：

```typescript
type SizeRecord<Size extends string> = `${Size}-Record`;

type Size = 'Small' | 'Middle' | 'Large';

// "Small-Record" | "Middle-Record" | "Large-Record"
type UnionSizeRecord = SizeRecord<Size>;
```

### 模板字符串类型的类型表现

由于模板字符串类型最终产物还是字符串字面量类型，因此只要插槽位置的类型匹配，字符串字面量类型就可以被认为是模板字符串类型的子类型。

```typescript
declare let v1: `${number}.${number}.${number}`;
declare let v2: '1.2.4';

v1 = v2;
```

如果反过来，`v2 = v1`很显然不成立。同样，模板字符串类型和模板字符串也拥有着紧密的关联：

```typescript
const greet = (to: string): `Hello ${string}` => {
  return `Hello ${to}`;
};
```

这个例子进一步体现了类型与值的紧密关联，通过模板字符串类型，能够进行更精确的类型描述。

### 结合索引类型与映射类型

基于 **keyof + 模板字符串类型**，可以基于已有的对象类型来实现精确到字面量的类型推导：

```typescript
interface Foo {
  name: string;
  age: number;
  job: Job;
}

type ChangeListener = {
  on: (change: `${keyof Foo}Changed`) => void;
}

declare let listener: ChangeListener;

// 提示并约束为 "nameChanged" | "numberChanged" | "jobChanged"
listener.on('');
```

除了索引类型，模板字符串类型也和映射类型有着奇妙的化学反应。

为了与映射类型实现更好的协作，TS 在引入模板字符串类型时支持了一个叫作 **重映射** 的新语法，基于模板字符串类型与重映射，可以实现一个此前无法想象的新功能：**在映射键名时基于原键名做修改**。

使用映射类型很容易复制一个接口：

```typescript
type Copy<T extends object> = {
  [K in keyof T]: T[K];
};
```

那么如何修改键名呢？

```typescript
type CopyWithRename<T extends object> = {
  [K in keyof T as `modified_${string & K}`]: T[K];
};

interface Foo {
  name: string;
  age: number;
}

// {
//   modified_name: string;
//   modified_age: number;
// }
type CopiedFoo = CopyWithRename<Foo>;
```

这里其实就是通过 `as` 语法，将映射的键名作为变量，映射到一个新的字符串类型。需要注意的是，由于对象的合法键名类型包括了 symbol，而模板字符串类型插槽中并不支持 symbol 类型，因此使用 `string & K` 来确保了最终交由模板插槽的值，一定会是合法的 string 类型。

### 专用工具类型

这些工具类型专用于字符串字面量类型，包括 **Uppercase、Lowercase、Capitalize** 与 **Uncapitalize** ，作用分别为：字符串大写、字符串小写、首字母大写与首字母小写：

```typescript
type Heavy<T extends string> = `${Uppercase<T>}`;
type Respect<T extends string> = `${Capitalize<T>}`;

type HeavyName = Heavy<'test'>; // "TEST"
type RespectName = Respect<'test'>; // "Test"
```

在重映射部分，将键名从 `name` 修改成了 `modified_name` 的形式，如果要修改成更习惯的小驼峰形式，此时就可以使用 Capitalize 工具类型：

```typescript
type CopyWithRename<T extends object> = {
  [K in keyof T as `modified${Capitalize<string & K>}`]: T[K];
};

interface Foo {
  name: string;
  age: number;
}

// {
//   modifiedName: string;
//   modifiedAge: number;
// }
type CopiedFoo = CopyWithRename<Foo>;
```

实际上，这是 TS 中首次引入了**能直接改变类型本身含义**的工具类型。跳转到源码定义时发现它们的定义是这样的：

```typescript
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;
```

intrinsic 代表了这一工具类型由 TS 内部进行实现，查看内部的源码发现：

```typescript
function applyStringMapping(symbol: Symbol, str: string) {
  switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
    case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
    case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
    case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
    case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
  }
  return str;
}
```

在这里字符串字面量类型被作为一个字符串值一样进行处理，这些工具类型通过调用了字符串的 toUpperCase 等原生方法实现。

### 模板字符串类型与模式匹配

模式匹配工具类型的核心理念就是对符合约束的某个类型结构，提取其某一个位置的类型，比如函数结构中的参数与返回值类型。而如果将一个字符串类型视为一个结构，就能够在其中也应用模式匹配相关的能力，而此前缺少的就是模板字符串类型的能力。

模板插槽不仅可以声明一个占位的坑，也可以声明一个要提取的部分：

```typescript
type ReverseName<Str extends string> = 
  Str extends `${infer First} ${infer Last}` ? `${Capitalize<Last>} ${First}` : Str;
```

一共在两处使用了模板字符串类型，首先是在约束部分，希望传入的字符串字面量类型是 `"Tom Hardy"` 这样的形式。注意，这里的空格也需要严格遵循，因为它也是一个字面量类型的一部分。对于符合这样约束的类型，使用**模板插槽 + infer 关键字**提取了其空格旁的两个部分（即名与姓），然后在条件类型中，将 infer 提取出来的值，再次使用模板插槽注入到了新的字符串类型中。

实际使用一下：

```typescript
type ReverseName1 = ReverseName<'Tom hardy'>; // "Hardy Tom"
```

如果传入的字符串字面量类型中有多个空格，这种情况下，模式匹配将只会匹配首个空格，即 `"A B C"` 会被匹配为 `"A"` 与 `"B C"` 这样的两个结构：

```typescript
type ReversedRes1 = ReverseName<'Vic One 111'>; // "One 111 Vic"
```

除了显式使用 infer 进行模式匹配操作以外，由于模板字符串的灵活性，甚至可以直接声明一个泛型来进行模式匹配操作：

```typescript
declare function handler<Str extends string>(arg: `Guess who is ${Str}`): Str;

handler('Guess who is vic'); // "vic"
handler('Guess who is '); // ""
handler('Guess who is  '); // " "

handler('Guess who was'); // Error
handler(''); // Error
```

### 扩展

**基于重映射的 PickByValueType**

在这一节了解了重映射这一能力，它使得可以在映射类型中去修改映射后的键名，而如果映射后的键名变成了 never，那么这个属性将不会出现在最终的接口结构中。可以基于重映射来实现结构处理工具类型，比如说 PickByValueType：

```typescript
type PickByValueType<T extends object, Type> = {
  [K in keyof T as T[K] extends Type ? K : never]: T[K]
}
```

在重映射中再次进行了条件类型判断，并在其成立时才重映射到原键名，否则只返回一个 never。类似的，也可以实现 OmitByType 等等。

## 类型编程新范式：模板字符串工具类型进阶

### 从最简单的模式匹配说起：Trim、Includes 等

最简单的模式匹配只有一层条件类型语句，也就意味着不需要对模式匹配的结果做结构转换等操作。对比到字符串类型变量的方法，也就是 trim（trimLeft、trimRight）、includes、startsWith 与 endsWith。

首先从 includes 看起，对应实现一个类型层面的版本：**判断传入的字符串字面量类型中是否含有某个字符串**：

```typescript
type Include<
  Str extends string, 
  Search extends string
> = Str extends `${infer _R1}${Search}${infer _R2}` ? true : false;
```

在 Include 类型中，我们在 Search 前后声明了两个 infer 插槽，但实际上并不消费 R1 与 R2，而只是判断字符串是否可以被划分为**要搜索的部分 + 其他部分**。来验证一下实际效果：

```typescript
type IncludeRes1 = Include<'test', 't'>; // true
type IncludeRes2 = Include<'test', '_t'>; // false
type IncludeRes3 = Include<'test', ''>; // true
type IncludeRes4 = Include<'  ', ''>; // true
type IncludeRes5 = Include<'', ''>; // false
```

在 IncludeRes5 中，发现对于空字符串 `''` 需要进行特殊的处理， `''.includes('')` 也应当是成立的，就像实际字符串中进行判断一样。因此需要新增额外处理：

```typescript
type _Include<
  Str extends string, 
  Search extends string
> = Str extends `${infer _R1}${Search}${infer _R2}` ? true : false;

type Include<Str extends string, Search extends string> = Str extends '' 
  ? Search extends '' 
    ? true
    : false
  : _Include<Str, Search>;
```

当字符串 Str 为空字符串时，判断 Search 是否是空字符串来直接决定返回结果。在 Str 不为空字符串时，才会真的进行 Include 的判断。在 Str 与 Search 均为空字符串的情况下，直接返回 true，否则才进行模式匹配。

提到模板字符串中的空字符串，就会想到 trim 三兄弟：去除其实部分空格的 trimStart，去除结尾部分空格的 tri mEnd，以及开头结尾空格一起去的 trim。

```typescript
// trimStart
type TrimLeft<V extends string> = V extends ` ${infer R}` ? R : V;

// trimEnd
type TrimRight<V extends string> = V extends `${infer R} ` ? R : V;

// trim
type Trim<V extends string> = TrimLeft<TrimRight<V>>;
```

这里的实现只能去掉一个空格，为了实现去掉多个空格，需要使用递归：

```typescript
// trimStart
type TrimLeft<V extends string> = V extends ` ${infer R}` ? TrimLeft<R> : V;

// trimEnd
type TrimRight<V extends string> = V extends `${infer R} ` ? TrimRight<R> : V;

// trim
type Trim<V extends string> = TrimLeft<TrimRight<V>>;
```

而类型版本的 StartWith 与 EndsWith 两个工具类型，和 Include 的实现非常接近。直接看 Start With 的最终实现与验证：

```typescript
type _StartWith<
  Str extends string, 
  Search extends string
> = Str extends `${Search}${infer _R}` ? true : false;

type StartWith<Str extends string, Search extends string> = Str extends '' 
  ? Search extends '' 
    ? true 
    : _StartWith<Str, Search>
  : _StartWith<Str, Search>;

type StartWithRes1 = StartWith<'test', 'te'>; // true
type StartWithRes2 = StartWith<'test', ''>; // true
type StartWithRes3 = StartWith<'test', '  '>; // false
type StartWithRes4 = StartWith<'', ''>; // true
type StartWithRes5 = StartWith<'  ', ''>; // true
```

### 结构转换：Replace、Split 与 Join

Include 判断是否能将字符串字面量划分为目标部分与其他部分，那 Replace 只需要将目标部分替换为新的部分，按照原本的结构组合好就行。

一切复杂的工具类型最终都可以转换为数个简单工具类型的组合。

```typescript
export type Replace<
  Str extends string, 
  Search extends string,
  Replacement extends string
> = Str extends `${infer Head}${Search}${infer Tail}` 
  ? `${Head}${Replacement}${Tail}` 
  : Str;
```

这里实际是先判断字符串字面量中是否包含 Search 部分（就像 Include 那样），在包含也就是结构符合时，将匹配得到的 Head 与 Tail 部分夹上 Replacement，这样就实现了一个类型版本的 Replace：

```typescript
// "我已经替换一段文本"
type ReplaceRes1 = Replace<'我在试着替换一段文本', '在试着', '已经'>;

// 不发生替换
type ReplaceRes2 = Replace<'我在试着替换一段文本', '???', '??'>;
```

如果遇到全量替换的场景，也就是 ECMAScript 2021 的 replaceAll 方法。那么如何在类型层面实现一个 replaceAll？可以使用递归方法：

```typescript
export type ReplaceAll<
  Str extends string, 
  Search extends string,
  Replacement extends string
> = Str extends `${infer Head}${Search}${infer Tail}` 
  ? ReplaceAll<`${Head}${Replacement}${Tail}`, Search, Replacement> 
  : Str;

// "mmm.baodu.com"
type ReplaceAllRes1 = ReplaceAll<'www.baodu.com', 'w', 'm'>;

// "www-baodu-com"
type ReplaceAllRes2 = ReplaceAll<'www.baodu.com', '.', '-'>;
```

如果将这两个类型合并在一起，再通过选项来控制是否进行全量替换：

```typescript
export type Replace<
  Input extends string, 
  Search extends string, 
  Replacement extends string, 
  ShouldReplaceAll extends boolean = false
> = Input extends `${infer Head}${Search}${infer Tail}` 
  ? ShouldReplaceAll extends true 
    ? Replace<
      `${Head}${Replacement}${Tail}`, 
      Search, 
      Replacement, 
      ShouldReplaceAll
    >
    : `${Head}${Replacement}${Tail}` 
  : Input;
```

在字符串类型值中还有一个常用的方法： split，它会将字符串按照确定的分隔符拆分成一个数组。

```typescript
export type Split<
  Str extends string, 
  Delimiter extends string
> = Str extends `${infer Head}${Delimiter}${infer Tail}` 
  ? [Head, ...Split<Tail, Delimiter>] 
  : Str extends Delimiter 
    ? [] 
    : [Str];

// ["test", " 1", " 2", " 3"]
type SplitRes1 = Split<'test, 1, 2, 3', ','>;

// ["t", "e", "s", "t", "1", "2", "3"]
type SplitRes2 = Split<'test123', ''>;
```

有两种情况需要注意。第一种，存在多处分割时，Split 类型进行到最后一次，即无法再分割时，需要直接将最后一部分给返回。第二种，对于空字符串作为分隔符，其表现为将字符串字面量按字母进行拆分，这同样与 Split 方法的实际表现一致。

在实际情况中，字符串可能就包含了多种可能的分隔符，即这里的 Delimiter 可以是一个联合类型 `"_" | "-" | " "`。在这种情况下，模板字符串中的模式匹配也能够生效，它会使用这里的多个分隔符依次进行判断，并在判断到其中一种就立刻成立：

```typescript
type Delimiters = '-' | '_' | ' ';

// ["iphone", "10", "128G"]
type SplitRes4 = Split<'iphone_10_128G', Delimiters>;
```

但需要注意的是，并不能在一个字符串中混用多种分隔符，在这种情况下由于联合类型在插槽中的排列组合特性，会得到一个诡异的结果：

```typescript
// ["iphone" | "iphone_10", "128G"] | ["iphone" | "iphone_10", "10", "128G"]
type SplitRes5 = Split<'iphone_10-128G', Delimiters>;
```

实际上，每次只能依据一种分隔符进行拆分才是符合预期的。在正常的变量命名中，通常只会使用一种分隔方式。确实使用了多种具有实际意义的分隔符时，应该进行多次拆分，如 CSS 的 BEM 命名方式（`Block__Element--Modifier`）下，经常会这样写类名：`footer__button--danger`。此时，应当先按照`__`拆出 Block，再按照 `--` 拆出 Modifier。

另外，基于 Split 类型还可以获取字符串长度：

```typescript
export type StrLength<T extends string> = Split<Trim<T>, ''>['length'];

type StrLengthRes1 = StrLength<'test'>; // 4
type StrLengthRes2 = StrLength<'a test'>; // 6
type StrLengthRes3 = StrLength<' '>; // 0
type StrLengthRes4 = StrLength<''>; // 0
```

Join 的作用和 Split 相反。Split 方法是**将字符串按分隔符拆分分成一个数组**，而 Join 方法则是**将一个数组中的所有字符串按照分隔符组装成一个字符串**。只需要通过递归依次取出每一个字符串单元，使用模板插槽组装即可：

```typescript
export type Join<
  List extends Array<string | number>, 
  Delimiter extends string
> = List extends [string | number, ...infer Rest] 
  ? // @ts-expect-error
    `${List[0]}${Delimiter}${Join<Rest, Delimiter>}` 
  : string;
```

这里的 Rest 类型无法被正确地推导，因此使用了 //@ts-expect-error 来忽略错误。

进行测试：

```typescript
// `test-a-example-${string}`
type JoinRes1 = Join<['test', 'a', 'example'], '-'>;
```

很明显，这是不对的。这是因为在递归到最后一次的时候：

```typescript
export type JoinTmp = [] extends [string | number, ...infer Rest] 
	?//@ts-expect-error
		`test-a-example-${Join<Rest, Delimiter>}`
	: string;
```

这个条件明显不成立，因此它返回了 string 类型，而这个 string 类型我们的本义是用来兜底：如果 Join 无法拼接一个列表，那至少要返回一个 string 类型。

要解决这种情况，需要额外处理一下空数组的情况：

```typescript
export type Join<
  List extends Array<string | number>, 
  Delimiter extends string
> = List extends [] 
  ? '' 
  : List extends [string | number, ...infer Rest] 
    ? // @ts-expect-error
      `${List[0]}${Delimiter}${Join<Rest, Delimiter>}` 
    : string;
```

但结果还是不太对：

```typescript
// "test-a-example-"
type JoinRes1 = Join<['test', 'a', 'example'], '-'>;
```

实际上，在进行到最后一项数组成员时，递归过程就应当被提前阻止。因此需要处理只剩下最后一项的情况：

```typescript
export type Join<
  List extends Array<string | number>, 
  Delimiter extends string
> = List extends [] 
  ? '' 
  : List extends [string | number] 
    ? `${List[0]}`  
    : List extends [string | number, ...infer Rest] 
      ? // @ts-expect-error
        `${List[0]}${Delimiter}${Join<Rest, Delimiter>}` 
      : string;

// "test-a-example"
type JoinRes1 = Join<['test', 'a', 'example'], '-'>;
```

**一个工具类型有时需要多次改进、多种边界情况处理，才能称为“可用”，尤其是在递归的情况下**。

### 最后一步：Case 转换

Case 转换一定是建立在**传入字符串已经拥有了一种 case 的情况**。

首先是 SnakeCase 转换到 CamelCase，也就是下划线转小驼峰。

```typescript
type SnakeCase2CamelCase<S extends string> = 
  S extends `${infer Head}${'_'}${infer Rest}` 
    ? `${Head}${SnakeCase2CamelCase<Capitalize<Rest>>}` 
    : S;

expectType<SnakeCase2CamelCase<'foo_bar_baz'>>('fooBarBaz');
```

解决了 SnackCase，举一反三，KebabCase（中划线）其实也解决了。

```typescript
type KebebCase2CamelCase<S extends string> = 
  S extends `${infer Head}${'-'}${infer Rest}` 
    ? `${Head}${KebebCase2CamelCase<Capitalize<Rest>>}` 
    : S;

expectType<KebebCase2CamelCase<'foo-bar-baz'>>('fooBarBaz');
```

将分隔符的能力进行抽象，支持任意的分隔符：

```typescript
type DelimiterCase2CamelCase<
  S extends string, 
  Delimiter extends string
> = S extends `${infer Head}${Delimiter}${infer Rest}` 
  ? `${Head}${DelimiterCase2CamelCase<Capitalize<Rest>, Delimiter>}` 
  : S;
```

验证一下效果：

```typescript
expectType<DelimiterCase2CamelCase<'foo-bar-baz', '-'>>('fooBarBaz');
expectType<DelimiterCase2CamelCase<'foo~bar~baz', '~'>>('fooBarBaz');
expectType<DelimiterCase2CamelCase<'foo bar baz', ' '>>('fooBarBaz');
```

这里还存在非常大的优化空间，比如可以让它自动处理分隔符。通常变量命名只会使用 `_` 和 `-` 作为分隔符，加上字面量中可能存在的空格，也就是说希望自动处理 `"-" | "_" | " "` 这三个分隔符。

很容易想当然地这样写：

```typescript
 type WordDelimiter = '-' | '_' | ' ';

type DelimiterCase2CamelCaseAuto<S extends string> = 
  S extends `${infer Head}${infer Delimiter}${infer Rest}` 
    ? Delimiter extends WordDelimiter 
      ? `${Head}${DelimiterCase2CamelCaseAuto<Capitalize<Rest>>}` 
      : S 
    : S;
```

由上一部分的知识，很明显这是错的。对于这种连续的 infer 插槽，其匹配策略是尽可能为前面的每个插槽匹配一个字符，然后将所有剩下的部分都交给最后一个插槽。

因此要实现一个自动分割的版本，还需要一些额外的工作，但思路是一致的：按照分隔符拆分，对除首个字符串以外的字符单元进行首字母大写处理以及组装。如果分隔符并不确定的情况下如何做呢？

可以利用上面讲到的 Split 类型：

```typescript
type Delimiter = '-' | '_' | ' ';

// ["foo", "bar", "baz"]
type SplitRes4 = Split<'foo_bar_baz', Delimiter>;
```

也就是说，可以使用 Split 将字符串拆分成数组，然后在数组中去处理第一项以外的其他成员：

```typescript
export type CamelCase<K extends string> = CamelCaseStringArray<
  Split<K, Delimiter>
>;
```

而 CamelCaseStringArray 这个类型，希望它能够将 `['foo', 'bar', 'baz']` 转化为 `['foo', 'Bar', 'Baz']`。也就是说这个数组可以分为两个部分，无需处理的第一项和全部首字母大写的其余项：

```typescript
type CamelCaseStringArray<Words extends string[]> = Words extends [
  `${infer First}`, 
  ...infer Rest
]
  ? `${First}${CapitalizeStringArray<Rest>}` 
  : never;
```

由于这里的 First 和 Rest 被视为两种不同的结构，因此需要再声明一个 CapitalizeStringArray 类型，它的作用就是将**递归地数组中所有地字符串单元转化为首字母大写形式**：

```typescript
type CapitalizeStringArray<Words extends any[]> = Words extends [
  `${infer First}`, 
  ...infer Rest
] 
  ? `${Capitalize<First>}${CapitalizeStringArray<Rest>}` 
  : '';
```

这样就得到了一个初具雏形的 Camel Case 智能版：

```typescript
export type Split<
  Str extends string, 
  Delimiter extends string
> = Str extends `${infer Head}${Delimiter}${infer Tail}` 
  ? [Head, ...Split<Tail, Delimiter>] 
  : Str extends Delimiter 
    ? [] 
    : [Str];
 
type Delimiter = '-' | '_' | ' ';

type CapitalizeStringArray<Words extends any[]> = Words extends [
  `${infer First}`, 
  ...infer Rest
] 
  ? `${Capitalize<First>}${CapitalizeStringArray<Rest>}` 
  : '';

type CamelCaseStringArray<Words extends string[]> = Words extends [
  `${infer First}`, 
  ...infer Rest
]
  ? `${First}${CapitalizeStringArray<Rest>}` 
  : never;

export type CamelCase<K extends string> = CamelCaseStringArray<
  Split<K, Delimiter>
>;
```

验证效果：

```typescript
expectType<CamelCase<'foo bar baz'>>('fooBarBaz');
expectType<CamelCase<'foo-bar-baz'>>('fooBarBaz');
expectType<CamelCase<'foo_bar_baz'>>('fooBarBaz');
```

这里的 CamelCase 其实还有一些需要改进的地方，比如首字母大写的 `Foo-bar-baz` 和全大写的 `FOO-BAR-BAZ` ，也需要转化为小驼峰形式的 `fooBarBaz` 。

最终实现如下：

```typescript
export type PlainObjectType = Record<string, any>;

export type WordSeparators = '-' | '_' | ' ';

export type Split<
  Str extends string, 
  Delimiter extends string
> = Str extends `${infer Head}${Delimiter}${infer Tail}` 
  ? [Head, ...Split<Tail, Delimiter>] 
  : Str extends Delimiter 
    ? [] 
    : [Str];

type CapitalizeStringArray<Words extends readonly any[], Prev> = Words extends [
  `${infer First}`, 
  ...infer Rest
] 
  ? First extends undefined 
    ? '' 
    : First extends ''
      ? CapitalizeStringArray<Rest, Prev> 
      : `${Prev extends '' ? First : Capitalize<First>}${CapitalizeStringArray<
          Rest, 
          First
        >}`
  : '';

type CamelCaseStringArray<Words extends string[]> = Words extends [
  `${infer First}`, 
  ...infer Rest
]
  ? Uncapitalize<`${First}${CapitalizeStringArray<Rest, First>}`> 
  : never;

export type CamelCase<K extends string> = CamelCaseStringArray<
  Split<K extends Uppercase<K> ? Lowercase<K> : K, WordSeparators>
>;
```

另外，虽然 Camel Case 只是对一维字符串字面量进行的转换，但由于重映射能力，可以被应用到对象类型层面：

```typescript
export type CamelCaseProperties<T extends PlainObjectType> = {
  [K in keyof T as CamelCase<string & K>]: T[K] extends object
    ? CamelCaseProperties<T[K]> 
    : T[K];
};

expectType<
  CamelCaseProperties<{ foo_bar: string; foo_baz: { nested_foo: string } }>
>({
  fooBar: '',
  fooBaz: {
    nestedFoo: '',
  },
});
```

 除了 Camel Case 以外，其实也很容易对应着实现智能版的 Delimiter Case、Snake Case 等。

## 工程层面的类型能力：类型声明、类型指令与命名空间

### 类型检查指令

在前端世界的很多工具中，其实都提供了**行内注释**的能力，用于支持在某一处特定代码**使用特殊的配置来盖掉全局配置**。最常见的即是 ESLint 与 Prettier 提供的禁用检查能力，如 `/* eslint-disable-next-lint */` 、 `<!-- prettier-ignore -->` 等。

TS 中同样提供了数个行内注释（这里我们称为类型指令），来进行单行代码或单文件级别的配置能力。这些指令均以 `// @ts-` 开头。

**ts-ignore 与 ts-expect-error**

`ts-ignore` 应该是使用最为广泛的一个类型指令了，它的作用就是直接禁用掉对下一行代码的类型检查：

```typescript
// @ts-ignore
const name: string = 599;
```

基本上所有的类型报错都可以通过这个指令来解决，但由于其本质上是 ignore 而不是 disable，也就意味着如果下一行代码并没有问题，那么使用 ignore 反而就是一个错误了。因此 TS 随后又引入了一个更严格版本的 ignore，即 `ts-expect-error`，它只有在下一行代码真的存在错误时才能被使用，否则会给出一个错误：

```typescript
// @ts-expect-error
const name: string = 599;

// @ts-expect-error 错误使用此指令，报错
const age: number = 599;
```

在这里第二个 expect-error 指令会给出一个报错：**无意义的 expect-error 指令**。

**在所有地方都不要使用 ts-ignore。**

这两个指令只能对单行代码生效，如果有非常多的类型报错要处理（比如正在将一个 JS 文件迁移到 TS），很显然不可能一个个为所有报错的地方添加上禁用检查指令。正如 ESLint 中可以使用 `/* eslint-disable-next-line */` 禁用下一行代码检查，也可以使用 `/* eslint-disable */` 禁用整个文件检查一样，TS 也提供了对整个文件生效的类型指令：`ts-check` 与 `ts-nocheck` 。

**ts-check 与 ts-nocheck**

首先看 ts-nocheck，可以把它理解为一个作用于整个文件的 ignore 指令，使用了 ts-nocheck 指令的 TS 文件将不再接受类型检查：

```typescript
// @ts-nocheck 以下代码均不会抛出错误
const name: string = 599;
const age: number = 'test';
```

那么 `ts-check` 呢？这看起来是一个多余的指令，因为默认情况下 TS 文件不是就会被检查吗？实际上，这两个指令还可以用在 JS 文件中。TS 并不是只能检查 TS 文件，对于 JS 文件它也可以通过类型推导与 JSDoc 的方式进行不完全的类型检查。

```typescript
// JavaScript 文件
let myAge = 18;

// 使用 JSDoc 标注变量类型
/** @type {string} */
let myName;

class Foo {
  prop = 599;
}
```

在上述代码中，声明了初始值的 myAge 与 `Foo.prop` 都能被推导出其类型，而无初始值的 myName 也可以通过 JSDoc 标注的方式来显式地标注类型。

但 JS 是弱类型语言，表现之一即是变量可以**被赋值为与初始值类型不一致的值**，如上面的例子进一步改写：

```typescript
let myAge = 18;
myAge = '90'; // 与初始值类型不同

// 使用 JSDoc 标注变量类型
/** @type {string} */
let myName;
myName = 599; // 与 JSDoc 标注类型不同
```

赋值操作在类型层面显然是不成立的，但因为是在 JavaScript 文件中，因此这里不会有类型报错。如果希望在 JS 文件中也能享受到类型检查，此时 `ts-check` 指令就可以登场了。

```typescript
// @ts-check
/** @type {string} */
const myName = 599; // 报错！

let myAge = 18;
myAge = '200'; // 报错！
```

这里的 `ts-check` 指令为 JS 文件也带来了类型检查，而同时还可以使用 `ts-expect-error` 指令来忽略掉单行的代码检查：

```typescript
// @ts-check
/** @type {string} */
// @ts-expect-error
const myName = 599; // OK

let myAge = 18;
// @ts-expect-error
myAge = '200'; // OK
```

而 `ts-nocheck` 在 JS 文件中的作用和 TS 文件其实也一致，即禁用掉对当前文件的检查。如果希望开启对所有 JS 文件的检查，只是忽略掉其中少数，此时需要在 TSConfig 中启用 `checkJs` 配置，来开启**对所有包含的 JS 文件的类型检查**，然后使用 `ts-nocheck` 来忽略掉其中少数的 JS 文件。

### 类型声明

类型声明实际上就是 `declare` 语法：

```typescript
declare var f1: () => void;

declare interface Foo {
  prop: string;
}

declare function  foo(input: Foo): Foo;

declare class Foo {}
```

可以直接访问这些声明：

```typescript
declare let otherProp: Foo['prop'];
```

但不能为这些声明变量赋值：

```typescript
// Error 不允许在环境上下文中使用初始值
declare let result1 = foo();

// OK Foo
declare let result2: ReturnType<typeof foo>;
```

这些类型声明就像在 TS 中的类型标注一样，会存放着特定的类型信息，同时由于它们并不具有实际逻辑，可以很方便地使用类型声明来进行类型兼容性的比较、工具类型的声明与测试等等。

除了手动书写这些声明文件，更常见的情况是 TS 代码在编译后生成声明文件：

```typescript
// 源代码
const handler = (input: string): boolean => {
  return input.length > 5;
}

interface Foo {
  name: string;
  age: number;
}

const foo: Foo = {
  name: 'vic',
  age: 18,
};

class FooCls {
  prop!: string;
}
```

这段代码在编译后会生成一个 `.js` 文件和一个 `.d.ts` 文件，而后者即是类型声明文件：

```typescript
// 生成的类型定义
declare const handler: (input: string) => boolean;

interface Foo {
    name: string;
    age: number;
}

declare const foo: Foo;

declare class FooCls {
    prop: string;
}
```

这样一来，别的文件或别的项目导入了这段代码，它们就能够从这些类型声明获得对应部分的类型，这也是类型声明的核心作用：**将类型独立于 `.js` 文件进行存储**。别人在使用这些代码的时候就能够获得这些额外的类型信息。同时，如果在使用别人没有携带类型声明的 `.js` 文件，也可以通过类型声明进行类型补全。

### 让类型定义全面覆盖你的项目

类型声明的核心能力：通过额外的类型声明文件，在核心代码以外去提供对类型的进一步补全。类型声明文件，即 `.d.ts` 结尾的文件，会自动地被 TS 加载到环境中，实现对应部分代码的类型补全。

声明文件中并不包含实际的代码逻辑，它做的事只有一件：**为 TS 类型检查与推导提供额外的类型信息**，而使用的语法仍然是 TS 的 declare 关键字。

首先是无类型定义的 npm 包，可以通过 declare module 的方式来提供其类型：

```typescript
import foo from 'pkg';

const res = foo.handler();
```

这里的 pkg 是一个没有类型定义的 npm 包（实际并不存在）。

```typescript
declare module 'pkg' {
	const handler: () => boolean;
}
```

现在 res 就具有了 boolean 类型！`declare module 'pkg'` 会为默认导入 `foo` 添加一个具有 handler 的类型，虽然这里的 `pkg` 根本不存在。也可以在 `declare module` 中使用默认导出：

```typescript
declare module 'pkg2' {
	const handler: () => boolean;
	export default handler;
}

import bar from 'pkg2';

bar();
```

在 `'pkg'` 的类型声明中，也可以使用 `export const`，效果是一致的，但由于对 `'pkg2'` 使用了默认导入，因此必须要有一个 `export default` 。

除了为缺失类型的模块声明类型以外，使用类型声明还可以为非代码文件，如图片、CSS文件等声明类型。

对于非代码文件，比如说 markdown 文件，假设希望导入一个 `.md` 文件，由于其本质和 npm 包一样是一条导入语句，因此可以类似地使用 declare module 语法：

```typescript
// index.ts
import raw from './note.md';

const content = raw.replace('NOTE', `NOTE${new Date().getDay()}`);

// declare.d.ts
declare module '*.md' {
	const raw: string;
	export default raw;
}
```

对于非代码文件的导入，更常见的其实是 `.css`、`.module.css`、`.png` 这一类，但基本语法都相似。

**DefinitelyTyped**

简单来说，`@types/` 开头的这一类 npm 包均属于 DefinitelyTyped，它是 TS 维护的，专用于为社区存在的**无类型定义的 JS 库**添加类型支持，常见的有 `@types/react` 、`@types/lodash` 等等。通过 DefinitelyTyped 来提供类型定义的包常见的有几种情况，如 Lodash 这样的库仍然有大量 JS 项目使用，将类型定义内置在里面不一定是所有人都需要的，反而会影响包的体积。还有像 React 这种不是用纯 JS / TS 书写的库，需要自己来手写类型声明（React 是用 Flow 写的，这是一门同样为 JS 添加类型的语言或者说语法）。

举例来说，只要安装了 `@types/react` ，TS 会自动将其加载到环境中（实际上所有 `@types/` 下的包都会自动被加载），并作为 react 模块内部 API 的声明。但这些类型定义并不一定都是通过 `declare module`，命名空间 namespace 其实也可以实现一样的能力。

先来看 `@types/node` 中与 `@types/react` 中分别是如何进行类型声明的：

```typescript
// @types/node
declare module 'fs' {
	export function readFileSync(/** 省略 */): Buffer;
}

// @types/react
declare namespace React {
	function useState<S>(): [S, Dispatch<SetStateAction<S>>];
}
```

可以看到，`@types/node` 中仍然使用 `declare module` 的方式为 `fs` 这个内置模块声明了类型，而 `@types/react` 则使用的是 `declare namespace`。

如果第三方库并不是通过导出来使用，而是直接在全局注入了变量，需要通过 `window.xxx` 的方式访问。

**扩展已有的类型定义**

对全局变量的声明，还是以 window 为例，实际上如果 Ctrl + 点击代码中的 window，会发现它已经有类型声明了：

```typescript
declare var window: Window & typeof globalThis;

interface Window {
	// ...
}
```

这行代码来自于 `lib.dom.d.ts` 文件，它定义了对浏览器文档对象模型的类型声明，这就是 TS 提供的内置类型，也是“出厂自带”的类型检查能力的依据。类似的，还有内置的 `lib.es2021.d.ts` 这种文件定义了对 ECMAScript 每个版本的类型声明新增或改动等等。

我们要做的，实际上就是在内置类型声明的基础上，再新增一部分属性。而别忘了，在 JS 中当访问全局变量时，是可以直接忽略 `window` 的：

```typescript
onerror = () => {};
```

反过来，在类型声明中，如果直接声明一个变量，那就相当于将它声明在了全局空间中：

```typescript
// 类型声明
declare const errorReporter: (err: any) => void;

// 实际使用
errorReporter("err!");
```

而如果想要将它显式的添加到已有的 `Window` 接口中呢？在接口一节中已经了解到，如果有多个同名接口，那么这些接口实际上是会被合并的，这一特性在类型声明中也是如此。因此，再声明一个 Window 接口即可：

```typescript
interface Window {
	userTracker: (...args: any[]) => Promise<void>;
}

window.userTracker("click!")
```

类似的，也可以扩展来自 `@types/` 包的类型定义：

```typescript
declare module 'fs' {
	export function bump(): void;
}

import { bump } from 'fs';
```

总结一下这两个部分，TS 通过 DefinitelyTyped，也就是 `@types/` 系列的 npm 包来为无类型定义的 JS npm 包提供了类型支持，这些类型定义的 npm 包内部其实就是数个 `.d.ts` 这样的声明文件。

而这些声明文件主要通过 declare / namespace 的语法进行类型的描述，可以通过项目内额外的声明文件，来实现为非代码文件的导入，或者是全局变量添加上类型声明。

### 三斜线指令

三斜线指令就像是声明文件中的导入语句一样，它的作用就是**声明当前的文件依赖的其他类型声明**。而这里的“其他类型声明”包括了 TS 内置类型声明（`lib.d.ts`）、三方库的类型声明以及自己提供的类型声明文件等。

三斜线指令本质上就是一个自闭合的 XML 标签，其语法大致如下：

```typescript
/// <reference path="./other.d.ts" />
/// <reference types="node" />
/// <reference lib="dom" />
```

**需要注意的是，三斜线指令必须被放置在文件的顶部才能生效。**

这里的三条指令作用其实都是声明当前文件依赖的外部类型声明，只不过使用的方式不同：分别使用了 path、type、lib 这三个不同属性。

使用 path 的 reference 指令，其 path 属性的值为一个相对路径，指向项目内的其他声明文件。而在编译时， TS 会沿着 path 指定的路径不断深入寻找，最深的那个没有其他依赖的声明文件会被最先加载。

```typescript
// @types/node 中的示例
/// <reference path="fs.d.ts" />
```

使用 types 的 reference 指令，其 types 的值是一个包名，也就是想引入的 `@types/` 声明，如上面的例子中实际上是在声明当前文件对 `@types/node` 的依赖。而如果代码文件（`.ts`）中声明了对某一个包的类型导入，那么在编译产生的声明文件（`.d.ts`）中会自动包含引用它的指令。

```typescript
/// <reference types="node" />
```

使用 lib 的 reference 指令类似于 types，只不过这里 lib 导入的是 TS 内置的类型声明，如下面的例子声明了对 `lib.dom.d.ts` 的依赖：

```typescript
// vite/client.d.ts
/// <reference lib="dom" />
```

而如果使用 `/// <reference lib="esnext.promise" />`，那么将依赖的就是 `lib.esnext.promise.d.ts` 文件。

这三种指令的目的都是引入当前文件所依赖的其他类型声明，只不过适用场景不同而已。

### 命名空间

假设一个场景，项目需要接入多个平台的支付 SDK，最开始只有微信支付和支付宝：

```typescript
class WeChatPaySDK {}

class ALiPaySDK {}
```

然后又多了美团支付、虚拟货币支付（比如 Q 币）、信用卡支付等等：

```typescript
class WeChatPaySDK {}

class ALiPaySDK {}

class MeiTuanPaySDK {}

class CreditCardPaySDK {}

class QQCoinPaySDK {}
```

随着业务的不断发展，项目中可能需要引入越来越多的支付 SDK，甚至还有比特币和以太坊，此时将这些所有的支付都放在一个文件内未免过于杂乱了。这些支付方式其实大致可以分为两种：现实货币与虚拟货币。此时就可以使用命名空间来区分这两类 SDK：

```typescript
export namespace RealCurrency {
	export class WeChatPaySDK {}
	
	export class ALiPaySDK {}
	
	export class MeiTuanPaySDK {}
	
	export class CreditCardPaySDK {}
}

export namespace VirtualCurrency {
	export class QQCoinPaySDK {}
	
	export class BitCoinPaySDK {}
	
	export class ETHPaySDK {}
}
```

注意，这里的代码是在 `.ts` 文件中的，此时它是具有实际逻辑意义的，也不能和类型混作一谈。

而命名空间的使用类似于枚举：

```typescript
const weChatPaySDK = new RealCurrency.WeChatPaySDK();
```

唯一需要注意的是，命名空间内部实际上就像是一个独立的代码文件，因此其中的变量需要导出以后，才能通过 `RealCurrency.WeChatPaySDK` 这样的形式访问。

命名空间的编译产物就像是上古时期里使用的伪模块化方案：

```js
export var RealCurrency;
(function (RealCurrency) {
    class WeChatPaySDK {
    }
    RealCurrency.WeChatPaySDK = WeChatPaySDK;
    class ALiPaySDK {
    }
    RealCurrency.ALiPaySDK = ALiPaySDK;
    class MeiTuanPaySDK {
    }
    RealCurrency.MeiTuanPaySDK = MeiTuanPaySDK;
    class CreditCardPaySDK {
    }
    RealCurrency.CreditCardPaySDK = CreditCardPaySDK;
})(RealCurrency || (RealCurrency = {}));
```

实际上，命名空间的作用也正是实现简单的模块化功能，在 TS 中引入它时（1.5版本），前端的模块化方案还处于混沌时期。

命名空间的内部还可以再嵌套命名空间，比如在虚拟货币中再新增区块链货币一类，此时嵌套的命名空间也需要被导出：

```typescript
export namespace VirtualCurrency {
	export class QQCoinPaySDK {}
	
	export namespace BlockChainCurrency {
		export class BitCoinPaySDK {}
		
		export class ETHPaySDK {}
	}
}

const ethPaySDK = new VirtualCurrency.BlockChainCurrency.ETHPaySDK();
```

类似于类型声明中的同名接口合并，命名空间也可以进行合并，但需要通过三斜线指令来声明导入。

```typescript
// animal.ts
namespace Animal {
	export namespace ProtectedAnimals {}
}

// dog.ts
/// <reference path="animal.ts" />
namespace Animal {
	export namespace Dog {
		export function bark() {}
	}
}

// corgi.ts
/// <reference path="dog.ts" />
namespace Animal {
	export namespace Dog {
		export namespace Corgi {
			export function corgiBark() {}
		}
	}
}
```

实际使用时需要导入全部的依赖文件：

```typescript
/// <reference path="animal.ts" />
/// <reference path="dog.ts" />
/// <reference path="corgi.ts" />

Animal.Dog.Corgi.corgiBark();
```

除了在 `.ts` 文件中使用以外，命名空间也可以在声明文件中使用，即 `declare namespace` ：

```typescript
declare namespace Animal {
	export interface Dog {}
	
	export interface Cat {}
}

declare let dog: Animal.Dog;
declare let cat: Animal.Cat;
```

但如果在 `@types/` 系列的包下，想要通过 namespace 进行模块的声明，还需要注意将其导出，然后才会加载到对应的模块下。以 `@types/react` 为例：

```typescript
export = React;
export as namespace React;
declare namespace React {
	// 省略了不必要的类型标注
	function useState<S>(initialState): [];
}
```

首先声明了一个命名空间 React，然后使用 `export = React` 将它导出了，这样就能够在从 react 中导入方法时，获得命名空间内部的类型声明，如 useState。

从这一个角度来看，`declare namespace` 其实就类似于普通的 `declare` 语法，只是内部的类型不再需要使用 `declare` 关键字（比如直接在 namespace 内部 `function useState(): []` 即可）。

而还有一行 `export as namespace React`，它的作用是在启用了 `--allowUmdGlobalAccess` 配置的情况下，允许将这个模块作为全局变量使用（也就是不导入直接使用），这一特性同样也适用于通过 CDN 资源导入模块时的变量类型声明。

除了这两个 namespace 使用，React 中还利用 namespace 合并的特性，在全局的命名空间中注入了一些类型：

```typescript
declare global {
	namespcace JSX {
		interface Element extends React.ReactElement<any, any> { }
	}
}
```

这也是为什么可以在全局使用 `JSX.Element` 作为类型标注。

### 仅类型导入

在 TS 中，当导入一个类型时其实并不需要额外的操作，和导入一个实际值是完全一样的：

```typescript
// foo.ts
export const Foo = () => {};

export type FooType = any;

// index.ts
import { Foo, FooType } from "./foo";
```

虽然类型导入和值导入存在于同一条导入语句中，在编译后的 JS 代码里还是只会有值导入存在，同时在编译的过程中，值与类型所在的内存空间也是分开的。

在这里只能通过名称来区分值和类型，但为每一个类型都加一个 Type 后缀太奇怪了。实际上，可以更好地区分值导入和类型导入，只需要通过 `import type` 语法即可：

```typescript
import { Foo } from "./foo";
import type { FooType } from "./foo";
```

这样会造成导入语句数量激增，如果想要抱持较少的导入语句数量又想区分值和类型导入，也可以使用同一导入语句内的方式（需要 4.6 版本以后才支持）：

```typescript
import { Foo, type FooType } from "./foo";
```

这实际上是个人编码习惯，即**对导入语句块的规范整理**。大致顺序如下：

* 一般最上面会是 React；
* 第三方 UI 组件，然后是项目内封装的其他组件；
* 第三方工具库，然后是项目内封装的工具方法；
* 类型导入，包括第三方库的类型导入、项目内的类型导入等；
* 样式文件，`CSS-IN-JS` 方案的组件应该被放在第二条中其他组件部分。

示例如下：

```typescript
import { useEffect } from 'react';

import { Button, Dialog } from 'ui';
import { ChildComp } from './child';

import { store } from '@/store';
import { useCookie } from '@/hooks/useCookie';
import { SOME_CONSTANTS } from '@/utils/constants';

import type { FC } from 'react';
import type { Foo } from '@/typings/foo';
import type { Shared } from '@/typings/shared';

import styles from './index.module.scss';
```

### 扩展：通过 JSDoc 在 JS 文件中获得类型提示

这里需要使用更强大一些的 JSDoc 能力：在 `@type {}` 中使用导入语句。

以 Webpack 为例：

```javascript
/** @type {import("webpack").Configuration} */
const config = {};

module.exports = config;
```

此时会发现已经拥有了如臂使指的类型提示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3074e724e8ef48f9ba02fae7e2c70b71~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

类似的，也可以直接进行导出：

```javascript
module.exports = /** @type { import('webpack').Configuration } */ ({});
```

当然，Webpack 本身也支持通过 ts 文件进行配置，在使用 TS 进行配置时，一种方式是简单地使用它提供的类型作为一个对象的标注。而目前更常见的一种方式其实是框架内部提供 `defineConfig` 这样的方法，让你能直接获得类型提示，如 Vite 中的做法：

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
	plugins: [react()]
});
```

## 在 React 中使用 TS ：内置类型与泛型坑位

### 项目初始化

这里使用 Vite 来进行项目搭建，在终端输入以下代码：

```
npx create-vite
```

输入项目名，选择 `react-ts` 模板即可。

### 项目配置

先观察基于 Vite 创建的初始项目里都包含了哪些配置。

首先是依赖，可以看到在  devDependencies 中包含了 `@types/react` 与 `@types/react-dom`。对于 `@types` 包的作用之前已经了解过，TS 会自动加载 `node_modules/@types` 下的类型定义来在全局使用。而除了这一点，当从 react 中导出一个类型：

```typescript
import { FC } from "react";
import type { FC } from "react";
```

实际上这个类型也来自于 `@types/react` 。接着是项目内的 `vite-env.d.ts` 声明文件，会发现它只有短短的一行：

```typescript
/// <reference types="vite/client" />
```

看看 `vite/client` 中都包含什么：

```typescript
/// <reference lib="dom" />
/// <reference path="./types/importMeta.d.ts" />

// CSS modules
type CSSModuleClasses = { readonly [key: string]: string }

declare module '*.module.css' {
  const classes: CSSModuleClasses
  export default classes
}
declare module '*.module.scss' {
  const classes: CSSModuleClasses
  export default classes
}
// ...

// CSS
declare module '*.css' {
  const css: string
  export default css
}
declare module '*.scss' {
  const css: string
  export default css
}
// ...


// Built-in asset types
// see `src/constants.ts`

// images
declare module '*.jpg' {
  const src: string
  export default src
}
// ...

// fonts
declare module '*.woff' {
  const src: string
  export default src
}

// ...
```

这里面包含的就是对于非实际代码文件导入的类型定义，包括了 CSS Modules、图片、视频等类型，通过这里的类型封装，在导入这些文件时就也能获得基本的类型保障。

三斜线指令并不是导入类型的唯一方式，可以使用 import 的方式来导入类型：

```typescript
// vite-env.d.ts
import * as ViteClientEnv from 'vite/client';
// 或使用 import type
import type * as ViteClientEnv from 'vite/client';
```

类型定义包与类型声明其实就是这个脚手架所进行的额外工作，也是日常开发中会重度依赖的部分。

### 组件声明

首先想想，在 React 中如何声明一个（函数）组件。最简单的方式肯定是直接声明一个函数：

```tsx
const Container = () => {
  return <p>这是一个函数！</p>;
};
```

对于组件的 props 类型，可以像在函数中标注参数类型一样：

```tsx
export interface IContainerProps {
  visible: boolean;
  controller: () => void;
}

const Container = (props: IContainerProps) => {
  return <p>这是一个函数！</p>;
};
```

属性默认值（defaultProps）也可以通过参数默认值的形式非常自然地进行声明：

```tsx
export interface IContainerProps {
  visible: boolean;
  controller: () => void;
}

const Container = ({
  visible = false,
  controller = () => {},
}: IContainerProps) => {
  return <p>这是一个函数！</p>;
};
```

这样做看起来很朴素，但 TS 仍然能把返回值类型推导出来，以上函数的类型可以被正确推导为 `() => JSX.Element`。

但这样只能说明它是一个函数，并不能从类型层面上标明它是一个 React 组件，也无法约束它必须返回一个合法的组件。可以加上显式的类型标注来确保它返回一个有效组件：

```tsx
const Container = (): JSX.Element => {
	return <p>这是一个段落！</p>;
};
```

JSX 是一个命名空间，来自于 `@types/react`。

除了这种方式，React 中还提供了 FC 这一类型来支持更精确的类型声明：

```tsx
import React from "react";

export interface IContainerProps {
  visible: boolean;
  controller: () => void;
};

const Container: React.FC<IContainerProps> = ({
  visible = false,
  controller = () => {},
}: IContainerProps) => {
  return <p>这是一个段落！</p>
};
```

FC 是 FunctionComponent 的缩写，而 FunctionComponent 同样被作为一个类型导出，其使用方式是一致的，接受的唯一泛型参数即为这个组件的属性类型。

来看看 FC 的声明是什么样的（实际上新版的 FunctionComponent 中已经没有 PropsWithChildren 了）：

```typescript
interface FunctionComponent<P = {}> {
	(props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
	propTypes?: WeakValidationMap<P> | undefined;
	contextTypes?: ValidationMap<any> | undefined;
	defaultProps?: Partial<P> | undefined;
	displayName?: string | undefined;
}
```

可以看到，代表着属性类型的泛型参数 P 实际上就是直接传给了类型别名 PropsWithChildren，而它其实就是为 Props 新增了一个 children 属性：

```typescript
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```

也就是说我们连这个组件的 children 都约束：

```tsx
const App = () => {
  return (
    <Container visible controller={() => {}}>
      <p>TypeScript + React!</p>
    </Container>
  );
};
```

**组件泛型**

使用简单函数和使用 FC 的重要差异之一就在于，使用 FC 时无法再使用组件泛型。组件泛型即指，为组件属性再次添加一个泛型，比如这样：

```tsx
import { PropsWithChildren } from "react";

interface ICellProps<TData> {
  field: keyof TData;
}

const Cell = <T extends Record<string, any>>(
  props: PropsWithChildren<ICellProps<T>>
) => {
  return <p></p>;
};

interface IDataStruct {
  name: string;
  age: number;
}

const App = () => {
  return (
    <>
      <Cell<IDataStruct> field='name'></Cell>
      <Cell<IDataStruct> field='age'></Cell>
    </>
  );
};
```

在 Cell 组件中，其 field 属性在接口结构中约束为 `keyof TData`，如在 App 中使用时，基于 IDataStruct 进行约束，此时 Cell 组件的 field 属性就**只能传入 name 与 age 两个值**。也就是说，我们可以为这个组件显式声明泛型参数，来获得填充属性时的类型提示与检查。

但很明显，使用 FC 并不能做到这一点。

### 泛型坑位

常见的泛型坑位主要还是来自于日常使用最多的 Hooks。

**useState**

首先是 useState，可以由输入值隐式推导或者显式传入泛型：

```tsx
const Container = () => {
  // 推导为 string 类型
  const [state1, setState1] = useState('test');
  // 此时类型为 string | undefined
  const [state2, setState2] = useState<string>();
};
```

需要注意的是在显式传入泛型时，如果像上面的例子一样没有提供初始值，那么 state2 的类型实际上会是 `string | undefined`，这是因为在 useState 声明中对是否提供初始值的两种情况做了区分重载：

```typescript
// 提供了默认值
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>];

// 没有提供默认值
function useState<S = undefined>(): [S | undefined, Dispatch<SetStateAction<S | undefined>>];
```

另一个常见的场景是对于在初始阶段是一个空对象的状态，可能会使用断言来这么做：

```typescript
const [data, setData] = useState<IData>({} as IData);
```

这么做的坏处在于，后续的调用方会认为这是一个完整实现了 IData 结构的对象，可能会出现遗漏的未赋值属性，此时也可以使用 Partial 类型标记它为可选：

```typescript
const [data, setData] = useState<Partial<IData>>({});
```

如果需要消费 useState 返回值的类型，可以搭配 ReturnType：

```typescript
// 相当于 useState(0) 的返回值类型
type State = ReturnType<typeof useState<number>>;
```

**useCallback 与 useMemo**

然后是 useCallback 与 useMemo，它们的泛型参数分别表示包裹的函数和计算产物，使用方式类似，也分为**隐式推导**与**显式提供**两种：

```typescript
const Container = () => {
  // 泛型推导为 (input: number) => boolean
  const handler1 = useCallback((input: number) => {
    return input > 123;
  }, []);

  // 显式提供为 (input: number, compare: boolean) => boolean
  const handler2 = useCallback<(input: number, compare: boolean) => boolean>(
    (input: number) => {
      return input > 123;
    },
    []
  );

  // 推导为 string
  const result1 = useMemo(() => {
    return 'some-expensive-process';
  }, []);

  // 显式提供
  const result2 = useMemo<{ name?: string }>(() => {
    return {};
  }, []);
};
```

通常情况下，不会主动为 useCallback 提供泛型参数，因为其传入的函数往往已经确定。而为 useMemo 提供泛型参数则要常见一些，因为可能希望通过这种方式来约束 useMemo 最后的返回值。

**useReducer**

useReducer 可以被视为更复杂一些的 useState，它们关注都是数据的变化。不同的是 useReducer 中只能由 reducer 按照特定的 action 来修改数据，但 useState 则可以随意修改。useReducer 有三个泛型坑位，分别为 reducer 函数的类型签名、数据的结构以及初始值的计算函数。

```tsx
const initialState = { count: 0 };

type Actions = 
  | {
    type: 'inc';
    payload: {
      count: number;
      max?: number;
    };
  }
  | {
    type: 'dec';
    payload: {
      count: number;
      min?: number;
    };
  };

function reducer(state: typeof initialState, action: Actions) {
  switch (action.type) {
    case 'inc':
      return {
        count: action.payload.max 
          ? Math.min(state.count + action.payload.count, action.payload.max) 
          : state.count + action.payload.count,
      };
    case 'dec': 
      return {
        count: action.payload.min 
          ? Math.max(state.count + action.payload.count, action.payload.min) 
          : state.count + action.payload.count,
      };
    default:
      throw new Error('Unexpected Action Received.');
  }
}

const Container = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      Count: {state.count}
      <button
        onClick={() => 
          dispatch({ type: 'dec', payload: { count: 599, min: 0 } })}
      >
        -(min: 0)
      </button>
      <button
        onClick={() => 
          dispatch({
            type: 'inc',
            payload: {
              count: 599,
              max: 599,
            },
          })}
      >
        +(max: 599)
      </button>
    </>
  );
};
```

在上面的例子中， useReducer 的泛型参数分别被填充为 reducer 函数的类型签名，以及其初始状态：

```typescript
type Reducer<S, A> = (prevState: S, action: A) => S;
type ReducerState<R extends Reducer<any, any>> = R extends Reducer<infer S, any> ? S : never;

function useReducer<R extends Reducer<any, any>>(
	reducer: R,
	initialState: ReducerState<R>,
): [ReducerState<R>, Dispatch<ReducerAction<R>>];
```

分析一下这里的填充：R 被填充为了一整个函数类型，而 `ReducerState<R>` 实际上就是提取了 reducer 中代表 state 的参数，即状态的类型，在这里即是 `{ count: number }` 这么一个结构。

需要注意的是，在 reducer 中其实也应用了此前提到过的**可辨识联合类型概念**，这里的 `action.type` 即为可辨识属性，通过 type 判断，就能在每一个 case 语句中获得联合类型对应分支的类型。

**useRef 与 useImperativeHandle**

useRef 的常见使用场景主要包括两种，存储一个 DOM 元素引用和持久化保存一个值。这两者情况对应的类型其实也是不同的：

```tsx
const Container = () => {
  const domRef = useRef<HTMLDivElement>(null);
  const valueRef = useRef<number>(599);

  const operateRef = () => {
    domRef.current?.getBoundingClientRect();
    valueRef.current += 1;
  };

  return (
    <div ref={domRef}>
      <p>ref 测试</p>
    </div>
  );
}
```

对于 domRef，此时其类型（current）会被推断为 `RefObject<HTMLDivElement>`，而 valueRef 的值类型则为 `MutableRefObject<number>`，这是完全符合预期的。因为并不会去修改挂载了 DOM 引用的 ref，而确实会修改值引用的 ref，所以后者会是 Mutable 的。

然而实际上，这一差异并不是通过判断是否被应用在了 DOM 引用来实现的，从 useRef 的类型定义可以看出，对于初始值为 null 的 useRef，其类型均会被推导为 RefObject：

```typescript
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

HTMLDivElement 这一类型来自于 TS 内置，在使用 ref 来引用 DOM 元素时，应当使用尽可能精确的元素类型，如 HTMLInputElement、HTMLIFrameElement 等，而不是 HTMLElement 这样宽泛的定义，因为这些精确元素定义的内部封装了更加具体的类型定义，包括 HTML 属性、事件入参等。

对于 useImperativeHandle，这个 hook 接受一个 ref、一个函数、一个依赖数组。这个函数的返回值会被挂载到 ref 上，常见的使用方式是用于是实现**父组件调用子组件方法**：子组件将自己的方法挂载到 ref 后，父组件就可以通过 ref 来调用此方法。

```tsx
import { 
  forwardRef, 
  useImperativeHandle, 
  useRef, 
  ForwardRefRenderFunction,
} from "react";

interface IRefPayload {
  controller: () => void;
}

const Parent = () => {
  const childRef = useRef<IRefPayload>(null);

  const invokeController = () => {
    childRef.current?.controller();
  };

  return (
    <>
      <Child ref={childRef}/>
      <button onClick={invokeController}>invoke controller!</button>
    </>
  );
};

interface IChildPropStruct {}

interface IExtendedRefPayload extends IRefPayload {
  disposer: () => void;
}

const Child = forwardRef<IRefPayload, IChildPropStruct>((props, ref) => {
  const internalController = () => {
    console.log('Internal Controller!');
  };

  useImperativeHandle<IRefPayload, IExtendedRefPayload>(
    ref,
    () => {
      return {
        controller: internalController,
        disposer: () => {},
      };
    },
    []
  );

  return <p></p>;
});
```

* IRefPayload 描述了将会在 ref 上挂载的对象结构。
* 在函数组件中，接受 ref 的函数组件（子组件）需要被 forwardRef 包裹才能正确接收到 ref 对象，其接受两个泛型参数，分别为 ref 的类型与此组件的属性类型。
* useImperativeHandle 中传入了 ref 以及一个返回两个方法的函数，它具有两个泛型参数，分别从传入的 ref 以及函数的返回值类型中进行类型推导。在这里显式传入了与推导不一致的第二个泛型参数，以此提供了额外的返回值类型检查。

### 内置类型定义

除了上面介绍的泛型坑位以外，在 React 中想要用好 TS 的另一个关键因素就是使用 `@types/react` 提供的类型定义，最常见的就是事件类型，比如输入框值变化时的 ChangeEvent 和 鼠标事件通用的 MouseEvent：

```tsx
import { useState } from "react";
import type { ChangeEvent, MouseEvent } from 'react';

const Container = () => {
  const [v, setV] = useState('test');

  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {};

  const handleClick = (event: MouseEvent<HTMLButtonElement>) => {};

  return (
    <>
      <input value={v} onChange={handleChange} />
      <button onClick={handleClick}>Click me!</button>
    </>
  );
};
```

需要注意的是，ChangeEvent 和 MouseEvent 上还具有一个泛型坑位，用于指定发生此事件的元素类型，可以在这里进一步传入 **HTMLButtonElement** 这样更精确的元素类型获得更严格的类型检查。

除了使用 ChangeEvent 作为参数类型，React 还提供了整个函数的类型签名，如 ChangeEventHandler：

```typescript
const handleChange: ChangeEventHandler<HTMLInputElement> = (e) => {};
```

由于上下文类型的存在，此时就无需再为 e 声明类型了，它会被自动推导为 `ChangeEvent<HTMLInputElement>` 。

类似的事件定义还有非常多，如 FormEvent、TouchEvent、DragEvent 等，但无需对所有定义都了解，在实际用到时再去导入即可。需要注意的是，由于 InputEvent 并非在所有浏览器都得到了支持，因此不存在对应的类型定义，可以使用 KeyboardEvent 来代替。

除了这些事件类型以外，还有一个常见的类型是在声明组件属性中的样式时会用到的，那就是 CSSProperties ，它描述了所有的 CSS 属性及对应的属性值类型，也可以直接用它来检查 CSS 样式时的值：

```tsx
import { CSSProperties } from "react";

export interface IContainerProps {
  style: CSSProperties;
}

const css: CSSProperties = {
  display: 'flex',
  alignContent: 'center',
  justifyContent: 'center',
};

const Container = ({ style }: IContainerProps) => {
  return <p style={style}>样式测试</p>
};
```

**其他内置类型**

这部分内置类型并不是日常开发中常用的，而是仅在组件库开发等场景时才会用到。

**ComponentProps**

当基于原生 HTML 元素去封装组件时，通常会需要将这个原生元素的所有 HTML 属性都保留下来作为组件的属性，此时肯定不能一个个声明所有属性，那么就可以使用 ComponentProps 来提取出一个元素上所有的属性：

```tsx
import { ComponentProps } from "react";

interface IButtonProps extends ComponentProps<'button'> {
  size?: 'small' | 'large';
  link?: boolean;
}

const Button = (props: IButtonProps) => {
  return <button {...props}>{props.children}</button>;
};
```

除了对原生 DOM 元素使用以外，这一用法在使用组件库时也有奇效，比如组件库只导出了这个组件而没有导出这个组件的属性类型定义，而又需要基于这个组件进行定制封装，此时就可以使用 ComponentProps 去提取它的属性类型：

```tsx
import { Button } from "ui-lib";
import type { ComponentProps } from 'react';

interface IButtonProps extends ComponentProps<Button> {
	display: boolean;
}

const EnhanceButton = (props: IButtonProps) => {
	return <Button {...props} >{props.children}</Button>;
};
```

由于 React 中 ref 的存在，有些时候会希望区分组件使用 ref 和没使用 ref 的情况，此时可以使用内置类型 ComponentPropsWithRef 或 ComponentPropsWithoutRef，其使用方式与 ComponentProps 一致。

ComponentProps 也可以用来提取一个 React 组件的属性，其内部实现对 HTML 元素和 React 组件这两种情况也做了区分：

```typescript
type ComponentProps<
	T extends keyof JSX.IntrinsicElements | JSXElementConstructor<any>
> = 
	// React 组件
	T extends JSXElementConstructor<infer P> 
		? P 
		: // HTML 元素
		T extends keyof JSX.IntrinsicElements 
			? JSX.IntrinsicElements[T] 
			: {};
```

**ReactElement 与 ReactNode**

在前面的例子中可能注意到了 ReactElement 与 ReactNode 这两个类型：

```typescript
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };

interface FunctionComponent<P = {}> {
	(props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
}
```

ReactElement 是 createElement、cloneElement 等 factory 方法的返回值类型，它本质上指的是一个有效的 JSX 元素，即 `JSX.Element`。而 ReactNode 可以认为包含了 ReactElement，它还包含 null、undefined 以及 ReactFragment 等一些特殊的部分，其类型定义如下：

```typescript
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```

### 其他工程实践

**项目中的类型声明文件**

在项目中使用一个专门的文件夹存放类型代码，其中又按照这些类型的作用进行了划分，其分布大致是这样的：

```text
PROJECT
├── src
│   ├── types
│   │   ├── shared.ts
│   │   ├── [biz].ts
│   │   ├── request.ts
│   │   ├── tool.ts
│   ├── typings.d.ts
└── tsconfig.json
```

依次来讲解这些类型声明文件的作用：

* `shared.ts`，被其他类型定义所使用的类型，如简单的联合类型封装、简单的结构工具类型等。

* `[biz].ts`，与业务逻辑对应的类型定义，比如 `user.ts` `module.ts` 等，推荐的方式是在中大型项目中尽可能按照业务模型来进行细粒度的拆分。

* `require.ts`，请求相关的类型定义，推荐的方式是定义响应结构体，然后使用 biz 中的业务逻辑类型定义进行填充：

  ```typescript
  import type { Status } from "./shared";
  
  export interface IRequestStruct<TData = never> {
  	status: Status;
  	code: number;
  	data: TData;
  }
  
  export interface IPaginationRequestStruct<TData = never> {
  	status: Status;
  	curPage: number;
  	totalCount: number;
  	hasNextPage: boolean;
  	data: TData[];
  }
  ```

  实际使用时：

  ```typescript
  import type { IPaginationRequestStruct } from "@/types/request";
  import type { IUserProfile } from "@/types/user";
  
  export function fetchUserList: Promise<IPaginationRequestStruct<IUserProfile>> {}
  ```

  通过这种方式，类型代定义之间就能够建立起清晰的、和业务逻辑一致的引用关系。

* `tool.ts`，工具类型定义，一般是推荐把比较通用的工具类型抽离到专门的工具类型库中，这里只存放使用场景特殊的部分。

* `typings.d.ts`，全局的类型声明，包括非代码文件的导入、无类型 npm 包的类型声明、全局变量的类型定义等等，也可以进一步拆分为 `env.d.ts` `runtime.d.ts` `module.d.ts` 等数个各司其职的声明文件。

另外并不需要将所有的类型定义都专门放到这个文件夹里，比如仅被组件自身消费的类型就应该使用就近原则，直接和组件代码一起即可。

**组件与组件类型**

在 React 父子组件中常见的一个场景是，父组件导入各个子组件，传递属性时会进行额外的数据处理，其结果的类型被这多个子组件共享，而这个类型又仅被父子组件消费，不应当放在全局的类型定义中。此时推荐的方式是，将这个类型定义在父组件中，子组件使用仅类型导入去导入这个类型，由于值空间与类型空间是隔离的，因此不需要担心循环引用：

```tsx
// Parent.tsx

import { ChildA } from "./ChildA";
import { ChildB } from "./ChildB";
import { ChildC } from "./ChildC";

// 被多个子组件消费的类型
export interface ISpecialDataStruct {}

const Parent = () => {
	const data: ISpecialDataStruct = {};
	
	return {
		<>
			<ChildA inputA={data} />
			<ChildB inputB={data} />
			<ChildC inputC={data} />
		</>
	}
}

// ChildA.tsx
import type { ISpecialDataStruct } from "./parent";

interface IAProp {
	inputA: ISpecialDataStruct;
}

export const ChildA: FC<IAProp> = (props) => {
	// ...
}
```

### 扩展：FC 并不是完美的

使用函数声明组件以及使用 FC 声明组件的两种形式的主要差异：

* 函数声明组件需要额外的返回值类型标注（`JSX.Element`）才能校验组件合法，并且可以再使用组件泛型来进一步确保类型安全。
* FC 可以简化函数的声明，但是无法使用组件泛型。

首先看 FC 的类型定义：

```typescript
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };

interface FunctionComponent<P = {}> {
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
}
```

会发现 FC 的属性中是默认包含了 children 这一属性的（对应到 Vue 中则是插槽 slot 的概念），但并不是所有时候组件中都会包含一个 children：

```typescript
const App = () => {
  return (
    <>
      <ContainerWithoutChildren />
      <ContainerWithChildren>linbudu</ContainerWithChildren>
    </>
  );
};
```

如果为 ContainerWithoutChildren 也传入了 children，虽然不会报错，但这个 children 实际上并不会渲染出来。

如果想让代码尽可能精确，应该区分这两种情况，即组件是否会接受 children 并消费。而在 FC 中并没有进行区分，因此 React 中又提供了 VFC，即 VoidFunctionComponent，它和 FC 的区别就在于属性中不包含 children：

```typescript
type VFC<P = {}> = VoidFunctionComponent<P>;

interface VoidFunctionComponent<P = {}> {
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

在 `@types/react` 18 版本后， FC 内部不再隐式包含 children 属性，因此 VFC 也就不再推荐使用。

在组件库中还有一个常见场景，即使用组件同时作为命名空间，如 `Table.Column`，`Form.Item` 这样，如果使用 FC，需要使用交叉类型补充上这些命名空间内的子组件：

```tsx
import * as React from 'react';

const Table: React.FC<{}> & {
  Column: React.FC<IColumnProps>;
} = () => {
  return <></>;
};

interface IColumnProps {}

const Column: React.FC<IColumnProps> = () => {
  return <></>;
};

Table.Column = Column;
```

但对于简单函数来说就不需要如此：

```tsx
const Table = (): JSX.Element => {
  return <></>;
};

interface IColumnProps {}

const Column = (props: IColumnProps): JSX.Element => {
  return <></>;
};

Table.Column = Column;
```

在这种情况下不需要通过额外的类型标注，因为就只是简单地把一个组件挂到这个组件的属性上。

最好不使用 FC，直接使用简单函数和返回值标注的方式。

## ESLint 约束 TS 代码：配置与规则集介绍

ESLint 的作用其实可以划分为两个部分：**风格统一**与**代码优化**。

### 基本的 ESLint 配置

最简单的方式就是通过 ESLint 自带的初始化功能，然后回答一系列问题即可。

```
npx eslint --init
npm init @eslint/config
```

如果选择了使用 TS，它会自动安装 `@typescript-eslint/` 一系列工具。

对于已有 ESLint 配置的项目，如果要配置 TypeScript ESLint ，安装以下依赖：

```
npm i @typescript-eslint/eslint-plugin @typescript-eslint/parser --save-dev
yarn add @typescript-eslint/eslint-plugin @typescript-eslint/parser --save-dev
pnpm i @typescript-eslint/eslint-plugin @typescript-eslint/parser --save-dev
```

然后更改 ESLint 配置：

```typescript
module.exports = {
	root: true,
	parser: '@typescript-eslint/parser',
	plugins: [
		'@typescript-eslint',
	],
	extends: [
		// ...其他已有的配置
		'plugin:@typescript-eslint/recommended',
	],
};
```

由于部分 TS ESLint 的规则和 ESLint 中基础的规则有冲突，需要修改配置文件的规则，最终的基本示例如下：

```js
module.exports = {
	root: true,
	extends: [
		'plugin:react/recommended',
		'plugin:@typescript-eslint/recommended',
		'prettier',
	],
	parser: '@typescript-eslint/parser',
	plugins: ['react', '@typescript-eslint'],
	rules: {
		'react/react-in-jsx-scope': 'off',
		indent: 'off',
		'@typescript-eslint/indent': ['error', 2],
		quotes: 'off',
		'@typescript-eslint/quotes': ['error', 'single'],
		semi: 'off',
		'@typescript-eslint/semi': ['error'],
	},
	settings: {
		react: {
			version: 'detect',
		},
	},
};
```

Indent 这条规则有一个需要注意的地方，其配置项还可以是 `'tab'` `space` 等，如果把上面的 2 改成 `'tab'`，大概率项目中会出现巨量报错。这是因为 Tab 和 Space 并不是等价的，也和编辑器的配置有关。

完成配置后，需要确定接受 ESLint 检查的项目文件，如可以只检查核心代码文件，包括 js/jsx，ts/tsx 文件。因此需要忽略掉部分文件，创建 `.eslintignore` 文件：

```ini
*.json
*.html
*rc.js
*.svg
*.css
```

在 package.json 中的 scripts 中添加以下命令：

```json
{
	"scripts": {
		"eslint": "eslint src/** --ext .js, .jsx, .ts, .tsx --cache",
		"eslint:fix": "npm run eslint -- --fix"
	},
}
```

`npm run eslint` 即是仅检查，而 `eslint:fix` 则是检查同时尽可能修复错误。这样就完成了基础的 TS ESLint 配置。但在实际项目中，光靠 ESLint 可没法确保代码质量。

### 配置 Prettier 与 Git Hooks

Prettier 同样是代码格式化工具，但和 ESLint 并不完全等价。除 JS/TS 代码文件以外，Prettier 也支持 CSS、Less 这样的样式文件，DSL 声明如 HTML、GraphQL 等等。一般习惯将除核心代码文件以外的部分，如 JSON、HTML、MarkDown 等全交给 Prettier 进行格式化。

对于 JS/TS 文件，Prettier 与 ESLint 的核心差异在于它并不包括 `no-xxx` （不允许某些语法），`prefer-xxx`（对于多种功能一致的语法，推荐使用其中某一种）这些**涉及具体代码逻辑**的规则，而是专注于 indent、quote、comma（逗号）、printWidth（每行允许的字符串长度）等规则。

首先安装 Prettier，如果想要让 Prettier 也参与格式化代码文件，还需要安装 eslint-config-prettier，这一配置包禁用了部分 ESLint 中会与 Prettier 产生冲突的规则。

```
npm install prettier eslint-config-prettier --save-dev
yarn add prettier eslint-config-prettier --save-dev
pnpm install prettier eslint-config-prettier --save-dev
```

创建 Prettier 配置文件 .prettierrc.js，选择一小部分常用的：

```js
module.exports = {
	// 单行最多 80 字符
	printWidth: 80,
	// 一个 Tab 缩进 2 个空格
	tabWidth: 2,
	// 每一行结尾需要有分号
	semi: true,
	// 使用单引号
	singleQuote: true,
	// 在对象属性中，仅在必要时才使用引号，如 "prop-foo"
	quoteProps: "as-needed",
	// 在 jsx 中使用双引号
	jsxSingleQuote: false,
	// 使用 es5 风格的尾缀风格逗号，即数组和对象的最后一项成员后也需要逗号
	trailingComma: "es5",
	// 大括号内首尾需要空格
	bracketSpacing: true,
	// HTML 标签（以及 JSX，Vue 模板等）的反尖括号 > 需要换行
	bracketSameLine: false,
	// 箭头函数仅有一个参数时也需要括号，如 (arg) => {}
	// 使用 crlf 作为换行符
	endOfLine: "crlf",
}
```

关于更多配置，参见 [Prettier 配置](https://link.juejin.cn/?target=https%3A%2F%2Fprettier.io%2Fdocs%2Fen%2Foptions.html)

同时为了避免和 ESLint 冲突，还需要通过 `eslint-config-prettier` 禁用掉部分 ESLint 规则，修改 ESLint 配置：

```js
module.exports = {
	extends: [
		'plugin:react/recommended',
		'plugin:@typescript-eslint/recommended',
		'prettier', // 新增这一行
	],
};
```

创建忽略文件 .prettierignore：

```init
build
dist
out
# 如果不希望 prettier 检查代码文件的话
# *.ts
# *.tsx
# *.jsx
# *.js
```

同时更新 NPM Scripts：

```json
{
	"scripts": {
		"eslint": "eslint src/** --no-error-on-unmatched-pattern --ext .js, .jsx, .ts, .tsx --cache",
		"eslint:fix": "npm run eslint -- --fix",
		"prettier": "prettier --check .",
		"prettier:fix": "prettier --write .",
		"lint": "npm run eslint && npm run prettier",
		"lint:fix": "npm run eslint:fix && npm run prettier:fix"
	},
}
```

类似的，`npm run prettier` 是仅检查，而 `prettier:fix` 才是进入修改。同时还增加 `lint` 和 `lint:fix` 来一次性执行两个工具。

即使配置了 ESLint 和 Prettier，还是可能出现提交代码不一样的情况。这是因为这些 scripts 需要手动执行，非常容易忘记。需要让开发每次提交代码时都自动执行一次格式化，从而确保提交的代码风格一致。

要实现这一能力，需要 Git Hooks 与 Lint Staged。

首先是 GIt Hooks，它和 React Hooks 不一样，更贴近生命周期的概念，即在某一个操作前后执行的额外逻辑。如要实现在 commit 前格式化，就可以使用 `pre-commit` 这个钩子，如果钩子执行失败，就不会真的执行 commit。常用的 Git Hooks 还有 `commit-msg`（可用于检查 commit 信息是否规范，如需要符合 `feat(core): enhancement` 这种格式）、`pre-push` 以及在服务端 Git 仓库执行的 `pre-receive`、`update`、`post-receive` 等。

直接写 GIt Hooks 不太优雅，可以通过 [Husky](https://link.juejin.cn/?target=https%3A%2F%2Ftypicode.github.io%2Fhusky%2F) 来实现相对简便的配置。关于各种初始化方式，可以阅读文档了解更多，此处只介绍自动安装的方式：

```
npx husky-init && npm install				# npm
npx husky-init && yarn							# yarn 1
yarn dlx husky-init --yarn2 && yarn	# tarn 2+
pnpm dlx husky-init && pnpm install	# pnmp
```

这样做只是安装了 Husky 以及配置了相关环境，实际上还没有添加 Git Hooks。Husky 也提供了快速创建的方式，直接把后面要执行的命令先添加进来：

```
npx husky add .husky/pre-commit './node_modules/.bin/lint-staged'
```

现在应该拥有了一个 .husky 文件夹，以及内部的 pre-commit 文件：

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

./node_modules/.bin/lint-staged
```

可以看到，实际上要执行的就是 lint-staged 这个命令，而 Lint Staged 的作用即是**找出添加到暂存区（git add）的文件，然后执行对应的 lint**，接着学习如何将它添加到项目里。

首先还是安装：

```
npm install --save-dev lint-staged
yarn add --save-dev lint-staged
pnpm install --save-dev lint-staged
```

然后在 package.json 中新增这段配置：

```json
{
	"lint-staged": {
		"*.{js,jsx,ts,tsx}": [
			"eslint --cache --fix",
			"prettier --write --list-different"
		],
		"*.{json,md,html,css,scss,sass,less,styl}": [
			"prettier --write --list-different"
		]
	},
}
```

这段配置的大意是，对于暂存区的核心代码文件，先使用 ESLint 格式化，再使用 Prettier 格式化，而对于其他文件，统一使用 Prettier 进行格式化。

加上 Prettier、Git Hooks 与 Lint Staged 后，项目约束才能说基本搞定了。虽然 Git Hooks 也可以通过 `git commit -m 'xx' --no-verify` 这种方式绕过去，但至少现在有办法让大家提交的代码都一致了。

### TS 下的 ESLint 规则集推荐

在这里介绍一批推荐使用的 TS ESLint 规则，包括其意图（如何约束代码）与配置等。这些规则分为两类：基础版于进阶版，基础版为约束程度较低的规则，而进阶版则较为严格。

如果想要使用现成的配置，也将下面介绍的规则发布到了 npm，首先安装配置集：

```
npm i eslint-config-ts-ruleset --save-dev
yarn add eslint-config-ts-ruleset --save-dev
pnpm i eslint-config-ts-ruleset --save-dev
```

然后在 ESLint 配置中启用：

```javascript
module.exports = {
	root: true,
	extends: [
		'plugin:@typescript-eslint/recommended',
		'prettier',
		// 基础规则
		'ts-ruleset',
		// 严格规则
		'ts-ruleset/strict',
	],
	parser: '@typescript-eslint/parser',
	parserOptions: {
		project: './tsconfig.json',
	},
	plugins: ['react', '@typescript-eslint'],
	rules: {
		'react/react-in-jsx-scope': 'off',
	},
};
```

TS ESLint 规则主要由四个部分组成：

* 仅在 ESLint 约束基础上支持了 TS 语法解析，如缩进 indent、单双引号 quote、逗号 comma 等
* 对语法的统一约束，比如类型断言有 as 和尖括号两种，可以通过规则来约束只能使用一种断言语法。以及对于某些对实际逻辑无影响，可加可不加的语法约束，如 for 循环和 `for...of` 的比较。
* 对类型标注的约束，如禁止某些类型被用于进行标注，以及在函数返回值处要求显式标注类型等
* 对能力的约束，如对于仅类型导入、类型声明的约束，以及非空断言、常量断言等功能的使用

基础部分的规则多是简单的语法检查和类型检查，因此不做讲解。

将严格规则组内的规则，进一步划分为**一般严格**与**较为严格**。

**一般严格组**

**语法统一约束**

**array-type**

TS 中同时支持使用 `Array<T>` 与 `T[]` 两种方式声明数组类型，此规则约束项目中对这两种数组类型声明的使用，包括仅使用 `Array<T>` 或 `T[]` 其中一种，或者对于原始类型与类型别名使用 `T[]`，对于联合类型、对象类型、函数类型等使用 `Array<T>`。

对于这种效果完全一致仅仅在使用上有差异的语法，需要的只是确定一个规范，然后在所有地方使用这一规范。

**await-thenable**

只允许对异步函数、Promise、PromiseLike 使用 await 调用（也就是实现了 Thenable 接口，存在 then 方法的对象），可以避免无意义的 await 调用，同时还能帮助发现某个异步函数忘记加 async 了。

**consistent-type-assertions**

TS 支持通过 `as` 与 `<>` 两种不同的语法进行类型断言，如：

```typescript
const foo = {} as Foo;
const foo = <Foo>{};

const foo = <const>[1, 2]; // 常量断言
const foo = [1, 2, 3] as const;
```

这一规则约束使用统一的类型断言语法，一般在 TSX 中使用 `as`，在其他时候尽可能的使用 `<>` ，原因则是 `<>` 更加简洁。

需要注意的是在 TSX 项目中使用 `<>` 断言会导致报错，因为此时无法像泛型那样通过 `<T extends Foo>` 来显式告知编译器这里是泛型语法而非一个 JSX 组件。

**consistent-type-definitions**

TS 支持通过 type 与 interface 声明对象类型，此规则可将其收束到统一的声明方式，即仅使用其中的一种。在绝大部分场景下，使用 interface 来声明对象类型，type 应当用于声明联合类型、函数类型、工具类型等，如：

```typescript
interface IFoo {}

type Partial<T> = {
	[P in keyof T]?: T[P];
};

type LiteralBool = "true" | "false";
```

原因主要有这么几点。

* 配合 naming-convention 规则（检查接口是否按照规范命名），要求接口的名称需要以大写字母 `I` 开头，能够在看见 `IFoo` 时立刻知道它是一个接口，看见 `Bar` 时立刻知道它是一个类型别名。额外配置如下：

  ```js
  {
  	"@typescript-eslint/naming-convention": [
  		"error",
  		{
  			"selector": "interface",
  			"format": ["PascalCase"],
  			"custom": {
  				"regex": "^I[A-Z]",
  				"match": false
  			}
  		}
  	]
  }
  ```

* 接口在类型编程中的作用非常局限，仅支持 extends、泛型等简单的能力，它也只应当被用于定义确定的结构体。而类型别名能够使用除 extends 以外所有的类型编程语法。同时，“类型别名”的含义也意味着，我们实际上是使用它来**归类类型**（联合类型）、**抽象类型**（函数类型、类类型）以及**封装工具类型**。

**prefer-for-of**

在使用 for 循环遍历数组时，如果索引仅仅用来访问数组成员，则应该替换为 `for...of`：

```typescript
// ×
for (let i = 0; i < arr.length; i++) {
	console.log(arr[i]);
}

// √
for (const x of arr) {
	console.log(x);
}
```

如果不是为了兼容性考虑，在仅访问索引的情况下，的确没有必要使用 for 循环，而应当使用语法更加简单的 `for...of`。

**prefer-nullish-coalescing 与 prefer-optional-chain**

 使用 `??` 而不是 `||`，使用 `a?.b` 而不是 `a && a.b`。这是因为，逻辑或 `||` 会将 0 与 `""` 视为 false 而导致错误地应用默认值，而 `??` 则只会在 null 和 undefined 时使用默认值。可选链 `?.` 相比于逻辑与 `&&` ，则能够带来更简洁的语法（尤其是在属性访问嵌套多层时，如 `res?.data?.status`），以及与 `??` 更好的协作：`const foo = a?.b?.c?.d ?? 'default';` 。

**consistent-type-imports**

约束使用 `import type {}` 进行类型的导入，如：

```typescript
// √
import type { CompilerOptions } from 'typescript';

// ×
import { CompilerOptions } from 'typescript';
```

`import type` 能够帮助更好地组织项目头部的导入结构。值导入与类型导入在 TS 中使用不同的空间来存放，因此无需担心循环依赖（所以可以**父组件导入子组件，子组件导入定义在父组件中的类型**这样）。

类似的，也可以通过 `consistent-type-exports` 规则，来约束只能使用 `export type` 导出类型：

```typescript
type Foo = string;

export type { Foo };
```

一个简单的、良好组织了导入语句的示例：

```typescript
import { useEffect } from 'react';

import { Button, Dialog } from 'ui';
import { ChildComp } from './child';

import { store } from '@/store';
import { useCookie } from '@/hooks/useCookie';
import { SOME_CONSTANTS } from '@/utils/constants';

import type { Foo } from '@/typings/foo';
import type { Shared } from '@/typings/shared';

import styles from './index.module.scss';
```

**non-nullable-type-assertion-style**

此规则要求在类型断言仅起到去空值作用，如将 `string | undefined` 类型断言为 `string` 时，将其替换为非空断言 `!`

```typescript
const foo:string | undefined = "foo";

// √
foo!;
// ×
foo as string;
```

非空断言的目的就是提供快速去除类型中 null、undefined 值的类型断言能力，这一规则的本质是检查**经过断言后的类型子集是否仅剔除了空值部分**。因此，无需担心对于多种有实际意义的类型分支的联合类型误判。

**promise-function-async**

返回 Promise 的函数必须被标记为 async。当函数显式返回了一个 Promise，就说明它应当是一个异步过程，就应当被标记为 async。

**prefer-literal-enum-member**

对于枚举成员值，只允许使用普通字符串、数字、null、正则，而不允许变量复制、模板字符串等需要计算的操作。虽然 TS 是允许使用各种合法表达式作为枚举成员的，但由于枚举的编译结果拥有自己的作用域，因此可能导致错误的赋值，如：

```typescript
const imOutside = 2;
const b = 2;
enum Foo {
	other = imOutside,
	a = 1,
	b = a,
	c = b,
}
```

这里 `c == Foo.b == Foo.c == 1` ，还是 `c == b == 2` ? 观察下编译结果：

```typescript
"use strict";
const imOutside = 2;
const b = 2;
var Foo;
(function (Foo) {
	Foo[Foo["outer"] = imOutside] = "outer";
	Foo[Foo["a"] = 1] = "a";
	Foo[Foo["b"] = 1] = "b";
	Foo[Foo["c"] = 1] = "c";
})(Foo || (Foo = {}));
```

编译结果也说明了，枚举对象内部也是有作用域的，优先级当然比外部作用域同名变量更高。

**prefer-as-const**

对于常量断言，使用 as const 而不是 `<const>`，这一点类似于上面约束类型断言语法的 consistent-type-assertions 规则。

**类型约束**

**no-unnecessary-type-arguments**

不允许与默认值一致的泛型参数，如：

```typescript
function foo<T = number>() {}
foo<number>();
```

与默认值一致的泛型参数是没有意义的，因为直接把鼠标悬浮上去就能看到实际应用的泛型类型。

**能力约束**

**ban-ts-comment**

禁止 `@ts-` 指令的使用，或者允许其在提供了说明的情况下被使用，如：

```
// @ts-expect-error 这里的类型太复杂，日后补上
// @ts-nocheck 未完成迁移的文件
```

此规则推荐与 `prefer-ts-expect-error` 一同搭配使用，让项目中不再出现 ts-ignore 指令。

**prefer-ts-expect-error**

使用 `@ts-expect-error` 而不是 `@ts-ignore` 。

`@ts-ignore` 与 `@ts-expect-error` 二者的区别主要在于，前者是 ignore，直接放弃了下一行的类型检查而无论下一行是否真的有错误，而后者则是期望下一行确实存在一个错误，并且会在实际不存在错误时反而抛出一个错误。

**no-extra-non-null-assertion**

不允许额外的重复非空断言：

```typescript
// ×
function foo(bar: number | undefined) {
	const bar: number = bar!!!;
}
```

额外的非空断言是无意义的，只要一次非空断言就能移除掉 null 与 undefined 类型了。

**no-non-null-asserted-nullish-coalescing**

不允许非空断言与空值合并同时使用，如 `bar! ?? tmp`。

非空断言可以用于移除掉一个类型中的 null 与 undefined，而空值合并则在左侧值为 null 或 undefined 时应用一个默认值，这么结合使用将非常有害，你的同事看到它会陷入思考：**bar 到底会不会有空值的时候？**

**no-non-null-asserted-optional-chain**

不允许非空断言与可选链同时使用，如 `foo?.bar!`。和上一条规则类似，属于非常有害的结合使用方式。同时也意味着对 `!` `??` `?.` 的理解存在着不当之处。

**no-unnecessary-type-assertion**

不允许与实际值一致的类型断言，如：`const foo = 'foo' as string`。类似于不允许默认值一致的泛型，这条规则的目的也是减少不必要的类型代码。

**no-unnecessary-type-constraint**

不允许与默认约束一致的泛型约束，如：`interface FooAny<T extends any> {}`。仍然是出于简化代码的考虑，在 TS 3.9 版本以后，对于未指定的泛型约束，默认使用 `unknown`，在这之前则是 `any`，这也就意味着没必要再多写 `extends unknown` 了。

**较为严格组**

**语法统一约束**

**method-signature-style**

方法签名的声明方式有 method 与 property 两种，区别如下：

```typescript
// method
interface T1 {
	func(arg: string): number;
}

// property
interface T2 {
	func: (arg: string) => number;
}
```

此规则将声明方式进行约束，推荐使用第二种的 property 方式。

method 方式类似于在 Class 中定义方法，而 property 则是就像定义普通的接口属性，只不过它的值是函数类型。推荐使用 property 的最重要原因是，通过使用 属性 + 属性值的方式定义，作为值的函数的类型能享受到更严格的类型校验（`strictFunctionTypes`），此配置会使用逆变而非协变的方式进行函数参数的检查。

**类型约束**

**ban-types**

禁止部分值被作为类型标注，此规则能够对每一种被禁用的类型提供特定的说明来触发此规则报错时给到良好的提示。

这条规则常见的场景是禁用 `{}`、`Function`、`object` 这些类型。

* 使用 `{}` 会导致寸步难行：`类型 {} 上不存在属性 'foo'`，所以用了 `{}` 大概率下面还需要类型断言回去或者变 any。同时别忘了 `{}` 作为万物起源的特性。
* `object` 的使用在大部分情况下都是错误的，它实际上表示所有非原始类型的类型。对于未知的对象类型，应使用 `Record<string, unknown>` 或者 `Record<string, any>`。
* 对于函数类型，应使用入参、返回值被标注出来的具体类型：`type SomeFunc = (arg1: string) => void`，或在入参与返回值类型未知的场景下使用 `type SomeFunc = (...args: any[]) => any`。

**no-empty-interface**

不允许定义空的接口，可配置为允许单继承下的空接口：

```typescript
// ×
interface Foo {}

// √
interface Foo extends Bar {}
```

没有父类型的空接口实际上就等于 `{}`，它显然是不对的。而单继承的空接口场景则更常见，比如可以先确定继承关系，再后续添加成员。

**no-explicit-any**

不允许显式的 any 类型。

由于很多场景下考虑到成本，还是会使用 any。因此实际上这条规则只被设置为 warn 等级。如果想要一个 any 都没有，推荐配合 tsconfig 的 `--noImplicitAny` 来检查隐式 any，来尽可能抹除代码中的 any。

**no-inferrable-types**

不允许不必要的类型标注，但可配置为允许类的属性成员、函数的属性成员进行额外标注。

```typescript
const foo: string = 'string';

class Foo {
	prop1: string = 'string';
}

function foo(a: number = 5, b: boolean = true) {
	// ...
}
```

对于普通变量来说，与实际赋值一致的类型标注确实是没有意义的，TS 的控制流分析能很好地进行类型自动推导。而对于函数参数与类属性，允许其进行类型标注则主要是为了确保一致性，即函数的所有参数（包括重载的各个声明）、类的所有属性都有类型标注，而不是仅为不存在初始值的参数/属性进行标注。

**explicit-module-boundary-types**

函数与类方法的返回值需要被显式指定，而不是依赖类型推导，如：

```typescript
const foo = (): Foo => {}
```

通过显式指定返回值类型，可以直观区分函数的功能，如是否携带副作用，以及是否有返回值等，同时显式指定的函数返回值类型也能在一定程度上提升 TS Compiler 性能。

**能力约束**

**no-unnecessary-boolean-literal-compare**

不允许对布尔类型变量的 === 比较，如：

```typescript
declare const someCondition: boolean;
if (someCondition === true) {
}
```

**no-throw-literal**

不允许直接 throw 一个字符串如：`throw 'err'`，只能抛出 Error 或基于 Error 派生类的示例，如：`throw new Error('Oops!')`。

抛出的 Error 示例能够自动的收集调用栈信息，同时借助 **[proposal-error-cause](https://link.juejin.cn/?target=http%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fgithub.com%2Ftc39%2Fproposal-error-cause)** 提案，还能跨越调用栈来附加错误原因传递上下文信息。

**switch-exhaustiveness-check**

switch 的判定变量为联合类型时，其每一个类型分支都需要被处理。如：

```typescript
type PossibleTypes = 'test1' | 'test2' | 'test3';

let value: PossibleTypes;
let result = 0;

switch (value) {
	case 'test1': {
		result = 1;
		break;
	}
	case 'test2': {
		result = 2;
		break;
	}
	case 'test3': {
		result = 3;
		break;
	}
}
```

联合类型变量中每一条类型分支可能都需要特殊的处理逻辑，此规则可以避免新增了一个类型分支却没有为它分配对应的处理逻辑。

在内置类型一节，还了解了通过 TS 中的 never 类型来实现编译时的检验：

```typescript
const strOrNumOrBool: string | number | boolean = false;

if (typeof strOrNumOrBool === 'string') {
	console.log('str!');
} else if (typeof strOrNumOrBool === 'number') {
	console.log('num!');
} else if (typeof strOrNumOrBool === 'boolean') {
	console.log('bool!');
} else {
	cosnt _exhaustiveCheck: never = strOrNumOrBool;
	throw new Error(`Unknown input type: ${_exhaustiveCheck}`);
}
```

**restrict-template-expressions**

模板字符串中的计算表达式其返回值必须是字符串，此规则可以被配置为允许数字、布尔值、可能为 null 的值以及正则表达式，或者也可以允许任意的值，但这样这条规则就形同虚设了。

在模板表达式中，非字符串与数字以外的值很容易带来潜在的问题，如：

```typescript
const arr = [1, 2, 3];
cosnt obj = { name: 'test' };

// 'arr: 1,2,3'
const str1 = `arr: ${arr}`;
// 'obj: [object Object]'
const str2 = `obj: ${obj}`;
```

无论哪种情况都不是想看到的。推荐在规则配置中仅开启 `allowNumber` 来允许数字，而禁止掉其他的类型，需要确保在其他类型变量填入模板插槽中时，进行一次具有实际逻辑的转化。

**prefer-reduce-type-parameter**

在泛型一节中曾经介绍过数组 reduce 方法的各种重载，比如下面这种：

```typescript
const arr: number[] = [1, 2, 3];

// 报错：不能将 number 类型的值赋值给 never 类型
const result = arr.reduce((prev, curr, idx, arr) => {
	return [...prev, curr]
}, []);
```

虽然知道 result 最终的结果会仍然是 `number[]` 类型，但这里仍然会有类型报错，原因就在于这里 reduce 通过 `[]` 进行类型推导，得到了一个 `never[]` 类型，要解决也很简单，只要给 reduce 显式传入泛型参数即可：

```typescript
const result = arr.reduce<number[]>((prev, curr, idx, arr) => {
	return [...prev, curr]
}, []);
```

这一规则就是为了确保在所有情况下都显式为 reduce 调用传入了泛型，这样就能自己确定它最终的返回值类型，而不是依赖类型推导了。

**triple-slash-reference**

这一规则的目的在于禁止使用三斜线指令，而是使用 import 来进行类型的导入。正如上一节所了解的那样：

```typescript
// × 不推荐使用
/// <reference types="vite/client" />

// √ 使用 import type 来声明对这个声明文件的依赖
import type * as ViteClientEnv from 'vite/client';
```

## 全链路 TS 工具库

### 开发阶段

这一部分的工具主要在项目开发阶段使用。

**项目开发**

* [ts-node](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FTypeStrong%2Fts-node) 与 [ts-node-dev](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwclr%2Fts-node-dev)：用于直接执行 .ts 文件。其中 ts-node-dev 基于 ts-node 和 node-dev（类似于 nodemon）封装，能够实现监听文件改动并重新执行文件的能力。

* [tsc-watch](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgilamran%2Ftsc-watch)：它类似于 ts-node-dev，主要功能也是监听文件变化然后重新执行，但 tsc-watch 的编译过程更明显，也需要自己执行编译后的文件。可以通过 onSuccess 与 onFailure 参数，来在编译过程成功与失效时执行不同的逻辑。

  ```
  ## 启动 tsc --watch，然后在成功时执行编译产物
  tsc-watch --onSuccess "node ./dist/server.js"
  
  ## 在失败时执行
  tsc-watch --onFailure "echo 'Beep! Compilation Failed'"
  ```

* [esno](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fesbuild-kit%2Fesno)，antfu 的作品。核心能力同样是执行 .ts 文件，但底层是 ESBuild 而非 tsc，因此速度上会明显更快。

* [typed-install](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Ftyped-install)，功能就是在安装包时自动去判断这个包是否有额外的类型定义包，并为你自动地进行安装。

* [suppress-ts-error](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fkawamataryo%2Fsuppress-ts-errors)，自动为项目中所有的类型报错添加 `@ts-expect-error` 或 `@ts-ignore` 注释，重构项目时很有帮助。

* [ts-error-translator](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmattpocock%2Fts-error-translator)，将 TS 报错翻译成更接地气的版本，并且会根据代码所在的上下文来详细说明报错原因，目前只有英文版本。

**代码生成**

* [typescript-json-schema](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FYousefED%2Ftypescript-json-schema)，从 TypeScript 代码生成 JSON Schema，如以下代码：

  ```typescript
  export interface Shape {
  	/**
  		* The size of the shape.
  		* 
  		* @minimum 0
  		* @TJS-type integer
  		*/
  	size: number;
  }
  ```

  会生成以下的 JSON Schema：

  ```json
  {
    "$ref": "#/definitions/Shape",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "definitions": {
      "Shape": {
        "properties": {
          "size": {
            "description": "The size of the shape.",
            "minimum": 0,
            "type": "integer"
          }
        },
        "type": "object"
      }
    }
  }
  ```

* [json-schema-to-typescript](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fbcherny%2Fjson-schema-to-typescript)，和上面那位反过来，从 JSON Schema 生成 TypeScript 代码：

  ```json
  {
    "title": "Example Schema",
    "type": "object",
    "properties": {
      "firstName": {
        "type": "string"
      },
      "lastName": {
        "type": "string"
      },
      "age": {
        "description": "Age in years",
        "type": "integer",
        "minimum": 0
      },
      "hairColor": {
        "enum": ["black", "brown", "blue"],
        "type": "string"
      }
    },
    "additionalProperties": false,
    "required": ["firstName", "lastName"]
  }
  ```

  ```typescript
  export interface ExampleSchema {
    firstName: string;
    lastName: string;
    /**
     * Age in years
     */
    age?: number;
    hairColor?: "black" | "brown" | "blue";
  }
  ```

需要注意的是， JSON Schema 并不是我们常见到的描述实际值的 JSON，它更像是 TS 类型那样的**结构定义**，存在着值类型、可选值、访问信息等相关信息的描述，因此他才能够与 TS 之间进行转换。

### 类型相关

以下工具库主要针对类型，包括提供通用工具类型与对工具类型进行测试。

* [type-fest](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Ftype-fest)
* [utility-types](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpiotrwitek%2Futility-types)，包含的类型较少
* [ts-essentials](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fts-essentials)
* [type-zoo](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpelotom%2Ftype-zoo)
* [ts-toolbelt](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmillsp%2Fts-toolbelt)，目前包含工具类型数量最多的一位
* [tsd](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Ftsd)，用于进行类型层面的单元测试，即验证工具类型计算结果是否是符合预期的类型
* [conditional-type-checks](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdsherret%2Fconditional-type-checks)，类似于 tsd，也是用于对类型进行单元测试

### 校验阶段

以下这些工具通常用于在项目逻辑中进行具有实际逻辑的校验（而不同于 tsd 仅在类型层面）。

**逻辑校验**

* [zod](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fcolinhacks%2Fzod)，核心优势在于与 TypeScript 的集成，如能从 Schema 中直接提取出类型：

  ```typescript
  import { z } from "zod";
  
  const User = z.object({
  	username: z.string(),
  });
  
  User.parse({ username: "Test" });
  
  // extract the inferred type
  type User = z.infer<typeof User>;
  // { username: string }
  ```

* [class-validator](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftypestack%2Fclass-validator)，TypeStack 的作品，基于装饰器来进行校验。

  ```typescript
  export class Post {
  	@Length(10, 20)
  	title: string;
  	
  	@Contains('hello')
  	text: string;
  	
  	@IsInt()
  	@Min(0)
  	@Max(10)
  	rating: number;
  	
  	@IsEmail()
  	email: string;
  }
  
  let post = new Post();
  post.title = 'Hello'; // 错误
  post.text = 'this is a great post about hell world'; // 错误
  post.rating = 11; // 错误
  post.email = 'google.com'; // 错误
  
  validate(post).then(errors => {
  	// 查看是否返回了错误
  	if (errors.length > 0) {
  		console.log('校验失败，错误信息：', errors);
  	} else {
  		console.log('校验通过！');
  	}
  });
  ```

* [superstruct](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fianstormtaylor%2Fsuperstruct)，功能与使用方式类似于 zod，更老牌一些。

* [ow](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fow)，用于函数参数的校验，通常在 CLI 工具里大量使用。

  ```typescript
  import ow from 'ow';
  
  const unicorn = input => {
  	ow(input, ow.string.minLength(5));
  	
  	// ...
  };
  
  unicorn(3);
  //=> ArgumentError: Expected `input` to be of type `string` but received type `number`
  
  unicorn('yo');
  //=> ArgumentError: Expected string `input` to have a minimum length of `5`, got `yo`
  ```

* [runtypes](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpelotom%2Fruntypes)，类似于 Zod，也是运行时的类型与 Schema 校验。

**类型覆盖检查**

* [typescript-coverage-report](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Falexcanessa%2Ftypescript-coverage-report)，检查你的项目中类型的覆盖率。
* [type-coverage](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fplantain-00%2Ftype-coverage)，前者的底层依赖，可以用来定制更复杂的场景。

### 构建阶段

以下工具主要在构建阶段起作用。

* [ESBuild](https://link.juejin.cn/?target=https%3A%2F%2Fesbuild.github.io%2F)，需要注意的是 ESBuild 和 TypeScript Compiler 还是存在一些构建层面的差异，比如 ESBuild 无法编译装饰器（但可以使用插件，对含有装饰器的文件回退到 tsc 编译）。
* [swc](https://link.juejin.cn/?target=https%3A%2F%2Fswc.rs%2F)，SWC 的目的是替代 Babel，因此它是可以直接支持装饰器等特性的。
* [fork-ts-checker-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Ffork-ts-checker-webpack-plugin)，Webpack 插件，使用额外的子进程来进行 TypeScript 的类型检查（需要禁用掉 ts-loader 自带的类型检查）。
* [esbuild-loader](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fprivatenumber%2Fesbuild-loader)，基于 ESBuild 的 Webpack Loader，放在这里是因为它基本可以完全替代 ts-loader 来编译 ts 文件。
* [rollup-plugin-dts](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Frollup-plugin-dts)，能够将你项目内定义与编译生成的类型声明文件重新进行打包。
* [Parcel](https://link.juejin.cn/?target=https%3A%2F%2Fparceljs.org%2F)，一个 Bundler，与 Webpack、Rollup 的核心差异是零配置，不需要任何 loader 或者 plugin 配置就能对常见基本所有的样式方案、语言方案、框架方案进行打包。

## 说说 TS 和 ECMAScript 之间那些事

### ECMAScript 与 TC39

ECMA 的全称是 [European Computer Manufacturers Association](https://link.juejin.cn/?target=https%3A%2F%2Fwww.ecma-international.org%2F)，即**欧洲计算机制造商协会**。这一国际组织的存在主要是维护各种计算机的相关标准，从硬件到软件到编程语言等等。

**而 ECMAScript 并不是一门语言，而是一门规范。**

ECMA 维护着数百条规范，这些规范的领域差异非常大。因此 ECMA 采用技术小组（**Technical Committee，也称为技术委员会**）的方式来管理这个规范，ECMAScript 对应的技术小组即是 TC39。

在 ECMAScript 中，一个提案被纳入规范要经历 5 个阶段。

* stage 0（**strawman**）：任何 TC39 的成员都可以提交。
* stage 1（**proposal**）：进入此阶段就意味着这一提案被认为是**正式**的了，需要对此提案的场景与API进行详尽描述。要想进入 Stage 1，需要一位 TC39 成员作为负责人（champion）对这个提案具体的**语法、语义和现有语法的冲突风险**都进行详细讨论分析。
* stage 2（**draft**）：要进入 Stage 2，需要完成**包含提案所有内容的标准文本的初稿**。
* stage 3（**candidate**）：这一阶段的提案只有在遇到了重大问题才会修改，需要撰写非常完善的规范文档。
* stage 4（**finished**）：这一阶段的提案将会被纳入到 ES 每年发布的规范之中，正式与大家见面。

将 ECMAScript 未纳入标准的语法进行降级（或将标准语法按照运行时环境降级）是它的核心功能之一。

### TypeScript 中的 ECMAScript 语法

目前在 TS 中，已经合入的 ECMAScript 语法主要有这么几个：

* 可选链 Optional Chainning，即 `?.` 语法。
* 空值合并 Nullish Coalescing，即 `??` 语法。
* 逻辑赋值 Logical Assignment，即 `&&=`, `??=` 这一类语法。
* 装饰器。
* 一些新增的方法，如 replaceAll 等。
* Class 相关，如基于 `#` 的私有成员标注等。

对于可选链、空值合并以及逻辑赋值，前两者在 TS 3.7 版本引入，逻辑赋值则在 4.0 版本被引入，它们早在 ECMAScript 2021 就被正式吸收。

**可选链 Optional Chainning**

在 JS 中，如果访问一个嵌套多层的属性，为了避免出现 `Cannot read property of undefined` 这样的错误，通常会使用**逻辑与** `&&` 语法来确保在某一层出现空值时及时短路掉访问：

```javascript
const inner = obj && obj.data && obj.data.innerProperty;
```

这种写法虽然丑陋，但确实能在一定程度上避免对空值的读取，但也只是一定程度上。

上面的代码有两个问题，当在某一步访问出现空值时，它返回的是上一步的值，而在属性读取过程中，通常希望的是如果某一环节短路了，返回一个 undefined。否则，如果下面还存在对 `inner` 进行真假值判断（`if(inner)`）的话，反而容易引发 Bug。

此外，逻辑与短路在属性嵌套过深时简直就是噩梦，使用可选链的 `?.` 语法，可以改写成这样：

```javascript
const inner = obj?.data?.innerProperty;
```

除了更简洁的写法以外，可选链也更加符合我们的预期：**它会在短路时返回一个 undefined **。可选链不仅能应用在属性访问，也可以用在计算属性访问以及方法调用上：

```typescript
obj?.[expr];
obj?.[++a];
// 对应到 obj.func && obj.func()
obj?.func();
```

在所有情况下，如果 `?.` 的左侧发生了短路，那么就会直接停止后续操作，比如不会去运行并计算表达式 `expr` 以及 `++a` 。

**空值合并 Nullish Coalescing**

 如果说可选链是为了取代**逻辑与**（`&&`），那么空值合并就是为了取代**逻辑或**（`||`）。而逻辑或的主要使用场景之一就是提供默认值，如：

```javascript
const foo = someValue || fallbackValue;
```

逻辑或会在 `||` 左边被判断为 false 时，执行右边的逻辑，在这里即是赋值行为。但由于 JavaScript 中无处不在的隐式转换，如果 `||` 左边是 `""`/ `0` / `false`，都会被视为 false（false 虽然是 false，但它也是个值！），而我们希望的是**仅在左边为 undefined 或 null 时，才去应用默认值**。

大部分情况下我们可以直接使用 `??` 代替 `||`：

```javascript
const foo = someValue ?? fallbackValue;
```

配合可选链：

```javascript
const bar = obj?.a?.b?.c() ?? fallbackValue;
```

空值合并就如它的名字一样，只会对真正意义上的空值（null 与 undefined）进行处理。

**逻辑赋值 Logical Assignment**

实际上，逻辑赋值是在复合赋值的基础上演进而来（或者说关系一致）的，都是将一个操作符和赋值符号结合在一起。比如我们最常见的复合赋值：

```javascript
a = a + b;
a += b;

a = a - b;
a -= b;

a = a * b;
a *= b;

// 还有除法运算，就不演示了
```

复合赋值其实就是先执行操作，再将操作结果赋值给左边的变量。如 `a += b` 就是执行 `a + b`，然后将结果赋值给 `a`。

而逻辑赋值也是一样：

```javascript
a = a || b;
a ||= b;

a = a && b;
a &&= b;
```

类比一下，逻辑赋值就是**先执行逻辑操作，然后将结果赋值给左边的变量**。这一语法其实在实际开发中有奇效，如 `a ||= b` 其实可以替代掉以下这段代码：

```javascript
if(!a) a = b;

// 或者
a = a ? a : b;
```

既然逻辑操作符可以，那么空值合并也是可以的：

```javascript
a = a ?? b;
a ??= b;
```

为了区分逻辑赋值，可以称其为**短路赋值**。短路赋值在一些需要懒初始化的场景中非常好用，比如：

```typescript
let arr: string[];

(arr ??= []).push("string");

// 等价于以下这段
arr = arr ?? []; // 假设 arr 有可能在多处被初始化
arr.push("string");
```

## 装饰器与反射元数据

**装饰器的本质其实就是一个函数**，只不过它的入参是提前确定好的。同时，TypeScript 中的装饰器目前**只能在类以及类成员上使用**。

装饰器通过 `@` 语法来使用：

```typescript
function Deco() { }

@Deco
class Foo {}
```

这样的装饰器只能起到固定的功能，实际上使用更多的是 Decorator Factory，即装饰器工厂的形式：

```typescript
function Deco() { 
  return () => {}
}

@Deco()
class Foo {}
```

在这种情况下，程序执行时会先执行 `Deco()` ，再用内部返回的函数作为装饰器的实际逻辑。

### 装饰器大起底

TypeScript 中的装饰器可以分为**类装饰器**、**方法装饰器**、**访问符装饰器**、**属性装饰器**以及**参数装饰器**五种，最常见的主要还是类装饰器、方法装饰器以及属性装饰器。

#### 类装饰器

类装饰器是直接作用在类上的装饰器，它在执行时的入参只有一个，那就是这个类本身（而不是类的原型对象）。因此可以通过类装饰器来覆盖类的属性与方法，如果你在类装饰器中返回一个新的类，它甚至可以篡改掉整个类的实现。

```typescript
@AddProperty('linbudu')
@AddMethod()
class Foo {
  a = 1;
}

function AddMethod(): ClassDecorator {
  return (target: any) => {
    target.prototype.newInstanceMethod = () => {
      console.log("Let's add a new instance method!");
    };
    target.newStaticMethod = () => {
      console.log("Let's add a new static method!");
    };
  };
}

function AddProperty(value: string): ClassDecorator {
  return (target: any) => {
    target.prototype.newInstanceProperty = value;
    target.newStaticProperty = `static ${value}`;
  };
}
```

这里通过 TypeScript 内置的 ClassDecorator 类型定义来进行类型标注，由于类装饰器只有一个参数，我们也不想使用过多的类型代码，这里我就直接 any 了。我们的函数返回了一个 ClassDecorator ，因此这个装饰器就是一个 Decorator Factory，在实际执行时需要以 `@Deco()` 的形式调用。

在 AddMethod 与 AddProperty 方法中，分别在 target、`target.prototype` 上添加了方法与属性。在这里 target 上的属性实际上是**静态成员**，也就是其实例上不会获得的方法，而 `target.prototype` 上的属性才是会随着继承与实例化过程被传递的**实例成员**。

来调用一下看看：

```typescript
const foo: any = new Foo();

foo.newInstanceMethod();
(<any>Foo).newStaticMethod();

console.log(foo.newInstanceProperty);
console.log((<any>Foo).newStaticProperty);
```

```text
Let's add a new instance method!
Let's add a new static method!
linbudu
static linbudu
```

我们在这里调用的方法并没有直接在 Foo 中定义，而是通过装饰器来强行添加！我们也可以在装饰中返回一个子类：

```typescript
const OverrideBar = (target: any) => {
  return class extends target {
    print() {}
    overridedPrint() {
      console.log('This is Overrided Bar!');
    }
  };
};

@OverrideBar
class Bar {
  print() {
    console.log('This is Bar!');
  }
}

// 被覆盖了，现在是一个空方法
new Bar().print();

// This is Overrided Bar!
(<any>new Bar()).overridedPrint();
```

在 React Class 组件时代，有许多功能也是通过装饰器实现的。如 Mobx 的 `@observer` 与 `@observable`，React-Redux 的 `@connect` 等。

#### 方法装饰器

方法装饰器的入参包括**类的原型**、**方法名**以及**方法的属性描述符**（PropertyDescriptor），而通过属性描述符你可以控制这个方法的内部实现（即 value）、可变性（即 writable）等信息。

能拿到原本实现，也就意味着可以在执行原本方法的同时，插入一段新的逻辑，比如计算这个方法的执行耗时：

```typescript
class Foo {
  @ComputeProfiler()
  async fetch() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('RES');
      }, 3000);
    });
  }
}

function ComputeProfiler(): MethodDecorator {
  const start = new Date();
  return (
    _target,
    methodIdentifier,
    descriptor: TypedPropertyDescriptor<any>
  ) => {
    const originalMethodImpl = descriptor.value!;
    descriptor.value = async function (...args: unknown[]) {
      const res = await originalMethodImpl.apply(this, args); // 执行原本的逻辑
      const end = new Date();
      console.log(
        `${String(methodIdentifier)} Time: `,
        end.getTime() - start.getTime()
      );
      return res;
    };
  };
}

(async () => {
  console.log(await new Foo().fetch());
})();
```

```text
fetch Time:  3003
RES
```

需要注意的是，方法装饰器的 target 是**类的原型而非类本身**。

#### 访问符装饰器

访问符装饰器并不常见，甚至访问符对于部分同学来说也是陌生的，但它其实就是 `get value(){}` 与 `set value(v)=>{}` 这样的方法，其中 getter 在你访问这个属性 `value` 时触发，而 setter 在你对 `value` 进行赋值时触发。访问符装饰器本质上仍然是方法装饰器，它们使用的类型定义也相同。

需要注意的是，访问符装饰器只能同时应用在一对 getter / setter 的其中一个，即要么装饰 getter 要么装饰 setter 。这是因为，不论你是装饰哪一个，装饰器入参中的属性描述符都会包括 getter 与setter 方法：

```typescript
class Foo {
  _value!: string;

  get value() {
    return this._value;
  }

  @HijackSetter('LIN_BU_DU')
  set value(input: string) {
    this._value = input;
  }
}

function HijackSetter(val: string): MethodDecorator {
  return (target, methodIdentifier, descriptor: any) => {
    const originalSetter = descriptor.set;
    descriptor.set = function (newValue: string) {
      const composed = `Raw: ${newValue}, Actual: ${val}-${newValue}`
      originalSetter.call(this, composed);
      console.log(`HijackSetter: ${composed}`);
    };
    // 篡改 getter，使得这个值无视 setter 的更新，返回一个固定的值
    // descriptor.get = function () {
    //   return val;
    // };
  };
}

const foo = new Foo();
foo.value = 'LINBUDU'; // HijackSetter: Raw: LINBUDU, Actual: LIN_BU_DU-LINBUDU
```

在这个例子中，我们通过装饰器劫持了 setter ，在执行原本的 setter 方法修改了其参数。同时，我们也可以在这里去劫持 getter（`descriptor.get`），这样一来在读取这个值时，会直接返回一个我们固定好的值，而非其实际的值（如被 setter 更新过的）。

#### 属性装饰器

属性装饰器在独立使用时能力非常有限，它的入参只有**类的原型**与**属性名称**，返回值会被忽略，但你仍然可以通过**直接在类的原型上赋值**来修改属性：

```typescript
class Foo {
  @ModifyNickName()
  nickName!: string;
  constructor() {}
}

function ModifyNickName(): PropertyDecorator {
  return (target: any, propertyIdentifier) => {
    target[propertyIdentifier] = '林不渡!';
    target['otherName'] = '别名林不渡!';
  };
}

console.log(new Foo().nickName);
// @ts-expect-error
console.log(new Foo().otherName);
```

```text
林不渡!
别名林不渡!
```

我们在原型对象上强行写入了属性，但这种方法实际上过于 hack，在后面会了解如何通过委托的方式来为一个属性注入值。

#### 参数装饰器

参数装饰器包括了构造函数的参数装饰器与方法的参数装饰器，它的入参包括**类的原型**、**参数名**与**参数在函数参数中的索引值（即第几个参数）**，如果只是单独使用，它的作用同样非常有限。

```typescript
class Foo {
  handler(@CheckParam() input: string) {
    console.log(input);
  }
}

function CheckParam(): ParameterDecorator {
  return (target, paramIdentifier, index) => {
    console.log(target, paramIdentifier, index);
  };
}

// {} handler 0
new Foo().handler('linbudu');
```

#### 装饰器的执行机制

装饰器的执行机制中主要包括**执行时机**、**执行原理**以及**执行顺序**这三个概念。

首先是执行时机，装饰器的本质就是一个函数，因此只要在类上定义了它，即使不去实例化这个类或者读取静态成员，它也会正常执行。很多时候，其实也并不会实例化具有装饰器的类，而是通过反射元数据的能力来消费。而装饰器的执行原理，我们可以通过编译后的代码来了解：

```typescript
@Cls()
class Foo {
  constructor(@Param() init?: string) { }

  @Prop()
  prop!: string

  @Method()
  handler(@Param() input: string) {

  }
}
```

这一段代码编译的产物会是这样的（经过简化）：

```javascript
"use strict";
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
   // ...
};
var __param = (this && this.__param) || function (paramIndex, decorator) {
    return function (target, key) { decorator(target, key, paramIndex); }
};

let Foo = class Foo {
    constructor(init) { }
    handler(input) {
    }
};
__decorate([
    Prop(),
], Foo.prototype, "prop", void 0);
__decorate([
    Method(),
    __param(0, Param()),
], Foo.prototype, "handler", null);
Foo = __decorate([
    Cls(),
    __param(0, Param()),
], Foo);
```

这里的 `__decorate` 方法，其实就是通过实际入参来判断当前到底执行的是哪种装饰器，然后执行对应的装饰逻辑。而观察这个方法调用的入参，会再次观察到这些装饰器的不同入参：**方法与属性装饰器是类的原型对象**，而**类装饰器才能获得这个类本身作为入参**。而属性装饰器应用时，这个属性还未被初始化（属性需要实例化才会有值），这也是为什么它无法像方法装饰器那样获取到值。

可以看到，上面的装饰器顺序依次是**实例上的属性、方法、方法参数**，然后是**静态的属性、方法、方法参数**，最后是**类以及类构造函数参数**。

而从这一编译结果中，我们还能观察到不同类型装饰器的**执行顺序**。首先是实例上的属性、方法、方法参数，然后是静态的属性、方法、方法参数，最后是类以及类构造函数参数。而装饰器的**应用顺序**则略有不同，**方法参数装饰器会先于方法装饰器应用**（`__param(0, Param())`）。

关于执行顺序与应用顺序，执行是**装饰器求值得到最终装饰器表达式**的过程，而应用则是**最终装饰器逻辑代码执行**的过程：

```typescript
function deco() {
  // 执行
  return () => {
    // 应用
  }
}
```

实际上，对于实例与静态的属性、方法装饰器而言，它们的执行与应用顺序其实**取决于它们定义的位置**，你可以在上面的例子里把方法定义在属性之前，就会发现执行顺序变成了**方法**-**方法参数**-**属性**，即先定义先执行。

在 TypeScript 官方文档中对应用顺序给出了详细的定义：

1. *参数装饰器*，然后依次是*方法装饰器*，*访问符装饰器*，或*属性装饰器*应用到每个实例成员。
2. *参数装饰器*，然后依次是*方法装饰器*，*访问符装饰器*，或*属性装饰器*应用到每个静态成员。
3. *参数装饰器*应用到构造函数。
4. *类装饰器*应用到类。

最后，再看一个例子，来更深刻地了解执行顺序与应用顺序：

```typescript
function Deco(identifier: string): any {
  console.log(`${identifier} 执行`);
  return function () {
    console.log(`${identifier} 应用`);
  };
}

@Deco('类装饰器')
class Foo {
  constructor(@Deco('构造函数参数装饰器') name: string) {}

  @Deco('实例属性装饰器')
  prop?: number;

  @Deco('实例方法装饰器')
  handler(@Deco('实例方法参数装饰器') args: any) {}
}
```

以上的代码输出是这样的：

```text
实例属性装饰器 执行
实例属性装饰器 应用
实例方法装饰器 执行
实例方法参数装饰器 执行
实例方法参数装饰器 应用
实例方法装饰器 应用
类装饰器 执行
构造函数参数装饰器 执行
构造函数参数装饰器 应用
类装饰器 应用
```

执行顺序就不再赘述，这里我们主要关注应用顺序。顺序大致是**实例属性-实例方法参数-构造函数参数-类**，好像不对，不是说参数装饰器先应用吗？这是因为在这个例子中，我们是先定义属性和属性装饰器的，因此属性装饰器会先应用。如果方法在前，可不就是方法参数装饰器先应用？

类装饰器是最后应用的。也就是说，如果我们在方法装饰器中标记某些信息，最终的类装饰器是可以消费到，并且基于此信息对类或类的实例进行某些操作的。如标记为 `@Deprecated` 的方法，我们在最终的类装饰器中可以将这些方法实现替换为一个报错！

#### 多个同类装饰器的执行顺序

可以使用多个同种装饰器，比如一个类上可以有好多个类装饰器：

```typescript
@Deprecated()
@User()
@Internal
@Provide()
class Foo {}
```

其顺序分为两步。首先，**由上至下**依次对装饰器的表达式求值，得到装饰器的实现，`@Internal` 中实现即为 Internal 方法，而 `@Provide()` 中实现则需要进行一次求值。

然后，这些装饰器的具体实现才会**从下往上**调用，如这里是 Provide、Internal、User、Deprecated 的顺序。从这个角度来看，甚至有点像洋葱模型：

```typescript
function Foo(): MethodDecorator {
  console.log('foo in');
  return (target, propertyKey, descriptor) => {
    console.log('foo out');
  };
}

function Bar(): MethodDecorator {
  console.log('bar in');
  return (target, propertyKey, descriptor) => {
    console.log('bar out');
  };
}

const Baz: MethodDecorator = () => {
  console.log('baz apply');
};

class User {
  @Foo()
  @Bar()
  @Baz
  method() {}
}

// foo in
// bar in
// baz apply
// bar out
// foo out
```

类似的，如果一个方法中的多个参数均存在装饰器，那么同样是 `Parma1 in` - `Param2 in `- `Param2 out` - `Param1 out` 的顺序，也就是**后面参数的装饰器逻辑**反而先执行。

**但我们通常不会在同种装饰器中进行存在依赖关系的操作。** 对于属性、参数装饰器来说，我们通常只进行信息注册，委托别人处理。对于方法装饰器来说，我们最多只进行方法执行前后的逻辑注入。而这些过程都应当是彼此独立的。

### 反射 Reflect

Reflect 在 ES6 中被首次引入，它主要是为了配合 Proxy 保留一份方法原始的实现逻辑。

```js
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target, name, value, receiver);
    if (success) {
      console.log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

Proxy 将修改这个对象的 set 方法，但可以通过 `Reflect.set` 方法获得原本的默认实现（不会被修改），先执行完默认实现逻辑再添加自己的额外逻辑。

Proxy 上的这些方法会一一对应到 Reflect 中（或者说 Reflect 中只有 Proxy 上方法的对应实现），如 defineProperty、deleteProperty、apply、get、set、has 等等。这些方法其实也可以在别的对象上找到，如 `Object.defineProperty`、`Function.prototype.apply` 等等，因此 Reflect 其实也起到了方法收拢的作用。

上面的 Proxy 对象的 set 方法是运行时才实际执行的，也就是说我们通过反射，在**运行时去修改了程序的行为**。这就是反射的核心要素：**在程序运行时去检查以及修改程序行为**，比如在代码运行时通过 `Reflect.construct` 实例化一个类，通过 `Reflect.setPrototypeOf` 修改对象原型指向，这些其实都属于反射 API 。

比如通过反射来实例化一个类：

```typescript
// 普通情况
const foo = new Foo()
foo.hello()

// 基于反射
const foo = Reflect.construct(Foo, [])
const hello = Reflect.get(foo, 'hello')
Reflect.apply(hello, foo, [])
```

反射的核心理念：**在程序运行时去检查以及修改程序行为**。

### 反射元数据 Reflect Metadata

想要使用反射元数据，你还需要安装 [reflect-metadata](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Frbuckton%2Freflect-metadata) ，并在入口文件中的顶部 `import "reflect-metadata"` 。

反射元数据提案（即 `"reflect-metadata"` 包）为顶级对象 Reflect 新增了一批专用于元数据读写的 API，如 `Reflect.defineMetadata`、`Reflect.getMetadata` 等。可以将元数据理解为**用于描述数据的数据**，如某个方法的参数信息、返回值信息就可称为该方法的元数据。

为类或类属性添加了元数据后，构造函数（或是构造函数的原型，根据静态成员还是实例成员决定）会具有 `[[Metadata]]` 属性，该属性内部包含一个 Map 结构，键为属性键，值为元数据键值对。也就是说，**静态成员的元数据信息存储于构造函数**，而**实例成员的元数据信息存储于构造函数的原型上**。

简单使用下元数据的注册与提取：

```typescript
import 'reflect-metadata';

class Foo {
  handler() {}
}

Reflect.defineMetadata('class:key', 'class metadata', Foo);
Reflect.defineMetadata('method:key', 'handler metadata', Foo, 'handler');
Reflect.defineMetadata(
  'proto:method:key',
  'proto handler metadata',
  Foo.prototype,
  'handler'
);
```

defineMetadata 的入参包括元数据 Key、元数据 Value、目标类 Target 以及一个可选的属性，在这里我们的三个调用分别是在 Foo、Foo.handler 以及 Foo.prototype 上注册元数据。而提取则可以通过 getMetadata 方法：

```typescript
// [ 'class:key' ]
console.log(Reflect.getMetadataKeys(Foo));
// ['method:key']
console.log(Reflect.getMetadataKeys(Foo, 'handler'));
// ['proto:method:key'];
console.log(Reflect.getMetadataKeys(Foo.prototype, 'handler'));

// class metadata
console.log(Reflect.getMetadata('class:key', Foo));
// handler metadata
console.log(Reflect.getMetadata('method:key', Foo, 'handler'));
// proto handler metadata
console.log(Reflect.getMetadata('proto:method:key', Foo.prototype, 'handler'));
```

实际上，反射元数据正是我们实现属性装饰器中提到的“委托”能力的基础。我们在属性装饰器中去注册一个元数据，然后在真正实例化这个类时，就可以拿到类原型上的元数据，以此对实例化完毕的类再进行额外操作。比如说，先通过元数据说明，这个属性需要获得变量 a 的值，在实例化时，我们发现有这个元数据，就会对应进行赋值操作。

正是考虑到这一点，反射元数据中直接就内置了基于装饰器的调用方式：

```typescript
@Reflect.metadata('class:key', 'METADATA_IN_CLASS')
class Foo {
  @Reflect.metadata('prop:key', 'METADATA_IN_PROPERTY')
  public prop: string = 'linbudu';

  @Reflect.metadata('method:key', 'METADATA_IN_METHOD')
  public handler(): void {}
}
```

`@Reflect.metadata` 装饰器会基于应用的位置进行实际的逻辑调用，如在类上装饰时以类作为 target 进行注册，而在静态成员与实例成员中分别使用构造函数、构造函数原型。

```typescript
const foo = new Foo();

// METADATA_IN_CLASS
console.log(Reflect.getMetadata('class:key', Foo));
// undefined
console.log(Reflect.getMetadata('class:key', Foo.prototype));

// METADATA_IN_METHOD
console.log(Reflect.getMetadata('method:key', Foo.prototype, 'handler'));
// METADATA_IN_METHOD
console.log(Reflect.getMetadata('method:key', foo, 'handler'));

// METADATA_IN_PROPERTY
console.log(Reflect.getMetadata('prop:key', Foo.prototype, 'prop'));
// METADATA_IN_PROPERTY
console.log(Reflect.getMetadata('prop:key', foo, 'prop'));
```

看起来我们现在拥有了实现委托的基本能力，但实际上这还不够。所有的元数据都需要我们提前定义好，如果我们希望直接用一些已有的信息作为元数据呢？比如下面这个例子：

```typescript
class UserService {
	@InjectModel()
	userModel: UserModel;
}
```

希望将 userModel 属性的类型 UserModel 作为一个元数据信息注入，同时我不会为 `@InjectModel()` 装饰器提供任何信息，那我们就束手无策了吗？

**反射允许程序去检视自身**，而属性类型作为程序的一部分，也应当是能被反射收集的。为了实现这一目的，反射元数据提案中还内置了基于类型的元数据，你可以通过 `design:type`、`design:paramtypes` 以及 `design:returntype` 这三个内置的元数据 Key，获取到类与类成员的类型、参数类型、返回值类型：

```typescript
import 'reflect-metadata';

function DefineType(type: Object) {
  return Reflect.metadata('design:type', type);
}
function DefineParamTypes(...types: Object[]) {
  return Reflect.metadata('design:paramtypes', types);
}
function DefineReturnType(type: Object) {
  return Reflect.metadata('design:returntype', type);
}

@DefineParamTypes(String, Number)
class Foo {
  @DefineType(String)
  get name() {
    return 'linbudu';
  }

  @DefineType(Function)
  @DefineParamTypes(Number, Number)
  @DefineReturnType(Number)
  add(source: number, input: number): number {
    return source + input;
  }
}

const foo = new Foo();
// [ [Function: Number], [Function: Number] ]
const paramTypes = Reflect.getMetadata('design:paramtypes', foo, 'add');
// [Function: Number]
const returnTypes = Reflect.getMetadata('design:returntype', foo, 'add');
// [Function: String]
const type = Reflect.getMetadata('design:type', foo, 'name');
```

需要注意的是，这一提案实际上并不依赖 TypeScript ，这些类型信息来自于运行时，而非我们的类型标注。同时这些内置元数据取出的值是装箱类型对象，如 String、Number 等。

TypeScript 为其进行了额外的支持，然后我们才可以获取到类型标注所对应的元数据，如：

```typescript
class Bar {
  @DefineType(Foo)
  prop!: Foo;
}

const bar = new Bar();
// [class Foo]
const type2 = Reflect.getMetadata('design:type', bar, 'prop');
```

这也是为什么我们需要启用 `emitDecoratorMetadata` 配置的原因之一。上面的装饰器执行机制代码中我们看到了编译后的装饰器代码，而启用 `emitDecoratorMetadata` 后，产物中会多出这些代码：

```js
var __metadata = (this && this.__metadata) || function (k, v) {
    if (typeof Reflect === "object" && typeof Reflect.metadata === "function") return Reflect.metadata(k, v);
};

__decorate([
    Prop(),
    __metadata("design:type", String) // 新增
], Foo.prototype, "prop", void 0);

__decorate([
    Method(),
    __param(0, Param()),
    __metadata("design:type", Function), // 新增
    __metadata("design:paramtypes", [String]), // 新增
    __metadata("design:returntype", void 0) // 新增
], Foo.prototype, "handler", null);

Foo = __decorate([
    Cls(),
    __param(0, Param()),
    __metadata("design:paramtypes", [String]) // 新增
], Foo);
```
