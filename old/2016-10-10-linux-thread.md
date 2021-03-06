---
layout: post
title: Linux Thread
comments: true
date: 2016-10-26 10:00:32+00:00
categories:
- Linux
- Tech
tags:
- linux
- thread
main-class: 'Linux'
color: '#006798'

introduction: 'The Note of APUE Thread'
---------------------------------------


## Thread Identification

```c
#include <pthread.h>

int pthread_equal(pthread_t tid1, pthread_t tid2);
```

- Returns: nonzero if equal, 0 otherwise

```c
#include <pthread.h>

pthread_t pthread_self(void);
```

- Returns: the thread ID of the calling thread

## Thread Creation
```c
#include <pthread.h>

int pthread_create(pthread_t *restrict tidp,
                   const pthread_attr_t *restrict attr,
                   void *(*start_rtn)(void),
                   void *restrict arg);
```

- Returns: 0 if OK, error number on failure


```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>

pthread_t ntid;


void printids(const char *s) {
    pid_t pid;
    pthread_t tid;

    pid = getpid();
    tid = pthread_self();
    printf("%s pid %u tid %u (0x%x)\n", s, (unsigned int) pid,
           (unsigned int) tid, (unsigned int) tid);
}

void *start_rtn(void *arg) {
    printids("new thread");
    return (void *) 0;
}

int main() {
    int err = pthread_create(&ntid, NULL, start_rtn, NULL);
    if (err != 0) {
        printf("can not create thread : %s\n", strerror(err));
    }
    printids("main thread");
    sleep(1);
    return 0;
}
```

- result
```
main thread pid 26921 tid 22664960 (0x159d700)
new thread pid 26921 tid 14526208 (0xdda700)
```


### Thread Termination

- Three ways to terminate thread
> The thread can simply return from the start routine. The return value is the thread's exit code.
> The thread can be canceled by another thread in the same process.
> The thread can call `pthread_exit`.

```c
#include <pthread.h>

void pthread_exit(void *rval_ptr);
```


```c
#include <pthread.h>

int pthread_join(pthread_t thread, void **rval_ptr);
```

- Returns: 0 if OK, error number on failure



```c
#include <pthread.h>
#include <stdio.h>
void* thr_fn1(void* arg){
    printf("I am thread 1 return 1\n");
    pthread_exit((void*)1);
}


void* thr_fn2(void* arg){
    printf("I am thread 2 return 2\n");
    pthread_exit((void*)2);
}

//ignore error handling
int main() {
    pthread_t p1,p2;
    void* pret;
    pthread_create(&p1,NULL,thr_fn1,NULL);
    pthread_join(p1,&pret);
    printf("return from p1 %d\n",(int)pret);

    pthread_create(&p2,NULL,thr_fn2,NULL);
    pthread_join(p2,&pret);
    printf("return from p2 %d\n",(int)pret);

    return 0;
}
```

```
I am thread 1 return 1
return from p1 1
I am thread 2 return 2
return from p2 2
```

- stack

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

struct foo {
    int a, b, c, d;
};

void print_foo(const char *msg, const struct foo *fp) {
    printf(msg);
    printf("  structure at 0x%x\n", (unsigned) fp);
    printf("  foo.a = %d\n", fp->a);
    printf("  foo.b = %d\n", fp->b);
    printf("  foo.c = %d\n", fp->c);
    printf("  foo.d = %d\n", fp->d);
}

void *thr_fn1(void *arg) {
    struct foo foo = {1, 2, 3, 4};
    print_foo("thread 1:\n", &foo);
    pthread_exit((void *) &foo);
}

void *thr_fn2(void *arg) {
    printf("thread 2: ID is %d\n", pthread_self());
    pthread_exit((void *) 0);
}

int main() {
    struct foo *fp;
    pthread_t p1, p2;
    pthread_create(&p1, NULL, thr_fn1, NULL);
    pthread_join(p1, (void *) &fp);
    sleep(1);
    pthread_create(&p2, NULL, thr_fn2, NULL);
    pthread_join(p2, NULL);
    sleep(1);
    print_foo("parent \n", fp);
    return 0;
}
```

```
thread 1:
  structure at 0x7f351f40
  foo.a = 1
  foo.b = 2
  foo.c = 3
  foo.d = 4
thread 2: ID is 2134189824
parent
  structure at 0x7f351f40
  foo.a = 0
  foo.b = 0
  foo.c = 1
  foo.d = 0
```


- heap

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

struct foo {
    int a, b, c, d;
};

void print_foo(const char *msg, const struct foo *fp) {
    printf(msg);
    printf("  structure at 0x%x\n", (unsigned) fp);
    printf("  foo.a = %d\n", fp->a);
    printf("  foo.b = %d\n", fp->b);
    printf("  foo.c = %d\n", fp->c);
    printf("  foo.d = %d\n", fp->d);
}

void *thr_fn1(void *arg) {
    struct foo *foo = malloc(sizeof(struct foo));
    foo->a = 1;
    foo->b = 2;
    foo->c = 3;
    foo->d = 4;
    print_foo("thread 1:\n", foo);
    pthread_exit((void *) foo);
}

void *thr_fn2(void *arg) {
    printf("thread 2: ID is %d\n", pthread_self());
    pthread_exit((void *) 0);
}

int main() {
    struct foo *fp;
    pthread_t p1, p2;
    pthread_create(&p1, NULL, thr_fn1, NULL);
    pthread_join(p1, (void *) &fp);
    sleep(1);
    pthread_create(&p2, NULL, thr_fn2, NULL);
    pthread_join(p2, NULL);
    sleep(1);
    print_foo("parent \n", fp);
    return 0;
}
```


```
thread 1:
  structure at 0x80008c0
  foo.a = 1
  foo.b = 2
  foo.c = 3
  foo.d = 4
thread 2: ID is 250177280
parent
  structure at 0x80008c0
  foo.a = 1
  foo.b = 2
  foo.c = 3
  foo.d = 4
```

### Thread Cancel

```c
#include <pthread.h>

int pthread_cancel(pthread_t tid);
```

- Returns: 0 if OK, error number on failure



> Note that `pthread_cancel()` doesn't wait for the thread to terminate. It merely makes the request.

### Thread Exit Handler
```c
#include <pthread.h>

void pthread_cleanup_push(void (*rtn)(void *),void *arg);

void pthread_cleanup_pop(int execute);
```

> Makes a call to pthread_exit
> Responds to a cancellation request
> Makes a call to pthread_cleanup_pop with a nonzero execute argument



```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void cleanup(void *arg) {
    printf("cleanup: %s\n", (char *) arg);
}

void *thr_fn1(void *arg) {
    printf("thread 1 start\n");
    pthread_cleanup_push(cleanup, "thread 1 first handler") ;
    pthread_cleanup_push(cleanup, "thread 1 second handler") ;
    printf("thread 1 push complete\n");
    if (arg)
        return ((void *) 1);
    pthread_cleanup_pop(0);
    pthread_cleanup_pop(0);
    return ((void *) 1);
}

void *thr_fn2(void *arg) {
    printf("thread 2 start\n");
    pthread_cleanup_push(cleanup, "thread 2 first handler") ;
    pthread_cleanup_push(cleanup, "thread 2 second handler") ;
    printf("thread 2 push complete\n");
    if (arg)
        pthread_exit((void *) 2);
    pthread_cleanup_pop(0);
    pthread_cleanup_pop(0);
    pthread_exit((void *) 2);
}

int main() {
    pthread_t p1, p2;
    void *ret;
    pthread_create(&p1, NULL, thr_fn1, (void *) 1);
    pthread_create(&p2, NULL, thr_fn2, (void *) 2);
    sleep(1);
    pthread_join(p1, &ret);
    printf("return from thread1 is : %d\n", (int) (ret));

    pthread_join(p2, &ret);
    printf("return from thread2 is : %d\n", (int) (ret));
    exit(0);
}
```

```
thread 1 start
thread 1 push complete
thread 2 start
thread 2 push complete
cleanup: thread 2 second handler
cleanup: thread 2 first handler
return from thread1 is : 1
return from thread2 is : 2
```


```c
#include <pthread.h>

int pthread_detach(pthread_t tid);
```

## Thread Synchronization

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int a = 0;

void* thr_run(void* arg){
    sleep(1);
    ++a;
    return (void*)0;
}

int main() {
    int count = 10000;
    pthread_t ts[count];
    for (int i = 0; i < count; ++i) {
        pthread_create(&ts[i],NULL,thr_run,NULL);
    }

    for (int i = 0; i < count; ++i) {
        pthread_join(ts[i],NULL);
    }
    printf("a=%d\n",a);
    return 0;
}
```

- result

```
a=9989
```

### Thread Mutex

```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *restrict mutex,
                       const pthread_mutexattr_t *restrict attr);

int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);

int pthread_mutex_trylock(pthread_mutex_t *mutex);

int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

- All return: 0 if OK, error number on failure

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
pthread_mutex_t mutex;
int a = 0;

void* thr_run(void* arg){
    pthread_mutex_lock(&mutex);
    ++a;
    pthread_mutex_unlock(&mutex);
    return (void*)0;
}

int main() {
    pthread_mutex_init(&mutex,NULL);
    int count = 10000;
    pthread_t ts[count];
    for (int i = 0; i < count; ++i) {
        pthread_create(&ts[i],NULL,thr_run,NULL);
    }

    for (int i = 0; i < count; ++i) {
        pthread_join(ts[i],NULL);
    }
    printf("a=%d\n",a);
    return 0;
}
```

```
a=10000
```

### ReaderWriter Locks

```c
#include <pthread.h>

int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
                        const pthread_rwlockattr_t *restrict attr);

int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
```

- Both return: 0 if OK, error number on failure

```c

#include <pthread.h>

int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

- Both return: 0 if OK, error number on failure

### Condition Variables

```c
#include <pthread.h>

int pthread_cond_init(pthread_cond_t *restrict cond,
                      pthread_condattr_t *restrict attr);

int pthread_cond_destroy(pthread_cond_t *cond);
```

```c
#include <pthread.h>

int pthread_cond_wait(pthread_cond_t *restrict cond,
                      pthread_mutex_t *restrict mutex);

int pthread_cond_timedwait(pthread_cond_t *restrict cond,
                           pthread_mutex_t *restrict mutex,
                           const struct timespec *restrict timeout);
```

## Thread Control


### Thread Limits

```c
/*************************************************************************
	> File Name: thread_limits.c
	> Author: Lion
	> Mail: lion.good.very.first@gmail.com
	> Created Time: Sat 05 Nov 2016 02:29:02 PM CST
 ************************************************************************/

#include<stdio.h>
#include <unistd.h>
#include <limits.h>

int main() {
#ifdef _SC_THREAD_DESTRUCTOR_ITERATIONS
    printf("PTHREAD_DESTRUCTOR_ITERATIONS=%d\n", sysconf(_SC_THREAD_DESTRUCTOR_ITERATIONS));
#endif

#ifdef _SC_THREAD_KEYS_MAX
    printf("PTHREAD_KEYS_MAX=0x%X\n", sysconf(_SC_THREAD_KEYS_MAX));
#endif

#ifdef _SC_THREAD_STACK_MIN
    printf("PTHREAD_STACK_MIN=0x%X\n", sysconf(_SC_THREAD_STACK_MIN));
#endif

#ifdef _SC_THREAD_THREADS_MAX
    printf("PTHREAD_THREADS_MAX=0x%X\n", sysconf(_SC_THREAD_THREADS_MAX));
#endif
    return 0;
}
```


- Results

```
PTHREAD_DESTRUCTOR_ITERATIONS=4
PTHREAD_KEYS_MAX=0x400
PTHREAD_STACK_MIN=0x4000
PTHREAD_THREADS_MAX=0xFFFFFFFF
```


###  Thread Attributes

```c
#include <pthread.h>

int pthread_attr_init(pthread_attr_t *attr);

int pthread_attr_destroy(pthread_attr_t   *attr);
```
- Table of Thread Attributes
| Name | Description    |
| :------------- | :------------- |
| detachstate      | detached thread attribute       |
| guardsize      | guard buffer size in bytes at end of thread stack       |
| stackaddr      | lowest address of thread stack       |
| stacksize      | size in bytes of thread stack       |

#### detachstate attribute

```c
#include <pthread.h>
int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr,
                                int *detachstate);
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
```

- Return : 0 if OK, error number on failure

```c
#include<stdio.h>
#include <pthread.h>

void *thr_fn(void *args) {

}

int main() {
    int err;
    pthread_t p;
    pthread_attr_t attr;

    err = pthread_attr_init(&attr);
    if (err != 0)
        return err;
    err = pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    if (0 == err)
        pthread_create(&p, &attr, thr_fn, NULL);
    pthread_attr_destroy(&attr);
    return 0;
}
```

#### thread stack attribute

```c
#include <pthread.h>

int pthread_attr_getstack(const pthread_attr_t *restrict attr,
                          void **restrict stackaddr,
                          size_t *restrict stacksize);

int pthread_attr_setstack(const p

```c
#include <pthread.h>

int pthread_attr_getstack(const pthread_attr_t *restrict attr,
                          void **restrict stackaddr,
                          size_t *restrict stacksize);

int pthread_attr_setstack(const pthread_attr_t *attr,
                          void *stackaddr, size_t *stacksize);
```
```c
#include<stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>

long fibonacci(int fibo) {
    if (fibo == 0 || fibo == 1)
        return 1;
    else
        return fibonacci(fibo - 1) + fibonacci(fibo - 2);
}

void *thr_fn(void *args) {
    int a = 0;
    int b = 1;
    long arr[1<<16] ;//alloc more than 64k on stack
    printf("ret = %d\n", fibonacci(20));
    return (void *) 0;
}

int main() {
    pthread_attr_t attr;
    int err;
    void *addr;
    size_t size;
    pthread_t p;
    if (err = pthread_attr_init(&attr)) {
        return err;
    }

    pthread_attr_getstacksize(&attr, &size);
    printf("size=0x%x\n", size);


    pthread_attr_setstacksize(&attr, 1024*64);//64k

    pthread_attr_getstacksize(&attr, &size);
    printf("size=0x%x\n", size);

    if (err = pthread_create(&p, &attr, thr_fn, NULL)) {
        return err;
    }

    sleep(3);

    pthread_attr_getstacksize(&attr, &size);
    printf("size=0x%x\n"l 11
```

```c, size);

    pthread_attr_getstack(&attr,&addr,&size);
    printf("size=0x%x\n", size);

    pthread_join(p, NULL);
    return 0;
}
```

- Results

```
size=0x800000
size=0x10000
Command terminated by signal 11
```

```c
#define	SIGSEGV		11	/* Segmentation violation (ANSI).  */
```

```c
#include <pthread.h>

int pthread_attr_getguardsize(const pthread_attr_t *restrict attr,
                                size_t *restrict guardsize);

int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
```

> The guardsize thread attribute controls the size of the memory extent after the end of the thread's stack to protect against stack overflow.

#### Mutex Attributes

```c
#include <pthread.h>
int pthread_mutexattr_init(pthread_mutexattr_t *attr);

int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
```
- Both return: 0 if OK, error number on failure

```c
#include <pthread.h>

int pthread_mutexattr_getpshared(const pthread_mutexattr_t* restrict attr,
                                 int *restrict pshared);

int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr,
                                 int pshared);
```

- Both return: 0 if OK, error number on failure

- `process-shared` attribute check `_POSIX_THREAD_PROCESS_SHARED`
- `PTHREAD_PROCESS_PRIVATE` is the default behavior.Within a process, multiple threads can access the same synchronization object
- `PTHREAD_PROCESS_SHARED` shared between multiple processes may be used for synchronization by those processes

- The `type` attribute

| Mutex type              | Relock without unlock?|Unlock when not owned? |Unlock when unlocked?  |
| :-------------          | :-------------        |:-------------         |:-------------         |
| PTHREAD_MUTEX_NORMAL    | deadlock              |undefined              |undefined              |
| PTHREAD_MUTEX_ERRORCHECK|returns error          |returns error          |returns error          |
| PTHREAD_MUTEX_RECURSIVE |allowed                |returns error          |returns error          |
| PTHREAD_MUTEX_DEFAULT   |undefined              |undefined              |undefined              |


```c
#include <pthread.h>

int pthread_mutexattr_gettype(const pthread_mutexattr_t *
                              restrict attr,
                               int *restrict type);

int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
```

- Both return: 0 if OK, error number on failure
