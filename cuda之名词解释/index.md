# cuda常见名词解释


---

<!-- more -->

## 常见名词

> `device`

device一般指GPU端，也叫设备端，一般以 `__device__` 前缀修饰的函数，函数的调用以及执行都需要在设备端进行

> `host`

host指CPU，也叫主机端，以 `__host__` 前缀修饰的函数，调用和执行都在主机端进行

> `kernel` & `kernel func`

全称是数据并行处理函数，一般也被成为核函数，个人理解，核函数负责主机与设备的交互，一般而言，核函数以 `__global__` 前缀修饰，代表着其可以从主机端调用，在设备端执行，这就完成了数据从主机端到设备端的传输

通常，我们使用以下形式，在主机端调用核函数

```cpp
kernel<<<Dg, Db, Ns, S>>>(param list);
```

kernel指核函数的名字，最后一部分就是核函数计算时需要传入的参数，<<< ... >>> 中的参数就是对grid，block以及共享内存等CUDA参数的设定，具体表示会在后面说明

> `SM` & `SP`

SM指的是 `Streaming MultiProcessing`

SP指的是 `Streaming Processing`

SM和SP都是硬件层次的术语

一块GPU上面，有非常多的SM，以Tesla P100 GPU举例，其基于Pascal架构的版本包含56个SMs

一个SM包含多个SP，这里的SP其实和我们常说的CUDA core是等同的

每一个SM有自己的指令缓存，L1缓存，共享内存

每个SM通过执行`block`来进行GPU的同步计算，但是每个SM每次只能执行一个`block`

> `grid` & `block` & `thread`

这三者是CUDA编程时经常会遇到的概念，从上而下的层级依次是 grid → block → thread

为了更加清晰地理解这些层级关系，自下而上依次介绍如下：

`thread`

thread是GPU工作的最小执行单元，是程序员可以控制的**最细粒度的并行单位**。每一个thread在运算单元上就对应一个**sp**，所以大家会常常会笼统的把sp数量等同于thread的并行数量，从而量化不同GPU的性能

关于`thread`，还有一个概念，在GPU中，`thread`通常是一组一组执行的，一般来说每组的`thread`数量为32，这32个`thread`也被称为`warp`(线程束)，同一个`warp`中的`thread`执行的指令都是相同的，只是处理不同的数据。除此之外，对于`warp`的分组，一般来说都是SM自动进行的，每个`warp`中的`thread`，其`id`都是连续的

`block`

`block`是CUDA执行代码时的基础单位，一个`block`由多个`thread`组成，具体参数是上述内核启动函数中传递的参数，也就是 <<< >>>中的 `Db` 参数，是一个Dim3类型的数据，具体包括x，y，z三个参数，分别对应行、列以及高度方向上的thread数量，这只是理想的参数，实际上，线程数还受到硬件计算能力的限制，不同计算能力的显卡，其允许的最大线程数量也不同，如下图

![Untitled.png](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/07/05-10-08-33-Untitled.png)

不同计算能力的显卡对应的最大允许Grid、block以及thread运行数量

`Grid`

`Grid`是由若干个`block`组成的，一般而言，一个`kernel`的程序就对应一个`grid`去执行

一个`grid`对应的参数有三个，同样在核函数中<<< >>>可以设置，对应第一个参数`Dg`，具体有x，y，z三个方向，不过z方向的数量**恒定为1**，所以一个`grid`中包括的`block`数量就是 `Dg.x` × `Dg.y` × 1

总结

总体来看，一个GPU由SM构成，而每个SM就是运行blocks中指令的硬件载体

软件方面，当我们在CUDA编程时，编写一个形如`kernel<<<Dg, Db, Ns, S>>>(param list);` 的核函数时，其在GPU中就会对应一个`grid`，而通过`Dg, Db`分配每个`grid`中包含多少个`blocks`以及每个`block`包含多少个`threads`，

最终，每个`block`会对应若干个由`threads`组成的`warp`，而硬件方面，每个`SM`就会运行对应`block` 中的每个`warp`，这就是整体的机制，为了方便理解，我画了一张草图来表示几者之间的关系

![](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/07/05-09-34-32-2021-07-05-09-34-23-image.png)

其实，仔细思考一下，对于编程人员来说，程序并行化的关键就在于设计一个方案，使得一个SM内的各种硬件单元都能有效利用，比如如何分配每个`block`中的线程数，让每个warp中尽可能都包含32个线程，进而合理利用资源，得到最理想的吞吐

## 关于

### 编程时实际线程数的计算

由于每个block里面的线程，在各自block内部都是有自己的线程id的，那么实际编程时如何计算对应每个block中的线程编号呢？

![Untitled 1.png](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/07/05-09-36-25-Untitled%201.png)

如上图所示，以一个维度上的grid、block以及thread三者关系为例

实际线程编号 = 当前block编号 × block在对应维度上的尺寸 + 当前block中的thread编号

代码如下：

```cpp
__global__
void add(int n, float *x, float *y)
{
  int index = blockIdx.x * blockDim.x + threadIdx.x;
  int stride = blockDim.x * gridDim.x;
  for (int i = index; i < n; i += stride)
    y[i] = x[i] + y[i];
}
```

### 算法测试耗时

![](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/07/05-09-47-41-2021-07-05-09-47-36-image.png)

分别测试了一个线程、一个block以及多个block下的算法耗时，可以看到非常明显的加速！

## 参考

认识什么是 `host code`  `device code` 以及 `kernel`

[An Easy Introduction to CUDA C and C++ | NVIDIA Developer Blog](https://developer.nvidia.com/blog/easy-introduction-cuda-c-and-c/)

其他基础概念了解，块、网格等

[An Even Easier Introduction to CUDA | NVIDIA Developer Blog](https://developer.nvidia.com/blog/even-easier-introduction-cuda/)
