---
title: "PTRACE_O_TRACEEXEC in ptrace"
date: 2020-01-16 15:19:00
tags: [Linux, ptrace]
---
ptrace's [manpage](https://man7.org/linux/man-pages/man2/ptrace.2.html) is a little confusing about the effect of the `PTRACE_O_TRACEEXEC` option:

First we have,
```
If the PTRACE_O_TRACEEXEC option is not in effect, all successful
calls to execve(2) by the traced process will cause it to be sent
a SIGTRAP signal, giving the parent a chance to gain control
before the new program begins execution.
```
and then, further down,
```
PTRACE_O_TRACEEXEC (since Linux 2.5.46)
        Stop the tracee at the next execve(2).  A
        waitpid(2) by the tracer will return a status value
        such that

        status>>8 == (SIGTRAP | (PTRACE_EVENT_EXEC<<8))

        If the execing thread is not a thread group leader,
        the thread ID is reset to thread group leader's ID
        before this stop.  Since Linux 3.0, the former
        thread ID can be retrieved with PTRACE_GETEVENTMSG.
```
Whether we have `PTRACE_O_TRACEEXEC` or not, the tracee/child will be stopped at its next `execve()`. The difference lies in what status does the tracer/parent gets from `wait()/waitpid()`.

**If PTRACE_O_TRACEEXEC**, the kernel will make sure that `status>>8 == (SIGTRAP | (PTRACE_EVENT_EXEC<<8))` (as the document mentions) for the `wait()/waitpid()` corresponding to the `execve()`.

**If NO PTRACE_O_TRACEEXEC**, the kernel will send a SIGTRAP (also as the document mentions) to the tracee/child, which will not be distinguishable with *other* SIGTRAPs, such as if the tracee/child `raise(SIGTRAP)`.

Consider the following (not actually compiling) program:

```cpp
void parent_tracer() {
    ptrace(PTRACE_SETOPTIONS, child, NULL, PTRACE_O_TRACEEXEC);

    int status;
    wait(&status);

    if (status == SIGTRAP)
        printf("child raise()");
    else {
        assert(status>>8 == (SIGTRAP | (PTRACE_EVENT_EXEC<<8)));
        printf("child execve()");
    }
}

void child_tracee() {
    if (rand() % 2)
        execve(...);
    else
        raise(SIGTRAP);
}
```