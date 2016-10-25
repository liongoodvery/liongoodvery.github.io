---
layout: post
title: Linux Signal
comments: true
date: 2016-10-17 10:00:32+00:00
categories:
- Linux
- Tech
tags:
- linux
- signal
main-class: 'Linux'
color: '#006798'

introduction: 'Linux Signal'

---

### kill and raise Functions

- `kill()`  sends a signal to a process or a group of processes.
- `raise()` sends a signal to itself.

```c
#include <signal.h>
int kill(pid_t pid, int signo);
int raise(int signo);
```

| pid     |  meanging     |
| :------------- | :------------- |
| pid>0      | send to the process whose process id is pid       |
| pid==0      | send to processes whose group id is the invoking process's group id      |
| pid==-1      | send to processes for which the invoing process has the permission to send       |
| pid<-1    | send to processes whoes group id is absolute value of pid   |

- What if the signo==0?

### alarm and pause Functions

- `alarm()` allows us to set a timer that will expire at a specified time in the future. When the timer expires, the SIGALRM signal is generated. If we ignore or don't catch this signal, its default action is to terminate the process.
- `pause()` function suspends the calling process until a signal is caught.

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
int pause(void);
```
- What is the return value?

```c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>
int main(){
    alarm(2);
    pause();
    fprintf(stderr,"exits \n");
    return 0;
}
```
- result

```
Command terminated by signal 14
```

```c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>
void alarm_handle(int signo){
    if(signo==SIGALRM){
        fprintf(stderr,"caught alarm\n");
    }

}
int main(){
    if(signal(SIGALRM,alarm_handle)==SIG_ERR){
        fprintf(stderr,"register error\n");
    }
    alarm(2);
    int ret = alarm(3);
    fprintf(stderr,"ret=%d \n",ret);
    pause();
    fprintf(stderr,"exits \n");
    return 0;
}
```

-result

```
ret=2
caught alarm
exits
```

A simple sleep
```c
#include <signal.h>
#include <unistd.h>

void sig_alrm(int signo) {
    /* nothing to do, just return to wake up the pause */
}

unsigned int sleep1(unsigned int nsecs) {
    if (signal(SIGALRM, sig_alrm) == SIG_ERR) {
        return nsecs;
    }
    alarm(nsecs);
    pause();
    return (alarm(0));
}

int main(int argc, char const *argv[]) {
    sleep1(3);
    return 0;
}
```

`setjmp()` demo

```c
#include <stdio.h>
#include <setjmp.h>

jmp_buf jump_buffer;

void func(void) {
    printf("Before calling longjmp\n");
    longjmp(jump_buffer, 1);
    printf("After calling longjmp\n");
}

void func1(void) {
    printf("Before calling func\n");
    func();
    printf("After calling func\n");
}

int main() {
    if (setjmp(jump_buffer) == 0) {
        printf("first calling set_jmp\n");
        func1();
    } else {
        printf("second calling set_jmp\n");
    }
    return 0;
}
```

result

```
first calling set_jmp
Before calling func
Before calling longjmp
second calling set_jmp
```

### Signal Sets

```c
#include <signal.h>
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signo);
```

```c
int sigismember(const sigset_t *set, int signo);
```

### sigprocmask Function

```
#include <signal.h>

int sigprocmask(int how, const sigset_t *restrict set,           sigset_t *restrict oset);
```

- Return : 0 if OK , -1 on error


```c
#include <signal.h>
#include <unistd.h>

int main() {
    sigset_t *set;
    sigfillset(set);
    sigprocmask(SIG_BLOCK, set, NULL);
    pause();
    return 0;
}
```


### sigpending Function
```c
#include <signal.h>

int sigpending(sigset_t *set);
```

- Return : 0 if OK , -1 on error


```c
//
// Created by lion on 10/24/16.
//
#include <signal.h>
#include <unistd.h>
#include <stdio.h>
void sig_quit(int signo){
    printf("\nCaught SIGQUIT\n");
    if (signal(SIGQUIT,SIG_DFL)==SIG_ERR){
        printf("canz not reset SIGQUIT");
    }
}
int main(){

    sigset_t old_set , new_set , pending_set;
    if (signal(SIGQUIT,sig_quit)==SIG_ERR){
        printf("can not catch SIGQUIT\n");
    }

    sigemptyset(&new_set);
    sigaddset(&new_set,SIGQUIT);
    if (sigprocmask(SIG_BLOCK,&new_set,&old_set)<0){
        printf("Block SIGQUIT Error\n");
    }

    printf("first sleep\n");
    sleep(5);

    if (sigpending(&pending_set)<0){
        printf("SIG pending Error\n");
    }

    if (sigprocmask(SIG_SETMASK,&old_set,NULL)<0){
        printf("reset error\n");
    }
    printf("second sleep\n");
    sleep(5);

    return 0;
}
```

result
```
first sleep
^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\^\
Caught SIGQUIT
second sleep
^\[1]    6117 quit   
```


### sigaction Function

NO FINISH

WAITING