## 示例代码
```nodejs
var http = require('http');

http.createServer(function(req, res) {
    res.writeHead(200, {
        'Content-Type': 'text/html'
    });
    res.write('<h1>Node.js</h1>');
    res.end('<p>Hello World, Zhaochenxi</p>');
}).listen(3000);
console.log("HTTP server is listening at port 3000.");
```

## 异步式IO
### 阻塞和线程
什么是阻塞（block）呢？线程在执行中如果遇到磁盘读写或网络通信（统称为I/O操作），通常要耗费较长的时间，这时操作系统会剥夺这个线程的CPU控制权，使其暂停执行，同时将资源让给其他的工作线程，这种线程调度方式称为**阻塞**。当I/O操作完毕时，操作系统将这个线程的阻塞状态解除，恢复其对CPU的控制权，令其继续执行。这种I/O模式就是通常的**同步式I/O（Synchronous I/O）**或**阻塞式I/O（Blocking I/O）**。

相应地，**异步式I/O（Asynchronous I/O）**或**非阻塞式I/O（Non-blocking I/O）**则针对所有I/O操作不采用阻塞的策略。当线程遇到I/O操作时，不会以阻塞的方式等待I/O操作的完成或数据的返回，而只是将I/O请求发送给操作系统，继续执行下一条语句。当操作系统完成I/O操作时，以事件的形式通知执行I/O操作的线程，线程会在特定时候处理这个事件。为了处理异步I/O，线程必须有事件循环，不断地检查有没有未处理的事件，依次予以处理。

### 回调
```nodejs
//**readfile.js**
function readFileCallBack(err, data) {
    if (err) {
        console.error(err);
    } else {
        console.log(data);
    }
}
var fs = require('fs');
fs.readFile('file.txt', 'utf-8', readFileCallBack);
console.log('end.');

//**readfilesync.js**
var fs = require('fs');
var data = fs.readFileSync('file.txt', 'utf-8');
console.log(data);
console.log('end.');

```

fs.readFile调用时所做的工作只是将异步式IO请求发送给了操作系统，然后立即返回并执行后面的语句，执行完以后进入事件循环监听事件。当fs接收到I/O请求完成的事件时，事件循环会主动调用回调函数以完成后续工作。

>ps.Node.js中，并不是所有的API都提供了同步和异步版本。Node.js不鼓励使用同步I/O。

### 同步式 I/O 和异步式 I/O 的特点

| 同步式 I/O（阻塞式） | 异步式 I/O（非阻塞式） |
| --- | --- |
| 利用多线程提供吞吐量 | 单线程即可实现高吞吐量 |
| 通过事件片分割和线程调度利用多核CPU | 通过功能划分利用多核CPU |
| 需要由操作系统调度多线程使用多核 CPU | 可以将单进程绑定到单核 CPU |
| 难以充分利用 CPU 资源 | 可以充分利用 CPU 资源 |
| 内存轨迹大，数据局部性弱 | 内存轨迹小，数据局部性强 |
| 符合线性的编程思维 | 不符合传统编程思维 |


## 模块
一个Node.js文件就是一个模块，这个文件可能是JavaScript代码、JSON或者编译过的C/C++扩展。
```nodejs
//module.js
var name;
exports.setName = function(thyname) {
    name = thyname;
}
exports.sayHello = function() {
    console.log('Hello ' + name);
}

//getmodule.js
var mymodule = require('./module');
mymodule.setName('Tracy');

var mymodule2 = require('./module');
mymodule2.setName('Tracy 2');

mymodule.sayHello();

//输出为Tracy 2 （单次加载）
```

```nodejs
//hello.js
function Hello() {
    var name;
    this.setName = function(thyName) {
        name = thyName;
    };
    this.sayHello = function() {
        console.log('Hello ' + name);
    };
};
module.exports = Hello;

//gethello.js
var Hello = require('./hello');
hello = new Hello();
hello.setName('BYVoid');
hello.sayHello();
```

### 包的安装和生成























