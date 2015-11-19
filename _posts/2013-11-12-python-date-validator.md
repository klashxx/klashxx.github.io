---
layout: post
title: Python Date format validator
permalink: python-date-validator
comments: true
---


**Simplicity** is always best and `Python` makes this task **insanely easy**.

We just only need to take advantage of the `strptime` function of the time module.

As usually in `Python`, the code is pretty self-explanatory:

{% highlight python %}
#!/usr/bin/python
"""date_validator.py"""
import time

def check_format(datec):
    # checks YYYYMMDD / YYYY-MM-DD / DDMMYYYY and MMDDYYYY formats

    format_ok = False
    for mask in ['%Y%m%d','%Y-%m-%d','%d%m%Y','%m%d%Y']:
    try:
        time.strptime(datec, mask)
        format_ok = True
        break
    except ValueError:
        pass

    if format_ok:
        print "Correct date !:%12s mask:%s" % (datec,mask)
    else:
        print "KO: %s" % datec
    return None

def main():
    check_format('11082011')
    check_format('12312010')
    check_format('13312010')
    check_format('20110811')
    check_format('40118841')
    check_format('2012-02-29')
    check_format('20110229')

if __name__=="__main__":
    main()
{% endhighlight %}


{% highlight text %}
./date_validator.py
Correct date !:    11082011 mask:%d%m%Y
Correct date !:    12312010 mask:%m%d%Y
KO: 13312010
Correct date !:    20110811 mask:%Y%m%d
KO: 40118841
Correct date !:  2012-02-29 mask:%Y-%m-%d
Correct date !:    20110229 mask:%d%m%Y
{% endhighlight %}

If we use the interpreter it couldn't be more clear.

Choose a good pair of `date` + `mask` and the result will be fine:

{% highlight python %}
>>> import time
>>> datec='20110811'
>>> time.strptime(datec,'%Y%m%d')
time.struct_time(tm_year=2011, tm_mon=8, tm_mday=11, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=3, tm_yday=223, tm_isdst=-1)
{% endhighlight %}

Otherwise:

{% highlight python %}
>>> time.strptime(datec,'%m%d%Y')
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
File "/usr/lib64/python2.6/_strptime.py", line 454, in _strptime_time
return _strptime(data_string, format)[0]
File "/usr/lib64/python2.6/_strptime.py", line 328, in _strptime
data_string[found.end():])
ValueError: unconverted data remains: 1
{% endhighlight %}

