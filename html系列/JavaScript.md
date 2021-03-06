# 浏览器模型

## 组成

- GUI 渲染线程

  **解析代码**：HTML代码解析为DOM，CSS代码解析为CSSOM（CSS Object Model）
  **对象合成**：将DOM和CSSOM合成一棵渲染树（render tree）
  **布局**：计算出渲染树的布局（layout）
  **绘制**：将渲染树绘制到屏幕

- 事件触发线程

  控制交互，响应用户。事件添加到待处理队列的队尾，等待JS引擎的处理

- 事件轮询处理线程

  轮询消息队列，event loop

- JavaScript引擎线程

  **javascript是单线程运行**。

  ==GUI渲染线程与JS引擎线程互斥JavaScript引擎线程==，JS执行的时间过长，会造成页面的渲染不连贯

  页面的下载和渲染都必须停下来等待js脚本执行完成，==操作的DOM节点必须存在才能js操作==。一般依赖库最先声明，执行脚本最后书写。

- 定时触发器线程

  单独线程来计时并触发定时更准确。setTimeout和setInteval

- 异步http请求线程

  XMLHttpRequest新开一个线程请求，将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件放到JS引擎的处理队列中等待处理。

  Chrome中打开一个网页相当于起了一个**进程**，每个**tab**网页都有由其独立的渲染引擎实例

  

## 解析页面

1. 开始解析HTML

   解析器将HTML转换为文档对象模型(**DOM**)

2. 获取外部资源

   遇到外部资源（如CSS或JavaScript文件）时，提取这些文件。 解析器在加载CSS文件时继续运行，此时会阻止页面渲染但仍解析，直到资源加载解析完

   解析器==先加载执行 JS 文件，阻塞渲染线程和HTML解析过程==。 defer` 和`async`允许同时执行

3. 解析 CSS 并构建CSSOM

   CSSOM 与 DOM一起构建渲染树

4. 执行 JavaScript

   JS和DOM被完全解析并准备就绪后就会 发生`document.DOMContentLoaded`事件。 所有资源(异步JavaScript，图像)出发`Window.load`事件

5. 合并 DOM 和 CSSOM 以构造渲染树

   **渲染树**是**DOM**和**CSSOM**的组合，表示将要渲染到页面上的所有内容

6. 计算布局和绘制

   渲染引擎从顶部开始一直向下遍历渲染树，计算应显示每个节点的坐标

## event loop机制

异步任务进入消息队列，指定回调函数。只有消息队列通知主线程，并且同步执行栈为空时，该消息对应的==回调函数==才能执行。

1. 所有同步任务都在主线程上执行，形成一个执行栈。主线程之外，还存在一个”任务队列”。
2. 只要异步任务有了运行结果，就在”任务队列”之中放置一个事件。
3. 一旦”执行栈”中的所有同步任务执行完毕，系统就会读取”任务队列”，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
4. 主线程不断重复上面的第三步。
   

<img src="eventlppo机制" alt="这里写图片描述" style="zoom: 33%;" />



主线程从”任务队列”中读取事件，这个过程是==循环不断==的，所以整个的这种运行机制又称为Event Loop（事件循环）

<img src="20201015205742198.gif#pic_center" alt="在这里插入图片描述" style="zoom:50%;" />

Task Queue 的异步任务分为   `微任务` 、`宏任务`；**微任务优先**
宏任务	setTimeout 、setInterval 、UI rendering
微任务	promise 、requestAnimationFrame          (Promise>setTimeout )



# JS

## 基本语法

### 基本类型

1. 反引号\`\`:用于模板字符串：**\`pre  ${varName}   suf\` **和   **多行字符串**
2. str：作为字符的数组
3. Symbol：类似str。Symbol(str)表示独一无二的值，解决命名冲突问题。不能与其他数据运算
4. js中所有数字Number用浮点表示，故而只能表示53位整数。超出2^53无法表示。n后缀表示安全的Integer

### 运算符

1. 比较运算符：始终坚持使用`===`比较，唯一能判断`NaN`的方法是通过`isNaN()`函数
2. boolean：null '' undefine转换为false  非空白转为true
3. 大多数情况用`null`。`undefined`仅仅在判断函数参数是否传递的情况下有用
4. 异步错误必须==回调函数中处理try-catch==

## 循环

`for-of`针对`iterable`:Map、Set、Array的**forEach**方法快捷遍历。==不支持**对象objec**t==，需要Object.keys才能

```js
map.forEach(function (value, key, map) {
    console.log(value);
}//Map支持k-v，但是Array、Set不支持
```

遍历**obj**：①for-in+hasOwnProperty()     ②for-of遍历properName

```js
for (var key in o) {
    if (o.hasOwnProperty(key)) {
        console.log(key); // 'name', 'age', 'city'
    }
}
for (let key of Object.keys(o)) {
    console.log(o[key]);
}
```

for-in 自身+继承  

## Array

length赋一个新的值会导致变化

默认根据str排序，数字也是

...可用于深拷贝数组



## object

1. ==所有属性key都是字符串(自动转换)==，value任意类型

2. 访问方式：①obj.proName    ②obj[  'proName '  **属性名str**  ]，不是str自动转换

3. `in`操作符判断obj是否有某一property，是否xiaoming自身拥有的，而不是来自原型可以用`hasOwnProperty()`

   ```javascript
   'toString' in xiaoming; // true
   xiaoming.hasOwnProperty('name'); // true
   xiaoming.hasOwnProperty('toString'); // false
   ```

4. Object.keys 遍历对象属性包括父类+本类， 仅可枚举属性，无原型

   **Object.getOwnPropertyNames**  获取属性包括父类+本类，包括可枚举和不可枚举的属性

   hasOwnProperty()   是否自身拥有（包括父+子）的，而不是原型得到的
   
   Object.is 判断两个值是否完全严格相等(===)
   Object.assign(base,after) 对象的合并，浅拷贝
   
   Object.keys() / values() 获取所有 keys / values，仅自身，无继承无Symbol
   Object.entries()    [ key,value ]  数组
   
   Object.fromEntries( map)： 从map构造obj
   
5. 函数形参、数组、对象支持尾逗号

## Map和Set

1. **KEY可为任意类型，而Object的property只能是str**
2. Map本质上时**二维数组**  const arr = [...map]输出二维数组的元素  `[keyn,  value1]`  ；Set可用Array初始化
3. **只能用get访问，不能[ ]**,    Object可以**[  ]**访问
4. get/has查键/delete/set方法操作
5. Map转为Json：key为字符串，转为对象json；key为其他，转为二维数组json



## 解构赋值

==设置默认值和函数形参配合使用==，可用于array(  )  对象{  }

```
let {name=默认值, 原属性名:新变量=默认值} = person;
```

支持...array / map / object 打散成序列，map打散成[ k,v]  , [k,v] ...

对象复制：property可直接引用变量，key为变量名，value为变量值

...args：生成参数数组

仅严格等于undefined才会赋默认值，null不会赋默认值

## 函数

### this问题

箭头函数完全修复了`this`的指向，`this`总是指向词法作用域，也就是外层调用者`obj`：

```js
getAge.apply(xiaoming, [参数数组]); //  this指向xiaoming, 参数为数组
getAge.call(null, 3, 5, 4); // this指向xiaoming, 参数为...
```



### 箭头函数

```js
// 两个参数:
(x, y) => x * x + y * y
// 无参数:
() => 3.14
// 可变参数:
(x, y, ...rest) => {
    return a;
}
```

箭头函数完全修复`this`的指向，无需apply和call。`this`总是指向**外层调用者`obj`**：

```js
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = () => new Date().getFullYear() - this.birth; // this指向obj对象
        return fn();
    }
};
obj.getAge(); // 25
```



### 高阶函数（针对Array）

```js
//map  (element, index可选, self可选) 
arr.map(i=>)

//filter   (element, index可选 , self可选) 
var r = arr.filter(function (x) {
    return x % 2 !== 0;
});

//reducecallback （执行数组中每个值的函数，包含四个参数）
    1、previousValue 积累值（上一次调用回调返回的值，或者是提供的初始值（initialValue））
    2、currentValue （数组中当前被处理的元素）
    3、index （当前元素在数组中的索引）可选
    4、array （调用 reduce 的数组）可选
initialValue （可选，作为第一次调用 callback 的第一个参数。）

arr.reduce(callback,[initialValue]):

//sort 类似java 正序返回-1
arr.sort(function (x, y) {
    if (x < y) {
        return 1;
    }
    if (x > y) {
        return -1;
    }
    return 0;
});
```



## Class(简化原型链代码)

**`static`**前缀：声明静态属性

**`get/set`**前缀：getter/setter，常用于合法性校验

```js
class Student {
    static age=100；//static
    name; //pubic
	#weight  //私有属性
    constructor(name) {
        this.name = name;
        this. #weight = 1000;
    }
	get price(){
		console.log(“价格属性被读取了");return 'iloveyou';
	}

	set price(newVa1){
		console.log('价格属性被修改了');
	}
}

class PrimaryStudent extends Student {
    constructor(name, grade) {
        super(name); // 记得用super调用父类的构造方法!
        this.grade = grade;
    }
}
```

## Promise  承诺内部有函数将来会执行，即内部有异步操作

==本身是同步，但是内部会触发异步方法==，Promise链根据不同状态进行不同调用。

> **Promise正常结束调用resolve( )方法，异常则调用reject( )方法**

### 状态转换

![img](v2-bcb0b896fc17b4f99b7ea9e4dfbd85d3_720w.jpg)

- pending: 初始状态，不是成功或失败状态。

- fulfilled: 意味着操作成功完成。

  **then()**：注册Promise正常结束的回调，多次then会根据顺序串行执行，参数为resolve(value )保存的value 

- rejected: 意味着操作失败。

  **catch()**：注册Promise失败的回调函数，异常可传递直至catch( )捕获，参数为reject(value )保存的value 

- fulfilled 和 rejected 状态只能由 pending 转化而来，两者之间不能互相转换。==只能转换一次==

### 构造方法

```js
new Promise(function (resolve, reject) { 
    setTimeout(function () {
        if (timeOut < 1) {
            resolve('200 OK');          //转换成fulfilled ，设置值，传递给then回调
        }
        else {
            reject('timeout in ');       //转换rejected ，设置值，传递给catch回调
        }
    }, timeOut * 1000);
})
Promise.resolve()    //生成fulfilled 状态的Promise
Promise.reject()	//生成rejected 状态的Promise
```

1. resolve(value )执行：转换成fulfilled状态，设置值value，传递给then回调，回调参数为保存的value值。
2. reject(value )执行：转换成rejected 状态，设置值value，传递给catch回调，回调参数为保存的value值。
3.  catch错误发生时专门捕获异常，==整条调用链==都可以被.catch捕获，用于==统一异常处理==
4. finally于调用链末尾，比如执行其回调

### 调用链

有多次then()可==串行处理==，then() catch()设置的回调函数有不同的返回值，==但都会处理成全新的Promise==

```js
job1.then(job2).then(job3).catch(handleError); //同步执行

Promise.reject().catch(function() {
  return 'Hello World';
})
.then(function(value) {
  console.log(`fulfilled: ${value}`); // 'fulfilled: Hello World'
})
.catch(function(value) {
  console.log(`rejected: ${value}`);
})
```

1. **then()和catch()返回普通对象，==全部==包装成resolve(fulfilled)状态的Promise对象，与原状态无关**
2. then()和catch()可以返回`指定状态`的Promise
3. then()和catch()抛出错误`时，==全部==包装成rejected(rejected)状态的Promise对象

### 并行多个Promise

1. `Promise.all([Promise数组])`：**所有**Promise都执行完毕才继续，==生成数组往后传递==
2. `Promise.race([Promise数组])`：**任意一个**Promise执行完就返回

```js
// 同时执行p1和p2，并在它们都完成后执行then:
Promise.all([p1, p2]).then(function (results) {
    console.log(results); // 获得一个Array: ['P1', 'P2']
});

Promise.race([p1, p2]).then(function (result) {
    console.log(result); // 'P1'
});  
```



### async await便于构造Promise异步执行逻辑，是基于promises的语法糖

1. 

2. **async **声明该函数是一个**异步**函数，==返回值自动包装成Promise(resolve)==。该函数内部触发异步操作，`但是同步执行非异步操作`。==可直接进行then操作==

3. **await** ***<u>只能放在 async 函数内部</u>***

   后跟Promise：==阻塞后面的代码，等待标识的Promise返回resolve值==，获取返回值 
   后跟非Promise：用Promise.resolve包装直接返回 

4. 当 async 函数中只要一个 await 出现 reject 状态，则后面的 await 都不会被执行，可.catch()统一异常处理

5. **await+Promise.all( [ Promise数组 ] )**：**并发**执行完毕返回结果Array

## 内置对象

### Json

1. JSON.stringify(xiaoming, ['name', 'skills'  属性名Array], '  '缩进);
2. 或重写`toJSON()`的方法，直接返回JSON应该序列化的数据：
3. JSON.parse(str)：反序列化

```js
var xiaoming = {
    name: '小明',
    age: 14,
    gender: true,
    height: 1.65,
    grade: null,
    'middle-school': '\"W3C\" Middle School',
    skills: ['JavaScript', 'Java', 'Python', 'Lisp'],
    toJSON: function () {
        return { // 只输出name和age，并且改变了key：
            'Name': this.name,
            'Age': this.age
        };
    }
};
JSON.stringify(xiaoming, ['name', 'skills'属性名Array], '缩进');
JSON.stringify(xiaoming); // '{"Name":"小明","Age":14}'

var obj = JSON.parse('{"name":"小明","age":14}', function (key, value) {
    if (key === 'name') {
        return value + '同学';
    }
    return value;
});
console.log(JSON.stringify(obj)); // {name: '小明同学', age: 14}
```

### Blob/File

Blob不可变、原始数据的文件对象。它的数据可以按文本或二进制的格式进行读取，可读

File 实现了Blob： \<input>标签返回`FileList`数组的组成

### FormData

生成键值对、键文件

```js
var formData = new FormData();
var formData = new FormData(formElement); //从已有form-dom生成

formData.append("username", "Groucho");
formData.append("accountnum", 123456); //数字123456会被立即转换成字符串 "123456"

// HTML 文件类型input，由用户选择
formData.append("userfile", fileInputElement.files[0]);
```

### Date

JavaScript的Date对象月份值从0开始，牢记0=1月，1=2月，2=3月，……，11=12月。

```js
var d = Date.parse('2015-06-24T19:49:22.875+08:00');//ISO标准格式创建
var d = new Date(1435146562875);					//时间戳创建
getTime()//获取时间戳
```

### FileReader对象(异步读取，需要回调)

- readAsText():读取**文本文件**返回String，默认UTF-8
- readAsBinaryString():读取二进制文件返回流，用于上传文件给后端
- readAsDataURL():解析文件为DaeaUrl
  - 无返回值
  - 参数Blob：可转换的文件(filedivnode.files[0])
- abort():中断读取
- 事件回调
  - **onabort**:读取文件中断时触发
  - **onerror**:读取错误时触发
  - **onload**:文件==读取成功==完成时触发
  - **onloadstart**:开始读取时触发
  - **onloadend**:读取完成时触发，==无论成功还是失败==
  - **onprogress**:读取文件过程中持续触发
- **result**：读取结果自动保存在FileReader对象的result属性。**事件回调函数中操作result**

### WebStorage存储键值对

- localStorage：全局有效

- sessionStorage：同一Tab有效，Tab关闭即清空

- setItem(k,v)       getItem(k)       removeItem(k)   clear()

  `target="_blank"`的A标签、``window.open`打开新窗口时，会把旧窗口sessionStorage**拷贝而非引用**。

  ==同一Tab刷新，sessionStorage扔有效==

# 模块化

## ES6

默认情况下，ES6模块中的**所有内容都是私有的**，并且以严格模式运行

暴露变量：①`exports default `  用**任意变量**来接收     ②`exports`  声明单个或**{ }的形式指定变量**按需导出，as别名

导入变量：① `import` 		② 浏览器下\<script type="module ">

​					③动态import(‘ ‘)返回Peomise

<u>`export`可多次导出单个变量或封装为对象统一导出</u>

```js
import { sum as addAll, mult as multiplyAll } from './lib.js';  //针对export

import * as lib from './lib.js';
import v from './lib.js';     								//只针对export default

export var firstName = 'Michael';

export default {
    name: 'zs',
    age: 20
}
export default info

btn.onclick = function(f
import('./hello.js').then(module => i
module.hello();
)}
```

1. ES6导入import的变量是 ==引用==，而非拷贝，==会**动态变化**==，(a,b,c引用d的某变量v，v一旦变化，abc中所有值都会变化)
2. import不论写在何处，最优先执行
3. 编译时运行导入的模块(整个模块的语句都会执行)，
4. module==仅执行一次==，不论import几次

## CommonJS

暴露变量 ①`exports`.varName=XX         ②`module.exports`={ }(实际上是同一个变量)

导入变量： `require`

1. `module.exports = xxx`的方式来输出模块变量万能
2. CommonJs import的变量是 ==拷贝==，而非引用。不会变化
3. 运行时加载，仅一次





