---
layout: post
title: Text processing performance,... Perl vs sed
permalink: perl-vs-sed
---

Here's the situation:

We have a large file to process, say `897872` lines (72 Mb), a small sample looks like this:

{% highlight text %}
TheWhisperers23Chapter 22    9781442305151    "WHISPERERS"    "CONNOLLY, JOHN" 2010
TheWhisperers23Chapter 22    9781442305151    "WHISPERERS"    "TORRES, FERNANDO" 2010
{% endhighlight %}

Our goal is to **re-order** and **re-case** the names to:

{% highlight text %}
TheWhisperers23Chapter 22    9781442305151    "WHISPERERS"    "John Connolly" 2010
TheWhisperers23Chapter 22    9781442305151    "WHISPERERS"    "Fernando Torres" 2010
{% endhighlight %}

## PERL

At firs glance `Perl` seems to be the rigth tool for this purpose.

First we need to find the rigth `regexp`.

The facts are:

- Names are enclosed between **double quotes**.
- We have a comma as surname – firstname separator.
- This two creates a unique pattern to search & replace.

Now we have alter the order, and the case of the name-surname.

One one of the approaches is to *memorize* portions of the pattern to do the right substitution.

So `CONNOLLY, JOHN` will match a quote, one singular letter ,the rest of the word,some spaces (or not) ,a comma, some spaces (or not), one sigular letter the rest of the letters of the name plus the final quote.

The translation to a `Perl` *regexp* could be:

`/"(\w)(\w+)\s*,\s*(\w)(\w+)"/`

We used four parentheses groups to memorize what whe have to change.

The complete substituion will be:

`s/"(\w)(\w*)\s*,\s*(\w)(\w*)"/"$3\L$4 \U$1\L$2"/`

**Note** the use of the upper/lowercase flags and how the order of the words is altered.

In `Perl` we can use the *infile* edition, so we can use this one-liner to get the requested result:

`perl -pi -e 's/"(\w)(\w*)\s*,\s*(\w)(\w*)"/"$3\L$4 \U$1\L$2"/;' file`

## SED

It will be pretty much the same, the `regexp` is:

`/"\([A-Z]\)\([A-Z]\{1,\}\) *, *\([A-Z]\)\([A-Z]\{1,\}\)"/`

In *standard* `sed` we don't have *handy* `perl` *word* flags `\w`, so whe have to use the *old fashioned* way.

The in-file substitution is impossible also.

The complete command will be:

{% highlight bash %}
sed -ei 's/"\([A-Z]\)\([A-Z]\{1,\}\) *, *\([A-Z]\)\([A-Z]\{1,\}\)"/"\3\L\4 \U\1\L\2"/' file
{% endhighlight %}

**PERFORMANCE**

Ok, before this experiment I would bet on `Perl`, but the results are clear ….

It’s a draw (if not a `sed` winning)

{% highlight bash %}
$ time -p perl -pi -e 's/"(\w)(\w*)\s*,\s*(\w)(\w*)"/"$3\L$4 \U$1\L$2"/' file
real 6.71
user 6.38
sys 0.32
{% endhighlight %}

{% highlight bash %}
$ time -p sed -ei 's/"\([A-Z]\)\([A-Z]\{1,\}\) *, *\([A-Z]\)\([A-Z]\{1,\}\)"/"\3\L\4 \U\1\L\2"/' file
real 6.51
user 6.19
sys 0.31
{% endhighlight %}

**Perl** execution flags `NOTES`:

>-iextension
>specifies that files processed by the construct are to be edited in-place. It does this by renaming the >input file, opening the output file by the same name, and selecting that output file as the default for >print statements. The extension, if supplied, is added to the name of the old file to make a backup >copy. If no extension is supplied, no backup is made. Saying “perl -p -i.bak -e “s/foo/bar/;” … ” is the >same as using the script:
>
>-p
>causes perl to assume the following loop around your script, which makes it iterate over filename >arguments somewhat like sed:
>
>while () {
>… # your script goes here
>} continue {
>print;
>}
>
>Note that the lines are printed automatically. To suppress printing use the -n switch. A -p overrides a ->n switch.
>
>-e commandline
>may be used to enter one line of script. Multiple -e commands may be given to build up a multi-line >script. If -e is given, perl will not look for a script filename in the argument list.

