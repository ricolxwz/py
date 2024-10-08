---
title: 线程
icon: material/spider-thread
comments: false
---

## 多进程 {#多进程}

在Python中, 我们可以利用多种方法在父进程中创建子进程. 当我们用`python`命令执行了一个`.py`文件的时候, 就创建了一个进程, 在这个`.py`文件中, 我们通过一些手段又创建了一些进程, 这些进程就被称为子进程. 

???+ tip "Tip"

    - 父进程是创建子进程的进程
    - 子进程继承/拷贝了父进程的环境, 但是无法直接访问父进程的变量、函数或者其他资源, 子进程的变量和父进程的变量是独立的.

        ???+ example "例子"

            === "例子1"

                定义:

                ```py
                import os
                import subprocess

                # 设置父进程的环境变量
                os.environ['MY_VAR'] = 'Hello from parent process!'

                # 启动子进程，运行命令 'printenv MY_VAR' 以打印环境变量 MY_VAR 的值
                process = subprocess.Popen(['printenv', 'MY_VAR'], stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)

                # 获取子进程的输出
                stdout, stderr = process.communicate()

                # 打印子进程的输出
                print('STDOUT:', stdout) 
                ```

                输出:

                ```
                $ python main.py
                Hello from parent process!
                ```
            
            === "例子2"

                定义: 

                ```py
                import multiprocessing

                def worker():
                    global counter
                    counter += 1
                    print(f'Worker counter: {counter}')

                if __name__ == "__main__":
                    counter = 0
                    process = multiprocessing.Process(target=worker)
                    process.start()
                    process.join()
                    print(f'Main process counter: {counter}')
                ```

                输出:

                ```
                Worker counter: 1
                Main process counter: 0
                ```

                `worker()`函数中的`counter`变量是从父进程那边继承/拷贝过来的变量, 加`global`的原因是这个是个函数, 如果不加`global`, `counter`就是局部变量, 如果加了`global`, 那么`counter`就和子进程环境中的那个从父进程拷贝过来的`counter`是同一个`counter`.

    - 如果子进程需要与父进程共享某些数据, 可以使用各种进程间通信机制, 如管道、队列、共享内存等
    - 子进程有自己独立的地址空间, 文件描述符表, 栈等资源

Python中创建子进程的模块有三个: [`os`](#osmodule), [`subprocess`](#subprocessmodule)和[`multiprocessing`](#multiprocessingmodule), 这三个模块各自有各自的特色, 在选用的时候要仔细甄别.

### `os`模块 {#os模块}

`os`模块的`fork()`函数适用于Unix/Linux系统的低级进程创建, 直接调用系统的`fork()`接口, 适用于简单的子进程创建, 但是不提供高级的通信和同步机制. 调用`os.fork()`函数, 会返回两次, 因为操作系统把当前进程(父进程)复制了一份(子进程), 然后, 分别在父进程和子进程内返回. 之后, 父进程和子进程都会执行后面的代码. 

子进程永远返回0, 而父进程返回子进程的pid. 

???+ tip "Tip"

    `os.getpid()`函数可以获取当前进程的pid. `os.getppid()`函数可以获取当前进程的父进程的pid. 

???+ example "例子"

    定义: 

    ```py
    import os

    print('Process (%s) start...' % os.getpid())
    pid = os.fork()
    if pid == 0:
        print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
    else:
        print('I (%s) just created a child process (%s).' % (os.getpid(), pid))
    ```

    运行:

    ```
    $ python main.py
    Process (876) start...
    I (876) just created a child process (877).
    I am child process (877) and my parent is 876.
    ```

    ???+ warning "注意"

        Windows没有提供`fork()`接口, 所以上述代码在Windows下无法运行.

### `subprocess`模块 {#subprocess模块}

`subprocess`模块适用于启动和管理外部命令行子进程, 能够和外部命令行子进程通过`communicate()`函数实现进程通信, 提供了发出`stdin`和接收`stdout`, `stderr`的功能. 其中的`Popen()`函数用于返回一个子进程对象.

???+ example "例子"

    定义:

    ```py
    import subprocess

    print('$ nslookup')
    p = subprocess.Popen(['nslookup'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    output, err = p.communicate(b'set q=mx\npython.org\nexit\n')
    print(output.decode('utf-8'))
    print('Exit code:', p.returncode)
    ```

    上述代码相当于在命令行执行命令`nslookup`然后手动输入:

    ```
    set q=mx
    python.org
    exit
    ```

    运行: 

    ```
    $ nslookup
    Server:		192.168.19.4
    Address:	192.168.19.4#53

    Non-authoritative answer:
    python.org	mail exchanger = 50 mail.python.org.

    Authoritative answers can be found from:
    mail.python.org	internet address = 82.94.164.166
    mail.python.org	has AAAA address 2001:888:2000:d::a6


    Exit code: 0
    ```

    ???+ tip "Tip"

        `[subprocess_instance].communicate()`方法是一种父进程和子进程间的进程间通信方式, 它是阻塞的, 意味着父进程会等待子进程完成之后才继续执行. 通信时一次性的, 即父进程发送数据到子进程, 然后读取子进程的输出, 不能持续进行双向通信, 适用于简单的父进程和子进程之间的进程间通信.


### `multiprocessing`模块 {#multiprocessing模块}

`multiprocessing`模块适用于并行处理和多进程编程, 提供了丰富的进程间通信和同步机制, 支持跨平台. 其中的`Process`类能够用于创建子进程对象.

???+ example "例子"

    定义: 

    ```py
    from multiprocessing import Process
    import os

    # 子进程要执行的代码
    def run_proc(name):
        print('Run child process %s (%s)...' % (name, os.getpid()))

    if __name__=='__main__':
        print('Parent process %s.' % os.getpid())
        p = Process(target=run_proc, args=('test',))
        print('Child process will start.')
        p.start()
        p.join()
        print('Child process end.')
    ```

    运行: 

    ```
    Parent process 928.
    Child process will start.
    Run child process test (929)...
    Process end.
    ```

    通过`[process].start()`启动子进程, 运行的代码在`run_proc`中. 

    ???+ note "笔记"

        `[process].join()`用于阻塞主进程, 直到子进程执行完毕.

        ???+ example "例子"

            ```py
            import multiprocessing
            import time

            def worker(number):
                print(f"Worker {number} started")
                time.sleep(2)
                print(f"Worker {number} finished")

            if __name__ == "__main__":
                processes = []
                for i in range(4):  # 创建4个子进程
                    process = multiprocessing.Process(target=worker, args=(i,))
                    processes.append(process)
                    process.start()

                for process in processes:
                    process.join()  # 等待所有子进程完成

                print("All workers completed")
            ```

            上述的例子中, 创建了4个子进程, 每个子进程执行`worker`函数, 如果任务都在主进程中顺序执行, 总共需要8秒完成. 而使用多进程同时执行4个任务, 只需要大约2秒的即可完成所有任务. `process.join()`函数用于阻塞当前的主进程, 当子进程全部执行完之后, 才打印"All workers completed".

            当然上述的代码还可以优化成使用[进程池](#进程池). 

### 进程池 {#进程池}

进程池由[multiprocessing模块](#multiprocessingmodule)的`Pool`类提供支持.

???+ example "例子"

    定义:

    ```py
    from multiprocessing import Pool
    import os, time, random

    def long_time_task(name):
        print('Run task %s (%s)...' % (name, os.getpid()))
        start = time.time()
        time.sleep(random.random() * 3)
        end = time.time()
        print('Task %s runs %0.2f seconds.' % (name, (end - start)))

    if __name__=='__main__':
        print('Parent process %s.' % os.getpid())
        p = Pool(4)
        for i in range(5):
            p.apply_async(long_time_task, args=(i,))
        print('Waiting for all subprocesses done...')
        p.close()
        p.join()
        print('All subprocesses done.')
    ```

    运行: 

    ```
    $ python main.py
    Parent process 669.
    Waiting for all subprocesses done...
    Run task 0 (671)...
    Run task 1 (672)...
    Run task 2 (673)...
    Run task 3 (674)...
    Task 2 runs 0.14 seconds.
    Run task 4 (673)...
    Task 1 runs 0.27 seconds.
    Task 3 runs 0.86 seconds.
    Task 0 runs 1.41 seconds.
    Task 4 runs 1.91 seconds.
    All subprocesses done.
    ```

    ???+ tip "Tip"
    
        `[pool_instance].apply_async()`用于提交任务, 任务是否立即被执行取决于当前进程池中工人的数量, 即进程池对象被创建时的参数`Pool([worker_number])`, 默认的大小是当前电脑CPU的核心数量. 

        ???+ example "例子"

            如在上述的例子中, 我们定义了进程池中工人的数量是4个, 那么最多同时只能执行4个任务, 任务0, 1, 2, 3是立即执行的, 而任务4是等待前面某个任务执行完成之后才执行. 如果没有明确工人的数量, 那么默认是CPU的核心数量, 比如说8核处理器, 那么至少提交9个任务才会看到上面的等待效果.

### 进程间通信

队列式进程间通信由[multiprocessing模块](#multiprocessingmodule)的`Queue`类提供支持, 此外还有管道式进程间通信, 这里不会详细介绍.

???+ example "例子"

    定义:

    ```py
    from multiprocessing import Process, Queue
    import os, time, random

    # 写数据进程执行的代码:
    def write(q):
        print('Process to write: %s' % os.getpid())
        for value in ['A', 'B', 'C']:
            print('Put %s to queue...' % value)
            q.put(value)
            time.sleep(random.random())

    # 读数据进程执行的代码:
    def read(q):
        print('Process to read: %s' % os.getpid())
        while True:
            value = q.get(True)
            print('Get %s from queue.' % value)

    if __name__=='__main__':
        # 父进程创建Queue，并传给各个子进程：
        q = Queue()
        pw = Process(target=write, args=(q,))
        pr = Process(target=read, args=(q,))
        # 启动子进程pw，写入:
        pw.start()
        # 启动子进程pr，读取:
        pr.start()
        # 等待pw结束:
        pw.join()
        # pr进程里是死循环，无法等待其结束，只能强行终止:
        pr.terminate()
    ```

    运行: 

    ```
    $ python main.py
    Process to write: 50563
    Put A to queue...
    Process to read: 50564
    Get A from queue.
    Put B to queue...
    Get B from queue.
    Put C to queue...
    Get C from queue.
    ```

    上述的例子中, 一个子进程用于向队列里面写数据, 另一个子进程用于由队列里面读数据.

## 多线程 {#多线程}

进程是由若干线程组成的, 一个进程至少有一个线程. 线程是操作系统直接支持的执行最小单元. 由于任何进程默认都会启动一个线程, 我们将该线程成为主线程, 其他有主线程创建出来的线程成为子线程. Python的线程是真正的Posix Thread不是模拟出来的线程. 

Python中创建线程的模块有两个: `_thread`和`threading`, `_thread`是低级模块, `threading`是高级模块, 对`_thread`进行了封装, 提供了一些更简单的接口, 一般情况下, 只需要使用`threading`模块.

???+ example "例子"

    定义: 

    ```py
    import time, threading

    # 新线程执行的代码:
    def loop():
        print('thread %s is running...' % threading.current_thread().name)
        n = 0
        while n < 5:
            n = n + 1
            print('thread %s >>> %s' % (threading.current_thread().name, n))
            time.sleep(1)
        print('thread %s ended.' % threading.current_thread().name)

    print('thread %s is running...' % threading.current_thread().name)
    t = threading.Thread(target=loop, name='LoopThread')
    t.start()
    t.join()
    print('thread %s ended.' % threading.current_thread().name)
    ```

    执行: 

    ```
    $ python main.py
    thread MainThread is running...
    thread LoopThread is running...
    thread LoopThread >>> 1
    thread LoopThread >>> 2
    thread LoopThread >>> 3
    thread LoopThread >>> 4
    thread LoopThread >>> 5
    thread LoopThread ended.
    thread MainThread ended.
    ```

    ???+ tip "Tip"

        `threading`模块的`current_thread()`函数, 用于返回当前线程的实例, 我们可以访问这个实例的`name`属性来查看它的名字, 子线程的名字是在创建的时候指定的, 主线程的名字为`MainThread`. 如果子线程没有起名字, 那么Python就会自动给线程命名为`Thread-1`, `Thread-2`, ....

### CPython中线程无法并行 {#CPython中线程无法并行}

由于[GIL锁](#GIL锁)的存在, Python的多线程虽然是真正的多线程, 但是在CPython解释器执行代码的时候, 多线程其实还是交替执行的, 也就是说没有并行执行, 只能在一个CPU核心上跑. 所以说在计算密集型任务中, 多线程的表现甚至还不如单线程, 因为有额外的线程切换的开销. 但是在IO密集型任务中, IO操作通常会导致线程阻塞, 在这种情况下, 阻塞线程将不占用CPU, GIL锁将释放给其他的线程, 这种情况下, 多线程的优势就比较明显.

### 线程的局部变量

在默认状态下, 同一个进程下的线程之间是可以互相访问, 修改数据的. 如果一个线程想要有自己的一份独立数据, 不希望其他线程访问, 就可以使用`threading`模块提供的`ThreadLocal`对象(需要在主线程中定义), 可以由`threading.local()`函数创建.

???+ example "例子"

    定义: 

    ```py
    import threading
    
    # 创建全局ThreadLocal对象:
    local_school = threading.local()

    def process_student():
        # 获取当前线程关联的student:
        std = local_school.student
        print('Hello, %s (in %s)' % (std, threading.current_thread().name))

    def process_thread(name):
        # 绑定ThreadLocal的student:
        local_school.student = name
        process_student()

    t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
    t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    ```

    执行: 

    ```
    $ python main.py
    Hello, Alice (in Thread-A)
    Hello, Bob (in Thread-B)
    ```

    全局变量`local_school`就是一个`ThreadLocal`对象, 每个子线程都可以读写它自己的`student`属性, 相互之间没有影响. 可以把`local_school`堪称全局变量, 但是每个属性如`local_school_student`都是线程的局部变量, 可以任意读写而互不干扰.

## 协程

协程将会在[这里](/foundation/async/#协程)讲到.

## 锁

锁的作用主要是在多线程或者多进程环境下, 确保在同一时间只有一个线程或者一个进程可以可以进入临界区, 解决了竞争的问题, 保护了数据的完整性. 在Python中, 进程和线程实现锁的方式稍有不同.

### 进程锁

进程锁主要依靠`multiprocessing`模块中的`Lock`类创建的锁对象来解决.

???+ example "例子"

    定义: 

    ```py
    import multiprocessing
    import time

    # 定义一个函数，用于多个进程执行的任务
    def worker(lock, shared_resource, index):
        with lock:
            print(f'Process {index} is accessing the shared resource...')
            shared_resource.value += 1
            time.sleep(1)
            print(f'Process {index} done. Shared resource value: {shared_resource.value}')

    if __name__ == '__main__':
        lock = multiprocessing.Lock()
        shared_resource = multiprocessing.Value('i', 0)  # 定义一个共享整数资源，初始值为0

        processes = []
        for i in range(5):
            p = multiprocessing.Process(target=worker, args=(lock, shared_resource, i))
            processes.append(p)
            p.start()

        for p in processes:
            p.join()

        print('All processes are done. Final value of shared resource:', shared_resource.value)
    ```

    === "解释1"

        ```py
        with lock:
            print(f'Process {index} is accessing the shared resource...')
            shared_resource.value += 1
            time.sleep(1)
            print(f'Process {index} done. Shared resource value: {shared_resource.value}')
        ```

        `with lock`是Python中的一种上下文管理器用法, `lock`对象中实现了`__enter__`和`__exit__`方法, 其中的`__enter__`方法定义为用于上锁, `__exit__`方法用于解锁, `with`语句确保在里面的代码块在执行开始前自动调用`__enter__`上锁, 执行完毕后自动调用`__exit__`方法解锁.

    === "解释2"

        ```py
        shared_resource = multiprocessing.Value('i', 0)
        ```

        `multiprocessing.Value`允许创建一个简单的对象用于进程间共享.

### 线程锁

线程锁主要依靠`threading`模块中的`Lock`类创建的锁对象来解决.

???+ example "例子"

    定义:

    ```py
    import threading
    import time

    # 工作线程函数
    def worker(index):
        global shared_resource
        with lock:
            print(f'Thread {index} is accessing the shared resource...')
            shared_resource += 1
            time.sleep(1)
            print(f'Thread {index} done. Shared resource value: {shared_resource}')

    if __name__ == '__main__':
        lock = threading.Lock()
        shared_resource = 0

        threads = []
        for i in range(5):
            t = threading.Thread(target=worker, args=(i,))
            threads.append(t)
            t.start()

        for t in threads:
            t.join()

        print('All threads are done. Final value of shared resource:', shared_resource)
    ```

    === "解释1"

        ```py
        with lock:
            print(f'Thread {index} is accessing the shared resource...')
            shared_resource += 1
            time.sleep(1)
            print(f'Thread {index} done. Shared resource value: {shared_resource}')
        ```

        `with lock`是Python中的一种上下文管理器用法, `lock`对象中实现了`__enter__`和`__exit__`方法, 其中的`__enter__`方法定义为用于上锁, `__exit__`方法用于解锁, `with`语句确保在里面的代码块在执行开始前自动调用`__enter__`上锁, 执行完毕后自动调用`__exit__`方法解锁.

    === "解释2"

        ```py
        shared_resource = 0
        ```

        子线程中的`shared_resource`对象指向的都是这个主线程中的`shared_resource`, 不需要创建什么别的对象来解释这个例子.

    ???+ warning "注意"

        由于线程之间共享进程的内存空间, 所以现存中所有的变量其实都是属于进程的, 而且都是同一个.

        ???+ example "例子"
            
            上述例子中的`shared_resource`, 它在主线程中创建, 在子线程中, 所有的`shared_resource`都指向的是同一块内存区域, 使用`global`的原因是需要在`worker()`函数内使用这个变量, 如果不加, 会变成一个局部变量, 和函数外的`share_resource`就不是同一个了, 如果加了, 指向的就是主线程的`shared_resource`所在的内存区域.

### GIL锁 {#GIL锁}

GIL锁, Global Interpreter Lock, 全局解释器锁, 是Python解释器中的一个互斥锁, 用于保护访问Python对象的底层实现, 简化了CPython的实现和维护. GIL锁确保了任何时候只有一个线程能够访问Python字节码, 这就意味着即使在多线程环境下, 多线程代码无法真正的并行执行.

???+ example "例子"

    定义: 

    ```
    import threading, multiprocessing

    def loop():
        x = 0
        while True:
            x = x ^ 1

    for i in range(multiprocessing.cpu_count()):
        t = threading.Thread(target=loop)
        t.start()
    ```

    假设在一个4核心CPU上运行上述代码, 你会发现CPU利用率仅有100%, 而不是400%, 这说明只有一个核心在真正工作. 但是我们明明启动了和CPU核心数相等的线程.

    这就是因为GIL锁的存在. GIL锁使得线程在执行之前必须获得GIL锁, 而解释器每执行100条字节码就会释放一次GIL锁让其他线程有机会执行. 所以在实际运行的时候, 各线程只能轮流执行, 而不能并行执行. 因此, 无论启动多少线程, 最终也只能有一个线程在某一时刻执行代码.

    与Python不同, C, C++或Java都没有GIL锁, 多线程可以真正并发执行, 比如上述代码换成C++版本执行, 利用率就是400%.

???+ tip "Tip"

    GIL锁的解决方法: 

    1. 使用[多进程](#多进程): 每个Python进程都有自己独立的GIL锁, 因此可以并行执行
    2. 使用C扩展: 使用C扩展模块来执行耗时的计算任务, 这些C扩展可以释放GIL锁, 从而允许其他线程并行执行
    3. 使用不同的解释器: 不用CPython解释器, 而使用替代解释器, 如`Jython, IronPython`等等, 没有GIL锁, 但是这些解释器不完全兼容CPython, 生态不如CPython丰富

### 锁的优点和缺点

=== "锁的优点"

    - 确保某段关键代码只能由一个进程/线程从头到尾完整地执行
    - 保证在同一个时间只有一个进程/线程进入临界区
    - 保护了共享资源的完整性

=== "锁的缺点"

    - 阻止了多线程的并行执行
    - 可能导致死锁, 需要操作系统介入

## 多进程 vs. 多线程 vs. 协程

要实现多任务, 通常我们会设计Master-Worker模式, Master负责分配任务, Worker负责执行任务, 在多任务环境下, 通常为一个Master, 多个Worker. 

- 如果用[多进程](#多进程)实现, 主进程为Master, 其他子进程为Worker
- 如果用[多线程](#多threading)实现, 主线程为Master, 其他线程为Worker

协程使用的不是Master-Worker模式, 它使用的是单个线程, 使用的是异步的方法来模拟执行多任务.

### 多进程优缺点

=== "多进程优点"

    - 稳定性高: 一个子进程崩溃了, 不会影响到主进程和其他子进程; 当然主进程崩溃了, 所有的子进程也会都崩溃.

=== "多进程缺点"

    - 开销巨大: 在Unix/Linux下, 用`os.fork()`建立子进程还行, 在Windows下创建子进程的开销巨大. 另外, 操作系统能够同时运行的进程数量也是有限的. 

### 多线程优缺点

=== "多线程优点"

    - 效率高: 由于共享相同的内存空间, 所以在上下文切换的开销较小, 不需要像进程切换那样的复杂操作.

=== "多线程缺点"

    - 稳定性差: 任何一个线程崩溃都有可能导致整个进程崩溃. 因为所有线程共享进程的内存空间.

### 协程优缺点

=== "协程优点"

    - 效率极高: 协程没有切换带来的额外开销, 因为在单个线程内实现模拟并行操作, 特别适用于IO密集型任务.
    - 轻量级: 协程非常轻量级, 可以在同一线程中运行成千上万的协程.
    - 易于管理: 协程的调度由程序控制, 而进程和线程的调度由操作系统内核控制, 减少了竞争和锁的使用, 简化了代码的复杂性.

=== "协程缺点"

    - 需要显式调度: 协程的切换需要显式在代码中标明, 需要开发者清楚地理解和管理协程
    - 异步处理复杂: 在复杂的协程网络中, 异常的处理和传播可能比较复杂.

[^1]: 多进程. (n.d.). Retrieved June 16, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017628290184064
[^2]: 多线程. (n.d.). Retrieved June 17, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017629247922688
[^3]: 进程 vs. 线程. (n.d.). Retrieved June 17, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017631469467456
[^4]: ThreadLocal. (n.d.). Retrieved June 17, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017630786314240
[^5]: 分布式进程. (n.d.). Retrieved June 17, 2024, from https://www.liaoxuefeng.com/wiki/1016959663602400/1017631559645600