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

在 js 中，void 操作符会强制将后面的函数声明转化为表达式。但是 ts 中 void 用于描述一个内部没有 return 语句，或者没有显式 return 一个值的函数的返回值。

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

对象类型描述使用 interface，可以理解为这个对象对外提供的接口结构。

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

可以对属性进行修饰，包括可选（Optional）与只读（Readonly）这两种。

**修饰接口属性**

```typescript
interface IDesc {
  name: string;
  age: number;
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

在 ts 中 Object 包含所有的类型（对于 undefined、null、void 0，需要关闭 strictNullChecks）。

与 Object 类似的还有 Boolean、Number、String、Symbol，这几个装箱类型（Boxed Types）同样包含一些超出预期的类型。如 String，其同样包括 undefined、null、void，以及代表的 拆箱类型（Unboxed Types）string。

**任何时候都不应该使用装箱类型。**

object 代表所有非原始类型的类型，即数组、对象与函数类型这些。

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

使用延迟求值的枚举值是有条件的。如果使用了延迟求值，那么没有使用延迟求值的枚举成员必须放在使用常量枚举值声明的成员之后（如上例），或者放在第一位。

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

这是因为使用 let 声明的变量是可以再次赋值的，因此对于 let 声明，**只需要推导至这个值从属的类型即可**。而 const 声明的原始类型变量将不再可变，因此可以一步到位收窄到最精确的字面量类型，但对象类型变量仍可变。

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

在 ts 中，一个没有返回值的函数，其返回类型应当被标记为 void。

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

对于 rest 参数的类型标记，由于其实际上是一个数组，应该使用数组类型进行标注：

```typescript
function foo(arg1: string, ...rest: any[]) { }
```

也可以使用元组类型进行标注：

```typescript
function foo(arg1: string, ...rest: [number, boolean]) { }
```

**重载**

想要实现与入参关联的返回值类型，可以使用 typescript 提供的函数重载签名。

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

* `function func(foo: number, bar: true): string`，重载签名一，传入 bar 的值为 true 时，函数返回值为 string 类型；
* `function func(foo: number, bar?: false): number`，重载签名二，传入 bar 的值为 false 时，函数返回值为 number 类型；
* `function func(foo: number, bar?: boolean): string | number`，函数的实现签名，会包含重载签名的所有可能情况；

需要注意的是：拥有多个重载声明的函数在被调用时，是按照重载的声明顺序往下查找的。

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

* public：此类成员在**类、类的实例、子类**中都能被访问；
* private：此类成员仅能在**类的内部**被访问；
* protected：此类成员仅能在**类与子类中**被访问，示例**不允许再访问受保护的成员**。

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

为了能够表示“任意类型”，ts 中提供了一个内置类型 any。

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

never 是一个“什么都没有”的类型，它甚至不包括空的类型，严格来说，**never 类型不携带任何的类型信息**，因此会在联合类型中被直接移除。

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

非空断言其实是类型断言地简化，使用`!`语法，即`obj!.func()!.prop`的形式标记前面的一个声明一定是非空的（实际上就是剔除了 null 和 undefined 类型）。

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

其应用位置类似于可选链：

```typescript
foo.func?.().prop?.toFixed();
```

不同的是，非空断言的运行时仍然会保持调用链，因此在运行时可能会报错。而可选链则会在某个部分收到 undefined 或 null 的时候直接短路掉，不会再发生后面的调用。

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