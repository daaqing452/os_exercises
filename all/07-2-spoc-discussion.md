# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)

第9题

	
	
信号量
```
#coding=utf-8

import random
import threading
import time


#	限定读上限为3
CanRead = threading.Semaphore(3)

#	写锁，在读的时候打开，保证不能写
CanWrite = threading.Semaphore(1)

#	读进入等待队列的锁，在写进入的时候打开，保证读不会跳到写的前面
CanReadIn = threading.Semaphore(1)

#	读个数
readerCnt = 0


#	暂停
def rest():
	time.sleep(random.randrange(1, 4))


#	读
class Reader(threading.Thread):
	def __init__(self, threadName):
		threading.Thread.__init__(self, name = threadName)
	def run(self):
		global CanRead, CanWrite, CanReadIn, readerCnt
		while True:
			#	判断读能进等待队列
			CanReadIn.acquire()
			CanReadIn.release()
			
			#	判断当前能否读
			CanRead.acquire()
			
			#	在读者数从0变成1的时候打开写锁
			if readerCnt == 0:
				CanWrite.acquire()
			readerCnt += 1
			
			print "%s read start\n" % (self.name)
			rest()
			print "%s read end\n" % (self.name)
			
			#	在读者数从1变成0的时候关闭写锁
			readerCnt -= 1
			if readerCnt == 0:
				CanWrite.release()
			
			CanRead.release()
			rest()


#	写
class Writer(threading.Thread):
	def __init__(self, threadName):
		threading.Thread.__init__(self,name = threadName)
	def run(self):
		global CanRead, CanWrite, CanReadIn, readerCnt
		while True:
			#	打开锁防止读进入等待队列
			CanReadIn.acquire()
			
			#	判断是否可写
			CanWrite.acquire()
			
			print "%s write start\n" % (self.name)
			rest()
			print "%s write end\n" % (self.name)
			
			CanWrite.release()
			CanReadIn.release()
			rest()

if __name__ == "__main__":
	for i in range(1, 5):
		thread = Reader("reader " + str(i))
		thread.start()
	for i in range(1, 2):
		thread = Writer("writer " + str(i))
		thread.start()
```

 
条件变量
```
#coding=utf-8

import random
import threading
import time


#	锁
Lock = threading.Condition()

#	活动读的个数
AR = 0

#	活动写的个数
AW = 0

#	等待读的个数
WR = 0

#	等待写的个数
WW = 0


#	暂停
def rest():
	time.sleep(random.randrange(1, 4))


#	读
class Reader(threading.Thread):
	def __init__(self):
		threading.Thread.__init__(self)
	def run(self):
		global Lock, AR, AW, WR, WW
		while True:
			Lock.acquire()
			
			#	变成等待读，直到没有写以及活动读小于等于2个
			while (AW + WW) > 0 or AR > 2:
				WR = WR + 1
				Lock.wait()
				WR = WR - 1
			
			#	转化为活动读
			AR = AR + 1
			Lock.release()
			
			print "%s read start\n" % (self.name)
			rest()
			print "%s read end\n" % (self.name)
			
			#	结束
			Lock.acquire()
			AR = AR - 1
			Lock.notifyAll()
			Lock.release()
			
			rest()
			

class Writer(threading.Thread):
	def __init__(self):
		threading.Thread.__init__(self)
	def run(self):
		global Lock, AR, AW, WR, WW
		while True:
			Lock.acquire()
			
			#	变成等待写，直到没有活动读和活动写
			while (AW + AR) > 0:
				WW = WW + 1
				Lock.wait()
				WW = WW - 1
				
			#	变成活动写
			AW = AW + 1
			Lock.release()
			
			print "%s write start\n" % (self.name)
			rest()
			print "%s write end\n" % (self.name)
			
			#	结束
			Lock.acquire()
			AW = AW - 1
			Lock.notifyAll()
			Lock.release()
			
			rest()

if __name__ == "__main__":
	for r in range(0, 5):
		r = Reader()
		r.start()

	for w in range(0, 2):
		w = Writer()
		w.start() 
```


2. (spoc)设计某个方法，能够动态检查出对于两个或多个进程的同步互斥问题执行中，没有互斥问题，能够同步等，以说明实现的正确性。
> 
https://github.com/MtMoon
