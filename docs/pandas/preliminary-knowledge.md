---
title: 预备知识
icon: material/meteor
comments: false
---

## Python基础

### 列表生成式

详情见[这里](/foundation/container/#列表生成式).

### 匿名函数

匿名函数即Lambda函数, 详情见[这里](/foundation/function/#Lambdafunction).

### `map`函数

`map`函数, 详情见[这里](/foundation/function/#mapfunction).

### `zip`函数

`zip`函数用于将多个可迭代对象中的索引值相同的元素打包成一个元组, 然后返回由这些元组组成的序列, 或者称一个zip对象.

???+ example "例子"

    ```
    [1]: L1, L2, L3 = list('abc'), list('def'), list('hij')
    [2]: list(zip(L1, L2, L3))
    [('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j')]
    [3]: tuple(zip(L1, L2, L3))
    (('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j'))
    [4]: zip(L1, L2, L3)
    <zip at 0x281f7952500>
    ```
    
???+ tip "Tip"

    - 如果各个可迭代对象中的元素个数不一致, 则返回序列中元组的长度和最短的可迭代对象的长度相同
    
        ???+ example "例子'

            ```
            [1]: L1, L2, L3 = list('abcyu'), list('def'), list('hij5')
            [2]: list(zip(L1, L2, L3))
            [('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j')]
            ```
            
    - 当需要两个列表建立字典映射时, 可以利用`zip`函数
    
        ???+ example "例子"
        
            ```
            [1]: L1, L2 = list('abc'), list('def')
            [2]: dict(zip(L1, L2))
            {'a': 'd', 'b': 'e', 'c': 'f'}
            ```
            
    - Python提供了`*`操作符用于对`zip`对象进行解压
    
        ???+ example "例子"
        
            ```
            [1]: L1, L2, L3 = list('abc'), list('def'), list('hij')
            [2]: zipped = list(zip(L1, L2, L3))
            [3]: zipped
            [('a', 'd', 'h'), ('b', 'e', 'i'), ('c', 'f', 'j')]
            [4]: list(zip(*zipped)) 
            [('a', 'b', 'c'), ('d', 'e', 'f'), ('h', 'i', 'j')]
            ```
        
### `enumerate`函数

`enumerate`函数用于接受一个可迭代对象, 在遍历的该可迭代对象的时候, 同时获取该元素的索引和值.
    
???+ exmple "例子"

    ```
    [1]: L = list("abcd")
    [2]: for index, value in enumerate(L):
             print(index, value)
    0 a
    1 b
    2 c
    3 d
    ```
    
???+ tip "Tip"

    用`zip`函数也可以实现与`enumerate`函数相同的功能.
    
    ???+ example "例子"
    
        ```
        [1]: L = list("abcd")
        [2]: for index, value in zip (range(len(L)), L):
                 print(index, value)
        0 a
        1 b
        2 c
        3 d
        ```
        
## NumPy基础

### 数组构建

详情见[这里](/numpy/array-creation).

### 数组索引

详情见[这里](/numpy/array-index).

### 广播

详情见[这里](/numpy/broadcast).

### 数组变形与合并

#### 转置

访问数组的`T`属性, 返回的数组是一个[视图](/numpy/view-copy/#视图).

???+ example "例子"

    ```
    [1]: np.eye(2,3)
    array([[1., 0., 0.],
           [0., 1., 0.]])
    [2]: np.eye(2, 3).T
    array([[1., 0.],
           [0., 1.],
           [0., 0.]])
    [3]: np.eye(2,3).T.base
    array([[1., 0., 0.],
           [0., 1., 0.]])
    ```

#### 合并

`np.r_()`表示上下合并, `np.c_()`表示左右合并.

- 两个二维数组合并

    ???+ example "例子"
    
        ```
        [1]: np.r_[np.eye(2,3),np.eye(2,3)]
        array([[1., 0., 0.],
               [0., 1., 0.],
               [1., 0., 0.],
               [0., 1., 0.]])
        [2]: np.c_[np.eye(2,3),np.eye(2,3)]
        array([[1., 0., 0., 1., 0., 0.],
               [0., 1., 0., 0., 1., 0.]])
        ```
        
- 一个一维数组和一个二维数组合并

    应当把一维数组视为列向量(二维), 在长度匹配的情况下只能使用左右合并`np.c_()`.
    
    ???+ example "例子"
    
        ```
        [1]: try:
                np.r_[np.array([0,0]),np.zeros((2,1))]
            except Exception as e:
                Err_Msg = e
        [2]: Err_Msg
        ValueError('all the input arrays must have same number of dimensions, but the array at index 0 has 1 dimension(s) and the array at index 1 has 2 dimension(s)')
        [3]: np.c_[np.array([0,0]),np.zeros((2,3))]
        array([[0., 0., 0., 0.],
               [0., 0., 0., 0.]])
        ```
        
- 两个一维数组合并

    ???+ example "例子"
    
        ```
        [1]: np.r_[np.array([0,0]),np.zeros(2)]
        array([0., 0., 0., 0.])
        [2]: np.c_[np.array([0,0]),np.zeros(2)]
        array([[0., 0.],
               [0., 0.]])
        ```
        
#### 变形

`np.reshape`函数能够把原数组按照新的形状重新排列. 在使用时, 有两种模式, 分别为`C`模式和`F`模式, 分别以逐行和逐列的顺序进行填充读取.

???+ example "例子"

    ```
    [1]: x = np.arange(8)
    [2]: x
    array([0, 1, 2, 3, 4, 5, 6, 7])
    [3]: x1 = x.reshape(2,4)
    [4]: x1
    array([[0, 1, 2, 3],
           [4, 5, 6, 7]])
    [5]: x2 = x1.reshape((4, 2), order="C") # 按照行读取和填充
    [6]: x2
    array([[0, 1],
           [2, 3],
           [4, 5],
           [6, 7]])
    [7]: x3 = x1.reshape((4, 2), order="F") # 按照列读取和填充
    [8]: x3
    array([[0, 2],
           [4, 6],
           [1, 3],
           [5, 7]])
    ```
    
???+ tip "Tip"

    当被调用的数组的形状是确定的情况下, 要将这个数组变形, 若你不知道某个维度的大小, 可以在`np.reshape`函数中将那个维度指定为`-1`. NumPy会自动计算这个维度的大小以确保在新形状下数组的元素数量和原数组的的元素数量一致.
    
    ???+ example "例子"
    
        ```
        [1]: x = np.arange(8)
        [2]: x
        array([0, 1, 2, 3, 4, 5, 6, 7])
        [3]: x1 = x.reshape(2,4)
        [4]: x1
        array([[0, 1, 2, 3],
               [4, 5, 6, 7]])
        [5]: x1.reshape((4, -1)) # NumPy自动根据"变形后元素数量一致"这条规律计算出空缺维度的大小
        array([[0, 1],
               [2, 3],
               [4, 5],
               [6, 7]])
        [6]: y = np.ones((3, 1))
        [7]: y
        array([[1.],
               [1.],
               [1.]])
        [8]: y.reshape(-1)
        array([1., 1., 1.])
        ```
        
### 常见函数/方法

#### `np.where`函数 

`np.where()`是一种条件函数, 可以指定满足条件与不满足条件位置对应的填充值.

???+ example "例子"

    ```
    [1]: a = np.array([-1, 1, -1, 0])
    [2]: np.where(a>0, a, 5)
    array([5, 1, 5, 5])
    ```
    
    对应位置为`True`时填充`a`的对应元素, 否则填充`5`.
    
#### `np.nonzero`函数

`np.nonzero`函数返回的时非零数的索引. 这个函数可用于[布尔数组索引](/numpy/array-index/#布尔array-index)到整数数组索引的转换.

???+ example "例子"

    ```
    [1]: a = np.array([-2,-5,0,1,3,-1])
    [2]: np.nonzero(a)
    (array([0, 1, 3, 4, 5]),)
    ```
    
#### `argmax/argmin`函数

- `argmax`函数返回最大值的索引

    ???+ example "例子"

        ```
        [1]: a = np.array([-2,-5,0,1,3,-1])
        [2]: a.argmax()
        4
        ```
    
- `argmin`函数返回最小值的索引

    ???+ example "例子"

        ```
        [1]: a = np.array([-2,-5,0,1,3,-1])
        [2]: a.argmin()
        1
        ```

#### `any/all`函数

- `any`函数当数组至少存在一个`True`或非零元素时返回`True`, 否则返回`False`

    ???+ example "例子"
    
        ```
        [1]: a = np.array([0, 1])
        [2]: a.any()
        True
        ```
        
- `all`函数当数组全为`True`或非零元素时返回`True`, 否则返回`False`

    ???+ example "例子"
    
        ```
        [1]: a = np.array([0, 1])
        [2]: a.all()
        False
        ```
        
#### `cumprod/cumsum`函数

- `cumprod()`表示累乘, 返回同长度的数组

    ???+ example "例子"
    
        ```
        [1]: a = np.array([1, 2, 3])
        [2]: a.cumprod()
        array([1, 2, 6])
        ```
        
- `cumsum()`表示累加, 返回同长度的数组

    ???+ example "例子"
    
        ```
        [1]: a = np.array([1, 2, 3])
        [2]: a.cumsum()
        array([1, 3, 6])
        ```
        
#### `np.diff`函数

`np.diff`函数表示每个元素的值是原数组中对应元素和前一个元素做差的值, 由于第一个元素为缺失值, 因此在默认参数情况下, 返回长度是原数组长度减1.

???+ example "例子"

    ```
    [1]: a = np.array([1, 2, 3])
    [2]: np.diff(a)
    array([1, 1])
    ```
    
#### 统计函数

常用的统计方法包括`max, min, mean, median, std, var, sum`. 统计函数有`np.quantile`.

???+ example "例子"

    ```
    [1]: x = np.arange(5)
    [2]: x
    array([0, 1, 2, 3, 4])
    [3]: x.max()
    np.int64(4)
    [4]: np.quantile(x, 0.5)   
    np.float64(2.0)
    ```
    
???+ tip "Tip"

    - 对于含有缺失值(`np.nan`)的数组, 上述函数的返回值也是缺失值. 如果要略过缺失值, 必须添加一个前缀`nan`, 上述几个方法/函数都有对应的`nan*`函数
    
        ???+ example "例子"
        
            ```
            [1]: x = np.array([1, 2, np.nan])
            array([ 1.,  2., nan])
            [2]: x.max()
            np.float64(nan)
            [3]: np.nanmax(x)
            np.float64(2.0)
            [4]: np.nanquantile(x, 0.5)
            np.float64(1.5)
            ```
        
    - [协方差](https://zh.wikipedia.org/wiki/%E5%8D%8F%E6%96%B9%E5%B7%AE)和[相关系数](https://zh.wikipedia.org/wiki/%E7%9B%B8%E5%85%B3%E7%B3%BB%E6%95%B0)可以用`np.cov()`和`np.corrcoef`函数计算
    
    - 统计方法/函数中的`axis`参数能够进行某一维度下的统计特征计算, 当`axis=0`时结果为列的统计指标, 当`axis=1`时结果为行的统计指标    
    
        ???+ example "例子"
        
            ```
            [1]: x = np.arange(1,10).reshape(3,-1)
            [2]: x
            array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
            [3]: x.sum(0)
            array([12, 15, 18])
            [4]: x.sum(1)
            array([ 6, 15, 24])
            ```
            
### 向量和矩阵的计算

#### 向量内积

使用`dot`函数.

???+ example "例子"

    ```
    [1]: a = np.array([1, 2, 3])
    [2]: b = np.array([1, 3, 5])
    [3]: a.dot(b)
    np.int64(22)
    ```
    
#### 矩阵乘法

使用`@`运算符.

???+ example "例子"

    ```
    [1]: a = np.arange(4).reshape(-1, 2)
    [2]: b = np.arange(-4, 0).reshape(-1, 2)
    [3]: a
    array([[0, 1],
           [2, 3]])
    [4]: b
    array([[-4, -3],
           [-2, -1]])
    [5]: a@b
    array([[ -2,  -1],
           [-14,  -9]])
    ```

[^1]: 第一章 预备知识—Joyful Pandas 1.0 documentation. (n.d.). Retrieved June 25, 2024, from https://inter.joyfulpandas.datawhale.club/Content/ch1.html