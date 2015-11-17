---
layout: post
title: PLAIN TEXT the language of *NIX
permalink: plain-text-language-of-nix
---

Let's face it, one of the main task of the IT administrator could possibly be text processing.

Why? Because text is everywhere, computer systems speak text in three different languages `STDOUT`, `STDERR` and `STDIN`.

We need to develop our skills to create log files miners, to adapt the ugly output of a program to meet our needs, etc...

Fortunately `*nix` offer may options to get our goals, so we have to try many tools until we find *the one* we get comfortable with.

Usually there's nothing wrong with that, nowadays, **most times**, the power of the machines allow us to use the tool we want focusing on the results.

But... what happens when we have to process a `4 GBs` file, or many files on production systems ?

Let’s illustrate this with an **example**.

You can use this `shell` script to create the sample text file:

{% highlight bash %}
$ i=1
$ while (( i<=10000000 )); do echo "Line: $i"; (( i += 1 )); done>dummy.txt
{% endhighlight %}

**NOTE**: the weight of the should be around `130 MB`.

Now we have a `10 million` lines flat text file.

Our task is simple, extract the line number `5000000`

Let’s *bench* a bunch of common ways:

{% highlight bash %}
$ time -p sed -n '5000000p' dummy.txt
Line: 5000000
real 1.12
user 1.06
sys 0.06
{% endhighlight %}

{% highlight bash %}
$ time -p awk 'NR==5000000{print;exit}' dummy.txt
Line: 5000000
real 1.09
user 1.05
sys 0.03
{% endhighlight %}

{% highlight bash %}
$ time -p perl -ne '$. == 5000000 && {print and exit}' dummy.txt
Line: 5000000
real 1.78
user 1.73
sys 0.04
{% endhighlight %}

{% highlight bash %}
$ time -p head -5000000 dummy.txt >dummy.txt.2
real 0.28
user 0.07
sys 0.20

$ time -p tail -1 dummy.txt.2
Line: 5000000
real 0.00
user 0.00
sys 0.00
{% endhighlight %}

{% highlight bash %}
$ time -p (head -5000000 dummy.txt |tail -1)
Line: 5000000
real 0.16
user 0.21
sys 0.10
{% endhighlight %}

The processing times are *quite similar*, but if you don't use the right logic …

{% highlight bash %}
$ time -p perl -ne '$. == 5000000 && print' dummy.txt
Line: 5000000
real 3.54
user 3.48
sys 0.06
{% endhighlight %}

{% highlight bash %}
$ time -p awk 'NR==5000000' dummy.txt
Line: 5000000
real 2.19
user 2.14
sys 0.05
{% endhighlight %}

**We get a double time here!** (*no exit after found the line*)

**Conclusion**: Don’t care too much about the tool, **care about your programming skills** or be ready to waste precious computing time.
