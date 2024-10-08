---
title: 数据索引
icon: material/dice-multiple-outline
comments: false
---

???+ info "信息"

    - 默认省略导入

        - `import numpy as np`
        - `import pandas as pd`

    - 缩写

        - DataFrame缩写为"DF"
        - Series缩写为"SE"

    - 默认使用数据集: [learn_pandas.csv](https://share.ricolxwz.io/machine-learning/dataset/learn_pandas.csv)

        已经导入:  `df = pd.read_csv('data/learn_pandas.csv', usecols=['School', 'Grade', 'Name', 'Gender', 'Weight', 'Transfer'])`

## 索引器

### DF的列索引 {#DF的列索引}

#### 单个索引

通过`[列名]`或者`.列名`可以取出单个索引对应的列, 返回值为SE. 

???+ example "例子"

    ```
    [1]: df['Name'].head()
    0      Gaopeng Yang
    1    Changqiang You
    2           Mei Sun
    3      Xiaojuan Sun
    4       Gaojuan You
    Name: Name, dtype: object
    [2]: df.Name.head()
    0      Gaopeng Yang
    1    Changqiang You
    2           Mei Sun
    3      Xiaojuan Sun
    4       Gaojuan You
    Name: Name, dtype: object
    ```

##### 多个索引

通过`[列名组成的列表]`可以取出多个索引对应的列, 返回值为DF.

???+ example "例子"

    ```
    [1]: df[['Gender', 'Name']].head()
    x	Gender	Name
    0	Female	Gaopeng Yang
    1	Male	Changqiang You
    2	Male	Mei Sun
    3	Female	Xiaojuan Sun
    4	Male	Gaojuan You
    ```

### SE的行索引

???+ info "信息"

    已经生成:

    `s = pd.Series([1, 2, 3, 4, 5, 6], index=['a', 'b', 'a', 'a', 'a', 'c'])`

#### 以字符串为索引的SE

##### 单个索引

通过`[item]`可以取出单个索引对应的元素.

- 如果SE中只有单个值和该索引对应, 则返回这个标量值
- 如果SE中有多个值和该索引对应, 则返回一个SE

???+ example "例子"

    ```
    [1]: s['a']
    a    1
    a    3
    a    4
    a    5
    dtype: int64
    [2]: s['b']
    2
    ```

##### 多个索引 

通过`[item的列表]`可以取出多个索引对应的元素.

???+ example "例子"

    ```
    [1]: s[['c', 'b']]
    c    6
    b    2
    dtype: int64
    ```

###### 两个索引之间的元素 {#取出两个索引之间的元素}

可以使用切片. 

???+ warning "注意"

    - 这里的切片与NumPy的[切片](/numpy/array-index/#切片索引)有一点不同, 会包含两个端点
    - 实际上, 也不算是真正意义上的切片, 这个切片中的索引是基于{==元素==}的, 而不是基于{==位置==}的
    - 这两个索引在整个索引中必须唯一

        ???+ tip "Tip"

            如果这两个索引的值在整个索引中存在重复, 那么需要经过排序才能使用切片.

            ???+ example "例子"

                ```
                [1]: try:
                        s['a': 'b']
                     except Exception as e:
                        Err_Msg = e
                [2]: Err_Msg
                KeyError("Cannot get left slice bound for non-unique label: 'a'")
                [3]: s.sort_index()['a':'b']
                a    1
                a    3
                a    4
                a    5
                b    2
                dtype: int64
                ```

???+ example "例子"

    ```
    [1]: s['c': 'b': -2]
    c    6
    a    4
    b    2
    dtype: int64
    ```

#### 以整数为索引的SE

在使用读入函数的时候, 如果不特别使用`index_col`参数指定对应的列作为索引, 那么会生成从0开始的整数索引作为默认索引. 当然, 任意一组符合长度要求的整数都可以作为索引.

和字符串一样, 如果使用`[int]`或`[int_list]`, 可以取出对应索引{==元素==}的值.

???+ example "例子"

    ```
    [1]: s[1]
    1    a
    1    c
    dtype: object
    [2]: s[[2, 3]]
    2    d
    3    b
    dtype: object
    ```

???+ warning "注意"

    - 这里的切片与NumPy的[切片](/numpy/array-index/#切片索引)相同
    - 这里的切片中整数的含义是索引的{==位置==}, 而不是索引本身

        ???+ example "例子"

            如索引`1, 3, 1, 2, 5, 4`, 各个索引的位置分别为`0, 1, 2, 3, 4, 5`.

            ```
            [1]: s[1:-1:2]
            3    b
            2    d
            dtype: object
            ```

### `loc`/`iloc`索引器

对于DF而言, 不仅能根据[列进行索引](#DF的列索引), 还能根据行进行索引. `loc`索引器和`iloc`索引器是一种对于DF索引来说更加普适的方法, 它既能行索引, 又能列索引.

- `loc`索引器: 基于{==元素==}的索引器
- `iloc`索引器: 基于{==位置==}的索引器 

???+ tip "Tip"

    对于SE来说, 也可以使用`loc`和`iloc`索引器.

#### `loc`索引器

`loc`索引器的一般形式是`loc[*, *]`, 其中第一个`*`代表行的选择, 第二个`*`代表列的选择. 如果省略第二个位置写作`loc[*]`, 这个`*`指行的筛选. 

其中`*`的位置一共有五类合法对象, 分别是:

- 单个元素
- 元素列表
- 元素切片
- 布尔列表
- 函数

下面将依次说明.

???+ info "信息"

    为了演示相应操作, 先利用`set_index`函数将`Name`列设为索引

    `df_demo = df.set_index('Name')`

##### `*`为单个元素

直接取出相应的行或列, 如果该元素在索引中重复则结果为DF, 否则为SE.

???+ example "例子"

    ```
    [1]: df_demo.loc['Qian Sun'] # 多个人叫此名字
                                      School      Grade  Gender  Weight Transfer
    Name                                                                        
    Qiang Sun            Tsinghua University     Junior  Female    53.0        N
    Qiang Sun            Tsinghua University  Sophomore  Female    40.0        N
    Qiang Sun  Shanghai Jiao Tong University     Junior  Female     NaN        N
    [2]: df_demo.loc['Quan Zhao'] # 此名字唯一
    School      Shanghai Jiao Tong University
    Grade                              Junior
    Gender                             Female
    Weight                               53.0
    Transfer                                N
    Name: Quan Zhao, dtype: object
    ```

也可以同时选择行与列.

???+ example "例子"

    ```
    [1]: df_demo.loc['Qian Sun', 'School'] # 返回SE
    Name
    Qiang Sun              Tsinghua University
    Qiang Sun              Tsinghua University
    Qiang Sun    Shanghai Jiao Tong University
    Name: School, dtype: object
    [2]: df_demo.loc['Quan Zhao', 'School'] # 返回单个元素
    'Shanghai Jiao Tong University'
    ```

##### `*`为元素列表

取出列表中所有元素值对应的行或列.

???+ example "例子"

    ```
    [1]: df_demo.loc[['Qiang Sun', 'Quan Zhao'], ['School', 'Gender']]
                                      School  Gender
    Name                                            
    Qiang Sun            Tsinghua University  Female
    Qiang Sun            Tsinghua University  Female
    Qiang Sun  Shanghai Jiao Tong University  Female
    Quan Zhao  Shanghai Jiao Tong University  Female
    ```

##### `*`为切片

参考[这里](#取出两个索引之间的元素), 这不是真正意义上的切片, 这个切片是基于{==元素==}的. 

???+ warning "注意"

    - 要求起始元素和终止元素是唯一的, 如果不唯一会报错.
    - 这里的切片会包含两个端点

???+ example "例子"

    ```
    [1]: df_demo.loc['Gaojuan You':'Gaoqiang Qian', 'School':'Gender']
                                          School      Grade  Gender
    Name                                                           
    Gaojuan You                 Fudan University  Sophomore    Male
    Xiaoli Qian              Tsinghua University   Freshman  Female
    Qiang Chu      Shanghai Jiao Tong University   Freshman  Female
    Gaoqiang Qian            Tsinghua University     Junior  Female
    ```

##### `*`为布尔列表  {#为布尔列表}

在实际的数据处理中, 根据条件筛选行是极其常见的, 此处传入`loc`的布尔列表与DF长度相同, 且列表为`True`的位置所对应的行会被选中, `False`则会被剔除.

???+ example "例子"

    ```
    [1]: df_demo.loc[df_demo.Weight>70].head()
                                          School      Grade Gender  Weight Transfer
    Name                                                                           
    Mei Sun        Shanghai Jiao Tong University     Senior   Male    89.0        N
    Gaojuan You                 Fudan University  Sophomore   Male    74.0        N
    Xiaopeng Zhou  Shanghai Jiao Tong University   Freshman   Male    74.0        N
    Xiaofeng Sun             Tsinghua University     Senior   Male    71.0        N
    Qiang Zheng    Shanghai Jiao Tong University     Senior   Male    87.0  
    ```

也可以用过`isin`函数返回的布尔列表等价列出.

???+ example "例子"

    ```
    [1]: df_demo.loc[df_demo.Grade.isin(['Freshman', 'Senior'])].head()
                                          School     Grade  Gender  Weight Transfer
    Name                                                                            
    Gaopeng Yang    Shanghai Jiao Tong University  Freshman  Female    46.0        N
    Changqiang You              Peking University  Freshman    Male    70.0        N
    Mei Sun         Shanghai Jiao Tong University    Senior    Male    89.0        N
    Xiaoli Qian               Tsinghua University  Freshman  Female    51.0        N
    Qiang Chu       Shanghai Jiao Tong University  Freshman  Female    52.0        N
    ```

对于复合条件, 可以用`|`(或), `&`(且), `~`(取反)的组合来实现.

???+ example "例子"

    ```
    [1]: condition_1_1 = df_demo.school == 'Fudan University'
    [2]: condition_1_2 = df_demo.Grade == 'Senior'
    [3]: condition_1_3 = df_demo.Weight > 70
    [4]: condition_1 = condition_1_1 & condition_1_2 & condition_1_3
    [5]: condition_2_1 = df_demo.School == 'Peking University'
    [6]: condition_2_2 = df_demo.Grade == 'Senior'
    [7]: condition_2_3 = df_demo.Weight > 80
    [8]: condition_2 = condition_2_1 & (~condition_2_2) & condition_2_3
    [9]: df_demo.loc[condition_1 | condition_2]
                               School     Grade Gender  Weight Transfer
    Name                                                               
    Qiang Han       Peking University  Freshman   Male    87.0        N
    Chengpeng Zhou   Fudan University    Senior   Male    81.0        N
    Changpeng Zhao  Peking University  Freshman   Male    83.0        N
    Chengpeng Qian   Fudan University    Senior   Male    73.0        Y
    ```

##### `*`为函数

???+ note "笔记"

    - 这里的函数, 必须以前面的四种合法形式之一为返回值
    - 函数的输入值为DF本身

???+ example "例子"

    ```
    [1]: def condition(x):
            condition_1_1 = x.School == 'Fudan University'
            condition_1_2 = x.Grade == 'Senior'
            condition_1_3 = x.Weight > 70
            condition_1 = condition_1_1 & condition_1_2 & condition_1_3
            condition_2_1 = x.School == 'Peking University'
            condition_2_2 = x.Grade == 'Senior'
            condition_2_3 = x.Weight > 80
            condition_2 = condition_2_1 & (~condition_2_2) & condition_2_3
            result = condition_1 | condition_2
            return result
    [2]: df_demo.loc[condition]
                               School     Grade Gender  Weight Transfer
    Name                                                               
    Qiang Han       Peking University  Freshman   Male    87.0        N
    Chengpeng Zhou   Fudan University    Senior   Male    81.0        N
    Changpeng Zhao  Peking University  Freshman   Male    83.0        N
    Chengpeng Qian   Fudan University    Senior   Male    73.0        Y
    ```

???+ tip "Tip"

    支持使用Lambda表达式, 返回值同样必须是先前提到的四种形式之一.

    ???+ example "例子"

        === "例子1"

            ```
            [1]: df_demo.loc[lambda x:'Quan Zhao', lambda x:'Gender']
            'Female'
            ```

        === "例子2"

            由于函数无法返回`<start>:<end>:<step>`的切片形式, 所以返回切片时要用slice对象进行包装.

            ```
            [1]: df_demo.loc[lambda x:slice('Gaojuan You', 'Gaoqiang Qian')]
                                                  School      Grade  Gender  Weight Transfer
            Name                                                                            
            Gaojuan You                 Fudan University  Sophomore    Male    74.0        N
            Xiaoli Qian              Tsinghua University   Freshman  Female    51.0        N
            Qiang Chu      Shanghai Jiao Tong University   Freshman  Female    52.0        N
            Gaoqiang Qian            Tsinghua University     Junior  Female    50.0        N
            ```

#### `iloc`索引器

`iloc`的使用和`loc`完全类似, 只不过是针对{==位置==}进行筛选, 在相应的`*`位置处一共也有五类合法对象. 

???+ example "例子"

    ```
    [1]: df_demo.iloc[1, 1] # 第二行第二列
    'Freshman'
    [2]: df_demo.iloc[[0, 1], [0, 1]] # 前两行前两列
                                           School     Grade
    Name                                                   
    Gaopeng Yang    Shanghai Jiao Tong University  Freshman
    Changqiang You              Peking University  Freshman
    [3]: df_demo.iloc[1: 4, 2:4] # 切片不包含结束端点, 这里的切片是真正的切片
                    Gender  Weight
    Name                          
    Changqiang You    Male    70.0
    Mei Sun           Male    89.0
    Xiaojuan Sun    Female    41.0
    [4]: df_demo.iloc[lambda x: slice(1, 4)] # 传入切片为返回值的函数
                                           School      Grade  Gender  Weight Transfer
    Name                                                                             
    Changqiang You              Peking University   Freshman    Male    70.0        N
    Mei Sun         Shanghai Jiao Tong University     Senior    Male    89.0        N
    Xiaojuan Sun                 Fudan University  Sophomore  Female    41.0        N
    ```

???+ warning "注意"

    在使用布尔列表的时候要特别注意, 不能传入SE而必须传入序列的`values`, 否则会报错. `values`的作用是取出SE的值, 即一个NumPy数组.

    ???+ example "例子"

        ```
        [1]: df_demo.iloc[(df_demo.Weight>80).values].head()
                                               School      Grade Gender  Weight Transfer
        Name                                                                            
        Mei Sun         Shanghai Jiao Tong University     Senior   Male    89.0        N
        Qiang Zheng     Shanghai Jiao Tong University     Senior   Male    87.0        N
        Qiang Han                   Peking University   Freshman   Male    87.0        N
        Chengpeng Zhou               Fudan University     Senior   Male    81.0        N
        Feng Han        Shanghai Jiao Tong University  Sophomore   Male    82.0        N
        ```

### `query`函数 

在Pandas中, 支持把字符串形式的查询表达式传入`query`函数来查询数据, 其表达式的执行结果必须返回布尔列表. 

在进行复杂索引的时候, 由于这种检索方式无需像[普通方法](#为布尔列表)一样重复使用DF的名字来引用列名. 一般而言会使代码长度在不降低可读性的前提下有所减少.

???+ example "例子"

    例如, 在`loc`索引器一节的[复合条件查询例子](#为布尔列表)可以如下改写: 

    ```
    [1]: df.query('((School == "Fudan University")&'
                  ' (Grade == "Senior")&'
                  ' (Weight > 70))|'
                  '((School == "Peking University")&'
                  ' (Grade != "Senior")&'
                  ' (Weight > 80))')
                    School     Grade            Name Gender  Weight Transfer
    38   Peking University  Freshman       Qiang Han   Male    87.0        N
    66    Fudan University    Senior  Chengpeng Zhou   Male    81.0        N
    99   Peking University  Freshman  Changpeng Zhao   Male    83.0        N
    131   Fudan University    Senior  Chengpeng Qian   Male    73.0        Y
    ```

???+ note "笔记"

    - 在`query`表达式中, 帮用户注册了所有来自DF的列名, 所有属于该列/SE的方法都可以被调用, 和正常的函数调用并没有区别

        ???+ example "例子"

            ```
            [1]: df.query('Weight > Weight.mena()').head()
                                    School      Grade            Name  Gender  Weight Transfer
            1               Peking University   Freshman  Changqiang You    Male    70.0        N
            2   Shanghai Jiao Tong University     Senior         Mei Sun    Male    89.0        N
            4                Fudan University  Sophomore     Gaojuan You    Male    74.0        N
            10  Shanghai Jiao Tong University   Freshman   Xiaopeng Zhou    Male    74.0        N
            14            Tsinghua University     Senior    Xiaomei Zhou  Female    57.0        N
            ```

    - 在`query`表达式中, 还注册了若干英语的字面用法, 帮助提高可读性. 如`or, and, or, in, not in`

        ???+ example "例子"

            ```
            [1]: df.query('(Grade not in ["Freshman", "Sophomore"]) and (Gender == "Male")').head()
                                       School   Grade           Name Gender  Weight Transfer
            2   Shanghai Jiao Tong University  Senior        Mei Sun   Male    89.0        N
            16            Tsinghua University  Junior  Xiaoqiang Qin   Male    68.0        N
            17            Tsinghua University  Junior      Peng Wang   Male    65.0        N
            18            Tsinghua University  Senior   Xiaofeng Sun   Male    71.0        N
            21  Shanghai Jiao Tong University  Senior  Xiaopeng Shen   Male    62.0      NaN
            ```

    - `==`和`!=`分别等价于`in`和`not in`

        ???+ example "例子"

            ```
            [1]: df.query('Grade == ["Junior", "Senior"]').head()
                                       School   Grade           Name  Gender  Weight Transfer
            2   Shanghai Jiao Tong University  Senior        Mei Sun    Male    89.0        N
            7             Tsinghua University  Junior  Gaoqiang Qian  Female    50.0        N
            9               Peking University  Junior        Juan Xu  Female     NaN        N
            11            Tsinghua University  Junior    Xiaoquan Lv  Female    43.0        N
            12  Shanghai Jiao Tong University  Senior       Peng You  Female    48.0      NaN
            ```

???+ tip "Tip"

    - 对于含有空格的列名, 需要使用`col name`的方式进行引用
    - 若要引用外部变量, 只需要在变量名前面加`@`

        ???+ example "例子"

            ```
            [1]: low, high =70, 80
            [2]: df.query('(Weight >= @low) & (Weight <= @high)').head()
                                       School      Grade            Name Gender  Weight Transfer
            1               Peking University   Freshman  Changqiang You   Male    70.0        N
            4                Fudan University  Sophomore     Gaojuan You   Male    74.0        N
            10  Shanghai Jiao Tong University   Freshman   Xiaopeng Zhou   Male    74.0        N
            18            Tsinghua University     Senior    Xiaofeng Sun   Male    71.0        N
            35              Peking University   Freshman      Gaoli Zhao   Male    78.0        N
            ```

### 随机抽样

如果把DF的每一行看做一个样本, 或者把每一列看做一个特征, 再把整个DF看作总体, 想要对样本或者特征进行随机抽样就可以用`sample`函数. 有时候在拿到大型数据集后, 想要对统计特征进行计算来了解数据的大致分布, 但是这很费事件. 同时, 由于许多统计特征在等概率不放回的简单随机抽样条件下, 是总体统计特征的无偏估计(例如在多次抽样后, 所有抽样均值的期望等于所有元素的均值).

`sample`函数的主要参数为`<n>, <axis>, <frac>, <replace>, <weights>`, 前三个指的是抽样数量, 抽样方向(0为行, 1为列)和抽样比例(0.3为从总体中抽出30%的样本); `<replace>`和`<weights>`分别是指是否放回和每个样本的抽样相对概率, 当`replace=True`表示有放回抽样. 

???+ example "例子"

    下面构造的`df_sample`以`value`值的大小为抽样概率进行有放回的抽样, 抽样数量为3.

    ```
    [1]: df_sample = pd.DataFrame({'id': list('abcde'), 'value': [1, 2, 3, 4, 90]})
    [2]: df_sample
      id  value
    0  a      1
    1  b      2
    2  c      3
    3  d      4
    4  e     90
    [3]: df_sample.sample(3, replace=True, weights=df_sample.value)
      id  value
    4  e     90
    4  e     90
    4  e     90
    ```

## 多级索引

### 多级索引及其表结构

???+ example "例子"

    为了更加清晰地说明具有多级索引的DF的结构, 下面新构造一张表.

    ```
    [1]: np.random.seed(0)
    [2]: multi_index = pd.MultiIndex.from_product([list('ABCD'), df.Gender.unique()], names=('School', 'Gender'))
    [3]: multi_column = pd.MultiIndex.from_product([['Height', 'Weight'], df.Grade.unique()], names=('Indicator', 'Grade'))
    [4]: df_multi = pd.DataFrame(np.c_[(np.random.randn(8,4)*5 + 163).tolist(), (np.random.randn(8,4)*5 + 65).tolist()],
                                 index = multi_index,
                                 columns = multi_column).round(1)
    [5]: df_multi
    Indicator       Height                           Weight                        
    Grade         Freshman Senior Sophomore Junior Freshman Senior Sophomore Junior
    School Gender                                                                  
    A      Female    171.8  165.0     167.9  174.2     60.6   55.1      63.3   65.8
           Male      172.3  158.1     167.8  162.2     71.2   71.0      63.1   63.5
    B      Female    162.5  165.1     163.7  170.3     59.8   57.9      56.5   74.8
           Male      166.8  163.6     165.2  164.7     62.5   62.8      58.7   68.9
    C      Female    170.5  162.0     164.6  158.7     56.9   63.9      60.5   66.9
           Male      150.2  166.3     167.3  159.3     62.4   59.1      64.9   67.1
    D      Female    174.3  155.7     163.2  162.1     65.3   66.5      61.8   63.2
           Male      170.7  170.3     163.8  164.9     61.6   63.2      60.9   56.4
    ```

    下面通过颜色区分上述DF. 与单层索引的DF一样, 具备元素值, 行索引和列索引三个部分. 其中, 这里的行索引和列索引都是`MultiIndex`类型, 只不过索引中的第一个元素为元祖而不是单层索引中的标量. 例如, 行索引的第四个元素为`("B", "Male")`, 列索引的第二个元素为`("Height", "Senior")`.

    ![](https://inter.joyfulpandas.datawhale.club/_images/multi_index.png)

    与单层索引类似, `MultiIndex`也具有名字属性. 图中的`School`和`Gender`分别对应了DF的第一层和第二层行索引的名字. `Indicator`和`Grade`分别对应了第一层和第二层列索引的名字.

    ???+ tip "Tip"

        - 索引的名字和属性分别可以通过`names`和`values`获得.

            ???+ example "例子"

                ```
                [1]: df_multi.index.names
                FrozenList(['School', 'Gender'])
                [2]: df_multi.columns.names
                FrozenList(['Indicator', 'Grade'])
                [3]: df_multi.index.values
                array([('A', 'Female'), ('A', 'Male'), ('B', 'Female'), ('B', 'Male'),
                    ('C', 'Female'), ('C', 'Male'), ('D', 'Female'), ('D', 'Male')],
                    dtype=object)
                [4]: df_multi.columns.values
                array([('Height', 'Freshman'), ('Height', 'Senior'),
                    ('Height', 'Sophomore'), ('Height', 'Junior'),
                    ('Weight', 'Freshman'), ('Weight', 'Senior'),
                    ('Weight', 'Sophomore'), ('Weight', 'Junior')], dtype=object)
                ```

        - 若要获得某一层的索引, 则需要通过`get_level_values`获取:

            ???+ example "例子"

                ```
                [1]: df_multi.index.get_level_values(0)
                Index(['A', 'A', 'B', 'B', 'C', 'C', 'D', 'D'], dtype='object', name='School')
                ```

### 多级索引中的`loc`/`iloc`索引器

???+ example "例子"

    熟悉了结构之后, 回到原表, 使用`set_index`函数将学校和年级作为索引, 此时的行为多级索引, 列为单级索引, 由于默认状态的列表不含名字, 因此对应于刚刚图中`Indicator`和`Grade`的索引名位置是空缺的.

    ```
    [1]: df_multi = df.set_index(["school", "Grade"])
    [2]: df_multi.head()
                                                       Name  Gender  Weight Transfer
    School                        Grade                                             
    Shanghai Jiao Tong University Freshman     Gaopeng Yang  Female    46.0        N
    Peking University             Freshman   Changqiang You    Male    70.0        N
    Shanghai Jiao Tong University Senior            Mei Sun    Male    89.0        N
    Fudan University              Sophomore    Xiaojuan Sun  Female    41.0        N
                                  Sophomore     Gaojuan You    Male    74.0        N
    ```

    多级索引使用`loc`/`iloc`索引器只需要将对应的标量改为元组就可以了.

    ???+ tip "Tip"

        在传入元组列表或单个元组或返回前两者的函数的时候, 需要先进行排序以避免性能警告.

        ???+ example "例子"

            ```
            [1]: with warnings.catch_warnings():
                     warnings.filterwarnings('error')
                     try:
                         df_multi.loc[('Fudan University', 'Junior')].head()
                     except Warning as w:
                         Warning_Msg = w
            [2]: Warning_Msg
            pandas.errors.PerformanceWarning('indexing past lexsort depth may impact performance.')
            ```

    ```
    [1]: df_sorted = df_multi.sort_index()
    [2]: df_sorted.loc[('Fudan University', 'Junior')].head()
                                        Name  Gender  Weight Transfer
    School           Grade                                         
    Fudan University Junior      Yanli You  Female    48.0        N
                     Junior  Chunqiang Chu    Male    72.0        N
                     Junior   Changfeng Lv    Male    76.0        N
                     Junior     Yanjuan Lv  Female    49.0      NaN
                     Junior  Gaoqiang Zhou  Female    43.0        N
    [3]: df_sorted.loc[[('Fudan University', 'Senior'), ('Shanghai Jiao Tong University', 'Freshman')]]
    School                        Grade       Name              Gender  Weight  Transfer
    Fudan University              Senior      Chengpeng Zheng   Female  38.0    N
                                  Senior      Feng Zhou         Female  47.0    N
                                  Senior      Gaomei Lv         Female  34.0    N
                                  Senior      Chunli Lv         Female  56.0    N
                                  Senior      Chengpeng Zhou    Male    81.0    N
                                  Senior      Gaopeng Qin       Female  52.0    N
                                  Senior      Chunjuan Xu       Female  47.0    N
                                  Senior      Juan Zhang        Female  47.0    N
                                  Senior      Chengpeng Qian    Male    73.0    Y
                                  Senior      Xiaojuan Qian     Female  50.0    N
                                  Senior      Quan Xu           Female  44.0    N
    Shanghai Jiao Tong University Freshman    Gaopeng Yang      Female  46.0    N
                                  Freshman    Qiang Chu         Female  52.0    N
                                  Freshman    Xiaopeng Zhou     Male    74.0    N
                                  Freshman    Yanpeng Lv        Male    65.0    N
                                  Freshman    Xiaopeng Zhao     Female  53.0    N
                                  Freshman    Chunli Zhao       Male    83.0    N
                                  Freshman    Peng Zhang        Female  NaN     N
                                  Freshman    Xiaoquan Sun      Female  40.0    N
                                  Freshman    Chunmei Shi       Female  52.0    N
                                  Freshman    Xiaomei Yang      Female  49.0    N
                                  Freshman    Xiaofeng Qian     Female  49.0    N
                                  Freshman    Changmei Lv       Male    75.0    N
                                  Freshman    Qiang Feng        Male    80.0    N
    [4]: df_sorted.loc[df_sorted.Weight > 70].head()
                                            Name Gender  Weight Transfer
    School           Grade                                           
    Fudan University Freshman       Feng Wang   Male    74.0        N
                     Junior     Chunqiang Chu   Male    72.0        N
                     Junior      Changfeng Lv   Male    76.0        N
                     Senior    Chengpeng Zhou   Male    81.0        N
                     Senior    Chengpeng Qian   Male    73.0        Y
    [5]: df_sorted.loc[lambda x:('Fudan University','Junior')].head()
                                        Name  Gender  Weight Transfer
    School              Grade                                         
    Fudan University    Junior      Yanli You  Female    48.0        N
                        Junior  Chunqiang Chu    Male    72.0        N
                        Junior   Changfeng Lv    Male    76.0        N
                        Junior     Yanjuan Lv  Female    49.0      NaN
                        Junior  Gaoqiang Zhou  Female    43.0        N
    ```

    ???+ warning "注意"

        在使用切片的时候需要注意, 在单级索引中只要切片端点元素是唯一的, 那么就可以进行切片; 但是在多级索引中, 无论元素在索引中是否重复出现, 都必须经过排序才能使用切片, 否则报错.

        ???+ example "例子"

            ```
            [1]: try:
                     df_multi.loc[('Fudan University', 'Senior'):].head()
                 except Exception as e:
                     Err_Msg = e
            [2]: Err_Msg
            pandas.errors.UnsortedIndexError('Key length (2) was greater than MultiIndex lexsort depth (0)')
            [3]: df_sorted.loc[('Fudan University', 'Senior'):].head()
                                                Name  Gender  Weight Transfer
            School           Grade                                           
            Fudan University Senior  Chengpeng Zheng  Female    38.0        N
                             Senior        Feng Zhou  Female    47.0        N
                             Senior        Gaomei Lv  Female    34.0        N
                             Senior        Chunli Lv  Female    56.0        N
                             Senior   Chengpeng Zhou    Male    81.0        N
            [4]: df_unique = df.drop_duplicates(subset=['School', 'Grade']).set_index(['School', 'Grade'])
            [5]: df_unique.head()
                                                               Name  Gender  Weight Transfer
            School                        Grade                                             
            Shanghai Jiao Tong University Freshman     Gaopeng Yang  Female    46.0        N
            Peking University             Freshman   Changqiang You    Male    70.0        N
            Shanghai Jiao Tong University Senior            Mei Sun    Male    89.0        N
            Fudan University              Sophomore    Xiaojuan Sun  Female    41.0        N
            Tsinghua University           Freshman      Xiaoli Qian  Female    51.0    
            [6]: try:
                     df_unique.loc[('Fudan University', 'Senior'):].head()
                 except Exception as e:
                     Err_Msg = e
            [7]: Err_Msg
            pandas.errors.UnsortedIndexError('Key length (2) was greater than MultiIndex lexsort depth (0)')
            [8]: df_unique.sort_index().loc[('Fudan University', 'Senior'):].head()
                                                    Name  Gender  Weight Transfer
            School            Grade                                              
            Fudan University  Senior     Chengpeng Zheng  Female    38.0        N
                              Sophomore     Xiaojuan Sun  Female    41.0        N
            Peking University Freshman    Changqiang You    Male    70.0        N
                              Junior             Juan Xu  Female     NaN        N
                              Senior          Changli Lv  Female    41.0        N
            ```

    ???+ tip "Tip"

        在多级索引中的元组中有一种特殊的用法, 可以对多层的元素进行交叉组合后索引. 

        ???+ example "例子"

            ```
            [1]: res = df_multi.loc[(['Peking University', 'Fudan University'], ['Sophomore', 'Junior']), :].head(10)
            [2]: res
                                                  Name  Gender  Height  Weight Transfer
            School            Grade                                                    
            Peking University Sophomore    Changmei Xu  Female   151.6    43.0        N
                              Sophomore   Xiaopeng Qin    Male   172.8     NaN        N
                              Sophomore         Mei Xu  Female   154.2    39.0        N
                              Sophomore    Xiaoli Zhou  Female   166.8    55.0        N
                              Sophomore       Peng Han  Female   147.8    34.0      NaN
                              Junior           Juan Xu  Female   164.8     NaN        N
                              Junior     Changjuan You  Female   161.4    47.0        N
                              Junior          Gaoli Xu  Female   157.3    48.0        N
                              Junior      Gaoquan Zhou    Male   166.8    70.0        N
                              Junior         Qiang You  Female   170.0    56.0        N
            ```

            ???+ warning "注意"

                - `loc/iloc[([exp1], [exp2], ..., [expN])]`和`loc/iloc[[exp1], [exp2], ..., [expN]]`是相等的, 后者是前者的语法糖. 所以上述的格式应该写成`(level_0_list, level_1_list), cols`, 而不能省略`cols`, 否则`level_1_list`会被作为列索引看待.

                    ???+ example "例子"

                        ```
                        [1]: res = df_multi.loc[(['Peking University', 'Fudan University'], ['Sophomore', 'Junior'])].head(10) # 这里省略了`cols`, 所以['Sophomore', 'Junior']被当成了列索引
                        [2]: res
                        KeyError: "None of [Index(['Sophomore', 'Junior'], dtype='object')] are in the [columns]"
                        ```

                - 注意和元组的列表区分, 它们的意义时不同的.

                    ???+ example "例子"

                        ```
                        [1]: res = df_multi.loc[[('Peking University', 'Junior'), ('Fudan University', 'Sophomore')]].head(10)
                        [2]: res
                                                               Name  Gender  Height  Weight Transfer
                        School            Grade                                                     
                        Peking University Junior            Juan Xu  Female   164.8     NaN        N
                                          Junior      Changjuan You  Female   161.4    47.0        N
                                          Junior           Gaoli Xu  Female   157.3    48.0        N
                                          Junior       Gaoquan Zhou    Male   166.8    70.0        N
                                          Junior          Qiang You  Female   170.0    56.0        N
                                          Junior       Chengli Zhao    Male     NaN     NaN      NaN
                                          Junior     Chengpeng Zhao  Female   156.0    44.0        N
                                          Junior      Xiaofeng Zhao  Female   159.9    46.0        N
                        Fudan University  Sophomore    Xiaojuan Sun  Female     NaN    41.0        N
                                          Sophomore     Gaojuan You    Male   174.0    74.0        N
                        ```

### `IndexSlice`对象

前面介绍的方法, 即使在索引不重复的时候, 也只能对元组整体进行切片, 而不能对每层进行切片, 也不允许讲切片和布尔列表混合使用, 引入`IndexSlice`对象就能解决这个问题.

该对象一共有两种形式: 

- `loc[idx[*, *]]`型
- `loc[idx[*, *], idx[*, *]]`型

???+ info "信息"

    为了方便演示, 下面构造一个索引不重复的DF:

    ```
    [1]: np.random.seed(0)
    [2]: L1 = ["A", "B", "C"], ["a", "b", "c"]
    [3]: mul_index1 = pd.MultiIndex.from_product([L1, L2], names=('Upper', 'Lower'))
    [4]: L3 = ["D", "E", "F"], ["d", "e", "f"]
    [5]: mul_index2 = pd.MultiIndex.from_product([L3, L4], names=('Big', 'Small'))
    [6]: df_ex = pd.DataFrame(np.random.randint(-9, 10, (9, 9)), index=mul_index1, columns=mul_index2)
    [7]: df_ex
    Big          D        E        F      
    Small        d  e  f  d  e  f  d  e  f
    Upper Lower                           
    A     a      3  6 -9 -6 -6 -2  0  9 -5
          b     -3  3 -8 -3 -2  5  8 -4  4
          c     -1  0  7 -4  6  6 -9  9 -6
    B     a      8  5 -2 -9 -8  0 -9  1 -6
          b      2  9 -7 -9 -9 -5 -4 -3 -1
          c      8  6 -5  0  1 -8 -8 -2  0
    C     a     -6 -3  2  5  9 -9  5 -6  3
          b      1  2 -5 -3 -5  6 -6  3 -5
          c     -1  5  6 -6  6  4  7  8 -4
    ```

    为了使用`IndexSlice`对象, 需要进行定义:

    ```
    [1]: idx = pd.IndexSlice
    ```
    
#### `loc[idx[*, *]]`型

这种情况不能进行多层分别切片, 前一个`*`表示行的选择, 后一个`*`表示列的选择, 与单纯的`loc`是类似的.

???+ example "例子"

    ```
    [1]: df_ex.loc[idx['C':, ('D', 'f'):]]
    Big          D  E        F      
    Small        f  d  e  f  d  e  f
    Upper Lower                     
    C     a      2  5  9 -9  5 -6  3
          b     -5 -3 -5  6 -6  3 -5
          c      6 -6  6  4  7  8 -4
    ```

另外, 也支持布尔序列的索引:

???+ example "例子"

    ```
    [1]: df_ex.loc[idx[:'A', lambda x:x.sum()>0]]
    Big          D     F
    Small        d  e  e
    Upper Lower         
    A     a      3  6  9
          b     -3  3 -4
          c     -1  0  9
    ```

#### `loc[idx[*, *], idx[*, *]]`型

这种情况能够分层进行切片, 前一个`idx`指代的是行索引, 后一个是列索引.

???+ example "例子"

    ```
    [1]: df_ex.loc[idx[:'A', 'b':], idx['E':, 'e':]]
    Big          E     F   
    Small        e  f  e  f
    Upper Lower            
    A     b     -2  5 -4  4
          c      6  6  9 -6
    ```

需要注意的是, 此时不支持使用函数.

???+ example "例子"

    ```
    [1]: try:
             df_ex.loc[idx[:'A', lambda x: 'b'], idx['E':, 'e':]]
         except Exception as e:
             Err_Msg = e
    [2]: Err_Msg
    KeyError(<function __main__.<lambda>(x)>)
    ```

### 多级索引的构造

前面提到了多级索引表的结构, 那么除了使用`set_index`(详情见[这里](#索引的设置与重置))之外, 如何自己构造多级索引呢?

常用的有`from_tuples`, `from_arrays`, `from_product`三种方法, 它们都是`pd.MultiIndex`对象下的函数.

#### `from_tuples`函数

`from_tuples`函数根据传入由元组组成的列表进行构造.

???+ example "例子"

    ```
    [1]: my_tuple = [('a','cat'),('a','dog'),('b','cat'),('b','dog')]
    [2]: pd.MultiIndex.from_tuples(my_tuple, names=['First','Second'])
    MultiIndex([('a', 'cat'),
                ('a', 'dog'),
                ('b', 'cat'),
                ('b', 'dog')],
               names=['First', 'Second'])
    ```

#### `from_arrays`函数

`from_arrays`函数根据传入列表中对应层的列表进行构造.

???+ example "例子"

    ```
    [1]: my_array = [list('aabb'), ['cat', 'dog']*2]
    [2]: pd.MultiIndex.from_arrays(my_array, names=['First','Second'])
    MultiIndex([('a', 'cat'),
                ('a', 'dog'),
                ('b', 'cat'),
                ('b', 'dog')],
               names=['First', 'Second'])
    ```

#### `from_product`函数

`from_product`函数根据给定多个列表的笛卡尔积进行构造.

???+ example "例子"

    ```
    [1]: my_list1 = ['a','b']
    [2]: my_list2 = ['cat','dog']
    [3]: pd.MultiIndex.from_product([my_list1, my_list2], names=['First','Second'])
    MultiIndex([('a', 'cat'),
                ('a', 'dog'),
                ('b', 'cat'),
                ('b', 'dog')],
               names=['First', 'Second'])
    ```

## 索引的常用方法

### 索引层的交换和删除

???+ info "信息"

    为了方便理解交换的过程, 这里构造了一个三级索引的例子:

    ```
    [1]: np.random.seed(0)
    [2]: L1, L2, L3 = ['A', 'B'], ['a', 'b'], ['alpha', 'beta']
    [3]: mul_index1 = pd.MultiIndex.from_product([L1, L2, L3], names=('Upper', 'Lower', 'Extra'))
    [4]: df_ex = pd.DataFrame(np.random.randint(-9,10,(8,8)), index=mul_index1, columns=mul_index2)
    [5]: df_ex
    Big                 C               D            
    Small               c       d       c       d    
    Other             cat dog cat dog cat dog cat dog
    Upper Lower Extra                                
    A     a     alpha   3   6  -9  -6  -6  -2   0   9
                beta   -5  -3   3  -8  -3  -2   5   8
          b     alpha  -4   4  -1   0   7  -4   6   6
                beta   -9   9  -6   8   5  -2  -9  -8
    B     a     alpha   0  -9   1  -6   2   9  -7  -9
                beta   -9  -5  -4  -3  -1   8   6  -5
          b     alpha   0   1  -8  -8  -2   0  -6  -3
                beta    2   5   9  -9   5  -6   3   1
    ```

索引层的交换由`swaplevel`和`reorder_levels`完成, 前者只能交换两个层, 而后者可以交换任意层, 两者都可以通过`axis`参数指定交换的是轴是哪一个, 即行索引或列索引.

???+ example "例子"

    ```
    [1]: df_ex.swaplevel(0,2,axis=1).head() # 列索引的第一层和第三层交换
    Other             cat dog cat dog cat dog cat dog
    Small               c   c   d   d   c   c   d   d
    Big                 C   C   C   C   D   D   D   D
    Upper Lower Extra                                
    A     a     alpha   3   6  -9  -6  -6  -2   0   9
                beta   -5  -3   3  -8  -3  -2   5   8
          b     alpha  -4   4  -1   0   7  -4   6   6
                beta   -9   9  -6   8   5  -2  -9  -8
    B     a     alpha   0  -9   1  -6   2   9  -7  -9
    [2]: df_ex.reorder_levels([2,0,1],axis=0).head() # 列表数字指代原来索引中的层
    Big                 C               D            
    Small               c       d       c       d    
    Other             cat dog cat dog cat dog cat dog
    Extra Upper Lower                                
    alpha A     a       3   6  -9  -6  -6  -2   0   9
    beta  A     a      -5  -3   3  -8  -3  -2   5   8
    alpha A     b      -4   4  -1   0   7  -4   6   6
    beta  A     b      -9   9  -6   8   5  -2  -9  -8
    alpha B     a       0  -9   1  -6   2   9  -7  -9
    ```

若想要删除某一层的索引, 可以使用`droplevel`.

???+ example "例子"

    ```
    [1]: df_ex.droplevel(1,axis=1)
    Big                 C               D            
    Other             cat dog cat dog cat dog cat dog
    Upper Lower Extra                                
    A     a     alpha   3   6  -9  -6  -6  -2   0   9
                beta   -5  -3   3  -8  -3  -2   5   8
          b     alpha  -4   4  -1   0   7  -4   6   6
                beta   -9   9  -6   8   5  -2  -9  -8
    B     a     alpha   0  -9   1  -6   2   9  -7  -9
                beta   -9  -5  -4  -3  -1   8   6  -5
          b     alpha   0   1  -8  -8  -2   0  -6  -3
                beta    2   5   9  -9   5  -6   3   1
    [2]: df_ex.droplevel([0,1],axis=0)
    Big     C               D            
    Small   c       d       c       d    
    Other cat dog cat dog cat dog cat dog
    Extra                                
    alpha   3   6  -9  -6  -6  -2   0   9
    beta   -5  -3   3  -8  -3  -2   5   8
    alpha  -4   4  -1   0   7  -4   6   6
    beta   -9   9  -6   8   5  -2  -9  -8
    alpha   0  -9   1  -6   2   9  -7  -9
    beta   -9  -5  -4  -3  -1   8   6  -5
    alpha   0   1  -8  -8  -2   0  -6  -3
    beta    2   5   9  -9   5  -6   3   1
    ```

### 索引属性的修改

通过`rename_axis`可以对索引层的名字进行修改, 常用的修改方式是传入字典的映射.

???+ example "例子"

    ```
    [1]: df_ex.rename_axis(index={'Upper': 'Changed_row'}, columns={'Other': 'Changed_Col'}).head()
    Big                       C               D            
    Small                     c       d       c       d    
    Changed_Col             cat dog cat dog cat dog cat dog
    Changed_row Lower Extra                                
    A           a     alpha   3   6  -9  -6  -6  -2   0   9
                      beta   -5  -3   3  -8  -3  -2   5   8
                b     alpha  -4   4  -1   0   7  -4   6   6
                      beta   -9   9  -6   8   5  -2  -9  -8
    B           a     alpha   0  -9   1  -6   2   9  -7  -9
    ```

通过`rename`可以对索引的值进行修改, 如果是多级索引需要指定修改的层号`level`.

???+ example "例子"

    ```
    [1]: df_ex.rename(columns={'cat': 'not_cat'}, level=2).head()
    Big                     C                       D                
    Small                   c           d           c           d    
    Other             not_cat dog not_cat dog not_cat dog not_cat dog
    Upper Lower Extra                                                
    A     a     alpha       3   6      -9  -6      -6  -2       0   9
                beta       -5  -3       3  -8      -3  -2       5   8
          b     alpha      -4   4      -1   0       7  -4       6   6
                beta       -9   9      -6   8       5  -2      -9  -8
    B     a     alpha       0  -9       1  -6       2   9      -7  -9
    ```

传入参数也可以是函数, 其输入值就是索引元素.

???+ example "例子"

    ```
    [1]: df_ex.rename(index=lambda x:str.upper(x), level=2).head()
    Big                 C               D            
    Small               c       d       c       d    
    Other             cat dog cat dog cat dog cat dog
    Upper Lower Extra                                
    A     a     ALPHA   3   6  -9  -6  -6  -2   0   9
                BETA   -5  -3   3  -8  -3  -2   5   8
          b     ALPHA  -4   4  -1   0   7  -4   6   6
                BETA   -9   9  -6   8   5  -2  -9  -8
    B     a     ALPHA   0  -9   1  -6   2   9  -7  -9
    ```

对于整个索引的元素替换, 可以利用迭代器实现.

???+ example "例子"

    ```
    [1]: new_values = iter(list('abcdefgh'))
    [2]: df_ex.rename(index=lambda x:next(new_values), level=2)
    Big                 C               D            
    Small               c       d       c       d    
    Other             cat dog cat dog cat dog cat dog
    Upper Lower Extra                                
    A     a     a       3   6  -9  -6  -6  -2   0   9
                b      -5  -3   3  -8  -3  -2   5   8
          b     c      -4   4  -1   0   7  -4   6   6
                d      -9   9  -6   8   5  -2  -9  -8
    B     a     e       0  -9   1  -6   2   9  -7  -9
                f      -9  -5  -4  -3  -1   8   6  -5
          b     g       0   1  -8  -8  -2   0  -6  -3
                h       2   5   9  -9   5  -6   3   1
    ```

???+ tip "Tip"

    - 可以使用定义在`index`属性上的`map`函数, 直接传入索引的元祖.

        ???+ example "例子"

            ```
            [1]: df_temp = df_ex.copy()
            [2]: new_idx = df_temp.index.map(lambda x: (x[0], x[1], str.upper(x[2])))
            [3]: df_temp.index = new_idx
            [4]: df_temp.head()
            Big                 C               D            
            Small               c       d       c       d    
            Other             cat dog cat dog cat dog cat dog
            Upper Lower Extra                                
            A     a     ALPHA   3   6  -9  -6  -6  -2   0   9
                        BETA   -5  -3   3  -8  -3  -2   5   8
                  b     ALPHA  -4   4  -1   0   7  -4   6   6
                        BETA   -9   9  -6   8   5  -2  -9  -8
            B     a     ALPHA   0  -9   1  -6   2   9  -7  -9
            ```

    - 关于`map`的另一个使用方法是对多级索引进行压缩.

        ???+ example "例子"

            ```
            [1]: df_temp = df_ex.copy()
            [2]: new_idx = df_temp.index.map(lambda x: (x[0]+'-'+x[1]+'-'+x[2]))
            [3]: df_temp.index = new_idx
            [4]: df_temp.head() # 单层索引
            Big         C               D            
            Small       c       d       c       d    
            Other     cat dog cat dog cat dog cat dog
            A-a-alpha   3   6  -9  -6  -6  -2   0   9
            A-a-beta   -5  -3   3  -8  -3  -2   5   8
            A-b-alpha  -4   4  -1   0   7  -4   6   6
            A-b-beta   -9   9  -6   8   5  -2  -9  -8
            B-a-alpha   0  -9   1  -6   2   9  -7  -9
            ```
    
    - 同样, 也可以使用`map`解压缩

        ???+ example "例子"

            ```
            [1]: new_idx = df_temp.index.map(lambda x:tuple(x.split('-')))
            [2]: df_temp.index = new_idx
            [3]: df_temp.head() # 三层索引
            Big         C               D            
            Small       c       d       c       d    
            Other     cat dog cat dog cat dog cat dog
            A a alpha   3   6  -9  -6  -6  -2   0   9
                beta   -5  -3   3  -8  -3  -2   5   8
              b alpha  -4   4  -1   0   7  -4   6   6
                beta   -9   9  -6   8   5  -2  -9  -8
            B a alpha   0  -9   1  -6   2   9  -7  -9
            ```

### 索引的设置与重置 {#索引的设置与重置}

???+ info "信息"

    为了说明本节的函数, 下面构造一个新表.

    ```
    [1]: df_new = pd.DataFrame({'A': list('aacd'), 'B': list('PQRT'), 'C': [1, 2, 3, 4]})
    [2]: df_new
       A  B  C
    0  a  P  1
    1  a  Q  2
    2  c  R  3
    3  d  T  4
    ```

索引的设置可以使用`set_index`完成, 主要参数是`append`, 表示是否保留原来的索引, 直接把心设定的添加到原索引的内层.

???+ example "例子"

    ```
    [1]: df_new.set_index('A')
       B  C
    A      
    a  P  1
    a  Q  2
    c  R  3
    d  T  4
    [2]: df_new.set_index('A', append=True)
         B  C
    A      
    0 a  P  1
    1 a  Q  2
    2 c  R  3
    3 d  T  4
    ```

可以同时指定多个列作为索引:

???+ example "例子"

    ```
    [1]: df_new.set_index(['A', 'B'])
         C
    A B   
    a P  1
      Q  2
    c R  3
    d T  4
    ```

???+ tip "Tip"

    如果想要添加的列没有出现在其中, 可以直接在参数中传入相应的SE:

    ???+ example "例子"

        ```
        [1]: my_index = pd.Series(list('WXYZ', name='D'))
        [2]: df_new = df_new.set_index(['A', my_index])
        [3]: df_new
             B  C
        A D      
        a W  P  1
          X  Q  2
        c Y  R  3
        d Z  T  4
        ```

`reset_index`是`set_index`的逆函数, 主要参数是`drop`, 表示是否要把去掉的索引层丢弃, 而不是添加到列中.

???+ example "例子"

    ```
    [1]: df_new.reset_index(['D'])
       D  B  C
    A         
    a  W  P  1
    a  X  Q  2
    c  Y  R  3
    d  Z  T  4
    [2]: df_new.reset_index(['D'], drop=True)
       B  C
    A      
    a  P  1
    a  Q  2
    c  R  3
    d  T  4
    ```

如果重置了所有索引, 会重新生成一个默认索引.

???+ example "例子"

    ```
    [1]: df_new.reset_index()
    Out[160]: 
       A  D  B  C
    0  a  W  P  1
    1  a  X  Q  2
    2  c  Y  R  3
    3  d  Z  T  4
    ```

### 索引的变形

在某些场合下, 需要对索引做一些扩充或者剔除, 更具体地要求是给定一个新的索引, 把原表中相应的索引对应元素填充到新索引构成的表中, 可以使用`reindex`函数.

???+ example "例子"

    ```
    [1]: df_reindex = pd.DataFrame({"Weight": [60, 70, 80], "Height": [176, 180, 179]}, index=['1001', '1003', '1002'])
    [2]: df_reindex
          Weight  Height
    1001      60     176
    1003      70     180
    1002      80     179
    [3]: df_reindex.reindex(index=['1001','1002','1003','1004'], columns=['Weight','Gender'])
          Weight  Gender
    1001    60.0     NaN
    1002    80.0     NaN
    1003    70.0     NaN
    1004     NaN     NaN
    ```

    这种需求经常出现在时间序列索引的时间点填充以及id编号的扩充. 另外, 需要注意的是原来表中的数据和新表中回根据索引自动对齐. 

还有一个与`reindex`功能类似的函数是`reindex_like`, 其功能是仿照传入的表索引来进行被调用表索引的变形.

???+ example "例子"

    ```
    [1]: df_existed = pd.DataFrame(index=['1001','1002','1003','1004'], columns=['Weight','Gender'])
    [2]: df_reindex.reindex_like(df_existed)
          Weight  Gender
    1001    60.0     NaN
    1002    80.0     NaN
    1003    70.0     NaN
    1004     NaN     NaN
    ```

## 索引运算

经常会有一种利用集合运算来取出符合条件行的需求. 由于集合的元素是互异的, 但是索引中可能有相同的元素, 先用`unique`去重后再进行运算.

???+ example "例子"

    ```
    [1]: df_set_1 = pd.DataFrame([[0,1],[1,2],[3,4]], index = pd.Index(['a','b','a'],name='id1'))
    [2]: df_set_2 = pd.DataFrame([[4,5],[2,6],[7,1]], index = pd.Index(['b','b','c'],name='id2'))
    [3]: id1, id2 = df_set_1.index.unique(), df_set_2.index.unique()
    [4]: id1.intersection(id2)
    Index(['b'], dtype='object')
    [5]: id1.union(id2)
    Index(['a', 'b', 'c'], dtype='object')
    [6]: id1.difference(id2)
    Index(['a'], dtype='object')
    [7]: id1.symmetric_difference(id2)
    Index(['a', 'c'], dtype='object')
    ```

???+ tip "Tip"

    若两张表需要做集合运算的列没有被设置为索引, 一种办法是先转成索引(如上), 运算后再恢复(如下), 另一种方法使用`isin`函数.

    ???+ example "例子"

        ```
        [1]: df_set_in_col_1 = df_set_1.reset_index()
        [2]: df_set_in_col_2 = df_set_2.reset_index()
        [3]: df_set_in_col_1[df_set_in_col_1.id1.isin(df_set_in_col_2.id2)]
          id1  0  1
        1   b  1  2
        ```

[^1]: 第三章 索引—Joyful Pandas 1.0 documentation. (n.d.). Retrieved June 29, 2024, from https://inter.joyfulpandas.datawhale.club/Content/ch3.html