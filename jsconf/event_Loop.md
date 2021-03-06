今天看到一个不错的视频是讲述js的[event-loop](https://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html)索性记录下来，方便下次查看。
 
 下面先看第一张图(js的 Runtime)
  ![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/001.png)
 
- 这里`heap`的作用是分配内存大小,而`stack`是处理上下文环境的地方,也就是执行代码程序的地方.
 
- 后来发现`setTimeout `或者`HTTP request`竟然没有在`stack`中执行,这里要研究一下喽。
 
 再来第二张图解释这个疑问

  ![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/002.png)
- 后来发现还有一些东西是代码运行所需要的(例如called web APIs, event loop 和 callback queue),而这些是由浏览器提供的。
- 而上面`setTimeout`, `HTTP`函数是则是在web APIs中运行的,这些函数的回调放到`callback queue`里面,通过event Loop 把在`callback queue`中的第一个取出来然后放到`stack`中执行。

---
现在是仔细分析一下

再来看第三张图 

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/003.png)

`one thread === one call stack === one thing at a time`

单线程  等于  只有一个调用栈  等于  同一时间只能做一件事情。

所以结论为:单线程的意思是同一时间只做一件事。

那么我们看第四张图来说明情况

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/004.png)

- 运行这个函数我们要牢记调用栈的规律:如果我们运行这个函数,这个函数进栈,
如果从函数中返回,该函数出栈。

- 运行函数先是`main()`进栈,`printSquare(4)`得到调用进栈,再调用`square(n)`函数，进栈;
最后`multiply(n, n)`进栈,最后按照顺序依次出栈。

1. 此外我们在浏览器开发中可能碰到过这样的代码

上图第五张图

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/005.png)


```
function foo() {
    throw new Error('Oops');
}

function bar() {
    foo();
}

function baz() {
    bar();
}

baz()

```
这里的输出结果是

```
/usr/local/bin/node test.js
/Users/zhangwenning/git/jfjun-cw/test.js:5
    throw new Error('Oops');
    ^

Error: Oops
    at foo (/Users/zhangwenning/git/jfjun-cw/test.js:5:11)
    at bar (/Users/zhangwenning/git/jfjun-cw/test.js:9:5)
    at baz (/Users/zhangwenning/git/jfjun-cw/test.js:13:5)
    at Object.<anonymous> (/Users/zhangwenning/git/jfjun-cw/test.js:16:1)
    
```
从而验证了上面函数进栈和出栈的顺序操作。

2. 在代码开发中可能遇到这样的问题 `RangeError: Maximum call stack size exceeded`

直接上图六

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/006.png)
还有一个例子:

```
function foo() {
    return foo();
}

foo();
```

通过`调用栈`出现问题,引出是什么原因导致程序变慢,奥,原来是`阻塞`。

上图片七

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/007.png)



这里引出`阻塞`的概念，我们谈论`阻塞`的概念,其实`阻塞`没有特定`概念`，

其实`阻塞`我们就是程序运行的慢。


引出图八

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/008.png)

这里有网络请求，并且是同步操作，所以执行的慢了，所以我们定义为`阻塞`了。

出现`阻塞`不可怕,但要有解决方案,而这里的解决方案就是`异步调用`。

看图九

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/009.png)

We log JSConfEU, clear, five seconds later somehow magically "there" appears on the stack

这张图反映了并发事件的由来。

然而开始说过js在同一个时间只能做一件事。

我们可以同时做多件事的原因是:浏览器不仅仅一个执行环境。

JavaScript Runtime 同一时间只能做一件事情,但是浏览器提供了一些API是，这些APIs是高效的线程,因而可以实现并发性。

这张图与node也是类似的,这里的web API是由c++API,而多线程是通过c++隐藏的。
  
I'm a single threaded single concurrent language  ‑‑ right. yeah, cool, I have a call stack, an event loop, a callback queue, and some other APIs and stuff
  
这里说明了js是单线程单进程的。有一个调用stack, 一个事件循环,一个回调函数队列和其他的api。

例子二,setTime(function(){}, 0);

上图

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/010.png)

设置定时器为0的含义是在`webapis`中不用等待处理了,webapis直接把回调函数推到`task queue`,

但是`event loop`还是要在`stack`执行完成所有的函数才会把`task queue`的第一个踢到`task`中

去。

例子三,异步请求的也是类似的

上图

![image](https://github.com/WenNingZhang/learnNode/blob/master/jsconf/event_Loop/011.png)

以上是js在浏览器中运行的原理,其实node与之也是类似的,

无非就是把webapis换成c++,用c++隐藏下面的多线程。

这里有一个例子是判断`setTimeout(fn, 0)`和`process.nextTick`的区别

```js
 setTimeout(function timeout() {
    console.log('TIMEOUT FIRED');
}, 0);


process.nextTick(function A() {
    console.log(1);
    process.nextTick(function B(){console.log(2);});
});

```
process.nextTick方法可以在当前"执行栈"的尾部----Event 
Loop（主线程读取"任务队列",或者主线程读取`callback queue`）之前----
触发回调函数。也就是说，它指定的任务总是发生在所有异步任务之前。

因此执行结果为:
```js
1
2
TIMEOUT FIRED
```








