# python中的time模块



![tp.web.random_picture](https://images.unsplash.com/photo-1516202180772-d888b16cf6dd?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1080&ixid=MnwxfDB8MXxyYW5kb218MHx8bGFuZHNjYXBlLHdhdGVyfHx8fHx8MTY0MjEzOTQ5OQ&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1920)




<!-- more -->


## 使用python中的time模块计算代码运行时间

在计算算法的耗时性时，需要通过一些方法计算代码实际运行的时间，在python中time模块集成了一些方法，可以很方便的计算耗时。

### 几种函数简介

time模块提供各种与时间相关的功能，如当前时间、时间戳等，下面只说明用来计算代码运行时间的几种常用函数，python环境为python3

#### time.time()

返回当前时间的时间戳(1970元年后的浮点秒数)，这个函数容易收到系统时间的影响，所以一般计算代码耗时都不会选择time函数。

#### time.perf_counter()

返回计时器的精准时间(系统的运行时间)，包含整个系统的睡眠时间．这里的睡眠时间指的是人工使用类似 time.sleep() 的额外时间。

使用时需计算两次之间差值。计算耗时时，如果没有人为的睡眠时间，可以使用这个函数。

#### time.process_time()

返回当前进程执行CPU的时间总和，不包含睡眠时间．使用时需计算两次之间差值作为代码运行时间。

### 代码测试

```python
from time import time, perf_counter, process_time, sleep


class TimeDemo:
    def __init__(self):
        pass

    def test_time(self):
        time0 = time()
        for i in range(10000000):
            i = 0
        sleep(2)
        time1 = time()
        print("time function : {0}".format(time1 - time0))

    def test_perf_cnt(self):
        time0 = perf_counter()
        for i in range(10000000):
            i = 0
        sleep(2)
        time1 = perf_counter()
        print("perf_counter function : {0}".format(time1 - time0))

    def test_process_time(self):
        time0 = process_time()
        for i in range(10000000):
            i = 0
        sleep(2)
        time1 = process_time()
        print("process_time function : {0}".format(time1 - time0))


if __name__ == '__main__':
    timedemo = TimeDemo()
    timedemo.test_time()
    timedemo.test_perf_cnt()
    timedemo.test_process_time()
```




