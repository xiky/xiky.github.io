---
layout:     post
title:      "Autoreleasepool"
subtitle:   " - "
date:       2019-11-02 20:00:00
author:     "Yxi"
header-img: "img/post-bg-2015.jpg"
catalog: true
<!-- tags:
    - 开发 -->
---

> “Yeah It's on. ”


## 前言
学无止境

<p id = "build"></p>
---

## 自动释放池
主要结构是`__AtAutoreleasePool`、`__AtAutoreleasePoolPage`
```cpp
class AutoreleasePoolPage 
{
    // EMPTY_POOL_PLACEHOLDER is stored in TLS when exactly one pool is 
    // pushed and it has never contained any objects. This saves memory 
    // when the top level (i.e. libdispatch) pushes and pops pools but 
    // never uses them.
#   define EMPTY_POOL_PLACEHOLDER ((id*)1)

#   define POOL_BOUNDARY nil
    static pthread_key_t const key = AUTORELEASE_POOL_KEY;
    static uint8_t const SCRIBBLE = 0xA3;  // 0xA3A3A3A3 after releasing
    static size_t const SIZE = 
#if PROTECT_AUTORELEASEPOOL
        PAGE_MAX_SIZE;  // must be multiple of vm page size
#else
        PAGE_MAX_SIZE;  // size and alignment, power of 2
#endif
    static size_t const COUNT = SIZE / sizeof(id);

    magic_t const magic;
    id *next;
    pthread_t const thread;
    AutoreleasePoolPage * const parent;
    AutoreleasePoolPage *child;
    uint32_t const depth;
    uint32_t hiwat;
}
```

```cpp
struct __AtAutoreleasePool {
  __AtAutoreleasePool() {
      atautoreleasepoolobj = objc_autoreleasePoolPush();
  }
  ~__AtAutoreleasePool() {
      objc_autoreleasePoolPop(atautoreleasepoolobj);
  }
  void * atautoreleasepoolobj;
};
```

```ojbc
{ 
    __AtAutoreleasePool __autoreleasepool; 
    Person *person = ((Person *(*)(id, SEL))(void *)objc_msgSend)((id)((Person *(*)(id, SEL))(void *)objc_msgSend)((id)((Person *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Person"), sel_registerName("alloc")), sel_registerName("init")), sel_registerName("autorelease"));
}
```

``` cpp
void * objc_autoreleasePoolPush(void)
{
    return AutoreleasePoolPage::push();
}

void objc_autoreleasePoolPop(void *ctxt)
{
    AutoreleasePoolPage::pop(ctxt);
}


void * _objc_autoreleasePoolPush(void)
{
    return objc_autoreleasePoolPush();
}

void _objc_autoreleasePoolPop(void *ctxt)
{
    objc_autoreleasePoolPop(ctxt);
}

// 可用此函数输入释放池，在外面 extern 这个函数，再直接调用
void_objc_autoreleasePoolPrint(void)
{
    AutoreleasePoolPage::printAll();
}
```

## 释放时机
`main`函数中的`@autoreleasepool`会在程序运行过程中一直存在，所以变量的自动释放池不是在main

iOS 在主线程的Runloop中创建了两个监听

- 第1个监听了`kCFRunLoopEntry`事件，会调用`objc_autoreleasePoolPush()`
- 第2个监听了`kCFRunLoopBeforeWaiting | kCFRunLoopBeforeExit`
    - 当触发`kCFRunLoopBeforeWaiting`时，会调用`objc_autoreleasePoolPop()`和`objc_autoreleasePoolPush()`
    - 当触发`kCFRunLoopBeforeExit`时，会调用`objc_autoreleasePoolPop()`