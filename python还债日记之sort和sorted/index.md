# Python还债日记之sort和sorted


------

<!-- more -->

## python中的`sort`与`sorted`的区别

Python 列表有一个内置的 [`list.sort()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#list.sort) 方法可以直接修改列表。还有一个 [`sorted()`](https://docs.python.org/zh-cn/3/library/functions.html#sorted) 内置函数，它会从一个可迭代对象构建一个新的排序列表

> `sort`和`sorted`使用了`Timsort`排序算法，时间复杂度和空间复杂度如下

![timsort1.png](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/04/06-19-53-42-timsort1.png)

![timsort2.png](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/04/06-19-53-47-timsort2.png)

### 1. `sort(*, key=None, reverse=False)`

- 针对python中的列表进行操作

- sort是对**原列表进行修改**，是一种原地操作，使用 `<` 进行比较，默认按**升序排序**

- *key* 指定带有一个参数的函数，用于从每个列表元素中提取比较键 (例如 `key=str.lower`)。 对应于列表中每一项的键会被计算一次，然后在整个排序过程中使用。 默认值 `None` 表示直接对列表项排序而不计算一个单独的键值

- *reverse* 为一个布尔值。 如果设为 `True`，则每个列表元素将按反向顺序比较进行排序，也就是输出**降序排序**结果

### 2. `sorted`(*iterable*, ***, *key=None*, *reverse=False*)

- 根据 *iterable* 中的项返回一个新的已排序列表，相比于`sort`只能针对python的列表对象，`sorted`可以针对任何可迭代对象使用（字典、元组等）

- 默认输出**升序排序**结果 

- 具有两个可选参数，它们都必须指定为关键字参数。

- *key* 指定带有单个参数的函数，用于从 *iterable* 的每个元素中提取用于比较的键 (例如 `key=str.lower`)。 默认值为 `None` (直接比较元素)。

- *reverse* 为一个布尔值。 如果设为 `True`，则每个列表元素将按反向顺序比较进行排序

