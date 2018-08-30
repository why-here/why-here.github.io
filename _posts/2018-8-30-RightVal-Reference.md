### 右值引用 / 转移语义 / 精确传递 / forward / emplace

大部分参考 [IBM developerWorks](https://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/index.html) 以及 [cppreference.com](https://zh.cppreference.com/w/%E9%A6%96%E9%A1%B5)

> 通俗的左值的定义就是非临时对象，那些可以在多条语句中使用的对象。所有的变量都满足这个定义，在多条代码中都可以使用，都是左值。右值是指临时的对象，它们只在当前的语句中有效。

> 右值引用也是引用，也是通过指针的传递实现的。[例子1](https://blog.csdn.net/xi_niuniu/article/details/50241227) / [例子2](http://kiritow.com/blog/archives/259)。右值引用是用来支持转移语义及精确传递。

> 转移语义可以将资源 ( 堆，系统对象等 ) 从一个对象转移到另一个对象，这样能够减少不必要的临时对象的创建、拷贝以及销毁。例子参考 [C++11 新特性](https://coolshell.cn/articles/5265.html)。

> 要实现转移语义，需要定义转移构造函数，还可以定义转移赋值操作符。对于右值的拷贝和赋值会调用转移构造函数和转移赋值操作符。

>  `std::move` ：编译器只对右值引用才能调用转移构造函数和转移赋值函数，而所有命名对象都只能是左值引用。`std::move` 这个函数以非常简单的方式将左值引用转换为右值引用。例如：

```C++
template <class T> 
swap(T& a, T& b) 
{ 
    T tmp(std::move(a)); // move a to tmp 
    a = std::move(b);    // move b to a 
    b = std::move(tmp);  // move tmp to b 
}
```

> Perfect Forwarding : 精确传递（完美转发）。精确传递适用于这样的场景：需要将一组参数原封不动的传递给另一个函数。“原封不动”不仅仅是参数的值不变，在 C++ 中，除了参数值之外，还有一下两组属性：左值／右值和 const/non-const（不能自动去掉 const 特性）。

``` C++
template <typename T> void forward_value(const T& val) { 
   process_value(val); 
} 
template <typename T> void forward_value(T& val) { 
   process_value(val); 
}

// 右值引用只需要定义一次，接受一个右值引用的参数，就能够将所有的参数类型原封不动的传递给目标函数。
template <typename T> void forward_value(T&& val) { 
   process_value(val); 
}
```

> std::forward : 用于手动精确传递。例如上面的例子，精确传递进 forward_value ,若想继续精确传递进 `process_value` 函数，就需要通过 std::forward 进行传递。见 [std::forward](https://zh.cppreference.com/w/cpp/utility/forward)

> EmplaceConstructible：（可原位构造）。以与提供给函数者准确相同的参数调用元素的构造函数。等效地调用 `c.emplace_back(std::forward<Args>(args)...);` 。 `emplace_back` 避免用 `push_back` 时的额外复制或移动操作。例子见 [std::vector::emplace_back](https://zh.cppreference.com/w/cpp/container/vector/emplace_back)