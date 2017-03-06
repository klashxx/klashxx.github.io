---
layout: post
title: The Killing machine
permalink: the-killing-machine
comments: true
---

Suppose we need to purge the processes in our system having two conditions:

- Running for  24 hours or more.
- Matching a _regexp_

This situation could be tricky sometimes.

[`killall` ](http://man7.org/linux/man-pages/man1/killall.1.html)

It will be the perfect tool when there's no need to inspect the command args, only process name is taken in consideration.

> -o, --older-than
>       Match only processes that are older (**started before**) the **time**
>       **specified**.  The time is specified as a float then a unit.  The
>       units are s,m,h,d,w,M,y for seconds, minutes, hours, days,
>       weeks, Months and years respectively.
>
> -r, --regexp
>       Interpret **process name** pattern as an extended regular expression.

[`pkill` ](http://man7.org/linux/man-pages/man1/pkill.1.html)

It solves the problem of  hiding parameters:

> -f flag can be used for args examination but there’s no way to select processes by elapsed time

But **does not provide** a way to sort processes by elapsed time.


### So ... Let’s build our custom solution.

Let's go back to our old friend [`ps` ](http://man7.org/linux/man-pages/man1/ps.1.html).

From the man page:

> -e              Select all processes.
>
> -o format       user-defined format.


AIX FORMAT DESCRIPTORS

>  This ps supports AIX format descriptors, which work somewhat like the formatting codes of printf(1) and printf(3).
>  For example, the normal default output can be produced with this:
>   `ps -eo "%p %y %x %c"`

>        CODE   NORMAL   HEADER
>        %C     pcpu     %CPU
>        %G     group    GROUP
>        %P     ppid     PPID
>        %U     user     USER
>        %a     args     COMMAND
>        %c     comm     COMMAND
>        %g     rgroup   RGROUP
>        %n     nice     NI
>        %p     pid      PID
>        %r     pgid     PGID
>        %t     etime    ELAPSED
>        %u     ruser    RUSER
>        %x     time     TIME
>        %y     tty      TTY
>        %z     vsz      VSZ

First things first, To _Kill_ a process, first, we need is its `PID`, then get how long has it been running and finally the command name and it's args.

This be accomplished by this format string using the codes mentioned in the table above:

`ps -eo "%p>~<%t>~<%a"`

**NOTE** : It’s important to choose a _complicated_ string as separator between our fields `>~<`, we don't want to find the same one inside the command name or the args garbling our data.


To process this output let’s compose an `awk`[^1] oneliner, step by step.

#### How to get processes running for + 24h?

In `ps` man page:

> `etime`  ELAPSED  elapsed time since the process was started, in the form `[[dd-]hh:]mm:ss`

So, **a _dash_ char in the second field** means that the program has been running for at least 24  hours.

Example:

```
$ ps -eo "%p>~<%t>~<%a" | awk -v '$2 ~ /-/' FS='>~<'
  528>~<49-04:37:37>~</sbin/udevd -d
  746>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log800 86400
  747>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log445 86400
  748>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log1447 86400
  749>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log450 86400
 2170>~<49-04:37:14>~</sbin/rsyslogd -i /var/run/syslogd.pid -c 5
 2204>~<49-04:37:14>~<irqbalance --pid=/var/run/irqbalance.pid
 2270>~<49-04:37:14>~</usr/sbin/mcelog --daemon
 6892>~<49-04:37:01>~</usr/sbin/snmpd -LS0-6d -Lf /dev/null -p /var/run/snmpd.pid
 6920>~<49-04:37:01>~<xinetd -stayalive -pidfile /var/run/xinetd.pid
```

**NOTE**: `FS` is set to the string used in `ps` format: `>~<`

#### Does the command line match our *regexp*?

Last step,  check if the `command` + `args` (`%a`) contains our `regexp`, for this example the *rotatelogs* string.

```
$ ps -eo "%p>~<%t>~<%a" | awk -v r="rotate.*access.*" '$2 ~ /-/ && $3 ~ r' FS='>~<'
  746>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log800 86400
  747>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log445 86400
  748>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log1447 86400
  749>~<21-08:21:52>~</dummys/apache/bin/rotatelogs -f /logs/access_log450 86400
```

Lets print only the pids.


```
$ ps -eo "%p>~<%t>~<%a" | awk -v r="rotate.*access.*" '$2 ~ /-/ && $3 ~ r{printf "%d ",$1}' FS='>~<'
  746 747 748 749
```

[`Bash command substituion` ](https://www.gnu.org/software/bash/manual/bashref.html#Command-Substitution) will make the final trick.

```
$ kill $(ps -eo "%p>~<%t>~<%a" |\
  awk -v r="rotate.*access.*" '$2 ~ /-/ && $3 ~ r{printf "%d ",$1}' FS='>~<')
```

[^1]: Check my *guide* [AWK power for your command line][english].

[english]: {{ site.baseurl }}/awk-power-for-your-cmd "AWK power for your command line"
