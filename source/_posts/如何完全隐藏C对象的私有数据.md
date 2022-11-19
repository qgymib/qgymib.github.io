---
title: 如何完全隐藏C对象的私有数据
date: 2021-08-10T14:15:06+08:00
tags:
---

## 背景

在C中对象的抽象常常使用struct中嵌套若干个函数指针的方式进行:

```c
typedef struct foo
{
    void*   priv;   // Private data
    void    (*bar)(struct foo* thiz);
} foo_t;
```

然而这里的`priv`字段其实是不需要的，接下来我们讨论如何去除这个字段，实现私有数据完全隐藏。

<!-- more -->

## 传统实现

在传统实现中，我们为了创建一个抽象对象的实例，工厂类往往会书写如下：

```c

typedef struct foo
{
    void*   priv;   // Private data
    void    (*bar)(struct foo* thiz);
} foo_t;

foo_t* create_foo(void)
{
    foo_t* obj = malloc(sizeof(foo_t));
    obj->priv = some_data;      // something private
    obj->bar = some_handler;    // function
    return obj;
}
```

## 改进版实现

### 认识 `container_of`

在改进版实现中，我们可以去除`priv`字段。为了实现这个操作，我们需要首先认识一个辅助宏 `container_of`：

```c
#if !defined(container_of)
#if defined(__GNUC__) || defined(__clang__)
#	define container_of(ptr, type, member) \
		({ \
			const typeof(((type *)0)->member)*__mptr = (ptr); \
			(type *)((char *)__mptr - offsetof(type, member)); \
		})
#else
#	define container_of(ptr, type, member) \
		((type *) ((char *) (ptr) - offsetof(type, member)))
#endif
#endif
```

`container_of` 的作用是通过结构体成员变量地址获取这个结构体的地址。举个例子：我们有如下结构体定义：
```c
struct example
{
    int field_a;
    int field_b;
};
```

现已知一个 `struct example` 的实例中 `field_b` 字段的地址，如何取得实例本身的地址呢？可通过如下代码进行：
```c
struct example* get_struct_addr(int* pointer)
{
    return container_of(pointer, struct example, field_b);
}
```

### 实现私有属性隐藏

基于 `container_of` 宏，我们就能实现私有属性隐藏：

```c
/**
 * @file: foo.h
 * 对外头文件
 */
typedef struct foo
{
    void    (*bar)(struct foo* thiz);
} foo_t;

```

```c
/**
 * @file: foo.c
 * 内部实现
 */
#include "foo.h"

typedef struct foo_impl
{
    foo_t   handle;     // 对外接口
    void*   priv;       // 私有属性
}foo_impl_t;

static void _bar(struct foo* thiz)
{
    /* 获取原始地址 */
    foo_impl_t* impl = container_of(thiz, foo_impl_t, handle);

    /* do something */
}

foo_t* create_foo(void)
{
    foo_impl_t* impl = malloc(sizeof(foo_impl_t));
    impl->priv = some_data; // 私有属性
    impl->handle.bar = _bar;

    /* 返回的是 foo_impl_t::handle 的地址 */
    return &impl->handle;
}

```
