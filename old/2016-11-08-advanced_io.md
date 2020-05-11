---
layout: post
title: Advanced Io
comments: true
date: 2016-11-08 16:00:32+00:00
categories:
- Linux
- Tech
tags:
- linux
- process
main-class: 'Linux'
color: '#006798'
introduction: 'Advanced Io Note'
---

## Nonblocking I/O

```c
#include<stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <apue.h>
int main() {
    char buf[500000];
    int ntowrite;
    ntowrite = read(STDIN_FILENO, buf, sizeof(buf));
    fprintf(stderr, "read %d bytes\n", ntowrite);
    set_fl(STDOUT_FILENO, O_NONBLOCK);
    char *ptr;
    int nwrite;
    while (ntowrite > 0) {
        errno = 0;
        nwrite = write(STDOUT_FILENO, buf, ntowrite);
        fprintf(stderr, "nwrite = %d, errno = %d\n", nwrite, errno);
        if (nwrite > 0) {
            ptr += nwrite;
            ntowrite -= nwrite;
        }
    }
    clr_fl(STDOUT_FILENO,O_NONBLOCK);
    return 0;
}
```


## Record Locking

```c
#include <fcntl.h>

int fcntl(int filedes, int cmd, ... /* struct flock *flockptr */ );
```

```c
struct flock {
     short l_type;   /* F_RDLCK, F_WRLCK, or F_UNLCK */
     off_t l_start;  /* offset in bytes, relative to l_whence */
     short l_whence; /* SEEK_SET, SEEK_CUR, or SEEK_END */
     off_t l_len;    /* length, in bytes; 0 means lock to EOF */
     pid_t l_pid;    /* returned with F_GETLK */
   };
```

### Requesting and Releasing a Lock

```c
#define read_lock(fd, offset, whence, len) \
            lock_reg((fd), F_SETLK, F_RDLCK, (offset), (whence), (len))
#define readw_lock(fd, offset, whence, len) \
            lock_reg((fd), F_SETLKW, F_RDLCK, (offset), (whence), (len))
#define write_lock(fd, offset, whence, len) \
            lock_reg((fd), F_SETLK, F_WRLCK, (offset), (whence), (len))
#define writew_lock(fd, offset, whence, len) \
            lock_reg((fd), F_SETLKW, F_WRLCK, (offset), (whence), (len))
#define un_lock(fd, offset, whence, len) \
            lock_reg((fd), F_SETLK, F_UNLCK, (offset), (whence), (len))
```

```c
#include "apue.h"
#include <fcntl.h>

int
lock_reg(int fd, int cmd, int type, off_t offset, int whence, off_t len)
{
    struct flock lock;

    lock.l_type = type;     /* F_RDLCK, F_WRLCK, F_UNLCK */
    lock.l_start = offset;  /* byte offset, relative to l_whence */
    lock.l_whence = whence; /* SEEK_SET, SEEK_CUR, SEEK_END */
    lock.l_len = len;       /* #bytes (0 means to EOF) */

    return(fcntl(fd, cmd, &lock));
}
```


### Testing for a Lock

```c
#define is_read_lockable(fd, offset, whence, len) \
          (lock_test((fd), F_RDLCK, (offset), (whence), (len)) == 0)
#define is_write_lockable(fd, offset, whence, len) \
          (lock_test((fd), F_WRLCK, (offset), (whence), (len)) == 0)
```

```c
#include "apue.h"
#include <fcntl.h>

pid_t
lock_test(int fd, int type, off_t offset, int whence, off_t len)
{
    struct flock lock;
    lock.l_type = type;     /* F_RDLCK or F_WRLCK */
    lock.l_start = offset;  /* byte offset, relative to l_whence */
    lock.l_whence = whence; /* SEEK_SET, SEEK_CUR, SEEK_END */
    lock.l_len = len;       /* #bytes (0 means to EOF) */

    if (fcntl(fd, F_GETLK, &lock) < 0)
        err_sys("fcntl error");

    if (lock.l_type == F_UNLCK)
        return(0);      /* false, region isn't locked by another proc */
    return(lock.l_pid); /* true, return pid of lock owner */
}
```

### DeadLock

```c
#include<stdio.h>
#include <apue.h>
#include <fcntl.h>

static void
lockabyte(const char *name, int fd, off_t offset) {
    if (writew_lock(fd, offset, SEEK_SET, 1) < 0)
        err_sys("%s: writew_lock error", name);
    printf("%s: got the lock, byte %ld\n", name, offset);
}

int main() {
    int fd;

    if ((fd = creat("file.tmp", FILE_MODE)) < 0) {
        err_sys("create file error");
    }
    if (write(fd, "ab", 2) != 2) {
        err_sys("write error");
    }

    TELL_WAIT();

    pid_t pid;
    if ((pid = fork()) < 0) {
        err_sys("fork error");
    } else if (pid == 0) {
        lockabyte("child", fd, 0);
        TELL_PARENT(getppid());
        WAIT_PARENT();
        lockabyte("child", fd, 1);
    } else {
        lockabyte("parent", fd, 1);
        TELL_CHILD(pid);
        WAIT_CHILD();
        lockabyte("parent", fd, 0);
    }

    return 0;
}
```

```c
#include "apue.h"

static volatile sig_atomic_t sigflag; /* set nonzero by sig handler */
static sigset_t newmask, oldmask, zeromask;

static void sig_usr(int signo)    /* one signal handler for SIGUSR1 and SIGUSR2 */
{
    sigflag = 1;
}

void TELL_WAIT(void) {
    if (signal(SIGUSR1, sig_usr) == SIG_ERR)
        err_sys("signal(SIGUSR1) error");
    if (signal(SIGUSR2, sig_usr) == SIG_ERR)
        err_sys("signal(SIGUSR2) error");
    sigemptyset(&zeromask);
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGUSR1);
    sigaddset(&newmask, SIGUSR2);

    /* Block SIGUSR1 and SIGUSR2, and save current signal mask */
    if (sigprocmask(SIG_BLOCK, &newmask, &oldmask) < 0)
        err_sys("SIG_BLOCK error");
}

void TELL_PARENT(pid_t pid) {
    kill(pid, SIGUSR2);        /* tell parent we're done */
}

void WAIT_PARENT(void) {
    while (sigflag == 0)
        sigsuspend(&zeromask);    /* and wait for parent */
    sigflag = 0;

    /* Reset signal mask to original value */
    if (sigprocmask(SIG_SETMASK, &oldmask, NULL) < 0)
        err_sys("SIG_SETMASK error");
}

void TELL_CHILD(pid_t pid) {
    kill(pid, SIGUSR1);            /* tell child we're done */
}

void WAIT_CHILD(void) {
    while (sigflag == 0)
        sigsuspend(&zeromask);    /* and wait for child */
    sigflag = 0;

    /* Reset signal mask to original value */
    if (sigprocmask(SIG_SETMASK, &oldmask, NULL) < 0)
        err_sys("SIG_SETMASK error");
}
```


- Result

```
parent: got the lock, byte 1
child: got the lock, byte 0
child: writew_lock error: Resource deadlock avoided
parent: got the lock, byte 0
```

### Implied Inheritance and Release of Locks
> Locks are associated with a process and a file. This has two implications. The first is obvious: when a process terminates, all its locks are released. The second is far from obvious: whenever a descriptor is closed, any locks on the file referenced by that descriptor for that process are released.

> Locks are never inherited by the child across a fork.
> Locks are inherited by a new program across an exec. Note, however, that if the close-on-exec flag is set for a file descriptor, all locks for the underlying file are released when the descriptor is closed as part of an exec.

### Advisory versus Mandatory Locking

```c
#include<stdio.h>
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv) {
    if (argc > 1) {
        int fd = open(argv[1], O_WRONLY);
        if (fd == -1) {
            printf("Unable to open the file\n");
            exit(1);
        }
        static struct flock lock;

        lock.l_type = F_WRLCK;
        lock.l_start = 0;
        lock.l_whence = SEEK_SET;
        lock.l_len = 0;
        lock.l_pid = getpid();

        int ret = fcntl(fd, F_SETLKW, &lock);
        printf("Return value of fcntl:%d\n", ret);
        if (ret == 0) {
            while (1) {
                scanf("%c", NULL);
            }
        }
    }
    return 0;
}
```


```
# mount -oremount,mand /
# touch advisory.txt
# touch mandatory.txt
# chmod g+s,g-x mandatory.txt
# ./file_lock advisory.txt
# ls >>advisory.txt
# ./file_lock mandatory.txt
# ls >>mandatory.txt
```
