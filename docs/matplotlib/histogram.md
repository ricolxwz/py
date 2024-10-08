---
title: 直方图
icon: material/chart-histogram
comments: false
---

???+ info "信息"

    默认省略导入:

    - `import numpy as np`
    - `import pandas as pd`
    - `import matplotlib.pyplot as plt`

可以使用`pyplot`中的`hist`函数来绘制柱形图.

`hist`函数语法格式如下:

```
matplotlib.pyplot.hist(x, bins, range, density, weights, cumulative, bottom, histtype, align, orientation, rwidth, log, color, label, stacked, **kwargs)
```

参数说明:

- `x`: 表示要绘制直方图的数据, 可以是一个一维数组或列表
- `bins`: 表示直方图的箱数, 默认为`10`
- `range`: 表示直方图的值域, 默认为`None`
- `density`: 表示是否将直方图归一化, 默认为`False`
- `weights`: 表示每个数据点的权重, 默认为`None`
- `cumulative`: 表示是否绘制累积分布图, 默认为`False`
- `bottom`: 表示直方图的起始高度, 默认为`None`
- `histtype`: 表示直方图的类型, 默认为`bar`
- `align`: 表示直方图箱子的对其方式, 默认为`mid`
- `orientation`: 表示直方图的方向, 默认为`vertical`
- `rwidth`: 表示每个箱子的宽度, 默认为`None`
- `log`: 表示是否在y轴上使用对数刻度, 默认为`False`
- `color`: 表示直方图的颜色
- `label`: 表示直方图的标签
- `stacked`: 表示是否堆叠不同的直方图
- `**kwargs`: 表示其他绘图参数

## 例子

### 普通例子

???+ example "例子"

    定义:

    ```
    data = np.random.randn(1000)

    plt.hist(data, bins=30, color='skyblue', alpha=0.8)

    plt.title('hist() Test')
    plt.xlabel('Value')
    plt.ylabel('Frequency')

    plt.show()
    ```

    输出:

    ![](https://img.ricolxwz.io/1d10ff65b389307a3472e916660216e2.png){:style="width:400px"}

### 多批数据

???+ example "例子"

    定义:

    ```
    data1 = np.random.normal(0, 1, 1000)
    data2 = np.random.normal(2, 1, 1000)
    data3 = np.random.normal(-2, 1, 1000)

    plt.hist(data1, bins=30, alpha=0.5, label='Data 1')
    plt.hist(data2, bins=30, alpha=0.5, label='Data 2')
    plt.hist(data3, bins=30, alpha=0.5, label='Data 3')

    plt.title('hist() TEST')
    plt.xlabel('Value')
    plt.ylabel('Frequency')
    plt.legend()

    plt.show()
    ```

    输出:

    ![](https://img.ricolxwz.io/3268de41cdc8eeb2093b53eed95ce055.png){:style="width:400px"}

### 结合Pandas

???+ example "例子"

    定义:

    ```
    random_data = np.random.normal(170, 10, 250)
    dataframe = pd.DataFrame(random_data)
    dataframe.hist()

    plt.title('hist() Test')
    plt.xlabel('X-Value')
    plt.ylabel('Y-Value')
    plt.show()  
    ```

    输出:

    ![](https://img.ricolxwz.io/65b38de07b39e5431bec2cbcff9f1290.png){:style="width:400px"}

[^1]: Matplotlib 直方图 | 菜鸟教程. (n.d.). Retrieved July 2, 2024, from https://www.runoob.com/matplotlib/matplotlib-hist.html