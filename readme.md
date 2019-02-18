# **python 多进程学习**
## **简介与前言**
- 单核CPU执行多任务原理：
  - 操作系统交替轮流执行各个任务。先让任务1执行0.01秒，然后切换到任务2执行0.01秒，再切换到任务3执行0.01秒...这样往复地执行下去。由于cpu的执行速度非常快，所以使用者的主观感受就是这些任务在并行地执行。
- 　多核cpu执行多任务的原理：
  - 由于实际应用中，任务的数量往往远超过cpu的核数，所以操作系统实际上是把这些多任务轮流地调度到每个核心上执行。
- 对于操作系统来说，一个应用就是一个进程。比如打开一个浏览器，它是一个进程；打开一个记事本，它是一个进程。每个进程有它特定的进程号。他们共享系统的内存资源。
- **进程是操作系统分配资源的最小单位。**
- 而对于每一个进程而言，比如一个视频播放器，它必须同时播放视频和音频，就至少需要同时运行两个“子任务”，进程内的这些子任务就是通过线程来完成。**线程是最小的执行单元。**
- **一个进程它可以包含多个线程，这些线程相互独立，同时又共享进程所拥有的资源。**
### **线程与进程的区别总结**
- 线程共享内存空间；进程的内存是独立的
- 同一个进程的线程之间可以直接交流；两个进程想通信，必须通过一个中间代理来实现
- 创建新线程很简单； 创建新进程需要对其父进程进行一次克隆
- 一个线程可以控制和操作同一进程里的其他线程；但是进程只能操作子进程
- 改变主线程（如优先权），可能会影响其它线程；改变父进程，不影响子进程

## ** 进程模块 multiprocessing **
- python中的多线程其实并不是真正的多线程，如果想要充分地使用多核CPU的资源，在python中大部分情况需要使用多进程
- multiprocessing是Python提供的一个跨平台的多进程模块，通过它可以很方便地编写多进程程序，在不同的平台（Unix/Linux, Windows）都可以执行
- multiprocessing支持子进程、通信和共享数据、执行不同形式的同步，提供了Process、Queue、Pipe、Lock等组件
#### **创建单个进程**
- 利用multiprocessing.Process对象可以创建进程

**Process介绍**
**构造方法**
  - Process([group [, target [, name [, args [, kwargs]]]]])
  - group: 线程组，目前还没有实现，库引用中提示必须是None；
  - target: 要执行的方法；
  - name: 进程名；
  - args/kwargs: 要传入方法的参数
**实例方法**
  - is_alive()：返回进程是否在运行。
  - join([timeout])：阻塞当前上下文环境的进程程，直到调用此方法的进程终止或到达指定的timeout（可选参数）。
  - start()：进程准备就绪，等待CPU调度。
  - run()：strat()调用run方法，如果实例进程时未制定传入target，这star执行t默认run()方法。
  - terminate()：不管任务是否完成，立即停止工作进程。
**属性**
    authkey
  - daemon：和线程的setDeamon功能一样（将父进程设置为守护进程，当父进程结束时，子进程也结束）。
  - exitcode(进程在运行时为None、如果为–N，表示被信号N结束）。
  - name：进程名字。
  - pid：进程号

**创建一个进程任务**
``` python
import multiprocessing
import time

def worker(interval):
    n = 5
    while n > 0:
        print("The time is {0}".format(time.ctime()))
        time.sleep(interval)
        n -= 1

if __name__ == "__main__":
    p = multiprocessing.Process(target = worker, args = (3,))
    p.start()
    print "p.pid:", p.pid
    print "p.name:", p.name
    print "p.is_alive:", p.is_alive()

```
程序输出：
```
p.pid: 1896
p.name: Process-1
p.is_alive: True
The time is Mon Feb 18 19:42:49 2019
The time is Mon Feb 18 19:42:52 2019
The time is Mon Feb 18 19:42:55 2019
The time is Mon Feb 18 19:42:58 2019
The time is Mon Feb 18 19:43:01 2019
```
**创建多个进程任务**
``` python
import multiprocessing
import time

def worker_1(interval):
    print("worker_1")
    time.sleep(interval)
    print("end worker_1")

def worker_2(interval):
    print("worker_2")
    time.sleep(interval)
    print("end worker_2")

def worker_3(interval):
    print("worker_3")
    time.sleep(interval)
    print("end worker_3")

if __name__ == "__main__":
    p1 = multiprocessing.Process(target = worker_1, args = (2,))
    p2 = multiprocessing.Process(target = worker_2, args = (3,))
    p3 = multiprocessing.Process(target = worker_3, args = (4,))

    p1.start()
    p2.start()
    p3.start()

    print("The number of CPU is:" + str(multiprocessing.cpu_count()))
    for p in multiprocessing.active_children():
        print("child   p.name:" + p.name + "\tp.id" + str(p.pid))
    print("END!!!!!!!!!!!!!!!!!")


```
程序输出
``` 
The number of CPU is:8
child   p.name:Process-1        p.id12848
child   p.name:Process-2        p.id4796
child   p.name:Process-3        p.id8988
END!!!!!!!!!!!!!!!!!
worker_1
worker_2
worker_3
end worker_1
end worker_2
end worker_3
```
**将进程定义为类**
- 进程p调用start()时，自动调用run()
``` python
import multiprocessing
import time

class ClockProcess(multiprocessing.Process):
    def __init__(self, interval):
        multiprocessing.Process.__init__(self)
        self.interval = interval

    def run(self):
        n = 5
        while n > 0:
            print("the time is {0}".format(time.ctime()))
            time.sleep(self.interval)
            n -= 1

if __name__ == '__main__':
    p = ClockProcess(3)
    p.start()
```
输出
```
the time is Mon Feb 18 20:17:20 2019 
the time is Mon Feb 18 20:17:23 2019
the time is Mon Feb 18 20:17:26 2019
the time is Mon Feb 18 20:17:29 2019
the time is Mon Feb 18 20:17:32 2019
```
**join()方法介绍**
- join()方法可以**等待子进程结束后再继续往下运行**，通常用于进程间的同步。
- 哪个子进程调用了join方法，**主进程就要等该子进程执行完后才能继续向下执行**



## ** 线程模块 threadding **


### **Python下多线程是鸡肋，推荐使用多进程！**



