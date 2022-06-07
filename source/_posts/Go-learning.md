---
title: 非Go程序员的Golang知识点总结
date: 2022-06-07 17:49:04
tags: [golang]
categories: Go
description: Golang learning
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1654595986/golang_cwzyke.jpg
---

由于项目上有Golang需求， 虽然照猫画虎的构建一个简单restful service不算难，不过熟悉一下Golang的基本知识还是很有必要。
这里记录下在学习过程中认为有价值的points， 以供日后查看。

<!--more-->

### Variables

- 变量首字母小写只有包可见(unexported), 大写则可被包外引用(exported)
- 声明变量未赋值会使用默认值
- 可以使用`var()`声明代码块来声明多个变量, 增加可读性

### Constants

- const变量申明是即需赋值
- const()代码块可声明多个变量
- 一般const变量使用全大写

### Data types

- 可以使用:=省去写var


### Convert Types

- strconv提供了从string到其他类型的转换, ParseXXX
- 同样也提供了从其他类型转string的方法, FormatXXX

### Switch

- 没有break, 默认break, 可以加fallthrough跳到下一级
- case支持条件表达式

### For loops

- `for k,v := range <map>`用来循环map
- `for k := range <map>`用来循环map的key
- `- for _,v := range <map>`用来循环map的value

### Panic & Recovery

- Go不能抛异常
- 如果发生Panic, 当前函数退出, defer继续执行, 调用该函数所在的协程会执行完其他方法直到程序崩溃
- 虽然go不能抛异常, 但可以返回结果+错误, 让调用者进行处理


### Array

- `[5]int`, `[...]string` 可以指定size或者不指定, 但是`[...]`赋值后确定size
- `[5]int{1: 10, 3: 20}` 可以给固定的位置赋值
- loop办法除了直接for还可以用跟map一样的range, 只是key是index
- `array[2:5]`可以filter index, 左闭右开

### Slice

- slice是的动态数组
- `make([]int, 1, 5)`可以生成一个初始size是1, capacity是5的slice
- 如果指定capacity, 则跟size一样
- 也可以直接使用`[]int{1, 2, 3}`来生成一个slice
- 还可以使用`new([50]int)[0:10]`, 创建一个size是10, capacity是50的slice, 这里的0:10可以是9:10, 则size为1
- 添加使用append并赋值给自己, `slice = append(slice, "1")`
- slice可以使用`slice1 := slice[start:end:cap]`进行切割, `length = end - start`, `cap = cap - start`


### Map

- 可以通过make来创建一个map, 也可以清空一个map, `make(map[string]int)`
- 也可以for range通过`delete(<map>, key)`来删除
- 对map的key或者value进行排序, 需要把值拿出到slice里, 通过sorts
- merge map需要循环添加


### Struct

- 初始化可以通过new(StructName), 或者直接构造
- new返回指针, 直接构造(struct literal, 即花括号)返回对象, 加&则也返回指针
- struct可以嵌套, 通过构造嵌套生成
- 如果struct成员做json字段, 可以通过Tag来重命名, `json:"name"`, 放在类型后面就行, 这里的``在go里表示raw文本
- 可以类似扩展的方法给struct加方法, `func (x XX) Info() string {}`
- 如果receiver的类型是指针, 则可赋值给struct对象, `func (x *XX) Info { x.name = "Jack" }`
- struct可以使用等号判断是否一样, 但是都是`值`比较


### Interface

- 可以定义struct或者简单类型, 如`type X int`, 来实现接口
- 如果接口方法全部实现, 则可以说该类型实现了该接口
- 一个定义的类型可以同时实现多个接口
- golang里分为具体类型(concrete type)与接口类型(interface type)
    - 具体类型包括ints, strings, arrays, slices, maps, pointers
    - 接口类型interface{}是一个指向空接口的指针
    - *interface{}是指向空接口的指针的指针, 是一个实体类型

### Ad hoc polymorphism

- 类似于接口的继承, interface里包含了其他的interface, 形成embedded

### goroutine

- goroutine的调度使用了GPM模式, 即协程, 处理器, 线程
- Processor包含了Goroutine的队列, 获取G必须先获取P
- G有一个全局的队列, 还有很多承载G队列的本地P列表, 最大个数通过GOMAXPROCS配置
- M默认个数为10000, 但实际上内核不一定支持这么大
- P的数量在启动的时候就创建好了
- 新建G时先放入本地P队列, 如果满了, 则分一半放入全局队列
- M如果自己有任务, 需获取P, 再从P中获取M, 如果没有空闲M, 则从全局或者其他P队列中偷
- 当线程无G可用, 会去其他线程绑定的P中偷G (working stealing)
- 当线程因为G阻塞, 则会把P转移给其他空闲M, 来执行其他的G, 如果没空闲的M, 则创建新的M (hand off)

### Channel

- channel只能通过`make(chan <Type>, <buffer size>)`创建
- 通过`<-`读写数据, 如 `xx <- "Hello"`, `yy := <-xx`
- unbuffered的channel, 发送跟接收都ready才会开始, 否则send或者receive就会等待
- buffered的channel如果没buffer则send会block, 如果没数据了receive会block
- `yy, result := <-xx`, result为false表示channel空了且empty, 通过`close(xx)`来在发送完关闭channel
- 通过select + case可以从channel读或者写东西, 如果没读到, 或这channel buffer已满无法写入, 则可以在default处理

### Const

- `const {}`可以定义一系列常量, 第一个标注itoa则剩下的自增
- 这样的const可以用来做Key来初始化map

### Concurrency

- 使用`sync.WaitGroup#Add()`加信号等待, `sync.WaitGroup#Done()`消信号
- 互斥锁`sync.Mutex`

### Pointer or not

- 对于channel, map, function, 因为他们的类型本身的size跟pointer一样, 切实际保存的就是指向数据的地址 所以没必要使用指针
- interface的大小一个是2 *pointer, 第一个pinter指向真实类型, 第二个pointer指向数据
- slice是3*pointer大小, 需要指向切片的首, 尾, 以及最后可以位 (reserved区域存放capacity之类的数据)
- 使用interface{}来吃所有类型
- 当值实现接口, 即`func (x X) xxx { }`, 则可以通过值赋值接口, 也可以通过指针赋值接口, 只是值赋值时有个拷贝, 再取指针
- 当指针实现接口, 即`func (x *X) xxx {}`, 则只能通过指针赋值
- 如果实现interface的是指针, 则参数不必定义为接口的指针, 因为拷贝的也是指针, 如果实现接口的是值, 则可定义指针, 避免值拷贝再取地址

### Log

- init函数会在main之前被调用
- log的初始化可以放在init里, 如SetPrefix, SetFlags
- 常用的flag有, `log.Ldate, log.Lmicroseconds, log.Lshortfile`等


### Some libs

- [struct validator: validator](github.com/go-playground/validator)
- [dynamic json parser](github.com/Jeffail/gabs)
- [statistics tool package: stats](github.com/montanaflynn/stats)
- [slice helper functions:  go-funk](github.com/thoas/go-funk)
- [html parser: goquery](github.com/PuerkitoBio/goquery)
- [common regex](github.com/mingrammer/commonregex)
- [image processing: imaging](github.com/disintegration/imaging)
- [chart generate: go-chart](github.com/wcharczuk/go-chart)
- [dynamic xml parser: xmlquery](github.com/antchfx/xmlquery)
- [datetime tools: now](github.com/jinzhu/now)


### Famous web framework

- [gin, 6.6k](https://github.com/gin-gonic/gin)
- [beego, 5.5k](https://github.com/beego/beego)
- [iris, 2.4k](https://github.com/kataras/iris)
- [echo, 2k](https://github.com/labstack/echo)
- [Fiber/fasthttp, 1.5k](https://github.com/valyala/fasthttp)

