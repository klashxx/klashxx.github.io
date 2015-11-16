---
layout: post
title: Print lines between two patterns , the awk way ...
permalink: awk-between-two-patterns
---

Example input file:

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

The *standard* way ..

```bash
awk '/OUTPUT/ {flag=1;next} /END/{flag=0} flag {print}' inputFile
```

    top 2
    bottom 1
    left 0
    right 0
    page 66

Self-explained indented code:

```bash
awk '
/OUTPUT/ {flag=1;next}        # Initial pattern found --> turn on the flag and read the next line
/END/    {flag=0}             # Final pattern found   --> turn off rhe flag
flag     {print}              # Flag on --> print the current line
' inputFile
```

The *first optimization* is to get rid of the print , in awk when a condition is true `print` is the default action , so when the flag is true the line is going to be echoed.

To delete de NEXT statement , in order o prevent printing the TAG line,  we need to activate the flag after the `OUTPUT` pattern discovery and after the flag evaluation.

A slight variation of the program flow and we're done:

```bash
awk '/END/{flag=0}flag;/OUTPUT/{flag=1}' inputFile
```
**PD**: What if we only want to print the lines enclosed between the OUTPUT && END tags ?
