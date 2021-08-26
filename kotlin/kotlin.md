# 特色

kotlin编译器十分强大，检查机制十分严格。可将源码译成jvm字节码等各种平台形式

与所有基于Java的框架完全兼容，支持函数式编程

**静态语言**但支持推断变量类型，**严格限制null**

脚本语言和编译语言的结合

垮平台的通用型语言，可开发各种原生应用，如Android、macOS、Windows、Javascript应用

支持直接编译成可以在Windows、Linux和macOS平台上运行的native原生二进制代码。

kotlin不能和java语法混合，groovy可以

默认最后一行结果为返回值

<img src="C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210825105631637.png" alt="image-20210825105631637" style="zoom:50%;" />

# 基本语法

## 变量

- 变量不仅是值，还可以是匿名函数、代码块。没有java static静态变量的概念
- 变量是==**强类型**==，同一变量不能赋值不同类型  `var 变量名[ ：类型 ] [ ? 允许null]`
- **支持变量类型推断**，但==**无初值必须指定类型**==。建议都赋值默认值
- ==**<u>默认变量使用前不能为null</u>**==。可为 **`null`** 要在**类型/返回值**声明时添加 **`?`**
- kotlin**一切都是类/对象**，包括java的void和函数，对应为**Unit**、**(形参类型...)->返回值类型**
- 常量
  - **`val`** 常量：只能赋值一次，可以先声明后赋值，但只能赋值一次
    - 同java：`private final static Class v`      +  `public final static getV( )`
  - **`const val`编译期常量**：  必须初始化赋值(编译时赋值)
    - **编译时赋值**
    - 只能在**函数外定义**(函数执行时才赋值)
    - <u>类型**只能**是常见的**基本数据类型**</u>：数字、String、Boolean
    - 同java： `public final static Class v=XXX` (初始化赋值全部搞定)
- 解构赋值：**`var ( narName ... )=支持解构的变量`**
- **`vararg`**：函数可变参数，***内部作为数组处理***而非集合。和具名参数配合<u>出现在任意位置</u>
  - **`*`**：可展开**数组**，**<u>不支持集合</u>**
- **val** p: String **by** lazy {    // compute the string }

### 特殊操作符

- **`?`**允许null类型声明

- 涉及null访问处理

  - **`? .`**安全调用/访问：**源对象为null直接返回**，终止调用/访问
  - **`? : 表达式/语句`**   Elvis  简化的<u>判空</u>表达式。<u>源对象**==null**执行表达式</u>
  - **`?.let ( 单参有返回值函数 )`**：<u>源对象**！=nul**l则执行该匿名函数</u>
  - **`! !.`**    :非null断言，源对象=null则抛出异常 `KotlinNullPointerException`，慎重使用
  - *

- 判断

  -  **`==`** ：同java **`equals( )`**

  - `===`：引用相等，同java ==

  - 判空操作和java一样  **! = null / == null**

  - **`in`**：是否range/array/list元素

  - **`is   !is`**  ：判断对象是否是指定类实例，属实则**无需强转类型**，同java instanceof。

    - **`as`**：强制类型转换，<u>失败**抛出异常**</u>
    - **`as ?`**：强制类型转换，<u>失败返回**`null`**</u>。注意变量声明必须允许null (**?**)

    

  - 

  - 

```kotlin
var result = telephone?.length ?: "18800008888"
varresult= telephone! !.length

if (a is String) {
	println("字符串长度:"+a.length)       //无需强转
}

var b : String = a as String			//失败抛异常
var b : String? = a as? String          //失败返回null，注意变量声明?


```

交换变量 var a = 1
var b = 2
a = b.also { b = a }

### 变量作用域

顶层：整个APP的全局作用域

- 顶层变量：同java类的 static类型，必须初始化赋值
- 顶层函数(**包级别函数**)：作用域为**包package**。同package直接使用；不同package需要导包。同java类的 static类型。
- 访问修饰符：无指定**默认public**，支持private

成员变量/函数：同java

局部变量/函数：方法内定义新的变量/函数。内部可访问/调用外部

没有package时，每个脚本文件有各自作用域，方法外的顶层非private变量构成共享的全局作用域.

建议使用package管理

### 异常处理

和java语法一致try-catch-finally，支持try-resource-catch

try、catch是表达式，可以返回一个值，try表达式的返回值是try代码块中的最后一个表达式的结果或者是所有catch代码块中的最后一个表达式的结果，finally代码块中的内容不会影响表达式的结果

throw抛出异常

自定义异常

继承Throwable类或其子类

```
class UnskilledException () : IllegalArgumentException("操作不当")

```

**`TODO( " reson " )`**：抛出异常 `NotImplementedError`,终止运行，返回`Nothing`

Java中有两种异常类型，一种是受检异常（ checked exception)，一种是非受检异常(unchecked exception），在编写Java代码时，由于编译器在编译时会检查受检异常，因此IDEA会提示进行try...catch操作。受检异常显得比较麻烦，一直以来争议比较大，可能会导致Java API变得复杂，在编写代码时，需要进行大量的try...catch操作，而Kotlin中相比于Java没有了受检异常，IDEA也不会提示进行try...catch操作。

#### 先决条件函数

kotlin内置的函数，类似assert，可校验关键逻辑

![image-20210825164717066](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210825164717066.png)

## 基本数据类型

java有基本数据类型和引用类型；

==kotlin**中一切都是引用数据类型。基本数据类型全为包装类型**。kotlin**编译器**把jvm**字节码**优化为java**基本数据类型**==

### 数字Int/Long/Float/Double/Byte/Short

初始化赋值**整数**：根据数字大小自动推断`Int/Long`，显式指定 `Long` 型值，追加 `L` 后缀

初始化赋值**浮点数**：一律默认为`Double`。显式指定为 `Float` 类型，添加 `f` 或 `F` 后缀

Kotlin ==**<u>一切都要显式转换</u>**==。小数字类型**不能**隐式转换为大数字类型， `Char` 不能直接当作数字。toXXX()

- 浮点和--->整数：toInt( )仅整数位   roundToInt( )四舍五入
- 支持下划线分割
- **`/`**  : 整数间的除法总是返回整数

- 



- 
- 

### Boolean

只有两个值true/false。严格Boolean

### Char

单引号包裹，不支持直接赋值给数字

## String

- `“  ”` ：普通字符串 ； `“”“  ”“”` ：原样非转义字符串。`' '`单引号针对Char
- 模板字符串：

  - **`${表达式/执行函数}`**：支持**表达式运算**，必须返回字符串
  - **`$变量`**：只能引用变量值
- **`==`** ：同 java **`equals( )`**，或kotlin `equals(" " ,false)`  
  - kotlin中`equals(" ",是否忽略大小写)` 默认false不忽略。等价于Java中的 equalsIgnoreCase()
- `===`：引用比较，涉及<u>jvm字符串常量池</u>机制
- String支持`[ ]`访问字符，支持`for-in`迭代
-  `Char` 不能直接当作数字，String[ ]为Char
- **`+`**不能隐式转换String

#### 常用方法

- **`.lenth`**：字符串长度
- first() / last() / get( index ):首 / 末 / 指定字符
- indexOf( Char )：返回index
- String.format (" 模板" , args)：支持`%dsf`模板格式输出字符串
- trim()/trimEnd()：去首/末空白
- substring(支持 IntRange )：接受range变量，开闭区间由Range确定
- split( )：返回`List<String>`,，可用于解构赋值
- replace(regex,单参函数)
- forEach：字符串遍历

## 集合类型

### 数组

定义：无需java new

1. 基本数据类型数组：**`xXXArrayOf( )`**，返回`xXXArray`。<u>不支持string</u>
2. 非基本类型：**`arrayOf( )`**   ，返回`Array<XXX>`。可自动推断，建议使用
3. **`*`**：可展开**数组**用于函数实参，**<u>不支持集合</u>**

#### 特殊方法属性

**`.size`**：数组长度

**`.indexOf(X)/indexOfFirst{}  lastIndexOf(X)/indexOfLast{}`**：查找第一个/最后一个匹配元素下标

.reversed() //数组内容反转

.count() //获取数组的容量，等价于Java中的 数组.length

**`arr.withIndex ()`**： 返回 **`( index , element )`** 

**`arr.forEachIndex ()`**： 返**` {index , element -> 代码块 }`** 

```
var int_array: IntArray = intArrayOf(1，2，3)
var int array1 : Array<Int> = arrayOf(1，2，3)
var string_array: Array<String> = arrayOf("Hello"， "World"， "!")

for ((index,i) in arr.withIndex ()){
	println("角标=$index元素=$i")
}
```

```
var int_array: IntArray = intArrayOf(1，2，3)
var int array1 : Array<Int> = arrayOf(1，2，3)
var string_array: Array<String> = arrayOf("Hello"， "World"， "!")

for ((index,i) in arr.withIndex ()){
	println("角标=$index元素=$i")
}
```



### List/Map/Set

![image-20210826163457605](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826163457605.png)

list支持解构赋值

![image-20210826163620719](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826163620719.png)



![image-20210826163921701](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826163921701.png)

![image-20210826164346981](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826164346981.png)



![image-20210826164414464](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826164414464.png)

![image-20210826164514540](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826164514540.png)









### 其他常用类型

- Any：kotlin中所有类的父类，没有父类默认Any是父类。提供`equals`、`hashCode`、`toString`。运行时映射成java Object
- Unit：没有返回值。同java void，由于kotlin一切都是类/对象，为解决泛型专门做的包装
- Nothing：有返回值但没有意义(同js undefined)。配合TODO( )抛出异常
- 函数类型：**函数本身作为返回值**，即某函数返回一函数。
  - `methodName(...)：（argTypr...）->返回函数的返回值类型`
- 



## 特殊表达式

kotlin中大多数语句都是表达式，返回值自动推断，无需指定声明

### if-else表达式

与java不同，它是**表达式，返回最后一行结果**。不支持三元运算符(如python)，if---else-- 代替

```kotlin
fun maxOf(a: Int, b: Int) = if (a > b) a else b
```

### range范围

- **`start. . end step X`** :  [start,end **]** 
- **`start until end step X`** :  [start,end **)** 
- **`start downTo end step X`** :  [end, start **]** 全闭区间
- **`range.step(X)`**：修改步长
- IntRange(start,end)：[start,end ] 全闭区间

### when表达式 : 是switch的加强

针对表达式的不同值匹配不同分支，**无需显示break**(即自动break)

匹配条件为Boolean可省略括号

**else必须书写**，除非是<u>枚举类型</u>全部有匹配条件，编译器会自动检测

```kotlin
var r : XXX = when ( 表达式/语句  ) {
	匹配条件  - >  代码块
	else	  - >  	代码块
}
//匹配条件为Boolean省略括号
when {
	a >b一>print ("a大于b")
    a b一>print ("a 小于b")else -> print ("a等于b")
}
```

   

## 控制

### for

<u>循环变量无须var声明</u>

通常针对Range、Array、List、Map、Set

```
for(循环遍量 in 可循环对象){

}
for ((index,i) in arr.withIndex ()){
	println("角标=$index元素=$i")
}

```

循环遍历无需var声明

```
if (-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
}
if (list.size !in list.indices) {
    println("list size is out of valid list indices range, too")
}


for (x in 1..10 step 2) {
    print(x)
}
println()
for (x in 9 downTo 0 step 3) {
    print(x)
}

for ((k, v) in map) {
    println("$k -> $v")
}
```



只读list /map    val list = listOf("a", "b", "c")   val map = mapOf("a" to 1, "b" to 2, "c" to 3)

 *in* 运算符来判断集合内是否包含某实例：

list.size

```
var list1=listOf(元素1，元素2，元素3)    //声明List时主要是通过 listOf()实现
for((index,value) in list.withIndex()){    //重点是 withIndex() 函数，index 接收索引，value 接收对应的值
    //DO  STH 
}

var map=TreeMap<键类型,值类型>()
map[key]=value

for ((k, v) in map) {
    println("$k -> $v")
}
```

## When表达式

类似于Java中的switch，严格分支执行，无须显式break

基本使用格式：

```

var result=when(变量){
    分支A -> 表达式(要有返回值，最终将值赋给result)
    else -> 表达式(要有返回值，最终将值赋给result)
}

when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}

fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```





# 函数

Kotlin中函数可以有普通的定义方式、可以用表达式函数体、可以把Lambda赋值给变量。函数可以放置在类的外面（顶层函数）、可以放置在方法的内部（嵌套函数）、可以作为参数传递、可以作为函数的返回值。函数的功能非常强大与灵活，并且地大大提升。Kotlin中的函数就是一等公民。

## 声明语法

- `可见性修饰符  fun  methodName（ v1 : Class1 [?]）[ : 返回值 ]`
- 无返回值时：①**`Unit`** (void，为解决泛型专门做特殊包装)           ②省略不写，默认Unit类型
- 函数执行**默认接受非null实参**，函数形参明加<u>**`？`**可接受null实参</u>
- 支持**默认参数**、**具名参数**、**可变参数 `vararg`**
- 具名函数中：**单个表达式作为函数体**，**不用`{ }`**包裹，无须指定返回值(**类型自动推断**)。
- 入口函数可无参数，名字必须是`main`
- **\` \` 反引号**：函数名含**特殊字符**(空格、<u>关键字</u>)，<u>声明</u>、<u>调用</u>时反引号包裹。解决java/kotlin互调关键字冲突

```kotlin
fun main (){}
fun main(args: Array<String>) {
    println("Hello World!")
}

fun sum(a:Int , b:Int):Int=a+b
```

### 修饰符

访问修饰符：无指定**默认public**，支持private

**`inline`** 内联函数：针对lambda函数，编译器将函数体复制每个调用处

**`tailrec`**尾递归优化：优化该尾递归函数，将尾递归函数转化为while循环，无栈溢出风险

## 匿名/lambda函数

**`var`**：==**匿名函数作为变量，变量类型 ： <u>(形参类型 ...) -> 返回值类型</u>**==，由声明时指定。常用于函数式编程和闭包

- 类型定义：

  - 显式指定形参类型放于Funcation类型定义，形参名放于函数定义
    - : **`var varName:(argType...) -> 返回类型 = {varName ... -> 语句块 }`**   
    - <u>仅一个形参：可省略参数定义</u>，**`it`**是隐含参数名
    - 常用于stream/lambda运算
  - 自动推断：**`var varname=  {varName :Type ...->语句块}`**
    - 变量可自动推断*函数签名类型*时，可省略类型声明
    - 常用于闭包写法、返回函数类型

- ==<u>语句块**不用 { }再次包裹**，返回值、类型是由最后一条语句决定。不能指定return语句</u>==

- 函数体中**变量声明没有括号**

- 匿名函数变量为**参数末尾**，语句块可写在外面。类似**<u>groovy闭包</u>**

  ```kotlin
  var j:(Int,Int) ->Int={x,y -> x+y}		//常用于stream/lambda运算
  var i={x : Int , y : Int ->x+y} 		//常用于闭包写法、返回函数类型
  ```

### `inline`函数内联

**`inline`** 内联：kotlin编译器提供针对lambda对象产生额外内存开销的问题，提供特殊优化机制。类似C的宏替换 。编译器将匿名函数体复制粘贴到每个函数调用处。

函数参数也会随之内联，参数有Lambda表达式时，Lambda表达式便不是一个函数的对象，从而也就无法当作参数来传递。使用`noline`修饰参数，禁止内联

不用内联复杂函数

**lambda递归函数不能内联**，宏替换无限循环，编译会发出警告

### 函数引用

**`：：具名`**(双冒号)： 具名函数----->实参，进行函数式编程。

## 特殊函数

**`TODO( " reson " )`**：抛出异常 `NotImplementedError`,终止运行，返回`Nothing`+

**`?.let ( 单参有返回值函数 )`**：应用于任何类型。源对象！=null则执行该函数，源对象==null不执行该函数

先决条件函数：kotlin内置的函数，类似assert，可校验关键逻辑  require/requireNotNull/error等

forEach/forEachIndexed：（element） /  ( index , element )

```kotlin
public inline fun TOD0(reason: String): Nothing = throw NotImplementedError

str=str?.let{ 
	if(it.isNotBlank ()){
		it.capitalize()
    }else{
		"butterf1y"
	}
}

```

## 高阶函数

在Standard类中的方法都可以通过`return@方法名`这种格式结束当前方法

![image-20210826170959977](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826170959977.png)

.apply  看作一个配置函数，<u>传入一个接收者</u>，然后调用一系列函数来配置它以便使用，提供lambda给apply函数执行，它会<u>返回配置处理好的接收者</u>。this隐式为调用者。传入的匿名函数没有形参，没有it

![image-20210826171859604](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826171859604.png)

```kotlin
val file2 = File ( "E://i have a dream_copy.txt").apply{
	setReadable (true)
	setWitable (true)
	setExecutable (false)
}
```

.let   闯入一个形参叫用着本身，it引用。null不执行

![image-20210826172720537](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826172720537.png)

.run

![image-20210826174140870](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826174140870.png)

支持执行函数引用    ::具名函数名

.with

![image-20210826174318499](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826174318499.png)



.also





# 类

成员访问修饰符：无指定**默认public**，支持private

类本身、类的成员函数默认`final`禁止继承/冲刺额，`open`开放继承/重写

## 构造函数 `constructor`

- **一个主**构造函数：
  - 位于**<u>类声明后</u>** `class 类名 [ constructor ] ([形参:Type...]){}`。没有其他修饰符可省略constructor。
  - **<u>未显式声明则默认生成 无参主构造函数</u>**
  - 主构造函数有参数时：**`init{ }`**代码块就是**函数体**。支持**`this`**
- **多个次**构造函数
  - 位于**<u>类体内</u>**
  - ==**必须调用主构造函数或其他次构造函数**==：`constructor ([形参:Type...]):this（参数列表)`
  - 声明的<u>新次构造函数</u>调用<u>已有的构造函数`:this(参数...)`</u>：
    - 编译器自动**按参数书写顺序赋值、调用已有构造函数**
    - 参数**顺序**必须**一致**。
    - **新的参数个数 >  已有的参数个数**
    - kotlin次构造函数会**调用涉及到的所有构造函数**，从==**主-->[次..]-->当前调用**==。不同于java 构造函数仅调用一次
- 声明构造函数时：**参数没有var修饰**

```kotlin
class 类名 constructor([形参1，形参2，形参3]){}
class 类名 ([形参：Type...]){} //无参数仍保留括号
class Workers constructor (name: String){
	var name: String
	init {
		this. name = name
		println("我叫$ iname} ")
	}
	constructor(name: String，age: Int) : this (name) {
		println("我叫$ iname}，我今年$ {age}岁。")
	}
	//新次构造参数个数 > 调用的参数个数
	constructor (name: String, age:Int，sex: String) : this(name,age)){
		println("我叫${name}，我今年$ {age}岁，我是${sex}")
	}
}
fun main(args: Array<String>){
	var pseron = Workers ("江小白"，18，"男")
}
```

## 继承

类本身、类的成员函数默认`final`禁止继承/冲刺额，`open`开放继承/重写。**类单继承，接口多实现**

`class 子类名：父类(构造函数) /接口名{ }`

`open/override`：子类中重写的==**属性、方法**==需要override修饰符。父方法默认final，需要加open修饰符

`super`：子类中访问父类成员

**`Any`**根类：kotlin中所有类的父类，没有父类默认Any是父类。提供`equals`、`hashCode`、`toString`。运行时映射成java Object

`abstract`：支持抽象类、抽象方法

实现接口`interface`只写接口名，可实现多个接口(逗号分隔)

## kotlin嵌套类 / inner内部类

嵌套类：无修饰符，**不能访问外部类**

内部类：`inner`修饰，**可以访问外部类**。同java <u>成员内部类</u>

## 特殊类

### enum枚举类

主构造函数参数声明由val

每个枚举常量仅一个实例，单例

### sealed密封类

限制取值，只能取声明的值。不同于枚举类，密封类子类可有多个实例

private构造函数

密封类适用于子类可数的情况，而枚举类适用于实例可数的情况。

由于密封类的子类可以不在该密封类中，但是必须与密封类在同一个件中

密封类的间接继承子类可以声明在其他文件中。

### data数据类

用于model、entity，保存数据

`data class  ([形参:Type...])`

主构造函数至少有一个参数，如果需要一个无参的构造函数，可以将构造函数中的参数都设置为默认值。
主构造函数中传递的参数必须用val或var来修饰。

不可用abstract、open、sealed或inner修饰。

所有属性getter/setter

自动生成equals()、hashCode()、toString()、componentN(、copy(等，这些方法也可以进行自定义。

### object 单例模式

有且只有一个实例

直接通过“类名.成员名”的形式调用类中的属性与函数，不需要创建该类的实例对象，这是因为通过object关键字创建单例类时，默认创建了该类的单例对象，因此在调用该类中的属性和函数时，不需要重新创建该类的实例对象。

### companion伴生对象

由于在Kotlin中没有静态变量，因此它使用了伴生对象来替代Java中的静态变量的作用。伴生对象是在类加载时初始化，生命周期与该类的生命周期一致。在Kotlin中，定义伴生对象是通过“companion"关键字标识的，由于每个类中有且仅有一个伴生对象，因此也可以不指定伴生对象的名称，并且其他对象可以共享生对象。伴生对象的语法格式如下:



![image-20210826160356162](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826160356162.png)

![image-20210826160444146](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826160444146.png)

## by委托

### 类委托

一个是委托类，一个是被委托类。在委托类中并没有真正的功能方法，该类的功能是通过调用被委托类中的方法实现的

### 属性委托

一个类的某个属性值不是在类中直接进行定义，而是将其委托给一个代理类，从而实现对该类的属性进行统一管理。

![image-20210826160839700](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210826160839700.png)

## 延迟加载

by lazy

### 类元数据

obj.javaClass：运行时对象类型

![image-20210824110201519](C:\Users\LiuJH\AppData\Roaming\Typora\typora-user-images\image-20210824110201519.png)

![image-20210824110306684](C:\Users\LiuJH\AppData\Roaming\Typora\typora-user-images\image-20210824110306684.png)

![image-20210824180256646](C:/Users/LiuJH/AppData/Roaming/Typora/typora-user-images/image-20210824180256646.png)



# 泛型



# 包级函数

不在类内，文件顶层

# 流操作

# 协程