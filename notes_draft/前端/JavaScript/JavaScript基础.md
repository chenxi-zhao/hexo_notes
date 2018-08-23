## 作用域
传统的类C语言,它们的作用域是block-level scope，块级作用域，花括号就是一个作用域。但是对于JavaScript而言，它的作用域是function-level scope，比如if条件/for条件语句中的变量在整个函数域内都有效。
```javascript
for (var i = 0; i < helpText.length; i++) {
    var item = 1;
}
// 等价于
var item = {};
for (var i = 0; i < helpText.length; i++) {
    item = 1;
}
```

### 作用域提升
#### 变量提升
>JavaScript中所有函数声明和变量声明都会提升到前面，变量声明在前面，赋值在生命过程之后。

```javascript
var x = 10;
function x(){};
console.log(x); // 10

// 实际上解释成
var x;
function x(){};
x = 10;
console.log(x); // 10
```

#### 函数提升
函数的声明方式主要由两种：声明式和变量式。

>声明式会自动将声明放在前面并且执行赋值过程。而变量式则是先将声明提升，然后到赋值处再执行赋值。

```javascript
function test() {
    foo(); // TypeError "foo is not a function"
    bar(); // "this will run!"
    var foo = function () { // function expression assigned to local variable 'foo'
        alert("this won't run!");
    }
    function bar() { // function declaration, given the name 'bar'
        alert("this will run!");
    }
}
test();
// 实际上解释成
function test() {
    var foo;
    var bar;
    bar = function () { // function declaration, given the name 'bar'
        alert("this will run!");
    }

    foo(); // TypeError "foo is not a function"
    bar(); // "this will run!"

    foo = function () { // function expression assigned to local variable 'foo'
        alert("this won't run!");
    }
}
test();

// 要注意的地方：带有命名的函数变量式声明，是不会提升到作用域范围
var baz = function spam() {};
baz(); // vaild
spam(); // ReferenceError "spam is not defined"
```


## 变量对象
### 基本类型(primitive types)
五种基本类型：
 - number
 - string
 - boolean
 - undefined
 - null

五种基本类型保存在内存中的栈中,大小固定,复制其变量时会创建这个值的一个副本。使用typeof区分。这些值是在底层上直接实现的，他们不是object，所以没有原型，没有构造函数。

### 引用类型
引用类型的值是对象,保存在堆内存中。引用类型的变量实际上是一个指针,它保存在栈中,指向堆内存中的对象,复制引用类型变量实际是复制该指针,所以他们都指向同一个对象,用instanceof确定一个值是哪种引用类型。

### 特殊包装类
 - Boolean
 - String
 - Number

```javascript
var c = new Boolean(true);
var d = new String('test');
var e = new Number(10);

// 转换成原始值
// 使用不带new关键字的函数
с = Boolean(c);
d = String(d);
e = Number(e);

// 重新转换成对象
с = Object(c);
d = Object(d);
e = Object(e);
```

### JavaScript原生对象及扩展
[JavaScript原生对象及扩展](https://segmentfault.com/a/1190000002634958)


### typeof

```text
Value               Class      Type
-------------------------------------
"foo"               String     string
1.2                 Number     number
true                Boolean    boolean
undefined           Undefined  undefined
new Function("")    Function   function
new String("foo")   String     object
new Number(1.2)     Number     object
new Boolean(true)   Boolean    object
new Date()          Date       object
new Error()         Error      object
[1,2,3]             Array      object
new Array(1, 2, 3)  Array      object
/abc/g              RegExp     object (function in Nitro/V8)
new RegExp("meow")  RegExp     object (function in Nitro/V8)
{}                  Object     object
new Object()        Object     object
```
返回值的类型只有六种：string/number/boolean/undefined/function/object

>instanceof操作符用来比较两个操作数的构造函数。只有在比较自定义的对象时才有意义。如果用来比较内置类型，将会和typeof操作符一样用处不大。

### Object.prototype.toString
因为这是唯一一个可依赖的方式。typeof 的一些返回值在标准文档中并未定义， 因此不同的引擎实现可能不同。
```javascript
Object.prototype.toString.call([])    // "[object Array]"
Object.prototype.toString.call({})    // "[object Object]"
Object.prototype.toString.call(2)    // "[object Number]"
```

### undefined 和 null 的区别
常见强类型语言中，一般有一个表示“空”的值，比如NULL，但是在JavaScript中，null和undefined都可表示空值，js中除了这两个值意外其他的都是对象（基本类型都有对应的包装类）。
>typeof null返回的是object，这是一个一直未修复的bug。

#### 相似之处
都是完全不可变的，没有属性和方法，也不能给其属性赋值。事实上,试图访问或定义一个属性将会引发一个类型错误（TypeError）。正如他们的名字暗示的那样，他们是完全无效的值。

#### 不同之处
一个重要的区别，服务于不同的目的和理由。区分这两个值，你可以认为undefined代表一个意想不到的没有值而null作为预期没有值的代表。
使用Object.prototype.toString.call()形式可以具体打印类型。

#### undefined
undefined实际上代表了不存在的值（non-existence of a value）。

有许多的方法产生一个undefined值的代码。它通常遇到当试图访问一个不存在的值时。在这种情况下，在JavaScript这种动态的弱类型语言中，只会默认返回一个undefined值，而不是上升为一个错误:
 - 任何声明变量时没有提供一个初始值，都会有一个为undefined的默认值
 - 当试图访问一个不存在的对象属性或数组项时，返回一个undefined值
 - 如果省略了函数的返回语句,返回undefined
 - 函数调用时未提供的值结果将为undefined参数值
 - void操作符也可以返回一个undefined值。像Underscore的库使用它作为一个防御式的类型检查，因为它是不可变的，可以在任何上下文依赖返回undefined
 - undefined是一个预定义的全局变量(不像null关键字)初始化为undefined值

#### null
通常用作一个空引用一个空对象的预期,就像一个占位符。typeof的这种行为已经被确认为一个错误，虽然提出了修正，出于后兼容的目的，这一点已经保持不变。 这就是为什么JavaScript环境从来没有设置一个值为null；它必须以编程方式完成。

使用null的情况:
 - DOM，它是独立于语言的，不属于ECMAScript规范的范围。因为它是一个外部API，试图获取一个不存在的元素返回一个null值，而不是undefined。
 - 如果你需要给一个变量或属性指定一个不变值，将它传递给一个函数，或者从一个函数返回null，null几乎总是最好的选择。
 - JavaScript使用undefined并且程序员应该使用null。
 - 通过分配null值，有效地清除引用，并假设对象没有引用其他代码，指定垃圾收集，确保回收内存。

#### Object.prototype.toString调用过程
 - 如果值是undefined，返回“[object Undefined]”。
 - 如果这个值为null，则返回“[object Null]”。
 - 让O作为调用ToObject同时传递this值作为参数的结果值。
 - 让class是O的内部属性[[Class]]的值。
 - 返回的结果连接三个字符串“[object ”，class，和“]”的结果的字符串值。


## 函数