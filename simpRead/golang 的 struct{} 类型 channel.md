> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/show58/p/12655396.html)

golang 的 struct{} 类型 channel

struct{} 是结构体类型的代表；
===================

struct{}{} 是结构体的值，并且值为空的代表
==========================

之前看代码的时候发现有如下定义的 channel，就觉得很诧异

```
var ch chan struct{}


```

这其中，struct{} 是个什么鬼。

实际上 struct{} 就是一种普通数据类型，只是没有具体的值而已。

常用用法
====

通常 struct{} 类型 channel 的用法是使用同步，一般不需要往 channel 里面写数据，只有读等待，而读等待会在 channel 被关闭的时候返回。

```
package main

import (
    "time"
    "log"
)

var ch chan struct{} = make(chan struct{})

func foo() {
    
    log.Println("foo() 111");
    time.Sleep(5 * time.Second)
    log.Println("foo() 222");
    close(ch)
    log.Println("foo() 333");
}

func main() {
    
    log.Println("main() 111");
    go foo()
    log.Println("main() 222");
    <-ch
    log.Println("main() 333");
}


```

运行结果为

```
2018/04/12 06:46:33 main() 111
2018/04/12 06:46:33 main() 222
2018/04/12 06:46:33 foo() 111
2018/04/12 06:46:38 foo() 222
2018/04/12 06:46:38 foo() 333
2018/04/12 06:46:38 main() 333


```

在 main 函数里面 ch 读操作一直等待 foo 调用 close(ch) 才返回。

注意啊，channel 对象一定要 make 出来才能使用。

往 chann struct{} 写入数据
=====================

另一个问题，我们能不能往 struct{} 类型的 channel 里面写数据呢，答案当然也是可以的。

```
package main

import (
    "time"
    "log"
)

var ch chan struct{} = make(chan struct{})

func foo() {
    ch <- struct{}{}
    log.Println("foo() 111");
    time.Sleep(5 * time.Second)
    log.Println("foo() 222");
    close(ch)
    log.Println("foo() 333");
}

func main() {
    
    log.Println("main() 111");
    go foo()
    log.Println("main() 222");
    <-ch
    log.Println("main() 333");
}


```

在 foo() 入口处给 ch 赋了一个值  
注意写法是 "struct{}{}"，第一个 "{}" 对表示类型，第二个 "{}" 对表示一个类型对象实例。

运行结果：

```
2018/04/12 06:50:16 main() 111
2018/04/12 06:50:16 main() 222
2018/04/12 06:50:16 foo() 111
2018/04/12 06:50:16 main() 333


```

由于在 foo() 启动的时候往 ch 里面写入了一个对象，所以在 main() 函数里面不需要等待 close(ch) 就能拿到一个值，因此 main() 函数可以马上退出，不需要等到 foo() 的 Sleep() 完成。

带缓冲的 chan struct{} 数据读写
=======================

另外也可以定义带缓冲的 channel

```
package main

import (
    "time"
    "log"
)

var ch chan struct{} = make(chan struct{}, 2)

func foo() {
    ch <- struct{}{}
    log.Println("foo() 000");
    ch <- struct{}{}
    log.Println("foo() 111");
    time.Sleep(5 * time.Second)
    log.Println("foo() 222");
    close(ch)
    log.Println("foo() 333");
}

func main() {
    var b struct{}
 
    log.Println("main() 111");
    go foo()
    log.Println("main() 222");
    a := <-ch
    log.Println("main() 333", a);
    b  = <-ch
    log.Println("main() 444", b);
    c := <-ch
    log.Println("main() 555", c);
}


```

运行结果为

```
2018/04/12 06:58:06 main() 111
2018/04/12 06:58:06 main() 222
2018/04/12 06:58:06 foo() 000
2018/04/12 06:58:06 foo() 111
2018/04/12 06:58:06 main() 333 {}
2018/04/12 06:58:06 main() 444 {}
2018/04/12 06:58:11 foo() 222
2018/04/12 06:58:11 foo() 333
2018/04/12 06:58:11 main() 555 {}


```

带两个缓冲大小的 channel；  
另外我们可以看到，其实也可以从 channel 里面读出数据来，但是这种数据显然没有实际意义。

作者：CodingCode

链接：https://www.jianshu.com/p/7f45d7989f3a

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。