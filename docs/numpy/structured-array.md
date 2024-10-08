---
title: 结构化数组
icon: material/abacus
comments: false
---

???+ info "信息"

    前置知识:
    
    - [数据类型](/numpy/datatype)
    - [数组构建](/numpy/array-creation)
    - [数组索引](/numpy/array-index): [字段索引](#字段索引)
        
## 简介 {#简介}

结构化数组是NumPy提供的一种数据结构, 允许用户将数据类型组织成命名字段序列. 这种数组的数据类型是由多个更简单的数据类型组合而成, 每个字段都有一个名称和数据类型.

???+ note "笔记"

    在结构化数据类型中, 每个子类型称为字段. 每个字段都有一个名称, 一个类型, 和一个形状. 官方说明见[这里](https://numpy.org/doc/stable/glossary.html#term-field)
    
    ???+ example "例子"
    
        ```
        x = np.array([('Rex', 9, 81.0), ('Fido', 3, 27.0)], dtype=[('name', 'U10'), ('age', 'i4'), ('weight', 'f4')])
        ```
        
        其中`('name', 'U10')`为字段, `('name', 'U10'), ('age', 'i4'), ('weight', 'f4')`为字段序列, 由于它们都有名称, 所以被称为"命名字段"/"命名字段序列".
        
        `x`是一个长度为2的一维数组, 其数据类型是一个包含三个字段的结构: 一个长度不大于10的字符串, 名字为`name`; 一个32位整数, 名字为`age`; 一个32位浮点数, 名字为`weight`.
        
## 字段索引 {#字段索引}

详情见[这里](/numpy/array-index/#字段索引).

## 结构化数据类型

### 结构化数据类型创建

可以使用函数`np.dtype`创建结构化数据类型, 总结如下:

1. 元祖列表

    每个元祖都有以下形式`([fieldname], [datatype], [shape])`, 其中`[shape]`是可选的. `[fieldname]`是一个字符串(如果含有标题, 则是一个元祖, 见下面的[字段标题](#字段标题)), `[datatype]`可以是能够转化成数据类型的东西, `[shape]`是一个元祖用来表示子数组的形状.
    
    ???+ example "例子"
    
        ```
        >>> np.dtype([('x', 'f4'), ('y', np.float32), ('z', 'f4', (2, 2))])
        dtype([('x', '<f4'), ('y', '<f4'), ('z', '<f4', (2, 2))])
        ```
        
    ???+ tip "Tip"
    
        - 如果`[filename]`是空字符串`''`, 则该字段将被赋予`f#`形式的默认名称, 其中`#`是字段的整数索引, 从左边的`0`开始计数.
        
            ???+ example "例子"
            
                ```
                >>> np.dtype([('x', 'f4'), ('', 'i4'), ('z', 'i8')])
                dtype([('x', '<f4'), ('f1', '<i4'), ('z', '<i8')])
                ```

        - 结构内字段的字节偏移量和结构的总大小会被自动确定.
            
2. 逗号分隔的字符串

    在此简写中, 任何[字符串`dtype`规范](https://numpy.org/doc/stable/reference/arrays.dtypes.html#specifying-and-constructing-data-types)都可以用于字符串中并使用逗号分隔. 结构内字段的字节偏移量和结构的总大小会被自动确定, 字段名称被赋予默认名称`f#`.
    
    ???+ example "例子"

        ```
        >>> np.dtype('i8, f4, S3')
        dtype([('f0', '<i8'), ('f1', '<f4'), ('f2', 'S3')])
        >>> np.dtype('3int8, float32, (2, 3)float64')
        dtype([('f0', 'i1', (3,)), ('f1', '<f4'), ('f2', '<f8', (2, 3))])
        ```
        
3. 字段参数数组字典
 
    这是最灵活的写法, 因为它允许控制字段的字节偏移量和结构的总大小.
    
    该字典有两个必须的键, "名称"和"格式", 四个可选的键, "偏移量", "项目大小", "对齐"和"标题". 
    
    ???+ example "例子"
    
        ```
        >>> np.dtype({'names': ['col1', 'col2'], 'formats': ['i4', 'f4']})
        dtype([('col1', '<i4'), ('col2', '<f4')])
        >>> np.dtype({'names': ['col1', 'col2'],
                      'formats': ['i4', 'f4'],
                      'offsets': [0, 4],
                      'itemsize': 12})
        dtype({'names': ['col1', 'col2'], 'formats': ['<i4', '<f4'], 'offsets': [0, 4], 'itemsize': 12})
        ```
        
    ???+ warning "注意"
    
        可以选择偏移量, 意味着字段之间有可能重叠, 可能会破坏数据. 特别的`np.object_`类型的字段不能与其他字段重叠, 因为存在破坏内部对象指针然后对其取消引用的风险. 可选的"对齐"值可以设置为`True`, 以使自动偏移量计算使用对其偏移量. 
        
4. 字段名称字典

    字段的建式字段名称, 值是指定类型和偏移量的元祖.
    
    ???+ example "例子"
    
        ```
        >>> np.dtype({'col1': ('i1', 0), 'col2': ('f4', 1)})
        dtype([('col1', 'i1'), ('col2', '<f4')])
        ```
        
施工中...

[^1]: 结构化数组—NumPy v1.26 手册—NumPy 中文. (n.d.). Retrieved June 25, 2024, from https://numpy.com.cn/doc/stable/user/basics.rec.html