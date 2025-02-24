---
title: 阅读笔记 ：jerryscript-internals
date: 2019-03-31 14:41:32
tags: jerryscript
categories: 源码阅读
copyright: true
---

> 阅读了jerryscript的[internals](http://jerryscript.net/internals/)文档，过程中做了一些笔记。主要记录的是ECMA部分。

### virtual machine

VM 是逐句执行byte-code指令的js翻译器。翻译开始的函数是`vm_run`。`vm_loop`是VM中非递归的主循环。也就是说在函数调用时它不会调用它自己进行递归但是有返回值，因此不会给栈带来负担。

### ECMA
#### 数据表示 data representation
- 最低2bit表示了数据类型type（simple,number,string,object）
- simple是预定义的常数，包括：undefined,null,true,false,empty(uninitialized value)
##### compressed pointers
16bits，排列成8byte（可以表示512Kb的内存，即Jerryscript的堆大小），可以扩展成32bits来表示更大的内存
##### number
IEEE 754标准。默认8 byte（double），也可以是4 byte（单精度）。通过CONFIG_ECMA_NUMBER_TYPE指定。
##### string
不仅仅是字符序列，也能表示numbers和magic id。read only memory中有一个表格用来存储常见的字符序列，保存的形式是（magic id，character sequence）对。对于表格中存在的字符序列，它对应的magic id也会被记录在表格中。这种机制可以加速property access，节省内存。
##### object/lexical environment
一个object可以是传统的数据对象，也可以是lexical environment对象。对象可以引用其他数据类型，并且可能存在循环引用，因此引用计数的机制不足以决定dead object。对于所有现存的对象有一个chain list，能够在垃圾回收的时候找到未被引用的对象。一个对象的gc-next指针就是指向它在chain list中的下一个对象。
lexical environment在jerry script中也当成对象（object）实现的，因为它包含像object一样的键值对（called binding）。这样可以简化实现并且减少代码量。

##### property of objects
对象中有一个包含了他们的性质的链表。这个链表实际上包含的是property pairs，为了节省内存：7bit的property + 2 bit 的type = 9bit，需要占用2 byte的空间。因此，将2个property与1个type放在一起就是 7bit + 7 bit + 2bit = 16bit, 同样也是2 byte的空间，提高了利用率。

- property hashmap：如果property对的数量超过一个限制（16对），property hashmap就被插入到pair list的头部，这样就可以通过这个hashmap来找到property，而不是通过线性便利链表来查找。
  - hashmap可以包含2^n个元素，查过了一个对象的property的个数，每个元素可以有3种类型的值：null,deleted,reference to the existing property
  - 对于对象包含的每个property，用hashmap查找一定可以找到。

- internal property：内部性质是一种特殊的性质，它携带了无法通过访问js代码获得的但是对于引擎本身来说很重要的信息。例如：
  - Class：class type
  - Code：指向函数的字节码的存放位置的指针
  - native code：指向本地函数代码存放位置的指针
  - PrimitiveValue for booean：存放boolean对象的值
  - PrimitiveValue for number：存放number对象的值
- Lcache：用来根据对象以及性质名寻找性质的hashmap。
  - 一个对象的所有性质并不一定都能在Lcache中找到，若找不到，则在property-list中寻找，找到后会被放入Lcache中。
- Collections：array-like，save memory。collection是一个以array为元素的链表。
- Exception Handling：为了实现异常处理，JerryScript函数的返回值可以指出它们的错误或者异常的操作。这些返回值实际上是ECMA的一些值，当错误出现时error bit会被置位。
- Value Management and Ownership：每个被引擎存储的ECMA值都会与一个虚拟的“ownership”相关联，ownership定义了ECMA值何时被释放或者何时传递给另一个函数。
  - 一开始，一个value由它的owner创建，owner有释放这个value的责任，当这个value作为一个参数被传递到另一个函数时，它的ownership不会跟着传递，那个函数必须创建一个value的拷贝。然而，当一个value作为返回值被传递的时候，它的ownership也会跟着传递，因此调用函数的caller必须负责释放它。