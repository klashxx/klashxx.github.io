---
layout: post
title: Print lines between two patterns , the awk way ...
permalink: awk-between-two-patterns
comments: true
---

*Post first published in [nixtip](https://nixtip.wordpress.com/2010/10/12/print-lines-between-two-patterns-the-awk-way/)*

Example input file:

{% highlight text %}
test -3
test -2
test -1
OUTPUT
top 2
bottom 1
left 0
right 0
page 66
END
test 1
test 2
test 3
{% endhighlight %}

The *standard* way ..

{% highlight awk %}
awk '/OUTPUT/ {flag=1;next} /END/{flag=0} flag {print}' infile
{% endhighlight %}

{% highlight text %}
top 2
bottom 1
left 0
right 0
page 66
{% endhighlight %}


Self-explained indented code:

{% highlight awk %}
awk '
/OUTPUT/ {flag=1;next} # Initial pattern found --> turn on the flag and read the next line
/END/    {flag=0}      # Final pattern found   --> turn off rhe flag
flag     {print}       # Flag on --> print the current line
' infile
{% endhighlight %}

The *first optimization* is to get rid of the print , in `awk` when a condition is true `print` is the default action , so when the flag is true the line is going to be *echoed*.

To delete de `NEXT` statement , in order o prevent printing the TAG line, we need to activate the flag after the `OUTPUT` pattern discovery and after the flag evaluation.

A slight variation of the program flow and we're done:

{% highlight awk %}
awk '/END/{flag=0}flag;/OUTPUT/{flag=1}' infile
{% endhighlight %}

**PD**: What if we only want to print the lines *enclosed* between the `OUTPUT` && `END` tags ?
