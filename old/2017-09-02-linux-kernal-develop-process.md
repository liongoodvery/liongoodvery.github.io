---
layout: post
title: linux_kernal_develop--process
comments: true
date: 2017-09-02 10:00:32+00:00
categories:
- Linux
tags:
- C
- Linux
- Kernal
main-class: 'C'
color: '#a0f798'
introduction: 'linux_kernal_develop--process'

---

# Linux Kernel Develop
## 进程
### 进程描述符与任务结构
- task_struct
- pid_t
- current
- thread_info


### Task State

- TASK_RUNNING
- TASK_INTERRUPTIBLE
- TASK_UNINTERRUPTIBLE
- __TASK_TRACED
- __TASK_STOPPED

### Manipulating the Current Process State
```c
 #include <linux/sched.h>
 set_task_state(task, state);
 set_current_state(state)
```

### Process Context
- User Space
- Kernel Space

### The Process Family Tree
- current
- init
- parent
- children
- process tree
- [list_entry](https://www.kernel.org/doc/htmldocs/kernel-api/API-list-entry.html)
- [list_for_each](https://www.kernel.org/doc/htmldocs/kernel-api/API-list-for-each.html)
- for_each_process



```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/sched.h>
#include <asm/current.h>


MODULE_LICENSE("Dual BSD/GPL");

static void printTaskInfo(struct task_struct* task);

static int hello_init(void)
{	
	struct task_struct *my_parent;
	struct task_struct *task;
	struct list_head *list;

	//current_info
	printTaskInfo(current);

	//parent
	my_parent= current->parent;
	printTaskInfo(my_parent);

	//init task
	printTaskInfo(&init_task);

	//children
	list_for_each(list,&init_task.children){
		task=list_entry(list,struct task_struct,sibling);
		printk(KERN_INFO"========children========\n");
		printTaskInfo(task);
	}
	//all_process
	for_each_process(task){
		printTaskInfo(task);
	}
	return 0;
}

static void printTaskInfo(struct task_struct* task)
{
	printk(KERN_INFO "comm=%s,pid=%d\n",task->comm,task->pid);
}

static void hello_exit(void)
{
	printk(KERN_ALERT "Goodbye, cruel world\n");
	printk(KERN_INFO "comm=%s,pid=%d\n",current->comm,current->pid);
	
}

module_init(hello_init);
module_exit(hello_exit);

```
### Process Creation

- Copy On Write
- Forking
    - fork()
    - vfork()
- Thread
    - Linux Implement
    - Kernel Thrad
    > [kthread_create](https://www.fsl.cs.sunysb.edu/kernel-api/re69.html)
    > [kthread_run](https://www.fsl.cs.sunysb.edu/kernel-api/re67.html)
    >[kthread_stop](https://www.fsl.cs.sunysb.edu/kernel-api/re71.html)


```c
    #define kthread_run(threadfn, data, namefmt, ...) \
({ \
struct task_struct *k; \
\
k = kthread_create(threadfn, data, namefmt, ## __VA_ARGS__); \
if (!IS_ERR(k)) \
wake_up_process(k); \
k; \
})
```

    
    
***

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kthread.h>
#undef SLEEP_MILLI_SEC
#define SLEEP_MILLI_SEC(nMilliSec)\
do { \
	long timeout = (nMilliSec) * HZ / 1000; \
	while(timeout > 0) \
		{ \
			timeout = schedule_timeout(timeout); \
		} \
	}while(0); 

	MODULE_LICENSE("Dual BSD/GPL");

	static void printTaskInfo(struct task_struct* task);
	static struct task_struct *kthread;
	static int fn(void* data);

	static int hello_init(void)
	{	
		kthread = kthread_run(fn,"Hello world","HelloWorld");
	}

	static void printTaskInfo(struct task_struct* task)
	{
		printk(KERN_INFO "comm=%s,pid=%d\n",task->comm,task->pid);

	}

	static void hello_exit(void)
	{
		if(kthread){
			kthread_stop(kthread);
		}
		printk(KERN_INFO "Goodbye\n");

	}
	static int fn(void* data)
	{
		char *mydata = kmalloc(strlen(data)+1,GFP_KERNEL);  
		memset(mydata,'\0',strlen(data)+1);  
		strncpy(mydata,data,strlen(data));  
		while(!kthread_should_stop())  
		{  
			SLEEP_MILLI_SEC(1000);  
			printk("%s\n",mydata);  
		}  
		kfree(mydata);  
		return 0;
	}
	module_init(hello_init);
	module_exit(hello_exit);
```
***
#### Process Termination


- Remove Process Descriptor
***
### Process Scheduling
- O(1) scheduler
- CFS
- Policy
    - I/O bound vs Processor bound
    - Process Priority
***
