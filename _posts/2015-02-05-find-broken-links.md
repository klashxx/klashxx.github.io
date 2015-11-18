---
layout: post
title: Where is my Broken Link
permalink: check-broken-links
---

This task *bugs* me quite often, there many methods to perform this job but want to share the options I consider **safer**.

Having this scenario:

{% highlight text %}
$ touch a b c
$ ln -s a a.link
$ ln -s b b.link
$ ln -s c c.link
$ rm b
{% endhighlight %}


{% highlight text %}
$ ll
total 0
-rw-r--r--   1 klashxx   klashxx   0 Feb 1 09:45 a
lrwxr-xr-x   1 klashxx   klashxx   1 Feb 1 09:45 a.link -> a
lrwxr-xr-x   1 klashxx   klashxx   1 Feb 1 09:45 b.link -> b
-rw-r--r--   1 klashxx   klashxx   0 Feb 1 09:45 c
lrwxr-xr-x   1 klashxx   klashxx   1 Feb 1 09:45 c.link -> c
{% endhighlight %}

## Find

{% highlight bash %}
find . -type l -exec test ! -e {} \; -print
{% endhighlight %}

`-type l` True if is a symbolic link file type.

`-exec test ! -e {}` *Test* if the file where the link points **DOES NOT** exists.

`-print` Show the broken link

**Outputs**

{% highlight text %}
./b.link
{% endhighlight %}

## Perl + bash

{% highlight bash %}
for link in *.link; do
  perl -se 'exit 5 unless (-e readlink($link));' -- -link=$link
  [[ $? -eq 5 ]] && echo "broken link: $link"
done
{% endhighlight %}

**Outputs**

{% highlight text %}
broken link: b.link
{% endhighlight %}

Let's explain the *mini* `Perl` program that makes the trick.


>-s
>
>enables rudimentary switch parsing for switches on the command line after the program name but before any filename arguments (or before an argument of –). Any switch found there is removed from @ARGV and sets the corresponding variable in the Perl program. The following program prints “1” if the program is invoked with a -xyz switch, and “abc” if it is invoked with -xyz=abc.
>
>-e commandline
>
>may be used to enter one line of program. If -e is given, Perl will not look for a filename in the >argument list. Multiple -e commands may be given to build up a multi-line script. Make sure to use >semicolons where you would in a normal program.

So... we use `–s` to pick link parameter and `–e` to execute the program.

**Program**

`exit 5 unless (-e readlink($link))` returns where is the symbolic link pointing.

**Arguments**

`--` *marks* the end of options and disables further option processing. Any arguments after the `—-` are treated as filenames and arguments (`bash` man).

`-link=$link` pass link variable to `Perl` program (see `–s` flag)

`-e` test if the file exists.
