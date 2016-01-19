---
layout: theme:post
title: "正确使用 Multiprocessing 的姿势"
date: 2015-12-31T09:46:40+08:00
---

最近在使用 arcpy 处理一些地理数据，为了提高处理性能，使用 multiprocessing 库进行并行化。在使用 multiprocessing 库的过程中，踩了不少坑，在此记录一下。

multiprocessing 是 python 的多进程并行库，我使用进程池 multiprocessing.pool 来自动管理进程任务。通过一下语句初始化 pool:
```python
multiprocessing.freeze_support()  # Windows 平台要加上这句，避免 RuntimeError
pool = multiprocessing.Pool()
```

假设我们要并行执行的任务是以下函数：
```python
def task(pid):
    # do something
    return result
```

然后在主函数调用：
```python
results = []
for i in xrange(0, 4):
    result = pool.apply_async(task, args=(i,))
    results.append(result)
```
`pool.apply_async` 采用异步方式调用 task，`pool.apply` 则是同步方式调用。同步方式意味着下一个 task 需要等待上一个 task 完成后才能开始运行，这显然不是我们想要的功能，所以采用异步方式连续地提交任务。在上面的语句中，我们提交了 4 个任务，假设我的 CPU 是 4 核，那么我的每个核运行一个任务。如果我提交多于 4 个任务，那么每个核就需要同时运行 2 个以上的任务，这回带来任务切换成本，降低了效率。所以我们设置的并行任务数最好等于 CPU 核心数， CPU 核可以通过下面语句得到：
```python
cpus = multiprocessing.cpu_count()
```

接下来我们使用 `result.get()` 来获取 task 的返回值：
```python
for result in results:
    print(result.get())
```
在这里不免有人要疑问，为什么不直接在 for 循环中直接 `result.get()`呢？这是因为`pool.apply_async`之后的语句都是阻塞执行的，调用 `result.get()` 会等待上一个任务执行完之后才会分配下一个任务。事实上，**获取返回值的过程最好放在进程池回收之后进行**，避免阻塞后面的语句。

最后我们使用一下语句回收进程池：
```python
pool.close()
pool.join()
```

最后附上完整的代码如下：
```python
def task(pid):
    # do something
    return result

def main():
    multiprocessing.freeze_support()
    pool = multiprocessing.Pool()
    cpus = multiprocessing.cpu_count()
    results = []

    for i in xrange(0, cpus):
        result = pool.apply_async(task, args=(i,))
        results.append(result)

    pool.close()
    pool.join()

    for result in results:
        print(result.get())
```
