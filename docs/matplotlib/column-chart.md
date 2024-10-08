---
title: 柱形图
icon: material/poll
comments: false
---

???+ info "信息"

    默认省略导入:

    - `import numpy as np`
    - `import matplotlib.pyplot as plt`

可以使用`pyplot`中的`bar`函数来绘制柱形图.

`bar`函数语法格式如下:

```
matplotlib.pyplot.bar(x, height, width, bottom, *, align, data, **kwargs)
```

参数说明:

- `x`: 浮点型数组, 柱形图的x轴数据
- `height`: 浮点型数组, 柱形图的高度
- `width`: 浮点型数组, 柱形图的宽度
- `bottom`: 浮点型数组, 底座的y坐标, 默认为`0`
- `align`: 柱形图与x坐标的对齐方式, `center`以x位置为中心, 这是默认值; `edge`讲柱形图的左边缘与x位置对齐. 要对齐右边缘的条形, 可以传递负数的宽度值以及`edge`
- `**kwargs`: 其他参数

## 例子

### 普通例子

- `bar`函数: 绘制水平柱形图
- `barh`函数: 绘制垂直柱形图

???+ example "例子"

    === "水平柱形图"

        定义:

        ```
        x = np.array(['Ricolxwz-1', 'Ricolxwz-2', 'Ricolxwz-3', 'Ricolxwz-4'])
        y = np.array([12, 22, 6, 18])

        plt.bar(x, y)
        plt.show()
        ```

        输出:

        ![](https://img.ricolxwz.io/02b390d48e3ab42cc9fefd641d48e87a.png){:style="width:400px"}

    === "垂直柱形图"

        定义:

        ```
        x = np.array(['Ricolxwz-1', 'Ricolxwz-2', 'Ricolxwz-3', 'Ricolxwz-4'])
        y = np.array([12, 22, 6, 18])

        plt.barh(x, y)
        plt.show() 
        ```

        输出:

        ![](https://img.ricolxwz.io/3d9918113dafe4dbe01f781ca8e14def.png){:style="width:400px"}

### 自定义柱的颜色

???+ example "例子"

    === "设置总体颜色"

        定义:

        ```
        x = np.array(['Ricolxwz-1', 'Ricolxwz-2', 'Ricolxwz-3', 'Ricolxwz-4'])
        y = np.array([12, 22, 6, 18]) 

        plt.bar(x, y, color="#4A4A45")
        plt.show()
        ```

        输出:

        ![](https://img.ricolxwz.io/32bb911b7ce2237e94433514a2964766.png){:style="width:400px"}

    === "设置各个柱的颜色"

        定义:

        ```
        x = np.array(['Ricolxwz-1', 'Ricolxwz-2', 'Ricolxwz-3', 'Ricolxwz-4'])
        y = np.array([12, 22, 6, 18]) 

        plt.bar(x, y, color=["#4A4A45", "#556B2F", "blue", "grey"])
        plt.show()
        ```

        输出:

        ![](https://img.ricolxwz.io/1ba81d2856cfb1b74386f4fe9f617e26.png){:style="width:400px"}

### 自定义柱的宽度

- `bar`函数: 设置`width`
- `barh`函数: 设置`height`

???+ example "例子"

    定义:

    ```
    x = np.array(['Ricolxwz-1', 'Ricolxwz-2', 'Ricolxwz-3', 'Ricolxwz-4'])
    y = np.array([12, 22, 6, 18])  

    plt.bar(x, y, width=0.2)
    plt.show()
    ```

    输出:

    ![](https://img.ricolxwz.io/359892a4b0f6ff5375e79e5e1c6d2bca.png){:style="width:400px"}

[^1]: Matplotlib 柱形图 | 菜鸟教程. (n.d.). Retrieved July 1, 2024, from https://www.runoob.com/matplotlib/matplotlib-bar.html