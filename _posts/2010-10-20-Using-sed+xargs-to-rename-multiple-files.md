---
layout: post
title: Using sed + xargs to rename multiple files
permalink: sed-xargs-to-rename
comments: true
---


Lets say that whe have a bunch of `txt` files and we need to rename to `sql`.

{% highlight bash %}
$ touch a.txt  b.txt  c.txt  d.txt  e.txt  f.txt
{% endhighlight %}

{% highlight bash %}$ ls
a.txt  b.txt  c.txt  d.txt  e.txt  f.txt
{% endhighlight %}

We can use `ls` combined with `sed` and `xargs` to achieve our goal.

{% highlight bash %}
$ ls | sed -e "p;s/\.txt$/\.sql/"|xargs -n2 mv
{% endhighlight %}

{% highlight bash %}
$ ls
a.sql  b.sql  c.sql  d.sql  e.sql  f.sql
{% endhighlight %}

How it works:

{% highlight bash %}
$ ls | sed -e "p;s/\.txt$/\.sql/"
a.txt
a.sql
b.txt
b.sql
c.txt
c.sql
d.txt
d.sql
e.txt
e.sql
f.txt
f.sql
{% endhighlight %}

The `ls` output *is piped* to `sed` , then we use the `p` flag to print the argument **without** modifications, in other words, the *original name of the file*.

The next step is use the substitute command to change file extension.

**NOTE**: We're using single quotes to enclose literal strings (the dot is a metacharacter if using double quotes scape it with a `backslash`).

The result is a combined output that consist of a sequence of `old_file_name` and `new_file_name`.

Finally we **pipe** the resulting feed through `xargs` to get the effective rename of the files.

{% highlight bash %}
$ ls | sed -e "p;s/.txt$/.sql/"|xargs -n2 mv
{% endhighlight %}

**PD**: Alternative path **to take care of spaces in the file names**:

{% highlight bash %}
$ touch "a a d.txt.txt" "b b b.txt" "c c.txt" d.txt e.txt f.txt
{% endhighlight %}

{% highlight bash %}
$ ls
a a d.txt.txt  b b b.txt      c c.txt        d.txt          e.txt          f.txt
{% endhighlight %}

Here's the CMD:

{% highlight bash %}
$ ls | awk '{gsub(/^|$/,"\"");print;gsub(/\.txt\"$/,".sql\"")}1' |xargs -n2 mv
{% endhighlight %}

Result:

{% highlight bash %}
$ ls
a a d.txt.sql  b b b.sql      c c.sql        d.sql          e.sql          f.sql
{% endhighlight %}

From the **man page**:

>DESCRIPTION
>
>xargs combines the fixed initial-arguments with arguments read from
>standard input to execute the specified command one or more times.
>The number of arguments read for each command invocation and the
>manner in which they are combined are determined by the options
>specified. [/sourcecode]
>
>The n parameter
>
>-n number      Execute command using as many standard input
>arguments as possible, up to number arguments
>maximum.  Fewer arguments are used if their total
>size is greater than size bytes, and for the last
>invocation if there are fewer than number
>arguments remaining.  If option -x is also coded,
>each number arguments must fit in the size[/sourcecode]
>
>The -n2 flag force xargs to take 2 arguments from the piped output each time and parses it to the mv command to get the job done.
