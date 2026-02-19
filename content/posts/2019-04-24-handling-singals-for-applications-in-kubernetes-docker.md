---
title: "Handling signals for applications running in kubernetes"
description: "Handling signals for applications running in kubernetes"
tags: [kubernetes, docker, devops]
comments: true
share: true
cover_image: '/content/images/2019/04/docker-k8s.jpg'
---

When the power goes off in a device in a linux based system, one can think of ways in which this event can be handled in the applications running on it. One thing to note is that, when you plug the power cable off, the power doesn't really go off immediately.

But this needs to be notified to processes so that they can handle such an event and save the state of the application (if any).

A few possible ways of doing so would be to
- Save everything in RAM to disk and then restore the contents stored on disk back to RAM when the startup happens next, but the problem with this approach is that, storing and retrieval on/from a disk is a slow process.
- Use a file to store the state of power, where 0 would denote that the power is on and 1 would mean that the power has gone off, but this approach would mean that the processes running in the system should keep track of this bit stored in the file.
- The kernel sends a signal like `SIGTERM` to the process and leaves it to the process on how it handles this signal.

How a container gets stopped is something which is very important or not, depending on your application of course.

I will discuss how does docker and kubernetes at large, and how the containers orchestrated via them handle signals very briefly in this blog post. 

## What is a signal

> A signal is a software interrupt and a way to communicate the state of a process(es) to another process, the OS and the hardware. 

By interrupt, we mean that whenever a signal is received by a process. It will stop doing whatever it is doing and handle it by either doing something about it or ignoring it. 

A few Signals and what they intend to do([credits www.usna.edu](https://www.usna.edu/Users/cs/aviv/classes/ic221/s16/lec/19/lec.html)) are listed below.

```sh
Signal     Value     Action   Comment
----------------------------------------------------------------------
SIGHUP        1       Term    Hangup detected on controlling terminal
                              or death of controlling process
SIGINT        2       Term    Interrupt from keyboard
SIGQUIT       3       Core    Quit from keyboard
SIGILL        4       Core    Illegal Instruction
SIGABRT       6       Core    Abort signal from abort(3)
SIGFPE        8       Core    Floating point exception
SIGKILL       9       Term    Kill signal
SIGSEGV      11       Core    Invalid memory reference
SIGPIPE      13       Term    Broken pipe: write to pipe with no
                              readers
SIGALRM      14       Term    Timer signal from alarm(2)
SIGTERM      15       Term    Termination signal
SIGUSR1   30,10,16    Term    User-defined signal 1
SIGUSR2   31,12,17    Term    User-defined signal 2
SIGCHLD   20,17,18    Ign     Child stopped or terminated
SIGCONT   19,18,25    Cont    Continue if stopped
SIGSTOP   17,19,23    Stop    Stop process
SIGTSTP   18,20,24    Stop    Stop typed at tty
SIGTTIN   21,21,26    Stop    tty input for background process
SIGTTOU   22,22,27    Stop    tty output for background process
```

Each signal has a default value and action defined. You can take a look at `sys/signal.h` and 
find out more about each and every other signal defined in it. 

## Passing the signal to a process from the command line

When you do a `ctrl+c`, it's the same as sending a `SIGINT` signal and when you type `ctrl+z`, it's
the same as sending `SIGTSTP`, and when we type `fg` or `bg` that is the same as sending a 
`SIGCONT` signal.

## Passing signals to containers

When you issue a `docker stop`, docker send `SIGTERM` to the process running as PID 1 inside the container
and waits for 10 seconds before it sends a `SIGKILL` to the kernel which will then straight terminate the
process, if the process hasn't terminated within that time frame. 

A `docker kill` will not give the container process, an opportunity to stop gracefully, but will straight ahead kill it. 

## How kubernetes handles signals

When you do a 

```sh
$ kubectl delete pods mypod
```

it will send a `SIGTERM` and then wait for a set number of seconds to send a `SIGKILL` to the process,
this period is known as the grace termination period of the pod and can be configured in the [podSpec](https://kubernetes.io/docs/api-reference/v1.9/#podspec-v1-core)

If your process doesn't handle `SIGTERM`, then it will get `SIGKILL`ed. Processes which are killed are 
immediately removed from the etcd and the API, without waiting for the process to actually terminate on the 
node. 

If you have anything which needs a graceful shutdown, you need to implement a handler for `SIGTERM`.

If there are more than one containers in a pod, they both will be sent a `SIGTERM` and one would want
to implement the right strategy on how they should be terminated. 

One mistake which is pretty common and something which can be missed is using the non-exec form of [`CMD`](https://docs.docker.com/engine/reference/builder/#cmd)
, for example, then your process is running as a child process of shell and not really running as root. 
It will be running as `/bin/sh -c myapplication`

The problem with this is that, shell will never forward this signal to the child process, which is your application
process and it will not be able to handle the `SIGTERM` for which you have written a handler for.

In any case, this would make your process be `SIGKILL`ed. Instead one should use the exec form CMD

```sh
/bin/sh -c myapplication
```

or use the exec form of [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint)

## Passing custom signals to container processes

Let's take an example of a sample application, which has a dockerfile like this

```
FROM alpine:3.5

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
RUN apk add --no-cache bash

ENTRYPOINT ["/entrypoint.sh"]
```

And the contents of your `entrypoint.sh` are

```sh
#!/bin/bash

trap "echo TERM" TERM
trap "echo HUP" HUP
trap "echo INT" INT
trap "echo QUIT" QUIT
trap "echo USR1; sleep 2; exit 0" USR1
trap "echo USR2" USR2

ps aux
tail -f /dev/null
```

Now if you build this container and run it in the background, if you try stopping the container
by doing `docker stop`. You will notice that the container process takes 10 seconds before the process 
dies and get `SIGKILL`ed

To avoid that, we do a signal rewrite using [dumb-init](https://github.com/Yelp/dumb-init) which runs a PID 1 for the docker
container. You can [read here](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/) about why running your application process as PID 1 is not usually a good idea.

```sh
FROM alpine:3.5

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
RUN apk add --no-cache bash

# Change 1: Download dumb-init
ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64 /usr/local/bin/dumb-init
RUN chmod +x /usr/local/bin/dumb-init

# Change 2: Make it the entrypoint.  The arguments are optional
ENTRYPOINT ["/usr/local/bin/dumb-init","--rewrite","15:10","--"]
CMD ["/entrypoint.sh"]
```

Now if you try issuing a `docker stop`, you will notice that the process exits out and it will get the
`USR1` signal and process it and then exit, which is what we wanted. 

## Signal rewriting for gracefully terminating apache2 and nginx

If you want to initiate a graceful shutdown of an nginx server, you should send a `SIGQUIT`. 
None of the Docker commands issue a `SIGQUIT` by default. So the solution for kubernetes is to 
rewrite the signal `SIGTERM` to a `SIGQUIT` for nginx.

```sh
## for full source, check https://github.com/Yelp/casper/blob/master/Dockerfile.opensource
FROM ubuntu:xenial

# Manually install dumb-init as it's not in the public APT repo
RUN wget https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64.deb
RUN dpkg -i dumb-init_*.deb

## Your application requirements

# Rewrite SIGTERM(15) to SIGQUIT(3) to let Nginx shut down gracefully
CMD ["dumb-init", "--rewrite", "15:3", "/code/start.sh"]
```

This will send the signal of `SIGQUIT` to nginx to gracefully handle termination

For gracefully terminating apache2, it requires a `SIGWINCH` signal to be passed to it. Which can be done by
passing the signal `28` to it.

```sh
## your dockerfile contents before this
CMD ["dumb-init", "--rewrite", "15:28", "/code/start.sh"]
```

## References

- [https://lasr.cs.ucla.edu/vahab/resources/signals.html](https://lasr.cs.ucla.edu/vahab/resources/signals.html)
- [https://www.ctl.io/developers/blog/post/gracefully-stopping-docker-containers/](https://www.ctl.io/developers/blog/post/gracefully-stopping-docker-containers/)
- [https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods)
- [https://httpd.apache.org/docs/2.4/stopping.html](https://httpd.apache.org/docs/2.4/stopping.html)
- [https://www.usna.edu/Users/cs/aviv/classes/ic221/s16/lec/19/lec.html](https://www.usna.edu/Users/cs/aviv/classes/ic221/s16/lec/19/lec.html)
- [http://jkamenik.github.io/blog/2017/10/21/docker-details---dumb-init/](http://jkamenik.github.io/blog/2017/10/21/docker-details---dumb-init/)
- [http://nginx.org/en/docs/control.html](http://nginx.org/en/docs/control.html)
- [http://httpd.apache.org/docs/2.2/stopping.html#gracefulstop](http://httpd.apache.org/docs/2.2/stopping.html#gracefulstop)
- [https://github.com/Yelp/casper/issues/21](https://github.com/Yelp/casper/issues/21)
- [https://unix.stackexchange.com/questions/80044/how-signals-work-internally](https://unix.stackexchange.com/questions/80044/how-signals-work-internally)
- [https://stackoverflow.com/questions/37374310/how-critical-is-dumb-init-for-docker](https://stackoverflow.com/questions/37374310/how-critical-is-dumb-init-for-docker)
