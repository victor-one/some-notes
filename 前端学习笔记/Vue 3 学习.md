# Vue 3 学习

## Vue安装方式

1. CDN引入；
2. JS文件导入；
3. npm;
4. vite。

## 声明式渲染

data property

## 插值

文本：Mustache 语法  --->  {{  }}

v-once

v-html(容易导致XSS攻击，因此只对可信内容使用HTML插值，绝不要将用户提供的内容作为插值。)

Mustache 语法不能在 HTML attribute 中使用，然而，可以使用 v-bind指令。

v-bind  : 动态绑定属性的内容。

v-on  @ 监听DOM事件

使用JS表达式：Mustache中可以使用；v-bind下也可以使用；

## 指令

v-bind 语法糖 : 

v-on 语法糖 @

## 动态参数

动态属性

\[attributeName\]

动态事件

\[mouseEvent\]

## data property 和方法

在定义methods时应当避免使用箭头函数，因为这会阻止Vue绑定恰当的this指向。

## 计算属性和侦听器

模板表达式用于简单运算，对于任何包含响应式数据的复杂逻辑都应该使用计算属性computed。

 computed只要依赖值不变就不会重新计算。

computed和methods的区别：

methods中的方法需要调用（加括号），JS表达式计算三次，计算属性执行一次（因为依赖值没变，会缓存结果），方法会计算三次。主要区别就是计算属性有缓存（基于响应依赖关系缓存，提高性能）。

## 计算属性的getter和setter

每一个计算属性中都有一个getter和setter。

get在调用computed属性的时候调用，set在设置或者更改计算属性的时候调用。

set中有一个newValue。

计算属性一般没有set方法，因此计算属性是只读属性。

## 侦听器

watch

用于监听数据的变化，可以执行异步操作或一些复杂的逻辑代码。

每当监听数据变化就调用这个函数。

侦听器可用于一个数据影响多个数据。

## watch对对象进行深度监听

v-model 数据的双向绑定。

watch有两种写法，一种写法为函数式，一种写法为对象式。

对象式可以有immediate属性，值为Boolean值，为真时表示初始化的时候调用handler中的函数。handler中写函数式中的函数。

**watch无法直接监听对象中的属性变化，需要使用深度监听deep**，值为Boolean值，表示是否深度监听。侦听器会一层层的向下遍历，给对象中的每个属性都加上侦听器。

若要对对象中某个属性单独加上侦听器，在前面使用如下格式“对象.属性”。使用字符串的形式进行优化，只会单独监听对象中对应的属性。

## class类名的对象的使用方式

用法：`:class="{类名: boolean}"`

和普通的类同时存在，不会起冲突，会合并。

绑定的对象不一定要写在模板里面，也可以写在data里面。

也可以使用computed

## class类名的数组使用方式

用法：`:class="[XX,XX]"`

数组语法不常用，对象语法常用。因为不能灵活操作类名。

数组和对象语法是可以结合使用的。

## style样式的多种操作方式

**绑定内联样式**

CSS property名使用小驼峰命名（camelCase）或短横线分割（kebab-case，记得用引号括起来）命名。

如果采用对象形式，key值放置CSS属性名，value放置属性值，属性值可来自于data中的属性。

**数组语法**

可以将多个样式对象应用到同一元素上。

## 条件渲染 v-if

v-if、v-else、v-else-if

后面可以接表达式，注意表达式最后返回结果必须为Boolean。

在template上使用v-if渲染分组。（包裹的template元素不会显示出来）

## 条件渲染 v-show 以及和 v-if 的区别

v-show 

用法和v-if大致一样，区别是带有v-show的元素始终会被渲染并保留在DOM中，v-show只是简单的切换元素的display CSS property。

v-show不支持template，也不支持v-else。

**v-if 与 v-show对比总结**

* v-if 只要后面的条件为false，对应的元素及子元素都不会被渲染，控制DOM元素的创建和销毁；
* v-show元素只是简单的对元素的display进行切换，并不会对DOM元素进行销毁。
* 当需要频繁切换状态的时候，使用v-show更好；当条件很少修改的时候或一次性的时候使用v-if。

## 列表渲染v-for

可以使用v-for指令基于一个数组来渲染一个列表。v-for指令需要使用 item in items ，items 是元数据数组，item是别名，代表数组中的每一个元素。使用时需要指定key。

在v-for中，还存在着一个可选参数 -- index，表示对应元素的下标。

也可以使用of 代替 in 作为分隔符，效果是一样的。

v-for使用对象，item表示键值，可选参数key表示键名，可选参数index表示索引。

## v-for 为什么要加key

为了给Vue一个提示，以便跟踪每个节点的身份，从而重用和重新排序现有的元素。

示例：v-for与checkbox联合使用。

key唯一标志。

如果没有key标识的情况，会随着位置进行渲染，当有key标识的情况下，会根据key进行渲染。

key的作用：快速找到节点，减少渲染次数，提高渲染性能。

## 数组更新检测

通过索引值去修改数组，Vue3可以。

push()：给数组末尾添加元素，可以添加多个元素，这是因为源码中传参使用的是`...item`的形式。

pop()：删除数组最末尾的元素，里面不需要写参数。

shift()：对数组的第一位进行删除（队列出队）。

unshift()：给数组的首位添加元素，可以添加多个元素。

splice()：可以用于删除元素、插入元素、替换元素。

删除元素：

第一个参数表示开始插入或开始删除的元素的下标；

第二个参数表示要删除的元素个数，若不传参数的话就会从开始位置一直删到最后。

插入元素：

第一个参数意义不变；

第二个参数传入0；

第三个参数表示要插入的元素。

替换元素：

第一个参数意义不变；

第二个参数表示要替换的元素个数；

第三个参数表示要替换的元素。

其实就是删除操作，第二个表示删除几个元素，第三个参数表示插入的元素。

sort()：排序，默认是从小到大。

reverse()：数组翻转。

## 事件处理

使用v-on指令来监听事件（通常简写为“@”）

建议在methods里面定义属性方法来使用。

若事件不传参的时候，在函数中写的参数为事件对象。

若要既传递参数，又有事件对象的话，使用`$event`作为实参。

事件处理程序中可以传递多个处理函数，使用逗号分隔即可。

## 事件修饰符

修饰符由点开头的指令后缀来表示

`.stop`：阻止事件冒泡。

`.prevent`：阻止默认行为。

`.once`：只会触发一次回调函数。

按键修饰符：如 `.enter`、`.tab` 等

可以通过keyCode（键盘编码）和keyAlias（键盘的简写）监听键盘的某一个键帽

## 表单输入绑定v-model 的原理

可以使用v-model指令在表单元素`<input>`、`<textarea>`及`<select>`上创建双向数据绑定。

**v-model 原理**：

本质是两个操作：

1、v-bind绑定一个value属性；

2、v-on给当前表单元素添加一个input事件。

绑定value属性：

```vue
<input type="text" :value="msg" @input="changeValue" />
```

input触发之后的函数：

```javascript
changeValue(e){
    this.msg = e.target.value;
}
```

## v-model表单控件的基本使用

**复选框**

单个勾选框，v-model为布尔值。

多个勾选框，需要有value值，可以绑定一个数组，选中的value将会进入到数组中。

**单选框**

同样需要有value值，将选中的value值赋值给变量。加了v-model之后就不需要再加 name 属性了。

**选项框**

**单选**

需要在option里有value。

**多选**

需要在select元素加上 multiple属性。v-model绑定的为一个数组。

## v-model修饰符的使用

**.lazy**：每次输入的时候都会实时更新，这样会消耗性能。这时候就可以使用 .lazy ，当输入框失去焦点，再去同步输入框中的数据。

**.number**：自动将输入框中的内容自动转换为数字类型。

**.trim**：自动过滤用户输入的首尾空白字符。

## vue的组件化开发

组件需要能够复用。

## vue组件的基本使用

组件的首字母一般是大写的。

通过import引入组件，在components中注册组件，利用标签使用组件。

项目的根组件为App.vue

组件是带有名称的可复用的实例，起名根据实际开发的功能模块起名。

在vue应用中使用单文件组件。

## 父组件和子组件

组件是单独功能模块的封装。

## 组件数据的存放

组件无法直接获得另一个组件中的数据。

data必须是一个函数，并返回一个对象。

**为什么data必须是一个函数，并返回一个对象？**

这样每个组件对象返回的都是一个新的对象，不会造成数据的污染。

## 父传子通过prop传值

在子组件中设置props选项，从而让子组件具有新的属性。可以使用数组形式，但在实际开发中，props以对象的形式列出。

这样可以对类型进行限制，且除了类型限制还可以设置默认值，是否为必传属性。

需要将属性后面设置为对象形式才可以进行上述操作。

如果有多种可能的类型，可以用数组的形式。

对象和数组的默认值必须从一个工厂函数返回。

**单向数据流**：父组件中prop的更新会影响到子组件，但子组件不行。

## 子组件通过自定义事件向父组件传值

在子组件中通过`$emit`来触发事件，第一个表示自定义事件的名称（在父组件中使用这个事件），第二个表示发送的事件参数。之后在父组件通过`v-on`监听子组件中自定义的事件，并触发父组件中的函数，函数有一个默认的value即为传递的子组件的值，将value值赋给父组件中的值即可。

## 父组件访问子组件-\$refs

父组件访问子组件，\$refs，开发中常用。

ref用来在父组件中给元素或者子组件注册引用信息，之后使用的时候在父组件中使用\$refs即可访问到对应组件中的值或方法。（非常强大）

在Vue2中可以使用\$children，但是Vue3中已经废弃了，因为返回的是一个数组。

子组件访问父组件，\$parent

子组件访问根组件，\$root

## 子组件访问根组件和父组件 -- \$parent -- \$root

实际开发中不推荐使用\$parent，因为考虑到组件的复用性，\$parent会经常改变。请使用props通过父组件传值。

## 插槽的基本使用

在移动端经常使用。

`<slot></slot>`

使用插槽的组件引用时不能写单标签，必须双标签。

## 具名插槽的使用

如果有多个值，同时放到组件中进行替换时，它们会一起作为替换元素。

如果有多个地方需要替换，需要多个插槽，此时为了区分插槽使用具名插槽，加上name属性。

使用的时候，使用template元素包裹内容，并在template中使用`v-slot: 对应插槽名字`，若不匹配就不替换。

在Vue2中直接在添加的元素里面使用`slot`。

插槽与模板具有相同的作用域。

父级模板里的所有内容都是在父级作用域中编译的，子模板里的所有内容都是在子作用域中编译的。

## 插槽备用内容和作用域插槽的使用

备用内容，会在没有提供内容的时候进行渲染。

作用域插槽：在slot中设置props，之后在使用组件时，在template上利用`v-slot:XXX="所有props组成的对象"`

作用域插槽：父组件替换插槽的标签，但是数据由子组件来提供。

将包含所有插槽prop的对象命名为slotProps。

## 组件之间的跨级通信-provide和inject

`provide(){return{AAA:this.aaa}}`

`inject:['AAA']`

provide和inject并不是响应式的。

如果想要响应式，可以使用响应式对象的方法，也就是说provide传的是一个对象，（可以理解为一个浅拷贝）

也可返回响应式数据（就是返回一个返回值为目标值的函数），inject中接受的就是函数，使用的时候记得加括号。

注意：mustache语法中不要写函数，因此在组件中使用计算属性来得到目标数据

## Vue生命周期钩子

![生命周期图示](https://v3.cn.vuejs.org/images/lifecycle.svg)

1. 实例化一个对象；
2. 初始化init，确定规则（events & lifecycle），数据代理还没有开始。之后执行beforeCreate()，此时示例还没有创建，拿不到DOM节点、数据、方法等；
3. 初始化响应式、数据监测、数据代理。之后执行created()，此时实例已经创建了，但是DOM还没有生成，也就是说页面还没有生成。
4. 判断是否有template选项。如果有，将template编译成一个渲染函数。如果没有，将它挂载的HTML编译成template。之后执行beforeMount()，表示组件挂载之前，此时页面还没有渲染，但是虚拟DOM已经生成了。但是对DOM做操作是没有用的。
5. 将编译好的HTML挂载到目标HTML位置。执行mounted()，DOM结构完成，页面可以显示出来。
6. 实时监听数据变化，当数据发生改变时，执行beforeUpdate()，（此时虚拟DOM已经更新了），将虚拟DOM重新渲染（通过对比节点得到不同，然后更新），更新完毕执行updated()。
7. 当app的unmount()执行时，实例被销毁，执行beforeUnmount()，此时原来的数据仍能使用，可以在里面写一些解绑事件监听或清除计时器。实例销毁之后，执行unmounted()。

最好能画出图。

## 组合式API的初体验

组合式API将同一个逻辑关注点的相关代码收集在一起。

setup()在组件被创建之前执行。在beforeCreate()、Creatted()，所以setup()可以替换掉这两个。

setup()中没有this，this不会指向实例。（因为setup的调用发生在`data` property、`computed` property 或 `methods`被解析之前。）

## Vue3组合式API在模板上的应用

通过return将setup()中的数据暴露出去。（此时数据不是响应式的）

setup中也可以直接写函数，同样需要return出去才能使用。

想要响应式数据需要通过ref来定义响应式变量。

ref是一个函数，它会返回一个带有value属性的对象。

模板会自动解析value值。

将值封装在对象中是为了保持JavaScript中不同数据类型的行为统一。

## Vue3组合式API在setup中定义变量

ref定义响应式变量

reactive定义响应式对象/数组

注意，通过ES6扩展运算符进行解构，会使得对象中的属性不是响应式的。

toRefs使解构后的数据重新获得响应式。

`...toRefs(obj)`，注意toRefs是一个函数，里面必须传一个对象进去。

也可以这样写`let {name, children} = toRefs(obj)`

## 在setup中使用watch

选项式API中watch的使用是以对象的形式。在setup中使用需要先引入，在里面的使用是以函数的形式，接受3个参数，侦听的响应式引用或getter函数、回调函数、可选配置项（不常用）。

同样无法直接侦听对象中的数据。需要引入watchEffect来进行侦听。

watchEffect中只有一个参数，即为回调函数。

注意，watchEffect不需要指定监听的属性，因为它会自动收集依赖。因为它在组件初始化的时候会执行一次回调函数，自动收集依赖。

**watch和watchEffect的区别**

1. watchEffect不需要有指定的监听属性，因为它会自动收集依赖，只要在回调中引用到了响应式的属性。只要这些属性发生改变，回调就会执行。watch只能监听指定的属性，做出回调函数的执行，可以侦听多个。
2. watch可以获取到新值和旧值，watchEffect拿不到。
3. watchEffect在组件初始化的时候就会自动执行一次用来收集数据，watch不需要，一开始就指定了。

## 在setup中使用computed

同样需要先引入。返回一个带有value属性的对象（和ref一样）。

## 生命周期钩子函数在setup中使用

需要先引入，需要前面加on。是一个函数，里面接受一个回调函数参数。生命周期钩子函数可以多次执行。

## setup中的参数props

setup中的props是响应式的。不可以使用ES6解构，会消除响应式。如果想要响应式，还是使用toRefs。注意，toRefs返回的还是一个带有value的对象。

## setup中的参数context

context是一个普通的JS对象，暴露了能在setup中有用的值。

对象中有attrs（非响应式对象）、slots（非响应式对象）、emit（方法）、公共property（函数）

setup还可以返回一个渲染函数。

可以使用`context.expose({})`将里面的方法暴露出去，在父组件中使用ref调用

## 在setup中使用provide-inject

在setup返回的refs在模板中使用会被自动浅解包，因此不应在模板中使用`.value`。

但是在setup中还是要使用的。

使用provide需要先导入，导入的是一个方法。允许两个参数，name（需要是字符串类型）和value。

inject需要在子组件中引入，有两个参数，name和默认值（可以不写）。

注意：此时不是响应式的，需要使用ref变为响应式。

## 在单文件组件（SFC）中使用组合式API的编译时语法糖

1. 引入组件不需要注册
2. 定义变量在模板中使用不需要暴露出去，模板直接使用。
3. 定义响应式变量，还是需要从Vue中引入，但是在setup中还是需要加`.value`

为什么不需要返回出去，因为顶层的绑定会被暴露给模板。

# vue-router

## vue-router引入和安装

路由核心：改变URL，但是页面不进行整体刷新。

路由理解为指向。

路由表是一个映射表，一个路由就是一组映射关系，key - value，key表示路由，value可以为function或者Component（组件）。

vue-router是基于路由和组件的，路由是用来设定访问路径，将路径和组件映射起来。

推荐，终端用来做安装操作。

## vue-router的基本使用

src下创建一个文件夹router，在里面新建一个文件`index.js`在里面进行路由集中管理。

首先引入路由组件，之后写一个路由映射表，创建路由实例（使用`createRouter`）并传递路由配置，设置history(hash模式：`createWebHashHistory`)和路由表。

导出路由，`export default router`。

在`main.js`引入路由。

**需要在挂载前先使用路由`use(router)`，不然会识别不了`router-link`和`router-view`**

**router-link**

在页面上表现为`<a>`标签。

使用一个自定义组件`router-link`来创建链接，这使得 Vue Router 可以在不重新加载页面的情况下更改URL，处理URL的生成以及编码。

使用`to = "路径"`的方法指向组件。

**router-view**

路由出口，占位符。路由匹配到的组件将渲染在这里。

## vue-router带参数的动态路由匹配

路由参数用`:`表示。当一个路由被匹配的时候，它的 params 的值将在每个组件中以`this.$route.params`的形式暴露出来。

`this.$route`表示当前活跃的路由对象。

当setup（组合式API）的时候，不能通过上述方法拿到。需要引入`useRoute`，之后使用的时候，方法为`useRoute().params.参数`。

## vue-router-404页面

写一个404页面，然后在路径中使用正则，`/:path(.*)`（匹配任意的）

## 路由正则与重复参数

在后面加括号，括号中写正则。

多个参数的时候`/:参数+`

参数可有可无的时候`/:参数*` 或者 `/:参数?`（但是?不支持多参数，不可以重复叠加）

## 嵌套路由

使用`children`来创建子路由，子路由的`path`不用加`/`（因为会自动加）

## 通过JS跳转页面

编程式导航

可以通过`$router`访问路由实例，需要使用`this.$router`

`$router`表示**全局路由实例对象**，可以通过push、forward、go方法进行页面操作。

通过`$router.push('/')`进行跳转。

也可以以传入对象形式进行跳转。`$router.push({path: "/"})`

传入对象的好处是可以带参数。

也可以写带`?`的形式，`this.$router.push({path:"/about", query: {name: "xxx"}})`

`$route`表示**当前活跃的路由对象**，可以通过这个拿到path、params、query（要传的参数）、name。

## 替换页面以及在历史堆栈中前进和后退

如果不想history在页面替换时添加记录，可以在`$router.push`里面添加一个属性`replace: true`

也可以使用`$router.replace()`

通过`$router.go(n)`，前进传入正数，后退传入负值。移动几步传递数字几。一般来说传 -1。

传 -1 相当于 `$router.back()` 后退

传 1 相当于`$router.forward()` 前进

## 命名路由和命名视图

使用方式：`:to="{name: 'xxx', params: {id: XXX}}"`

命名视图：

使用`componennts:{}`，然后在`router-view`中设置`name`属性。

## 重定向和别名

使用`redirect`后面接上要访问的路径`redirect: '/'`

也可以在后面使用命名路由`redirect: {name: 'XXX'}`

也可以使用方法`redirect: (to)=>{return {path: 'XXX'}}`，里面的`to`表示当前路由实例。

别名：`alias`

多个别名可以使用数组的形式。

## 路由组件传参

使用props传递给路由组件

在路由表中设置属性`props: true`

然后用`props:['XXX']`在选项式API中获取参数，这样就可以通过`this.xxx`使用了。

在组合式API中使用`defineProps`和`defineEmits`来声明`props`和`emits`

在命名视图中，使用`props:{default: true, ShopFooter: true}`使组件可以拿到props。

## 不同的历史模式

hash模式：`createWebHashHistory()`,不会向服务器发送请求。

history模式：`createWebHistory()`，需要适当的服务器配置。

二者的区别有无`#`

## 路由守卫

通过跳转或者取消的方式守卫导航。

**全局前置守卫**：`router.beforeEach`，里面是一个回调函数，里面有三个参数`to`、`from`和`next`。

next相当于to和from之间的一个关卡，next是一个函数，相当于一个通行证。

**每路守卫**：`beforeEnter`（路由独享的守卫），参数一样，to指的是要进入的路由，from代表从哪个路由进入的。

**组件内的守卫**：

* `beforeRouteEnter`：路由进入组件之前
* `beforeRouteUpdate`：路由更新组件之前
* `beforeRouteLeave`：路由离开组件之前

作为函数使用。里面也有to、from，`beforeRouteEnter`里面拿不到this（实例对象），此时需要使用next里面的回调函数中的参数拿到（参数表示路由实例，相当于this）。

## 路由懒加载

当路由被访问的时候才会加载路由组件。

使用`() => import('......')`引入组件

## 状态管理

vuex做的工作就是集中管理状态。

不使用vuex的时候使用provide/inject 跨级通信进行状态管理。

新建一个store文件夹，里面写一个index.js，里面存放状态集中管理的数据。

利用ref、reactive实现响应式。

```js
import {reactive} from 'vue'

const store = {
  state:reactive({
    msg: "helloworld"
  }),
  updateMsg:function(){
    this.state.msg='你好'
  }
}

export default store
```

## 使用fetch获取数据

fetch属于原生JS，是http数据请求的一种方式。

axios是第三方库

fetch会返回一个promise对象，通过then进行处理获得回调之后的数据。需要使用`res.json()`将响应回来的body解析成JSON的promise对象。之后再通过一个then得到数据

可以在状态管理里面进行集中管理。

## 使用axios获取数据

对Ajax的封装

基于promise的http库

## vite使用proxy解决跨域问题

同源策略是浏览器的保护机制。

proxy代理

访问地址使用target配置，表示替换的服务端地址。

`changeOrigin:true`表示开启代理，允许跨域请求数据

`rewrite:path = path.replace(/^\/path/,'')`，通过正则表达式重写路径

## 通过vue_cli创建项目

## 认识vuex

状态、视图、操作，单向数据流

将组件与组件的关系变成了组件与仓库的关系。

## vuex的基本使用--state

vuex应用的核心就是store（仓库）。

vuex的状态存储是响应式的。

不能直接改变store中的状态。改变store中的状态的唯一途径就是显式地提交（commit）mutation。

createStore

`$store`

`state(){return {xxx:xxx}}`

## vuex的基本使用--mutation

不要使用其他方式，因为这样vuex跟踪不到数据的变化。

里面函数都有一个参数state，表示上面state返回的的state对象。

mutation使用`$store.commit('xxxx')`触发函数。

commit中第一个参数为函数，第二个为接收值（载荷），一般来说载荷应该是一个对象。

mutation必须是一个同步函数

## getters

可以理解为仓库中的计算属性。

如果state中的状态需要过滤或者其他操作放在getters里面。

里面第一个参数是state，第二个参数是getters，表示当前store中的getters对象

## action

action类似于mutation，不同的是：action提交的是mutation，而不是直接更改状态；action包含任意异步操作。

通过`$store.dispatch`触发

在vue_cli中使用pathRewrite设置重写路径。

action中默认传的是context，是与store实例具有相同的属性和方法的对象。

不能直接在action中直接更改state，要使用mutation进行更改。使用`context.state`。

第二个参数也是载荷。

## module

getters通过`$store.getters`可以直接获取

state通过`$store.state.模块名.状态名`获取

mutation触发提交函数，通过`$store.commit`即可，actions也是一样的。

getters里面函数第二个参数是getters，指的是store实例的getters对象。第三个参数是rootState，指的是根节点实例的state。

## 辅助函数

mapState 辅助函数，状态映射

传在computed里面，可以使用箭头函数，也可以使用字符串参数。

名字一样的话，可以传字符串数组。

如果想有自己私有的属性，computed使用对象形式，mapState使用扩展运算符`...mapState`

mapMutations、mapGetters、mapActions使用同理。

## 用module

命名空间，`namespaced: true`

此时getters使用需要使用`{{$store.getters['a/userNameAge']}}`

mutations需要使用`this.$store.commit('a/updateUserName')`

如果配合辅助函数使用：

`...mapMutations('a',['updateUserName'])`

然后执行函数，直接用`this.updateUserName()`

## v-for 和 v-if 为什么不建议一起使用

指令优先级，v-for优先级高于v-if，如果两者放在一起会先执行v-for，循环出列表，然后使用v-if进行判断，造成性能浪费。

应该使用计算属性，在里面进行过滤，`arr.filter`，然后不使用v-if。

## 在Vue中如何使用自定义指令

两种方法：全局注册、局部注册

局部注册使用`directives`

```js
directives:{//自定义指令
  focus:{// v-focus
    mounted:function(el){// 钩子函数表示被绑定的元素插入父节点的时候调用
      // el表示指令绑定的元素
      // binding：对象，属性name：指令名，value：传递给指令的值。
      el.focus()//获取焦点
    }
  }
}
```

全局注册使用

`Vue.directive("指令名字","对象数据也可以是一个指令函数")`

第二个参数和上面一样。

## 什么是$nextTick

将回调函数延迟在下一次DOM更新数据之后调用。

Vue是异步渲染的框架，数据更新之后DOM是不会立刻渲染的，因此获取`innerHTML`不会立刻获取数据。

因此可以借助定时器来获取，或者使用`this.$nextTick(() => {})`

`$nextTick`会在DOM渲染之后触发，用于获取最新的DOM节点。

使用场景：

1. 在生命周期函数created中进行DOM操作，一定要放到nextTick中执行
2. 在数据变化后要执行某个操作，而这个操作需要使用随数据变化而变化的DOM结构时，这个操作需要放到nextTick中