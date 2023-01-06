---
title: cutest 技术分析
date: 2023-01-06T14:17:25+08:00
---

## 前言

最近又花了一些时间将 C 语言单元测试框架 [cutest][1] 完善了一下，已经基本可以覆盖绝大部分 [googletest][2] 的常用功能。

由于 [cutest][1] 主要用于 C 语言，为了达到类似 [googletest][2] 的能力，用了不少技术手段。现在来分析一下。

## 测试用例自动注册

[cutest][1] 的一大功能是测试用例自动注册，即如下测试代码能够自动注册运行：

``` c
TEST(foo, bar)
{
    ASSERT_NE_STR("hello", "world");
}
```

上述功能的核心技术是使一段代码在 `main()` 函数执行之前执行，以便自动注册测试代码。

[cutest][1] 使用了三项技术，通过 `TEST_INITIALIZER` 宏判断运行环境并选择对应技术。

### C++全局变量初始化

当处于C++环境中时，[cutest][1] 采用类似 [googletest][2] 中全局类的初始化方法执行，即：

``` cpp
#define TEST_INITIALIZER(f) \
    void f(void); \
    struct f##_t_ { f##_t_(void) { f(); } }; f##_t_ f##_; \
    void f(void)
```

如上的代码中定义了一个全局变量结构 `s_##fixture##_##name`，并在结构初始化函数中调用用户定义函数 `f_##fixture##_##name`。虽然C++手册中没有明确定义，但是大部分编译器初始化全局变量的时间点在 `main()` 函数执行之前，这就保证了所有的测试用例在 `main()` 函数执行之前就已经自动注册。

### GCC/Clang 的构造属性

当使用 gcc/clang 时，这两种编译器支持 `__attribute__((constructor))` 属性。使用这个属性修饰的函数会在 `main()` 函数执行之前执行：

```c
#define TEST_INITIALIZER(f) \
    void f(void) __attribute__((constructor)); \
    void f(void)
```

### MSVC 的特殊 section

Windows平台的可执行文件有一个特殊 secion `.CRT$XCU`，存放在此 section 中的函数会在 `main()` 函数启动之前运行：

```c
#pragma section(".CRT$XCU",read)
#define TEST_INITIALIZER2_(f,p) \
    void f(void); \
    __declspec(allocate(".CRT$XCU")) void (*f##_)(void) = f; \
    __pragma(comment(linker,"/include:" p #f "_")) \
    void f(void)
#ifdef _WIN64
#    define TEST_INITIALIZER(f) TEST_INITIALIZER2_(f,"")
#else
#    define TEST_INITIALIZER(f) TEST_INITIALIZER2_(f,"_")
#endif
```

注意这里还需要区分是否是64位运行环境，这是由于不同环境在编译时生成的符号表不同：32位环境生成的符号会自动添加一个下划线。

## 参数化测试用例解析

当定义了参数化用例时，`--test_list_tests` 命令行选项会输出对用的参数：

```c
typedef struct test_p_2_data
{
    int a;
    int b;
    int c;
} test_p_2_data_t;

TEST_PARAMETERIZED_DEFINE(example, test_p_structure, test_p_2_data_t, { 1, 2, 3 }, { 2, 3, 5 });

TEST_P(example, test_p_structure)
{
    TEST_PARAMETERIZED_SUPPRESS_UNUSED;
}
```

```
$ ./example --test_list_tests
example.
  test_p_structure/0  # <test_p_2_data_t> { 1, 2, 3 }
  test_p_structure/1  # <test_p_2_data_t> { 2, 3, 5 }
```

这里存在两个技术难点：
1. 如何知道这里存在2个用例？
2. 如何准确输出2个用例的参数内容？

### 用例数量统计

为了准确知道参数化测试的用例数量，我们需要用到可变数组的初始化方法：

```c
static TYPE s_parameterized_userdata[] = { __VA_ARGS__ };
```

如上代码将用户参数静态初始化为一个数组，则数组元素数量就是用例数量。上述代码展开后就变成：

```c
static test_p_2_data_t s_parameterized_userdata[] = {
    { 1, 2, 3 }, { 2, 3, 5 }
};
```

此时 `sizeof(s_parameterized_userdata)/sizeof(s_parameterized_userdata[0])` 即为用例数量。

### 用例参数解析

用例参数的内容是无法在编译期直接获知的。为了能够输出各个用例的内容，我们需要在运行期解析：

1. 首先我们通过字符串记录用户参数的完整内容：`"{ 1, 2, 3 }, { 2, 3, 5 }"`。
2. 随后我们需要在运行期对于这个字符串进行词法分析。这里不用担心分析一场，因为既然能够通过编译，一定说明词法是正确的。词法分析的内容依照优先级分为三类：字符串、结构体、其他。
3. 依照词法分析结果，选择对用的用户字符串输出。

## 终端颜色按场景渲染

在普通的支持彩色输出的程序中，如果我们将其重定向至文件，可以看到文件中有类似乱码的存在；而类似的问题在 [cutest][1] 中不存在，这是为什么？

在 Linux 中，为了输出彩色颜色，我们需要添加一些控制字符，例如 `\033[0;33m` 等，这些特殊字符是无法通过文本编辑器正常显示的，正是它们造成了乱码。

而 [cutest][1] 通过 `isatty()` 接口对输出设备进行了判断（代码来源于 [googletest][2]），仅当输出设备为 `tty` 时，才输出颜色，否则仅输出文本。

<!-- links -->
[1]: https://github.com/qgymib/cutest
[2]: https://github.com/google/googletest
