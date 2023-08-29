---
title: Rust Basic
date: 2023-08-29 16:37:46
tags: [Rust]
categories: Rust
description: Rust basic concept
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1693298600/rust_gyjkfr.webp
---

学习总结了一些Rust基础知识, 不包括Thread与Asyc Program, 后面如果有涉猎, 会单独出来.

<!--more-->

- Introduction
    - 无运行时, 无垃圾回收
    - No null and exception
    - 更先进的包管理 cargo
    - 没有资源竞争
    - rust的包叫crate, 在crates.io
- Memory
    - stack, heap, smart pointer, pointer
    - 使用GDB去debug, 需要添加, [Ref](https://doc.rust-lang.org/cargo/reference/profiles.html#split-debuginfo)
        ```    
        [profile.dev] 
        split-debuginfo = "packed"
        ``` 
    - mac使用gdb有限制, 无法run, 切换lldb后可行
    - [commands map](https://lldb.llvm.org/use/map.html)
    - 需要sudo启动
- Variables
    - Booleans, Characters, Integers, Floats
    - Integer
        - u8 i8, u16 i16, u32 i32, u64 i64, u128 i128
        - usize isize 64bit or 32bit
    - Float
        - f32 f64
    - Booleans
        - bool (1 byte) 
    - Char
        - 4 bytes
        - full unicode value
- Functions
    - 使用snake case来命名, 即小写加下划线
    - 函数最后一行不加`;`可以直接做返回值
    - 如果需要fast return可继续使用`return`
    - 函数调用加`!`是一个宏调用
        - `println!`
        - `dbg!`
    - 可以使用cargo expand来查看宏
- Mutability
    - let与let mut
    - 自动类型推断
- Standard Library
    - https://doc.rust-lang.org/std/
- Ownership
    - 每一个变量自己持有自己
    - 跳出作用于就自己析构(drop function)
        - RAII resource acquisition is initialization 
    - 同一时间只能有一个owner
        - 指针类型如果copy, 之前的指针就无法使用, 会编译错误
        - 作为形参传入函数也会pass ownership, 编译会报错
        - 使用引用`&`((ampersand))做参数就可以Borrow Ownership, 如果需要对这个参数进行操作, 需要`mut &`
        - 同一个指针对象的Mutable borrow只能有一个
        - 这样就能在编译的时候消除资源竞争
        - 编译器自动检测immutable borrow中间不能加入mutable borrow
        - 可以有多个immutable的引用, 但是只能有一个mutable的引用
- Result
    - 封装了成功与失败的结果
    - unwrap会直接获取成功, 如果失败程序会崩溃
    - 在编译器会检测是否正确处理了异常
    - enum Ok / Err
- Option
    - 包装可以为空的类型Option<XXX>
    - enum Some / None
    - 可以通过`ok_or`转换为Result
    - 可以通过`if let Some(i) = XXX`从中获取值
- Structs
    - struct构造函数使用`::`来初始化
    - 构造函数返回类型与初始化可以直接使用`Self`关键字来代替类名
    - `impl Name {}`来实现function
    - `&self`作为函数参数来获取类的指针, 需要是引用, 否则就borrow了
    - 第一个参数是`&self`是类方法, 否则就是静态方法
- Strings
    - parse可以根据类型智能转换
    - `&str`是immutable string slice类型, 如果数据不需要改变， 可以使用slice取代String，避免Heap的拷贝，节省空间
    - `&string[x..y]`可以对borrow的string进行slice
    - String会在堆上开辟空间, 并且可以自动伸缩
    - 如果不想copy, 则需要使用`&str`来做slice指向需要的区域
    - 定义的静态字符串就是`&str`
    - rust使用utf8编码, slice必须包含整个字符, 但是下标只能是byte级别, 如果slice到未完整的字符, 会报`panicked character boundary`错误
- Enums
    - 默认枚举实际内存中是数字从0开始
    - 如果显式指定数字, 则后面定义的累加, 前面的仍然从0开始
    - 枚举可以承载不同的类型在内, 最终在struct里占用的空间以最大的为准
- Module
    - mod xxx { }
    - 所有内部定义的都是private, 如果公开需要加pub
    - mod可以嵌套
    - 嵌套的mod可以使用`super::`引用上一级
    - 同时使用`crate::`可以获取root level的module
    - 每一个文件都是一个module, 文件名就是mod的名字, 所以就免去了写mod
    - 如果是嵌套的, 则可以使用文件夹, 但是需要在文件夹里添加`mod.js`来export里面的module
    - 使用`mod Name`来引用, 类似以import
    - 使用`use ModName::StructName`来声明, 类似与命名空间声明, 便可直接使用
    - `std`默认就被引用了, 所以不需要自己添加`mod std`
- Loops
    - loop可以加label
    - `'lable: loop {} condition;`
- Match
    - 类似于switch或者when
    - `match variable { case1 => xxx }`
    - 枚举值内的元素可以使用元组(tuple)来接多个值
    - 如果不需要使用, 使用下划线
    - `_ =>`可以接收default值
    - 多个case并列在一起使用`case1 | case2`
- Array
    - `[initial; size]`定义array的初始化值与长度
    - initial值决定了array内数据的类型
- Trait
    - 类似于interface, 或者extension, 可以给已有struct扩展方法
    - 实现方式 `impl TraitName for StructName`
    - 还可以定义一个Associate Type `type XXX;`来当泛型用, 当你只期望有一种扩展的时候可以使用
    - 如果使用了泛型, 则可以扩展多种次, 每次使用trait方法时候, 需要指定类型帮助推断
    - `unimplemented!()` 或者 `todo!()`可以supress未实现异常
- TryFrom
    - 它是一个trait, 有一个静态方法`try_from`, 实现了从指定的泛型<T>转换到你扩展的struc上, `impl TryFrom<Input> for Struct`
    - 它还还转自动实现`TryInto`, 不同的是, 里面是一个对象方法, `try_into`.
    - 它们都有一个协同类型定义Error类型
- Custom Error
    - 实现Error, 它是一个trait, 并且需要同时实现Display跟Debug
    - 可以通过`write!(formatter, "{}", xxx)`宏来写输出
    - Display与Debug实现相同, Error无需实现任何方法
- ErrorHandling
    - `Result`提供`or`方法来对error进行转换
    - 使用`?`, 如果正确返回结果, 如果错误异常
        - 可以使用`or`转换Error类型到Err
        - 也可以给自己Error实现`From<XXError>`
- Lifetime Parameter
    - compiler会check引用的生命周期
    - 因为有垃圾回收的语言会根据引用数量来判断是否release
    - 所以如果有引用的使用，必须提供足够的信息(类似MetaData)让静态检查可以判断引用使用的scope
    - 在`Struct`上通过范型`<'name>`来声明一个lifetime，name通常是单个字母
    - 如果函数只有一个lifetime param, 那么可以通过推断, 不需要显示的声明
    - 实现trait的时候, 需要在`impl`后面声明, 再在`Struct`后面加上lifetime
    - `Trait`的方法参数也需要声明, 以便实现里可以推断, 如:
      ```
      impl <'a> Trait for Struct<'a> {  
          fn test(str: &'a str)   { 
              ... 
          } 
      }`
      ```
- Silent Warnings
    - 可以对指定文件通过`#![allow(xxx)]`添加编译参数
    - 比如`dead_code`, `unused_variables`等
    - 这上面的Pang(!)表示该应用于该Module以及所有子Module
- Type Infer
    - rust编译器的推断是整体性的
    - 一行范型的推断可以考下一条语句来完成
    - 比如`HashMap::new()`可以不指定类型, 但是下一句使用该Map可以帮助上一句推断
- 指针赋值
    - 使用*Field_Name = Object
    - 只要地址类型匹配, 则可以安全赋值
    - 比如替换成其他的子类
- Debug Output
    - 可以使用dbg!来print debug信息
    - 但是对象必须实现Debug trait
    - 如果想直接用基础类型作为输出, 可以在对象上加`#[derive(Debug)]`
    - 需要给对象所有层的对象都加上
    - 比较推荐的是给所有的对象都加上以便调试
- 自动提示
    - 如果一个rs文件没有被添加到`mod.rs`, VSCode不会提示
    - 比如自动实现某Trait所有方法, 自动complete`match`的所有case
- Copy & Clone
    - `as`可以将指针指向位置转基础类型, 或者转换成已经实现的Trait
    - 转换为基础类型时需要对指针指向位置进行copy/clone
    - 可以通过`#[derive(Copy, Clone)]`实现
- Dynamic/Static Dispatch
    - 参数Trait前面加上`dyn`, 如`&mut dyn Trait`, 为动态代理
    - 加`impl`则为编译器静态绑定, 编译器会扫描所有的调用, 然后生成不同的函数
- 签名与声明
    - 原则: immutable borrow(`x: &T`), mutable borrow(`x: &mut T`), move((`x: T`)
    - 这些原则都是在冒号后面决定的, 它决定了变量的签名类型
    - 如果在变量前面添加`mut`, 这个只是在变量本身被赋予了mutable权限, 并没有改变签名
    - 引用的可读性是无法升级变更的, 因为它是签名, 无法从不可读变为可读
    - 有点像`const*`与`*const`, 一个是对内容的限制, 一个是对指针的限制
- Getter方法
    - 函数命名不需写get
    - 返回是成员的引用
    - Optional则通过`as_ref()`转成Option<&T>
- 环境变量
    - 通过`env::var("Name")`可以读取环境变, 返回Result
    - 通过`env!`变异期直接读取环境变量返回`&str`, 如果不存在则会直接编译错误
    - 通过`format`可以进行拼接