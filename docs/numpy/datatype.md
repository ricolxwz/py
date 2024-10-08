---
title: 数据类型
icon: material/cards-playing-club-outline
comments: false
---

## 标量和向量

- 标量: 只有一个数, 不具有方向性, 比如说NumPy的nd数组中的一个元素
- 向量: 由标量组成, 具有方向性, 比如说NumPy的nd数组

### 标量和向量属性的一致性 {#NumPy中标量和向量的一致性}

在NumPy中, 为了消除混合标量和nd数组操作时产生的不一致性, 我们将nd数组(向量)中的元素(标量)也视为一个nd数组(向量), 这可以通过保持标量和向量的属性一致实现, 也就是说由`np.[type]([value])`创建的标量具有和向量相同的属性和方法.

正因为如此, 我们可以称由`np.[type]([value])`创建的标量特别的称为"数组标量". 更多的属性可以在[这里](https://numpy.org/doc/stable/reference/arrays.scalars.html#attributes)找到, 更多的方法可以在[这里](https://numpy.org/doc/stable/reference/arrays.scalars.html#methods)找到.

???+ example "例子"

    ```
    >>> import numpy as np
    >>> x = np.int8(10)
    >>> y = np.array([1, 2, 3], dtype=np.int8)
    >>> x.dtype
    dtype('int8')
    >>> isinstance(x.dtype, np.dtype)
    True
    >>> y.dtype
    dtype('int8') 
    >>> isinstance(y.dtype, np.dtype)
    True
    ```

## `np.[type]`类

`np.[type]`是一个类, `np.[type]([value])`用于创建标量, 或"数组标量".

???+ tip "Tip"

    Python自带的`int`也是一个类,由`int`创建的对象是Python标量, 不是数组标量. 这两种标量的属性有很大不同.

???+ example "例子"

    ```
    >>> import numpy as np
    >>> x = np.int8(8)
    >>> type(x)
    <class 'numpy.int8'>
    >>> isinstance(x, np.int8)
    True
    >>> type(np.int8)
    <class 'type'>
    >>> z = 10
    >>> type(z)
    <class 'int'>
    >>> isinstance(z, int)
    True
    ```

### 别名类

|名称|C语言中的类型|描述|
|-|-|-|
|`np.int_`|`long`|一般是`np.int32`或`np.int64`(取决于系统是32位还是64位)|
|`np.intc`|`int`|一般是`np.int32`或`np.int64`(取决于系统是32位还是64位)|
|`np.intp`|`ssize_t`|一般是`np.int32`或`np.int64`(取决于系统是32位还是64位)|
|`np.float_`|`double`|`float64`类型的简写|
|`np.complex_`|`double complex`|`complex128`类型的简写|

### 固定类

|名称|C语言中的类型|描述|
|-|-|-|
|`np.bool_`|`bool`|布尔数据类型|
|`np.int8`|`int8_t`|整数 -128 ~ 127|
|`np.int16`|`int16_t`|整数 -32768 ~ 32767|
|`np.int32`|`int32_t`|整数 -2147483648 ~ 2147483647|
|`np.int64`|`int64_t`|整数 -9223372036854775808 ~ 9223372036854775807|
|`np.uint8`|`uint8_t`|无符号整数 0 ~ 255|
|`np.uint16`|`uint16_t`|无符号整数 0 ~ 65535|
|`np.uint32`|`uint32_t`|无符号整数 0 ~ 4294967295|
|`np.uint64`|`uint64_t`|无符号整数 0 ~ 18446744073709551615|
|`np.float16`|无|半精度浮点数, 包括: 1个符号位, 5个指数位, 10个尾数位|
|`np.float32`|`float`|单精度浮点数, 包括: 1个符号位, 8个指数位, 23个尾数位|
|`np.float64`|`double`|双精度浮点数, 包括: 1个符号位, 11个指数位, 52个尾数位|
|`np.complex64`|`float complex`|复数, 表示双32位浮点数(实数部分和虚数部分)|
|`np.complex128`|`double complex`|复数, 表示双64位浮点数(实数部分和虚数部分)|

### 层级

`np.[type]`类有一个层次结构来表示继承关系. 

![层级](https://numpy.org/doc/stable/_images/dtype-hierarchy.png)

???+ example "例子"

    ```
    >>> import numpy as np
    >>> x = np.int8(10)
    >>> isinstance(x, np.generic)
    True
    >>> isinstance(x, np.bool_)
    False
    >>> isinstance(x, np.number)
    True
    ```

## `np.dtype`类

`np.dtype`是一个类, 由`np.dype([base_type])`创建的对象被用来表示nd数组中元素或者数组标量的类型, 类似于C语言中的结构体, 可以定义基类, 定义字节数量, 定义是小端还是大端等等. 我们可以通过对象的`type`属性知道它的基类.

```
class numpy.dtype(dtype, align=False, copy=False[, metadata])
```

???+ warning "注意"

    - 在创建数组标量的时候, 其`dtype`属性会被自动赋值为一个`np.dtype([type])`对象. 详情见[这里](#NumPy中标量和向量的一致性).
    - 在创建nd数组的时候, 其`dtype`属性会被自动赋值为一个`np.dtype(np.float64])`对象, 或者也可以手动赋值. 

???+ example "例子"

    ```
    >>> import numpy as np
    >>> dt = np.dtype(np.int32)
    >>> data = np.array([1, 2, 3], dtype=dt)
    >>> data
    array([1, 2, 3], dtype=int32)
    >>> data.dtype
    dtype('int32')
    >>> data.dtype.type
    <class 'numpy.int32'>
    ```

[^1]: NumPy 数据类型 | 菜鸟教程. (n.d.). Retrieved June 20, 2024, from https://www.runoob.com/numpy/numpy-dtype.html
[^2]: Data type objects (dtype)—NumPy v2.0 Manual. (n.d.). Retrieved June 20, 2024, from https://numpy.org/doc/stable/reference/arrays.dtypes.html
[^3]: Scalars—NumPy v2.0 Manual. (n.d.). Retrieved June 20, 2024, from https://numpy.org/doc/stable/reference/arrays.scalars.html
[^4]: ricolxwz. (2024, June 20). Difference between np.[type] class and np.dtype class [Forum post]. Stack Overflow. https://stackoverflow.com/q/78645442/19538012