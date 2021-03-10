---
layout:     post
title:      "Category & Load & Initialize源码"
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

## Category & Load & Initialize

### category
编译之后的底层结构是`struct category_t`， 里面存储着分类的对象方法、类方法、属性、协议信息
在运行时，runtime会将具体的数据，合并到类信息中
```cpp
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }
    
    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

每一个分类，都是一个`struct category_t`，会在runtime里，遍历每一个分类，添加到类信息中的rw_t表中，拿到这些表
```c
    method_list_t **mlists = (method_list_t **)
        malloc(cats->count * sizeof(*mlists));
        
    property_list_t **proplists = (property_list_t **)
        malloc(cats->count * sizeof(*proplists));
        
    protocol_list_t **protolists = (protocol_list_t **)
        malloc(cats->count * sizeof(*protolists));
```

将所有分类信息，附加到类信息，将原方法移动到数组中的最后
` memmove(array()->lists + addedCount, array()->lists, oldCount * sizeof(array()->lists[0]));`

将分类中的方法复制到原方法位置
`memcpy(array()->lists, addedLists, addedCount * sizeof(array()->lists[0]));`

所以分类方法会覆盖原方法，多个分类方法的调用(消息机制)，根据编译顺序确定

## Category & Extention
Class Extension 在编译时，它的数据就已经存在类信息中
Category，在运行时，动态添加

### Category & Load
- 为什么Load方法没有覆盖
    - 因为分类的方法是消息发机，根据isa查找到具体的类，再去遍历方法列表，找到具体的方法进行调用
    - Load方法是内存中，直接通过函数指针找到函数地址值，直接调用

## Load
会在runtime加载类、分类时调用
每个类、分类的Load方法，只会调用一次
Load方法是内存中，直接通过函数指针找到函数地址值，直接调用

- 调用顺序
  1. 先调用类的+ load
  按照编译顺序，先编译先调用
  调用子类的+ load前，会先调用父类的+ load

  2. 再调用分类的+ load
  按照编译顺序，先编译先调用

## Initialize
通过消息机制调用
会在类第一次接收到消息时调用

- 调用顺序
  会先调用父类的+ initialize, 再调用子类的+ initialize，不够严谨，有可能不会调用子类的
  (先初始化父类，再初始化子类，每个类只会初始化一次)
  
```
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver)
{
    // 如果需要初始化，并且 类cls 没有初始化
    if (initialize  &&  !cls->isInitialized()) {
        runtimeLock.unlock();
        _class_initialize (_class_getNonMetaClass(cls, inst));
        runtimeLock.lock();
        // If sel == initialize, _class_initialize will send +initialize and 
        // then the messenger will send +initialize again after this 
        // procedure finishes. Of course, if this is not being called 
        // from the messenger then it won't happen. 2778172
    }
}
```

``` 
void _class_initialize(Class cls)
{
    // Make sure super is done initializing BEFORE beginning to initialize cls.
    // See note about deadlock above.
    // 递归初始化父类
    supercls = cls->superclass;
    if (supercls  &&  !supercls->isInitialized()) {
        _class_initialize(supercls);
        }
}
```

```
    {
            // 真正的初始化
            callInitialize(cls);

            if (PrintInitializing) {
                _objc_inform("INITIALIZE: thread %p: finished +[%s initialize]",
                             pthread_self(), cls->nameForLogging());
            }
    }
}
```

```
void callInitialize(Class cls)
{
    ((void(*)(Class, SEL))objc_msgSend)(cls, SEL_initialize);
    asm("");
}
```

## Load & Initialize 区别
1. 调用方式
    - load 是通过函数指针，直接调用
    - initialize 是通过消息发送机制

2. 加载时刻
    - load 是在类、分类加载的时候调用，只会调用一次
    - initialize 是类第一次接收到消息时调用 