### 进程

#### multiprocessing

run()和start()的区别：

1. run()不会创建子进程执行任务，它只会使用当前进程执行。
2. start()是创建一个子进程，然后执行run()方法。

全局变量不能被多进程共享：

```python
import multiprocessing
import time
import os

# 全局变量不能被多进程共享
m = 0

def task1(*param):
    global m
    while True:
        time.sleep(1)
        print("Task 1")
        print("current pid:", os.getpid())
        print("parent pid:", os.getppid())
        print("param:", *param)
        m += 1
        print("m:", m)


def task2():
    global m
    while True:
        time.sleep(1)
        print("Task 2")
        m += 2
        print("m:", m)


if __name__ == "__main__":
    # args和kwargs是给函数传递的参数
    t1 = multiprocessing.Process(target=task1, name="task_1", args=(1,3))
    t2 = multiprocessing.Process(target=task2)
    t1.start()
    t2.start()
    print("main:", os.getpid())
    # 主进程打印子进程pid
    print("t1:", t1.pid)
    print("t2:", t2.pid)
    print('-' * 30)
```

#### 自定义进程

```python
import multiprocessing
import time
import os


class MyProcess(multiprocessing.Process):
    # 重写run方法
    def run(self):
        n = 1
        while True:
            time.sleep(1)
            print("进程%s，n = %d" % (self.name, n))
            n = n + 1
            # 执行5次后关闭
            if n == 5:
                self.terminate()
            print(os.getpid())


if __name__ == "__main__":
    print("parent", os.getpid())
    t1 = MyProcess(name="t1")
    t1.start()
    t2 = MyProcess(name="t2")
    t2.start()
```

#### 进程池

> 进程池包括阻塞式和非阻塞式。

非阻塞式：

```python
import multiprocessing, time, random, os


def task(task_name):
    print("Starting task %s:" % task_name)
    start_timestamp = time.time()
    time.sleep(random.random() * 2)
    end_timestamp = time.time()
    return task_name, end_timestamp - start_timestamp, os.getpid()


def callback(params):
    task_name, consuming_time, pid = params
    print("  %s Took time %fs. The pid is %s." % (task_name, consuming_time, pid))


if __name__ == "__main__":
    # 创建5个进程
    pool = multiprocessing.Pool(processes=5)

    # 如果有进程完成任务变成空闲，会执行进程池中的剩余任务
    for i in range(8):
        # 非阻塞式
        # func是任务函数，callback是任务完成的回调
        pool.apply_async(func=task, kwds={"task_name": "Task %d" % (i + 1)}, callback=callback)

    pool.close()  # 关闭进程池，使其不添加新的任务
    pool.join()  # 主进程阻塞等待子进程的退出， join方法要在close或terminate之后使用

    print("Bye bye~")
    
"""
Outputs:
Starting task Task 1:
Starting task Task 2:
Starting task Task 3:
Starting task Task 4:
Starting task Task 5:
Starting task Task 6:
  Task 1 Took time 0.116347s. The pid is 46471.
  Task 4 Took time 0.260033s. The pid is 46470.
Starting task Task 7:
Starting task Task 8:
  Task 5 Took time 0.529314s. The pid is 46473.
  Task 7 Took time 0.591754s. The pid is 46470.
  Task 6 Took time 1.115715s. The pid is 46471.
  Task 8 Took time 0.707028s. The pid is 46473.
  Task 2 Took time 1.733969s. The pid is 46472.
  Task 3 Took time 1.979868s. The pid is 46474.
Bye bye~
"""
```

阻塞式：

```python
import multiprocessing, time, random, os


def task(task_name):
    print("Starting task %s:" % task_name)
    start_timestamp = time.time()
    time.sleep(random.random() * 2)
    end_timestamp = time.time()
    print("  %s Took time %fs. The pid is %s." % (task_name, end_timestamp - start_timestamp, os.getpid()))


if __name__ == "__main__":
    # 创建5个进程
    pool = multiprocessing.Pool(processes=5)

    # 如果有进程完成任务变成空闲，会执行进程池中的剩余任务
    for i in range(8):
        # 阻塞式没有callback
        pool.apply(func=task, kwds={"task_name": "Task %d" % (i + 1)})

    pool.close()  # 关闭进程池，使其不添加新的任务
    pool.join()  # 主进程阻塞等待子进程的退出， join方法要在close或terminate之后使用

    print("Bye bye~")
    
"""
Outputs:
Starting task Task 1:
  Task 1 Took time 1.793565s. The pid is 46434.
Starting task Task 2:
  Task 2 Took time 1.371809s. The pid is 46435.
Starting task Task 3:
  Task 3 Took time 0.684377s. The pid is 46438.
Starting task Task 4:
  Task 4 Took time 1.410749s. The pid is 46436.
Starting task Task 5:
  Task 5 Took time 1.717518s. The pid is 46437.
Starting task Task 6:
  Task 6 Took time 1.030143s. The pid is 46434.
Starting task Task 7:
  Task 7 Took time 0.700300s. The pid is 46435.
Starting task Task 8:
  Task 8 Took time 1.696361s. The pid is 46438.
Bye bye~
"""
```

### 进程间通信

> 进程本身不能通信，但是可以利用数据结构共享数据。

queue库示例：

```python
import queue

q = queue.Queue(maxsize=5)

print(q.empty())  # True

q.put("t1")
q.put("t2")
q.put("t3")
q.put("t4")
q.put("t5")

print(q.full())  # True

# 由于队列满了，此时进程被阻塞，队列等待空位以添加新项
# get()也有同样的性质：如果队列为空，进程会被阻塞
# get()和put()都有timeout参数
q.put("t6")  
```

通过multiprocessing.Queue。略

### 线程

#### 基本使用

```python
import threading, time


def download(n):
    imgs = ['cat', 'dog', 'flower']
    for i in imgs:
        print("Downloading %s%d..." % (i, n))
        time.sleep(1)
        print("  %s%d Downloaded." % (i, n))


if __name__ == '__main__':
    t = threading.Thread(target=download, args=(1,))
    t2 = threading.Thread(target=download, args=(2,))

    t.start()
    t2.start()
```

#### 全局变量可以被多线程共享


```python
import threading
import time
import os

m = 0


def task1():
    global m
    while True:
        time.sleep(1)
        m += 1
        print("Task 1", "m:", m)


def task2():
    global m
    while True:
        time.sleep(1)
        m += 2
        print("Task 2", "m:", m)


if __name__ == "__main__":
    t1 = threading.Thread(target=task1)
    t2 = threading.Thread(target=task2)
    t1.start()
    t2.start()
```

看下面的情况：

```python
import threading
import time
import os

m = 0


def task1():
    global m
    for i in range(1000000):
        m += 1
    print("Task 1")
    print("  m:", m)


def task2():
    global m
    for i in range(1000000):
        m += 1
    print("Task 2")
    print("  m:", m)


if __name__ == "__main__":
    t1 = threading.Thread(target=task1, name="task_1")
    t2 = threading.Thread(target=task2)
    t1.start()
    t2.start()

"""
Outputs:
Task 2
  m: 1234926
Task 1
  m: 1352098
"""
```

为什么线程可以共享数据，但是这里的数据是错误的呢？

因为：

如果多线程要共享数据，则必须要**线程同步**，才能使数据不会在资源抢占时发生不同步。实现线程同步、数据安全，就是给线程加锁。

Python默认规则是，只要用线程，默认就加锁(GIL, Global Interpreter Lock)，所以可以实现线程同步。而且如果数据很大时，锁又回失效。这是CPython解释器的缺点。（可以使用numpy这样的第三方库做计算。）

加锁使得python的threading的多线程其实是伪多线程。


总结：耗时操作(比如爬虫、IO)使用多线程，计算密集型使用多进程（因为进程直接与CPU打交道）。

#### 线程锁的使用

```python
import threading

m = 0

lock = threading.Lock()


def task1():
    global m
    for i in range(1000000):
        lock.acquire()
        m += 1
        lock.release()
    print("Task 1", "m:", m)


def task2():
    global m
    for i in range(1000000):
        lock.acquire()
        m += 1
        lock.release()
    print("Task 2", "m:", m)


if __name__ == "__main__":
    # args和kwargs是给函数传递的参数
    t1 = threading.Thread(target=task1, name="task_1")
    t2 = threading.Thread(target=task2)
    t1.start()
    t2.start()

"""
Outputs:
Task 2 m: 1745951
Task 1 m: 2000000
"""
```

#### 死锁

示例：

```python
from threading import Thread, Lock
import time

lock1 = Lock()
lock2 = Lock()


class MyThread1(Thread):
    def run(self):
        lock1.acquire()
        print("Thread 1 got lock 1.")
        time.sleep(1)
        lock2.acquire()
        print("Thread 1 got lock 2.")
        lock2.release()
        lock1.release()


class MyThread2(Thread):
    def run(self):
        lock2.acquire()
        print("Thread 2 got lock 2.")
        time.sleep(1)
        lock1.acquire()
        print("Thread 2 got lock 1.")
        lock1.release()
        lock2.release()


if __name__ == '__main__':
    t1 = MyThread1()
    t2 = MyThread2()
    t1.start()
    t2.start()
```

解决方式是：给`acquire()`添加`timeout`参数

### 生产者和消费者

```python
from queue import Queue
from threading import Thread

q = Queue()


def produce():
    for i in range(10):
        q.put(i)
    print('生产任务完毕！')
    """
    不写join，该函数立刻结束；
    写了join，所有任务都task_done时，join才会取消阻塞
    """
    q.join()
    print(produce.__name__, '函数结束！')


def consumer():
    for i in range(10):
        print('消费：', q.get())
        q.task_done()  # 每次get后都要调用task_done()

    print(consumer.__name__, '函数结束！')


pro = Thread(target=produce)
con = Thread(target=consumer)

pro.start()
con.start()

con.join()
pro.join()

print('主进程结束')

"""Outputs:
生产任务完毕！
消费： 0
消费： 1
消费： 2
消费： 3
消费： 4
消费： 5
消费： 6
消费： 7
消费： 8
消费： 9
consumer 函数结束！
produce 函数结束！
主进程结束
"""
```

### 协程

> coroutine。比线程更小。

在Python中可以使用生成器来实现协程。相关库有greenlet、gevent。

```python
# https://www.liaoxuefeng.com/wiki/897692888725344/923057403198272
import time


def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        time.sleep(1)
        r = '200 OK'


def produce(c):
    next(c)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()


if __name__ == '__main__':
    c = consumer()
    produce(c)
```