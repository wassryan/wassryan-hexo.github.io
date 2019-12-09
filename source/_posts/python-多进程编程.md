---
title: 'Python-多进程编程'
date: 2019-12-02 14:56:23
tags: [Python]
---
这篇文章会讲一下python中的多线程和多进程

## 前话

- python解释器中存在GIL（全局解释器锁），保证同一时刻只有一个线程可以执行代码，所以python中的多线程其实并不是真正的多线程。
  - GIL：每个线程执行时都需要先获取GIL，保证同一时刻只有一个线程在执行代码
  - 遇到IO操作时，CPU空闲的时候，会释放

- 多线程&多进程

  - 多线程适合：IO密集型（文件操作、网络爬虫）

  - 多进程适合：CPU密集型（循环计算）

    **为什么**：`IO密集型`，CPU大部分时间是在等待，此时多CPU资源是无用的，由于pythonGIL机制，线程阻塞时，该线程会释放CPU给新的线程使用，则充分利用了CPU；相反`CPU密集型`，CPU资源多，可分配给更多进程使用。

## 多线程

在一个进程中，允许几段代码`并发`运行。

#### 编写多线程

- 定义`目标函数`，传入`线程类`
- 继承`线程类`，覆写`类函数`

```python
# 法1
import threading
def func(times,name,ret):
    ...
if __name__=='__main__':
    thread_pool=[] # 线程池
    res={} # 线程返回的值
    th_1=threading.Thread(target=func,args=[3,'th_1',ret],name='th_1')
    th_2=threading.Thread(target=func,args=[5,'th_2',ret],name='th_2')
    thread_pool.append(th_1)
    thread_pool.append(th_2)
    for th in thread_pool:
        th.start() # 启动线程
    for th in thread_pool:
        th.join() # 阻塞主线程直到子线程执行完
    print(ret)
```

* `start()`：表示启动该线程
* `join()`：阻塞主进程直至所有子进程执行完毕

```python
# 法2
import threading
class ThreadChild(threading.Thread): # 继承线程类
    def __init__(self, times, name, ret):
        # 扩写父类的初始化，首先调用父类的初始化
        threading.Thread.__init__(self) 
        self.times = times
        self.name = name
        self.ret = ret
        return
    def run(self):
        # 覆盖重写函数 run
        ...
        return

if __name__ == '__main__':
    thread_pool = []
    ret = {}
    th_1 = ThreadChild(times=3, name='th_1', ret=ret)
    th_2 = ThreadChild(times=5, name='th_2', ret=ret)
    thread_pool.append(th_1)
    thread_pool.append(th_2)
    for th in thread_pool:
        th.start()
    for th in thread_pool:
        th.join()
    print(ret)
```

#### 资源竞争

**注意：**线程间的Queuebenign通信，进程的Queue可以通信。

多线程共享一个进程的`内存`，所以在并发的时候会出现资源竞争。

对于共同的参数，要读写时，需要加`锁Lock`

```python
self.ret_lock.acquire()
...对共享参数的操作
self.ret_lock.release()
```



## 多进程

在有多核的电脑上`并行计算`

#### 编写多进程

```python
import math
import datetime
import multiprocessing as mp

def train_on_parameter(name, param):
    result = 0
    for num in param:
        result += math.sqrt(num * math.tanh(num) / math.log2(num) / math.log10(num))
    return {name: result}

if __name__ == '__main__':
    start_t = datetime.datetime.now()
    num_cores = int(mp.cpu_count())
    print("本地计算机有: " + str(num_cores) + " 核心")
    pool = mp.Pool(num_cores) # 最大可允许多少个进程
    param_dict = {'task1': list(range(10, 30000000)),
                  'task2': list(range(30000000, 60000000)),
                  'task3': list(range(60000000, 90000000)),
                  'task4': list(range(90000000, 120000000)),
                  'task5': list(range(120000000, 150000000)),
                  'task6': list(range(150000000, 180000000)),
                  'task7': list(range(180000000, 210000000)),
                  'task8': list(range(210000000, 240000000))}
    results = [pool.apply_async(train_on_parameter, args=(name, param)) for name, param in param_dict.items()]
    results = [p.get() for p in results]

    end_t = datetime.datetime.now()
    elapsed_sec = (end_t - start_t).total_seconds()
    print("多进程计算 共消耗: " + "{:.2f}".format(elapsed_sec) + " 秒")
```

<div align=center width=80%>
  <img width=80% src="/images/multiprocess.png" >
</div>

- **异步调度**：`apply_async()`是非阻塞异步的，不等待子进程执行完，主进程会继续执行。函数返回的是`ApplyResult类`
- **调度结果**：调用`ApplyResult类`的`get()函数`（非异步函数），即`get()`会阻塞到类结果返回之后，才执行

#### 资源共享

多进程每个进程有独立的内存，进程之间可以通过Pipe，Queue和Manager的方式，共享对象。

- 管道Pipe

  定义一个`双向的pipe`

  ```python
  import multiprocessing as mul
  
  def proc1(pipe):
      pipe.send('hello')
      print('proc1 rec:', pipe.recv())
  
  def proc2(pipe):
      print('proc2 rec:', pipe.recv())
      pipe.send('hello, too')
  
  # Build a pipe
  pipe = mul.Pipe()
  if __name__ == '__main__':
      # Pass an end of the pipe to process 1
      p1 = mul.Process(target=proc1, args=(pipe[0],))
      # Pass the other end of the pipe to process 2
      p2 = mul.Process(target=proc2, args=(pipe[1],))
      p1.start()
      p2.start()
      p1.join()
      p2.join()
  ```

  管道对象是一个含有`两个元素的表`，每个元素代表管道的一端，一端用`send()`来传送对象，另一端用`recv()`来接收。

- 队列Queue

  **注意：**需要使用进程库提供的Queue，其他的队列不能在进程之间通信。

  允许多个进程放入，多个进程取出，使用`multiprocessing.Queue(maxsize)`创建

  - 放入：`queue.put(info)`
  - 取出：`out=queue.get()`

- 管理员Manager

  创建共享的数据容器，容器种类包含了所有python自带的数据类

  ```python
  import math
  import datetime
  import multiprocessing as mp
  
  def train_on_parameter(name, param, result_dict, result_lock):
      result = 0
      for num in param:
          result += math.sqrt(num * math.tanh(num) / math.log2(num) / math.log10(num))
      with result_lock:
          result_dict[name] = result # 把结果保存在字典中，而不是直接返回
      return
  
  if __name__ == '__main__':
      start_t = datetime.datetime.now()
      num_cores = int(mp.cpu_count())
      print("本地计算机有: " + str(num_cores) + " 核心")
      pool = mp.Pool(num_cores)
      param_dict = {'task1': list(range(10, 30000000)),
                    'task2': list(range(30000000, 60000000)),
                    'task3': list(range(60000000, 90000000)),
                    'task4': list(range(90000000, 120000000)),
                    'task5': list(range(120000000, 150000000)),
                    'task6': list(range(150000000, 180000000)),
                    'task7': list(range(180000000, 210000000)),
                    'task8': list(range(210000000, 240000000))}
      manager = mp.Manager()
      managed_locker = manager.Lock() # 创建Lock
      managed_dict = manager.dict()
      results = [pool.apply_async(train_on_parameter, args=(name, param, managed_dict, managed_locker)) for name, param in param_dict.items()]
      results = [p.get() for p in results]
      print(managed_dict)
      end_t = datetime.datetime.now()
      elapsed_sec = (end_t - start_t).total_seconds()
      print("多线程计算 共消耗: " + "{:.2f}".format(elapsed_sec) + " 秒")
  ```

  <div align=center width=80%>
    <img width=80% src="/images/multiprocess_manager.png" >
  </div>

