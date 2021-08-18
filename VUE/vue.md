<img src="C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210818150934024.png" alt="image-20210818150934024" style="zoom:25%;" />

# 模板语法

## 内容指令

### 文本显示

`{{ }}`双大括号：支持**简单表达式**(三元运行，基本运算)，不支持语句

`v-once `:	仅显示一次初始内容。数据改变时，插值处的内容不更新

`v-html`：	输出真正的 HTML

```html
<div id="counter">
  Counter: {{ counter }}  
</div>

<span v-once>这个将不会改变: {{ msg }}</span>

<span v-html="rawHtml"></span></p>

{{ number + 1 }} 
{{ ok ? 'YES' : 'NO' }}    三元运算
{{ message.split('').reverse().join('')}}
```

### 标签属性 v-bind  ( : )

`v-bind : attrName = varName `：设置*H5标签* 属性，支持**转换boolean**

`v-bind : [ 动态attr变量名 ]= varName ` ：绑定的是变量，此**变量值=属性名称字符串**。**不支持运算**

```html
<div v-bind:id="'list-' + id"></div>
<button v-bind:disabled="isButtonDisabled">支持boolean</button>

<a v-bind:[attributeName变量  值为真正的元素属性名]="url"> ... </a>
```

## 事件处理指令`v-on`   ( @ )

1. `v-on : / @ 事件名 = 函数名 / 函数( 参数) / 简单表达式 `：支持click   keyup 和 快捷键
2. `v-on:[动态事件名变量] = "函数名"` ： 绑定的是变量，此**变量值=事件名字符串**。**不支持运算**

- `$event`：特殊实参，代表**事件本身**
- **仅绑定函数名**默认传入`$event`参数，回调函数可用一个参数接受
- 自定义事件$event=value
- 支持**逗号分隔**的多处理方法

```html
<div id="basic-event">
  <button @click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
<a v-on:click="doSomething"> ... </a> 
<a @click="doSomething('hello ')"> ... </a>
<a v-on:[eventName]="doSomething"> ... </a>
```

### 事件修饰符

#### 通用

- `.stop`：     event.stopPropagation()   ***阻止事件冒泡***
- `.prevent`   ：    event.preventDefault()    **不执行默认动作**，如submit表单 href超链接
- `.capture`：       使用事件捕获模式
- `.self`：      当前元素自身时触发处理函数
- `.once`： ：    只会触发一次
- `.passive`：永远不会调用 `preventDefault()` 改善的滚屏性能，有利于移动端

```html
<!-- 阻止单击事件继续传播 -->
<a @click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a @click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form @submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<div @click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<div @click.self="doThat">...</div>
```

## 渲染指令

1.  `v-if = " boolVar" ` ，`v-else`  ，`v-else-if`：根据绑定的**布尔变量决定是否渲染DOM**
2.  `v-show = " boolVar"` ：**元素总是会被渲染**，单地切换元素的 CSS `display`
3. `v-for = "value  in arr/obj "`**==`:key`==** : 循环遍历**当前标签**，支持**组件props传递**

- `v-if`用于`<template>` 元素：条件渲染
- `v-show` 不支持 `<template>` 元素，也不支持 `v-else`。
- `v-if` 有更高的切换开销， `v-show` 有更高的初始渲染开销
- `v-if` 具有比 `v-for` 更高的优先级
- 在 `v-for` 必须设置唯一 `key` attribute

### 实时监测变更

- 数组操作方法实时监测：`push()` `pop()` `shift()` `unshift()` `splice()` `sort()` `reverse()`
- 新数组替换旧数组：`filter()`、`concat()`  `slice()`
- **排序**建议计算属性**computed**

```html
<ul id="array-rendering">
  <li v-for="item in itemsArray">    
  <li v-for="(item, index) in items">
    {{ item.message }}
  </li>
</ul>

<ul id="v-for-object" class="demo">
  <li v-for="value in myObject">    
  <li v-for="(value, name) in myObject">
  <li v-for="(value, name, index) in myObject">
    {{ value }}
  </li>
</ul>
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>

<!--组件有独立的作用域，使用 props传递值-->
<my-component
  v-for="(item, index) in items"
  :item="item"
  :index="index"
  :key="item.id"
></my-component>
```

## 表单指令 v-model

在 `<input>`、`<textarea>` 及 `<select>` ==双向数据绑定==。根据控件类型自动更新元素。本质是语法糖，监听用户的输入事件以更新数据。默认oninput

- radio select 默认绑定value   
- Checkbox单选默认boolean绑定，value绑定需设置true-value和false-value。支持多选数组
- 支持绑定内联对象`:value="{ number: 123 }"`

```html
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option :value="{ number: 123 }">123</option>
</select>

// 当被选中时
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

### 修饰符

 `.lazy`：改为onchage离开焦点更新

 `.number`：数值类型

 `.trim`：去除首尾空白字符

## 样式指令

### `:class = obj/array`动态地切换 class

①内联对象 `{attrName:bool,...}`  ② 直接绑定对象obj  ③ 数组`[attrStr1,attrStr2]`

绑定对象：peoperty=attrName value只能是boolean            绑定数组可以是str，切换class只能用数组

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
/>
<!-- 支持直接绑定obg或计算属性-->
<div :class="classObject"></div>
data() {	^
  return {  |
    classObject: {
      active: true,
      'text-danger': false
}}}

<!-- 支持直接绑定变量数组[ ]:str-->
<div :class="[activeClass, errorClass]"></div>
data() {
  return {
    activeClass: 'active',
    errorClass: 'text-danger'
}}
<!-- 切换class-->
<div :class="[isActive ? activeClass : '', errorClass]"></div>
```

### :style = obj/array

`:style = obj`：绑定样式对象，支持内联写法

`:style = [ obj1 , obj2 ...]`：样式对象 组成的 数组

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

<div :style="styleObject"></div>
data() {
  return {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
}
<div :style="[baseStylesObj, overridingStylesObj]"></div>

<!-- 多选一，不支持自动跳过-->
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

# 组件

各个组件被组织成一个嵌套的、可重用的组件树，与DOM树类似

## 组件创建

##### 挂载根组件

```js
Vue.createApp({ })//根组件属性
    .component('SearchInput', SearchInputComponent)   //声明全局组件
	.directive('focus', FocusDirective)  
    .use(LocalePlugin)   //使用插件  
    .mount('#app')js
```

在挂载的节点内部使用自定义标签

**全局注册**和**局部注册**

## 组件交互

为了把v-for迭代数据传递到组件里，我们要使用 props：

# Composition API

### `setup(props,{ attrs, slots, emit } )` 组件选项

setup在==Created之前==执行(**beforeCreate-setup-Created**),不能访问  this data  computed  methods

- **`props`只读**，是响应式对象(toRefs解构)不可修改；
- `context` 是普通 JavaScript 对象(es6解构)，暴露：attrs   slots   emit
- `attrs` 和 `slots` 始终最新值，以 `attrs.x` 或 `slots.x` 的方式引用 property
- 内部设置生命周期回调函数


### 响应式变量

Proxy劫持修改操作，在页面实时显示。分为**响应式对象reactive()**和**响应式变量ref()**

#### reactive(obj)

声明响应式对象，**劫持**对象**内部变量**的**赋值**操作，用包装后的变量对内部属性名赋值

#### ref(baseVar)

声明响应式变量，ref(X)内部包装为reactive( ( value : X } )的响应式对象。故而==\<script>中.value==修改，\<template>中直接引用

#### 响应式数组

- ref( [ ] ): 用 push等支持劫持的方法操作
- reactive( { propertyName : [ ] } ): 对propertyName **直接赋值**

#### `toRefs`响应式对象-变量转换

将**reactive** 对象**每个属性** 转成 **ref变量**  ，生成对应的普通对象

- reactive：使用`toRefs(reactiveObj)`***解构***，不能ES6解构(会消除 prop 的响应性)
- 常用于return { ...toRefs( reactiveObj ) }

```js
{
  foo: Ref<number>,
  bar: Ref<number>
}
```

#### 非递归响应式变量

- reactive ref：递归监听 
- shallowRef 、shallowReactive、shallowReadonly ：仅仅劫持第一层

#### `readonly(reactive/ref/普通obj)`

返回原始对象的**递归只读**代理，对内部*<u>所有层</u>*都是*<u>只读</u>*

### 监听

1. `watchEffect( ( ) => )`:立即执行传入的函数，并响应式监听其依赖
2. `停止监听`：执行watchEffect/watch`返回的函数`
3. `watch(ref / reactive , ( newValue,oldValue ) => { } )`：内部操作会自动监听，默认懒执行。更精确
   - 懒执行副作用；
   - 访问**新旧值**newValue/oldValue，数组同时**侦听多个源**

```js
const counter = ref(0)
watch(counter, (newValue, oldValue) => {
  console.log('The new counter value is: ' + counter.value)
})

watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})

const stop=watchEffect(() => {
  // 依赖追踪
  console.log(copy.count)
})

// 停止监听
stop()
```

### 计算

`computed( ( ) => )`：传入getter返回**只读响应式引用**

`computed( {( ) => , ( val ) => } )`：可手动修改**响应式引用**

```js
const count = ref(1)

const plusOne = computed(() => count.value + 1)

const plusOne = computed({
  get: () => count.value + 1,
  set: (val) => {
    count.value = val - 1
  },
})
```

### 生命周期

onInvalidate   清除副作用：未完成的异步



# Router

注册全局守卫beforeEach，获取vuex的store.state的值时，要写在main.js中。因为写在router/index.js时，store未挂载

#Vuex

setup()获取store的值，要使用计算属性computed(()=>useStore().state.***)