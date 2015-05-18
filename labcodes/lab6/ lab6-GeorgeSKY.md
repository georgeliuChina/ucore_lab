# OS Lab6 实验报告



刘相P14206021

### 练习0
---
1. 	<b>添加lab1~lab5原有代码，并进行适当更新。</b>

	> * alloc_proc() in proc.c
	```
	proc->rq = NULL;
    	proc->run_link.prev = proc->run_link.next = NULL;
    	proc->time_slice = 0;
    	proc->lab6_run_pool.left = proc->lab6_run_pool.right = proc->lab6_run_pool.parent = NULL;
    	proc->lab6_stride = 0;
   	 proc->lab6_priority = 0;	
	```

	> * trap_dispatch() in trap.c
	```
	ticks ++;
	sched_class_proc_tick(current);	
	```

### 练习1
---
1.	<b>理解并分析sched_class中各个函数指针的用法，并结合RR算法描述ucore的调度执行过程</b>
	
	> * 共有5个函数指针，用法如下
	```
	init：初始化正在运行的队列
	enqueue：将正在运行队列加入一个进程
	dequeue：将正在运行队列中的一个进程取出
	pick_next：选择队列中的stride参数最小的一个进程
	proc_tick：每个时钟中断时调用，表示一个进程时间片完成，操作进程剩余时间片
	```

	> * ucore的调度执行过程如下
	```
	1 维护一个就绪队列rq，需要调度时执行schedule函数
	2 将当前进程插入队列
	3 选择stride最小的作为next
	4 将next从队列取出
	5 变换当前执行的进程至next
	```
2.	<b>说明如何设计实现“多级反馈队列调度算法”</b>
	
	> * 维护多个队列，每个队列的优先级不同，MAX_TIME_SLICE不同。
	> * 对不同队列中的进程按次序进行调用
	> * 对不同队列中的prioity设置为不同的值，运行时长不同

### 练习2
---
1.	<b>完成stride算法的实现</b>
	
	> * 定义一个最大步长数，方便之后的步长与优先级之间的转化，此处取32位有符号整数中的最大值
	```
	#define BIG_STRIDE   0x7FFFFFFF
	```

	> * stride_init() 初始化函数
	```
	list_init(&(rq->run_list));
    	rq->lab6_run_pool = NULL;
    	rq->proc_num = 0;
	```

	> * stride_enqueue() 入队列函数（使用基于斜堆的优先队列）
	```
	rq->lab6_run_pool = skew_heap_insert(rq->lab6_run_pool, &(proc->lab6_run_pool), proc_stride_comp_f);
	if (proc->time_slice == 0 || proc->time_slice > rq->max_time_slice) {
          proc->time_slice = rq->max_time_slice;
    	}
    	proc->rq = rq;
    	rq->proc_num ++;
	```

	> * stride_dequeue() 出队列函数（使用基于斜堆的优先队列）
	```
	rq->lab6_run_pool =skew_heap_remove(rq->lab6_run_pool, &(proc->lab6_run_pool), proc_stride_comp_f);
    	rq->proc_num --;
	```

	> * stride_pick_next() 选择下一个运行的进程（使用基于斜堆的优先队列）
	```
	if (rq->lab6_run_pool == NULL) return NULL;
    	struct proc_struct *p = le2proc(rq->lab6_run_pool, lab6_run_pool);
    	if (p->lab6_priority == 0)
        	p->lab6_stride += BIG_STRIDE;
    	else p->lab6_stride += BIG_STRIDE / p->lab6_priority;
    	return p;
	```

	> * stride_proc_tick() 产生时钟中断
	```
	if (proc->time_slice > 0) {
        	proc->time_slice --;
    	}
    	if (proc->time_slice == 0) {
        	proc->need_resched = 1;
    	}
	```




