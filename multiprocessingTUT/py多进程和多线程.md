- [1. threading--多线程](#1-threading--多线程)
  - [1.1. 获得线程信息](#11-获得线程信息)
  - [1.2. 基本语法--创建任务](#12-基本语法--创建任务)
    - [1.2.1. **语法**<br>](#121-语法)
    - [1.2.2. **实例**](#122-实例)
  - [1.3. join](#13-join)
    - [1.3.1. 实例](#131-实例)
  - [1.4. queue](#14-queue)
    - [1.4.1. 语法](#141-语法)
    - [1.4.2. 实例](#142-实例)
  - [1.5. GIL](#15-gil)
  - [1.6. lock](#16-lock)
    - [1.6.1. 语法](#161-语法)
    - [1.6.2. 实例](#162-实例)
- [2. multiprocessing--多核运行](#2-multiprocessing--多核运行)
  - [2.1. 与threading异同](#21-与threading异同)
  - [2.2. multiprocessing 和 threading 速度比较](#22-multiprocessing-和-threading-速度比较)
  - [2.3. lock 和 multiprocessing.Value() 实例](#23-lock-和-multiprocessingvalue-实例)

# 1. threading--多线程

##  1.1. 获得线程信息
``` python
{
    print(threading.active_count()) # 返回已经激活的线程
    print(threading.enumerate()) # 返回所有线程
    print(threading.current_thread()) # 返回当前线程
}
```
<br>

## 1.2. 基本语法--创建任务

### 1.2.1. **语法**<br>
<p><code>thread = threading.Thread(target=,args=(),name=,)</code></p>
<p><code>target</code>: 调用的函数</p>
<p><code>args</code>: 调用函数传入的实参</p>
<p><em>注意，上面仅仅是创建一个线程任务，但是还未运行，用<code>thread.start()</code>来开始thread这个线程</em></p>
<br>

### 1.2.2. **实例**  
``` python
{
    import threading

    def thread_job():
        print('This is a thread of %s' % threading.current_thread())

    def main():
        thread = threading.Thread(target=thread_job,)
        thread.start()

    if __name__ == '__main__':
        main()
}
```
<br>

**运行结果:**
<code>This is a thread of <Thread(Thread-1 (thread_job), started 1408)></code>

## 1.3. join 
**如果在start一个thread任务后面还有其他代码，那么thread任务还在执行的时候其他代码也同时执行。<br>如果要避免这种情况(等thread先执行完再往后运行非thread程序),可以用`thread.join()`**

### 1.3.1. 实例
** 没有用`thread.join()`:
``` python
{
    import threading
    import time

    def T1_job():
        print('T1 start\n')
        for i in range(10):
            time.sleep(0.1)
        print('T1 finish\n')

    def T2_job():
        print('T2 start\n')
        print('T2 finish\n')

    def main():
        threda1 = threading.Thread(target=T1_job, name='T1')
        thread2 = threading.Thread(target=T2_job, name='T2')
        thread1.start()
        thread2.start()

        print('all done\n')

    if __name__ == '__main__':
        main()
}
```
<br>

**运行结果:**
```
T1 start

T2 start

all done

T2 finish

T1 finish
```
<br>

** 使用`join()`  
``` python
{
    import threading
    import time

    def T1_job():
        print('T1 start\n')
        for i in range(10):
            time.sleep(0.1)
        print('T1 finish\n')

    def T2_job():
        print('T2 start\n')
        print('T2 finish\n')

    def main():
        threda1 = threading.Thread(target=T1_job, name='T1')
        thread2 = threading.Thread(target=T2_job, name='T2')
        thread1.start()
        thread2.start()
        ##############
        thread1.join()
        thread2.join()
        ##############

        print('all done\n')

    if __name__ == '__main__':
        main()

}
```
<br>

**运行结果:**<br>
```
T1 start

T2 start

T2 finish

T1 finish

all done
```
<br>

## 1.4. queue
**thread多线程任务调用的函数不能用返回值，可以用queue代替，把返回的结果put进入queue中**<br>

### 1.4.1. 语法
``` python
    from queue import Queue
    q = Queue()  # 定义q值
    def fun(q):
        """
        代码...
        value = ...
        """
        q.put(value)  # 把返回值放入q中
    for _ in range(4):
        fun(q)
    for _ in range(4):
        q.get() # 一个个取出q值
```
<br>  

### 1.4.2. 实例
``` python 

    import threading
    import time
    from queue import Queue

    def job(l,q):
        for i in range(len(l)):
            l[i] = l[i]**2
        q.put(l)

    def multithreading():
        q = Queue()
        threads = []
        data = [[1,2,3],[3,4,5],[4,4,4],[5,5,5]]
        for i in range(4):
            t = threading.Thread(target=job, args=(data[i], q))
            t.start()
            threads.append(t)

        for thread in threads:
            thread.join()
        results = []

        for _ in range(4):
            results.append(q.get())
        print(results)

    if __name__ == '__main__':
        multithreading()
```
<br>

**运行结果:**
`[[1, 4, 9], [9, 16, 25], [16, 16, 16], [25, 25, 25]]`

## 1.5. GIL
***对于`threading`这个包，虽然创建了多线程，但是多线程并不是同时运行的，同一时间只有一个线程运行，在不同时间内多个任务交替运行，每个线程处理一种任务。***

## 1.6. lock
***运行两个线程任务时，两个任务交替运行时会相互干扰（比如两个任务都要Print时，输出会混乱）,用lock可以解决这个问题***  
### 1.6.1. 语法
``` python
    import threading
    lock = threading.Lock() # 创建lock

    def job(lock):
        lock.acquire()
        """代码..."""
        lock.release()

    t = threading.Thread(target=job,args=(lock)) # 传递lock
    
    t.start()
    t.join()
```
<br>

### 1.6.2. 实例 
``` python
    import threading

    def job1():
        global A, lock
        lock.acquire()
        for i in range(10):
            A += 1
            print('job1', A)
        lock.release()

    def job2():
        global A, lock
        lock.acquire()
        for i in range(10):
            A += 10
            print('job2', A)
        lock.release()

    if __name__ == '__main__':
        lock = threading.Lock()
        A = 0
        t1 = threading.Thread(target=job1)
        t2 = threading.Thread(target=job2)
        t1.start()
        t2.start()
        t1.join()
        t2.join()
```
**运行结果:**  
```
    job1 1
    job1 2
    job1 3
    job1 4
    job1 5
    job1 6
    job1 7
    job1 8
    job1 9
    job1 10
    job2 20
    job2 30
    job2 40
    job2 50
    job2 60
    job2 70
    job2 80
    job2 90
    job2 100
    job2 110
```

# 2. multiprocessing--多核运行
## 2.1. 与threading异同
- [x] 一样是`x.start()` 开始运行任务，`x.join()` 等待任务结束再往下运行
- [x] 一样有Lock方法 `(lock = threading.Lock()) / lock = multiprocessing.Lock()`
- [ ] `threading: threading.Thread(target=, args=(), name=)`
`multiprocessing: multiprocessing.Process(target=, arge=(), name=)` 
- [ ] multiprocessing自带`Queue (q = multiprocessing.Queue())`，但是threading需要导入queue包 `(from queue import Queue; q = Queue())`，其中`multiprocessing`自带的`Queue()`可以给`threading`的任务用
- [ ] `multiprocessing`如果多个进程共用一个global变量需要如下方式创建：<br>`var = multiprocessing(类型，值)`,比如 `var = multiprocessing('i',0)` 代表创建一个数值为0的四字节整型
另外还可以用`multiprocessing.Array(类型，值)`创建**一维数组** (ps:只能是一维数组)
- [ ] `multiprocessing.Pool()`可以创建一个pool自动分配进程，并且可以不用Queue()获得返回值，可以用`return`
``` python
    import multiprocessing as mp

    def job(x):
        return x*x

    def multicore():
        pool = mp.Pool(processes=2) # 可以指定进程数,不指定的话不填
        res = pool.map(job, range(10))
        print(res)
        res = pool.apply_async(job, (2,)) # apply_async一次只能运行一个任务
        print(res.get())
        # 下面用法和pool.map()一致
        multi_res =[pool.apply_async(job, (i,)) for i in range(10)]
        print([res.get() for res in multi_res])

    if __name__ == '__main__':
        multicore()
```
<br>
**输出结果**
```
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
4
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```
<br>


## 2.2. multiprocessing 和 threading 速度比较
**多核用四核 (八线程)，多线程用四线程**
``` python

    import multiprocessing as mp
    import threading as td
    import time
    N = 10000000

    def job(q):
        q.put(sum(i+i**2+i**3 for i in range(N)))

    def multicore():
        q = mp.Queue()
        cores = []
        for _ in range(4):
            p = mp.Process(target=job,args=(q,))
            p.start()
            cores.append(p)
        for core in cores:
            core.join()
        res = sum(q.get() for _ in range(len(cores)))
        print("multicore calculated result:",res)

    def multithread():
        q = mp.Queue()
        threads = []
        for _ in range(4):
            t = td.Thread(target=job,args=(q,))
            t.start()
            threads.append(t)
        for thread in threads:
            thread.join()
        res = sum(q.get() for _ in range(len(threads)))
        print("multithread calculated result:",res)

    def normal():
        print("normal calculated result",sum(i+i**2+i**3 for _ in range(4) for i in range(N)))

    if __name__ == "__main__":
        t1 = time.time()
        multicore()
        t2 = time.time()
        multithread()
        t3 = time.time()
        normal()
        t4 = time.time()
        print("multicore time:",t2-t1)
        print("multithread time:",t3-t2)
        print("normal time:",t4-t3)
```
<br>
**运行结果:**  
```
multicore calculated result: 9999999333333433333320000000
multithread calculated result: 9999999333333433333320000000
normal calculated result 9999999333333433333320000000
multicore time: 4.598999977111816
multithread time: 17.36699938774109
normal time: 17.513998985290527
```
<br>

## 2.3. lock 和 multiprocessing.Value() 实例
``` python
    import multiprocessing as mp 
    import time

    def job(v,num,lock):
        lock.acquire()
        for _ in range(10):
            v.value += num
            print(v.value)
        lock.release()

    def multicole():
        lock = mp.Lock()
        v = mp.Value('i',0)
        p1 = mp.Process(target=job,args=(v,1,lock))
        p2 = mp.Process(target=job,args=(v,10,lock))
        p1.start()
        p2.start()
        p1.join()
        p2.join()

    if __name__ == "__main__":
        multicole()
```
<br>
**运行结果:**  
```
1
2
3
4
5
6
7
8
9
10
20
30
40
50
60
70
80
90
100
110
```
temo