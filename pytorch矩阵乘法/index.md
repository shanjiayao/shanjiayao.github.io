# Pytorch中的矩阵乘法


<!--more-->

### `Pytorch` 矩阵乘法

关于广播机制，[参考](https://shmilywh.github.io/2021/01/13/python%E4%B8%AD%E7%9A%84%E5%B9%BF%E6%92%AD%E6%9C%BA%E5%88%B6/)

1. `torch.mul`
   
   ```python
   torch.mul(input, other, *, out=None)
   ```
   
   按位相乘，对维度的要求是，两个张量尽量维度对齐，或者是可以遵循广播机制
   
   比如：
   
   ```python
   >>> a = torch.randn(4, 1)
   >>> a
   tensor([[ 1.1207],
           [-0.3137],
           [ 0.0700],
           [ 0.8378]])
   >>> b = torch.randn(1, 4)
   >>> b
   tensor([[ 0.5146,  0.1216, -0.5244,  2.2382]])
   >>> torch.mul(a, b)
   tensor([[ 0.5767,  0.1363, -0.5877,  2.5083],
           [-0.1614, -0.0382,  0.1645, -0.7021],
           [ 0.0360,  0.0085, -0.0367,  0.1567],
           [ 0.4312,  0.1019, -0.4394,  1.8753]])
   ```

2. `torch.mm`
   
   ```python
   torch.mm(input, mat2, *, out=None) → Tensor
   ```
   
   矩阵相乘，，注意只支持二维的张量
   
   这个函数不支持广播机制，也就是说，必须完全按照矩阵相乘的顺序输入

3. `torch.matmul`
   
   ```python
   torch.mm(input, mat2, *, out=None) → Tensor
   ```
   
   也是矩阵相乘，不过与 `mm`不同，这个函数支持广播机制
   
   也就是说输入的两个张量维度不一定需要完全按照矩阵乘法顺序输入
   
   比如，输入的张量是 (j×1×n×m) ，另外一个张量是 (k×m×p)，则会自动按照维度，广播到 (j×k×n×m) 和 (j×k×m×p)，然后计算矩阵相乘，得到 (j×k×n×p)

