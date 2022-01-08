# Pytorch-nn与functional的区别


------

<!-- more -->

## `Pytorch`中的 `nn` 与` nn.functional`

> 这两者其实本质上功能是一致的，都是用于开发者调用`pytorch`中的接口以搭建自己的网络
> 
> 有一点不同的是，`nn`继承于`nn.Module`，是对`nn.functional`中定义的函数的类封装，所以`nn.functional`更加灵活，更加底层

| Attributes        | `torch.nn.Xxx`    | `torch.nn.functional.xxx` |
|:-----------------:|:-----------------:|:-------------------------:|
| 类型                | class             | function                  |
| 自身属性              | 继承`nn.Module`相关属性 | /                         |
| 调用方式              | 先实例化，再调用实例化对象     | 直接传参                      |
| 支持`nn.Sequential` | 是                 | 否                         |
| `dropout`兼容性      | 自动关闭`dropout`     | 需手动传参关闭                   |
| `weight`管理        | 自动                | 手动定义相关权重                  |

官方文档中有写到，需要学习参数的网路结构，最好使用 `nn.Xxx`的方式定义，便于管理权重参数，其他像`maxpool`,` loss func`, `activation func`等不需要学习参数的结构，可以直接使用 `nn.functional.xxx`调用底层函数，使用更加灵活方便

