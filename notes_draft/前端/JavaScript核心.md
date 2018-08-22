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
