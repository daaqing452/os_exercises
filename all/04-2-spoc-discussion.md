#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象
> 设在t时刻，S(t)是大小为n的栈中的内容，S’(t)是大小为n+k的栈中的内容，其中k>0,B(t)为新入栈的元素 只需证明对于任意t，S(t)属于S’(t)，且任意S(t)中元素a，对应到S‘(t)中元素a1，满足a的位置小于等于a1的栈位置,即可证明物理页面数量增加的缺页率不会降低。 数学归纳法： 在初始情况下，S(0)与S’(0)都为空，满足任意S(0)中元素a，对应到S’(0)中元素a1 假设在t-1时刻满足S（t-1）属于S’(t-1)，且任意S(t-1)中元素a，对应到S’(t-1)中元素a1，满足a的位置小于等于a1的栈位置 在t时刻，对于对B(t)页的页面访问请求，可能出现以下三种情况 【情况1】：B(t)属于S(t),且B(t)属于S’(t),则经过这一步，需要将S(t)与S’(t)中的B(t)页都置于栈顶部，因为S1的栈大小大于S(t),所以对于B(t)元素，还是满足B(t)在S(t)中的位置，小于B(t)在S’(t)中的位置。对于原有在S(t)与S’(t)中的元素，在B(t)元素前的元素位置不变，在B(t)后的元素位置整体前移，大小关系保持不变。所以这种情况下依旧满足条件。 【情况2】：B(t)不属于S(t),且B(t)属于S’(t)。B(t)不属于S(t),就在栈顶压入元素，B(t)位置为n；在S‘(t)中找到B(t)，将B(t)元素移至栈顶，依旧满足S(t)中B(t)位置小于等于S‘(t)中位置。对于原有在S(t)中的元素，整体前移了一位，S’(t)中元素B(t)前的不变，B(t)后的整体前移1位，所以整体大小关系依旧满足。 【情况3】：B(t)不属于S(t),且B(t)不属于S‘(t),则都在栈尾部加入B(t)，S(t)中位置为n，S’(t)中位置为n+k。同时对于被弹出栈的元素，如果弹出元素相同，则依旧满足。如果弹出元素不同，因为S(t)中的对应元素位置小于等于S‘(t)的，所以S’(t)中弹出的元素必然已经不属于S(t)了。所以弹出后依旧满足S(t)属于S‘(t).即在这种情况下依旧满足假设。 由于假设的存在，S(t)属于S’(t),即不会出现B(t)属于S(t),B(t)不属于S‘(t)的情况。 综上所述，由数学归纳法得，对任意时刻t，S(t)属于S’(t),且任意S(t)中元素a，对应到S’(t)中元素a1，满足a的位置小于等于a1的栈位置。即对任意时刻，对S’(t)的缺页数量不会大于S(t)。即物理页数量增加，缺页率不会上升。即LRU算法不会出现belady现象。

(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)

> ```
import random

maxx = 10
n = 100
window = 8
work = []
mem = set([])

miss = 0
for i in range(0, n) :
    now = random.randint(0, maxx)

    if not (now in mem) :
        miss = miss + 1

    mem.add(now)

    work.append(now)
    if len(work) > window :
        old = work[0]
        if work.count(old) == 1 :
            mem.remove(old)    
        work = work[1:]

    # print "memo: " + str(mem)
    # print "work: " + str(work)
    # print ""
    
print miss * 1.0 / n
```

## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
