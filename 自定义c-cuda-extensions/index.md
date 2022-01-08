# 自定义C++/CUDA扩展


----

<!-- more -->

## 官网文档 [Pytorch 1.7]

[Custom C++ and CUDA Extensions - PyTorch Tutorials 1.7.1 documentation](https://pytorch.org/tutorials/advanced/cpp_extension.html)

[Custom C++ and CUDA Extensions - PyTorch Tutorials 1.7.1 documentation CN](https://pytorch.apachecn.org/docs/1.0/cpp_extension.html)

## 自定义扩展的必要性

Pytorch虽然已经使用了NVIDIA cuDNN、Intel MKL和NNPACK这些底层来加快训练速度，但是在某些情况下，比如我们要实现一些特定算法，光靠组合Pytorch已有的操作是不够的。这是因为Pytorch虽然在特定操作上经过了很好的优化，但却并不见得适合我们自定义的操作，所以，作为一名程序员，应该了解如何将自己的网络或者操作以底层C++的代码实现，这是很重要的

除此之外，如果pytorch代码需要与C++代码进行交互，也需要自己编写对应的C++扩展

## 操作步骤

### 1. **如何用C++实现我们的扩展/操作？**

头文件包括：`#include <torch/extension.h>`  以及定义对应的函数名

```python
#include <torch/extension.h> //这一句是无论要实现任何op都必须添加的
#include <vector>

//前向传播
torch::Tensor my_op_forward(const torch::Tensor& x, const torch::Tensor& y);
//反向传播
std::vector<torch::Tensor> my_op_backward(const torch::Tensor& gradOutput);
```

源文件包括：网络的定义，包括前向传播、反向传播以及pybind11将c++代码绑定到python的部分

```python
#include "my_op.h"

torch::Tensor my_op_forward(const torch::Tensor& x,                             const torch::Tensor& y) {     
     AT_ASSERTM(x.sizes() == y.sizes(), "x must be the same size as y");
     torch::Tensor z = torch::zeros(x.sizes());
     z = 3 * x - y;
     return z; } 

std::vector<torch::Tensor> my_op_backward(const torch::Tensor& gradOutput) {
     torch::Tensor gradOutputX = 3 * gradOutput * torch::ones(gradOutput.sizes());
     torch::Tensor gradOutputY = -1 * gradOutput * torch::ones(gradOutput.sizes());
     return {gradOutputX, gradOutputY}; } 

// pybind11 绑定 
PYBIND11_MODULE(**TORCH_EXTENSION_NAME**, m) {
     m.def("forward", &my_op_forward, "MY_OP forward");
     m.def("backward", &my_op_backward, "MY_OP backward"); 
}
```

1. `pybind11`是python中用来和c++11通信的库
2. TORCH_EXTENSION_NAME不需要指定，这个定义在运行 setup.py 脚本文件时，对应的扩展名会传给这个定义，这样避免两者匹配不上
3. 这里网络的定义可以调用`pytorch/extension` 的库，也就是ATen，也可以通过自定义的`cuda kernels`实现

### 2. **如何编译C++代码为python可以识别的文件？**

> 对应两种方案，`setuptools` 或者 `torch.utils.cpp_extension.load()` 这里详细介绍前者

编写对应的 setup.py，构建pytorch的c++扩展，利用python的 `setuptools` 来编译并加载C++代码，这样，执行setup.py

```python
from setuptools import setup
import os
import glob
from torch.utils.cpp_extension import BuildExtension, CppExtension

# 头文件目录
include_dirs = os.path.dirname(os.path.abspath(__file__))
# 源代码目录 
source_file = glob.glob(os.path.join(working_dirs, 'src', '*.cpp'))

setup(
    name='test_cpp',  # 模块名称 与.cpp中的TORCH_EXTENSION_NAME对应
    ext_modules=[CppExtension('test_cpp', sources=source_file, include_dirs=[include_dirs])],
    cmdclass={
        'build_ext': BuildExtension
    }
)
```

在终端执行

`python setup.py install`

这一步其实是包含了build+install执行的是先编译链接动态链接库，然后将构建好的文件以package的形式安装存放再当前开发环境的package的集中存放处，这样就相当于生成了一个完整的package了。和其他的如numpy，torch这些package没什么两样。

执行后，目录下会生成三个目录`build/` `dist/` `***.egg-info/` ，除此之外，一个名子类似 ***-0.0.0-py3.6-linux-x86_64.egg 的文件也会出现在当前python的环境中`site-package` ，具体的编译输出如下：

- output
  
  ```python
    running install
    running bdist_egg
    running egg_info
    writing lltm.egg-info/PKG-INFO
    writing dependency_links to lltm.egg-info/dependency_links.txt
    writing top-level names to lltm.egg-info/top_level.txt
    reading manifest file 'lltm.egg-info/SOURCES.txt'
    writing manifest file 'lltm.egg-info/SOURCES.txt'
    installing library code to build/bdist.linux-x86_64/egg
    running install_lib
    running build_ext
    building 'lltm' extension
    gcc -Wsign-compare -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I~/local/miniconda/lib/python3.6/site-packages/torch/lib/include -I~/local/miniconda/lib/python3.6/site-packages/torch/lib/include/TH -I~/local/miniconda/lib/python3.6/site-packages/torch/lib/include/THC -I~/local/miniconda/include/python3.6m -c lltm.cpp -o build/temp.linux-x86_64-3.6/lltm.o -DTORCH_EXTENSION_NAME=lltm -std=c++11
    cc1plus: warning: command line option ‘-Wstrict-prototypes’ is valid for C/ObjC but not for C++
    g++ -pthread -shared -B ~/local/miniconda/compiler_compat -L~/local/miniconda/lib -Wl,-rpath=~/local/miniconda/lib -Wl,--no-as-needed -Wl,--sysroot=/ build/temp.linux-x86_64-3.6/lltm.o -o build/lib.linux-x86_64-3.6/lltm.cpython-36m-x86_64-linux-gnu.so
    creating build/bdist.linux-x86_64/egg
    copying build/lib.linux-x86_64-3.6/lltm_cuda.cpython-36m-x86_64-linux-gnu.so -> build/bdist.linux-x86_64/egg
    copying build/lib.linux-x86_64-3.6/lltm.cpython-36m-x86_64-linux-gnu.so -> build/bdist.linux-x86_64/egg
    creating stub loader for lltm.cpython-36m-x86_64-linux-gnu.so
    byte-compiling build/bdist.linux-x86_64/egg/lltm.py to lltm.cpython-36.pyc
    creating build/bdist.linux-x86_64/egg/EGG-INFO
    copying lltm.egg-info/PKG-INFO -> build/bdist.linux-x86_64/egg/EGG-INFO
    copying lltm.egg-info/SOURCES.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
    copying lltm.egg-info/dependency_links.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
    copying lltm.egg-info/top_level.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
    writing build/bdist.linux-x86_64/egg/EGG-INFO/native_libs.txt
    zip_safe flag not set; analyzing archive contents...
    __pycache__.lltm.cpython-36: module references __file__
    creating 'dist/lltm-0.0.0-py3.6-linux-x86_64.egg' and adding 'build/bdist.linux-x86_64/egg' to it
    removing 'build/bdist.linux-x86_64/egg' (and everything under it)
    Processing lltm-0.0.0-py3.6-linux-x86_64.egg
    removing '~/local/miniconda/lib/python3.6/site-packages/lltm-0.0.0-py3.6-linux-x86_64.egg' (and everything under it)
    creating ~/local/miniconda/lib/python3.6/site-packages/lltm-0.0.0-py3.6-linux-x86_64.egg
    Extracting lltm-0.0.0-py3.6-linux-x86_64.egg to ~/local/miniconda/lib/python3.6/site-packages
    lltm 0.0.0 is already the active version in easy-install.pth
  
    Installed ~/local/miniconda/lib/python3.6/site-packages/lltm-0.0.0-py3.6-linux-x86_64.egg
    Processing dependencies for lltm==0.0.0
    Finished processing dependencies for lltm==0.0.0
  ```

此外，使用 `pip list` 或者 `conda list` 查看包列表时，可以找到对应的包已经安装了，并且有对应的版本信息以及来源等

### 3. **如何通过python调用编译好的扩展/操作？**

 在上述操作完成后，虽然python/conda环境下已经有了相对应的库，但通过python进行import 是会报错的，如下

> undefined symbol: _ZTIN3c1021AutogradMetaInterfaceE

原因是编译好的包还需要进一步封装才能在python中调用，具体做法如下：

- 在setup.py 相同路径下新建一个py文件，内容为：
  
  ```python
    from torch.autograd import Function
    import torch
    import test_cpp
  
    class _TestFunction(Function):
        @staticmethod
        def forward(ctx, x, y):
            """
            It must accept a context ctx as the first argument, followed by any
            number of arguments (tensors or other types).
            The context can be used to store tensors that can be then retrieved
            during the backward pass."""
            return test_cpp.forward(x, y)
  
        @staticmethod
        def backward(ctx, gradOutput):
            gradX, gradY = test_cpp.backward(gradOutput)
            return gradX, gradY
  
    # 封装成一个模块（Module）
    class Test(torch.nn.Module):
        def __init__(self):
            super(Test, self).__init__()
  
        def forward(self, inputA, inputB):
            return **_TestFunction.apply**(inputA, inputB)
  ```
  
    可以看到，主要就是继承pytorch中的父类，新建对应的类，进而做了一些接口上的封装，具体就是使用 `torch.autograd.Function` 来将这个扩展写成一个函数，方便在构建网络的时候调用。最后就在合适的地方使用`Function.apply(*args)` ，就完成了一个自定义扩展了！

## 注意

### 1. 关于C++底层代码的编写，有两种方案

**第一种形式**，用`pytorch/extension` 的库，也就是ATen，加速效果尚好，需要以类似`at::sigmoid`的形式将pytorch的接口API复现一遍

[Pytorch学习 (二十一) ------自定义C++/ATen扩展_Hungryof的专栏-CSDN博客](https://blog.csdn.net/hungryof/article/details/88857607)

效果展示

```python
python     Forward: 187.719 us | Backward 410.815 us
extension  Forward: 149.802 us | Backward 393.458 us
```

**第二种形式**，使用自定义的`cuda kernels`来进一步加速，详见

[Custom C++ and CUDA Extensions](https://pytorch.apachecn.org/docs/1.0/cpp_extension.html)

[Custom C++ and CUDA Extensions - PyTorch Tutorials 1.7.1 documentation](https://pytorch.org/tutorials/advanced/cpp_extension.html#writing-a-mixed-c-cuda-extension)

### 2. 关于 `setup.py`

python库打包分发的详细攻略（setup.py编写）见 [程序包的打包和分发](https://blog.konghy.cn/2018/04/29/setup-dot-py/) 

### 3. 关于扩展包的封装

- 注意一定要有前向传播和反向传播的定义

- 实现`torch.autograd.Function` 子类时，注意其前向传播和反向传播，都需要有`ctx`参数
  
    大体上来说就是，这个 ctx 变量会在前向传播时，保存一些涉及到计算梯度的信息，然后在反向传播时辅助计算梯度，如
  
  ```python
    class Sigmoid(Function):
        @staticmethod
        def forward(ctx, x): 
            output = 1 / (1 + torch.exp(-x))
            ctx.**save_for_backward**(output)
            return output
        @staticmethod
        def backward(ctx, grad_output): 
            output,  = ctx.**saved_tensors**
            grad_x = output * (1 - output) * grad_output
            return grad_x
  ```

- 前向传播的输入参数和反向传播的输出参数**数量必须一致**
  
    如果有些变量不需要求导，就直接返回None即可

### 4. 关于梯度计算的验证

pytorch提供了`torch.autograd.gradcheck()` 函数来检测计算的梯度是否合理

如上述的sigmoid梯度计算，可以通过如下代码检验

```python
# tensor([-0.4646, -0.4403,  1.2525, -0.5953], requires_grad=True)
test_input = torch.randn(4, requires_grad=True)     
torch.autograd.gradcheck(Sigmoid.apply, (test_input,), eps=1e-3)    # pass
torch.autograd.gradcheck(torch.sigmoid, (test_input,), eps=1e-3)    # pass
torch.autograd.gradcheck(Sigmoid.apply, (test_input,), eps=1e-4)    # fail
torch.autograd.gradcheck(torch.sigmoid, (test_input,), eps=1e-4)    # fail
```

我们发现：eps 为 1e-3 时，我们编写的 Sigmoid 和 torch 自带的 builtin Sigmoid 都可以通过梯度检查，但 eps 下降至 1e-4 时，两者反而都无法通过。而一般直觉下，计算数值梯度时， eps 越小，求得的值应该更接近于真实的梯度。这里的反常现象，是由于机器精度带来的误差所致：test_input的类型为torch.float32，因此在 eps 过小的情况下，产生了较大的精度误差（计算数值梯度时，eps 作为被除数），因而与真实精度间产生了较大的 gap。将test_input换为float64的 tensor 后，不再出现这一现象。这点同时提醒我们，在编写backward时，要考虑的数值计算的一些性质，尽可能保留更精确的结果

```python
test_input = torch.randn(4, requires_grad=True, **dtype=torch.float64**)    # tensor([-0.4646, -0.4403,  1.2525, -0.5953], dtype=torch.float64, requires_grad=True)
torch.autograd.gradcheck(Sigmoid.apply, (test_input,), eps=1e-4)    # pass
torch.autograd.gradcheck(torch.sigmoid, (test_input,), eps=1e-4)    # pass
torch.autograd.gradcheck(Sigmoid.apply, (test_input,), eps=1e-6)    # pass
torch.autograd.gradcheck(torch.sigmoid, (test_input,), eps=1e-6)    # pass
```

具体介绍见

[PyTorch 源码解读之 torch.autograd](https://zhuanlan.zhihu.com/p/321449610)

### 5. 关于运行设备

由于Pytorch的C++库 ATen可以同时适用于CPU和GPU，所以只需要传给封装好的函数对应cuda形式的张量，就可以调用GPU加速运算了

### 6. 关于`torch.utils.cpp_extension.load()` 实时编译扩展库

这种方式调用pytorch中的api，是一种动态编译和加载扩展的方式

代码如下

```cpp
from torch.utils.cpp_extension import load
lltm_cpp = load(name="lltm_cpp", sources=["lltm.cpp"])
```

[Custom C++ and CUDA Extensions - PyTorch Tutorials 1.7.1 documentation](https://pytorch.org/tutorials/advanced/cpp_extension.html#jit-compiling-extensions)

## 参考

https://zhuanlan.zhihu.com/p/100459760

