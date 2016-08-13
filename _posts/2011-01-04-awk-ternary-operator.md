---
layout: post
title: Use of the ternary operator, an awk example.
permalink: ternary-operator
comments: true
---

*Post first published in [nixtip](https://nixtip.wordpress.com/2011/04/01/use-of-the-conditional-operator-an-awk-example/)*

Use of the ternary operator, an `awk` example.

Ok, suppose this source file:

{% highlight bash %}
$ cat infile
21/tcp   closed ftp
22/tcp   open   ssh
23/tcp   closed telnet
80/tcp   closed http
90/tcp   closed dnsix
95/tcp   closed supdup
100/tcp  closed newacct
162/tcp  closed snmptrap
205/tcp  closed at-5
335/tcp  closed unknown
435/tcp  closed mobilip-mn
555/tcp  closed dsf
8080/tcp closed http-proxy
8081/tcp closed blackice-icecap
{% endhighlight %}

Our mission will be to get a formatted output `port:state:service` like this

{% highlight text %}
21:1:ftp
22:0:ssh
23:1:telnet
80:1:http
{% endhighlight %}
… and so on …

All closed ports should be marked as `1` the rest will be `0`.

Like always, in `*nix` system we have plenty of tools (and approaches) to get the expected result, lets try the `awk` way…

At first sight we can identify **three fields** in our input file and tree tasks be solved.

1. Get rid of the slash + `tcp` string of the first field.
2. Change the value of the second field for `1` or `0`.
3. Field separator should be `:`

A simply text replacing, is a straightforward way to get the expected result:

{% highlight text %}
$ awk '{sub(/\/.*closed +/,":1:");sub(/\/.*open +/,":0:")}1' infile
{% endhighlight %}

Here’s the internals:

- We look for a string started by as **slash** (note de escape char `\/`) followed by any number of any character (dot + star `.*`) ,followed by the string `closed` and ended by any number of space chars `*` and replace it with `:1:` .For the first line:
`21/tcp   closed ftp` will be replace for `:1:`

- Same thing for “open” in this case “:0:” will be the substitution string , example: `22/tcp   open   ssh` will be replace for `:0:`

Our initial tasks get solved ,but we can refine our efforts.

Let’s use the conditional operator.

`expr ? action1 : action2`

Its pretty straight forward : if `expr` then `acction1` is performed/evaluated , if not `action2`.

For our example , field two must change to `1` if it's value is `closed`, if not it should be `1`.

The needed conditional operator:

`$2=="closed" ? "1" : "0"`

Depending of second field value, our program will perform a different action, in this case its returning a string : `1` or `0`.

At this point, a variable is needed to store it:

`n= $2=="closed" ? "1" : "0"`

Finally we perform the text substitution:

{% highlight bash %}
awk '{n= $2=="closed" ? "1" : "0";sub(/\/.*(open|closed) +/,":"n":")}1' infile
{% endhighlight %}

**Note** that we reduce the calls to the sub function to just one.

**A final (and total different) approach** , *field substitution* instead of text replacing.

Remember our tasks:

a) Get rid of the slash+`tcp` string of the first field.
b) Change the value of the second field for `1` or `0`
c) Field separator should be `:`

Our input file has *naturally* three fields (by the default `awk` `FS` ):

{% highlight text %}
21/tcp   closed ftp
22/tcp   open   ssh
23/tcp   closed telnet
{% endhighlight %}

It’s clear that we can think in a **four fields based line**, if we add the slash `/` to our field separators by using a `regex` as `FS='( *)|(/)'` where `( *)` represents any number of spaces as separator and `(/)` represents the slash:

So:

{% highlight awk %}
awk '{print $1,$2,$3,$4}' OFS='>' FS='( *)|(/)'  infile|head -3
{% endhighlight %}

{% highlight text %}
21>tcp>closed>ftp
22>tcp>open>ssh
23>tcp>closed>telnet
{% endhighlight %}

Note that the Output Field Separator `OFS` is changed to `>` for clarify.

Now, we want to get rid of the second field, technically is not possible, but we can assign the **null value** (empty string) to it:

{% highlight awk %}
awk '{$2=""}1' OFS='>' FS='( *)|(/)'  infile|head -4
{% endhighlight %}

{% highlight text %}
21>>closed>ftp
22>>open>ssh
23>>closed>telnet
80>>closed>http
{% endhighlight %}

**Attention**, the use of the `print` statement is not needed, `awk` will print the input line if the result of applying the inner statements to the current input line is true.

The assignment `$2=""` is not an action statement but we force a true return by placing `1` at the end of the program.

If we set the OFS to null value:

{% highlight awk %}
awk '{$2=""}1' OFS= FS='( *)|(/)'  infile|head -4
{% endhighlight %}

{% highlight text %}
21closedftp
22openssh
23closedtelnet
80closedhttp
{% endhighlight %}

We’re close to or goal, the **last step** is to process the third field:

`$3=="closed" ? ":1:" : ":0:"`

Like we saw before we need to assign it to a variable,… look the trick:

`$3= $3=="closed" ? ":1:" : ":0:"`

We say , hey! change `$3 depending of its previous value.
So :

{% highlight awk %}
awk '{$2="";$3=$3=="closed" ? ":1:" : ":0:"}1' OFS= FS='( *)|(/)'  infile|head -4
{% endhighlight %}

{% highlight text %}
21:1:ftp
22:0:ssh
23:1:telnet
80:1:http
{% endhighlight %}

A **final optimization**, the conditional operator performs always an action that imply the print statement, so:

{% highlight awk %}
awk '{$2="";$3=$3=="closed" ? ":1:" : ":0:"}1' OFS= FS='( *)|(/)' infile
{% endhighlight %}

Is equivalent to:
{% highlight awk %}
awk '$2="";$3=$3=="closed" ? ":1:" : ":0:"' OFS= FS='( *)|(/)'  infile
{% endhighlight %}

{% highlight text %}
21:1:ftp
22:0:ssh
23:1:telnet
80:1:http
90:1:dnsix
95:1:supdup
100:1:newacct
162:1:snmptrap
205:1:at-5
335:1:unknown
435:1:mobilip-mn
555:1:dsf
8080:1:http-proxy
8081:1:blackice-icecap
{% endhighlight %}

**We’re done**.
