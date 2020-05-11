---
layout: post
title: Daemon Processes
comments: true
date: 2016-11-07 16:00:32+00:00
categories:
- Linux
- Tech
tags:
- linux
- process
main-class: 'Linux'
color: '#006798'
introduction: 'Daemon Processes Note'
---

### Coding Rules

> The first thing to do is call umask to set the file mode creation mask to 0.
> Call fork and have the parent exit.
> Call `setsid` to create a new session.
> Change the current working directory to the root directory.
> Unneeded file descriptors should be closed.
> Some daemons open file descriptors 0, 1, and 2 to /dev/null so that any library routines that try to read from standard input or write to standard output or standard error will have no effect.

```c
#include "apue.h"
#include <syslog.h>
#include <fcntl.h>
#include <sys/resource.h>

void daemonize(const char *cmd)
{
    int                 i, fd0, fd1, fd2;
    pid_t               pid;
    struct rlimit       rl;
    struct sigaction    sa;
    /*
     * Clear file creation mask.
     */
    umask(0);

    /*
     * Get maximum number of file descriptors.
     */
    if (getrlimit(RLIMIT_NOFILE, &rl) < 0)
        err_quit("%s: can't get file limit", cmd);

    /*
     * Become a session leader to lose controlling TTY.
     */
    if ((pid = fork()) < 0)
        err_quit("%s: can't fork", cmd);
    else if (pid != 0) /* parent */
        exit(0);
    setsid();

    /*
     * Ensure future opens won't allocate controlling TTYs.
     */
    sa.sa_handler = SIG_IGN;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    if (sigaction(SIGHUP, &sa, NULL) < 0)
        err_quit("%s: can't ignore SIGHUP");
    if ((pid = fork()) < 0)
        err_quit("%s: can't fork", cmd);
    else if (pid != 0) /* parent */
        exit(0);

    /*
     * Change the current working directory to the root so
     * we won't prevent file systems from being unmounted.
     */
    if (chdir("/") < 0)
        err_quit("%s: can't change directory to /");

    /*
     * Close all open file descriptors.
     */
    if (rl.rlim_max == RLIM_INFINITY)
        rl.rlim_max = 1024;
    for (i = 0; i < rl.rlim_max; i++)
        close(i);

    /*
     * Attach file descriptors 0, 1, and 2 to /dev/null.
     */
    fd0 = open("/dev/null", O_RDWR);
    fd1 = dup(0);
    fd2 = dup(0);

    /*
     * Initialize the log file.
     */
    openlog(cmd, LOG_CONS, LOG_DAEMON);
    if (fd0 != 0 || fd1 != 1 || fd2 != 2) {
        syslog(LOG_ERR, "unexpected file descriptors %d %d %d",
               fd0, fd1, fd2);
        exit(1);
    }
}
```

===

![The BSD syslog facility](../assets/img/BSDSyslogFacility.jpg)


```c
#include <syslog.h>

void openlog(const char *ident, int option, int facility);

void syslog(int priority, const char *format, ...);

void closelog(void);

int setlogmask(int maskpri);
```
- `ident` is normally the name of the program
- `option` is a bitmask specifying various options . Probably one of `LOG_CONS`, `LOG_NDELAY`, `LOG_NOWAIT`, `LOG_ODELAY`, `LOG_PERROR`, and `LOG_PID`.
- The reason for the facility argument is to let the configuration file specify that messages from different facilities are to be handled differently.
- The priority argument is a combination of the `facility`  and a `level`, shown in Figure 13.5.


### Single-Instance Daemons


### Daemon Conventions

> If the daemon uses a lock file, the file is usually stored in /var/run.The name of the file is usually name.pid.
> If the daemon supports configuration options, they are usually stored in /etc. The configuration file is named name.conf.
> Daemons can be started from the command line, but they are usually started from one of the system initialization scripts (/etc/rc* or /etc/init.d/*).
> If a daemon has a configuration file, the daemon reads it when it starts, but usually won't look at it again.
