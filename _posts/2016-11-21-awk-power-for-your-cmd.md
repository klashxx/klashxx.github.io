---
layout: post
title: awk, **power** to your command line
permalink: awk-power-to-your-cmd
comments: true
---

### A beginners guide to the *nix swiss army knife

<hr>


> AWK is a language similar to PERL, only considerably more elegant.
>
> - Arnold Robbins


#### *What ??*

> AWK is a programming language designed for text processing and typically used as a data extraction and reporting tool.
> It is a standard feature of most Unix-like operating systems.

#### ``awk``award name ...

> its name is derived from the surnames of its authors – Alfred **A**ho, Peter **W**einberger, and Brian **K**ernighan.


### So ``awk`` ...

* Searchs for lines that contain certain patterns in files or the standard input.

* Mostly used for data extraction and reporting like summarizing information from the output of other utility programs.

* ``C-like`` syntax.

* Data Driven: it's describe the data you want to work with and then what action to do when you find it.


```` shell
pattern { action }
pattern { action }
````

<hr>

### The Basics

#### How to Run it

* If the program is **short**:

```` shell
awk 'program' input-file1 input-file2
````
**Note:** Beware of shell quoting issues[^1].


```` shell
cmd | awk 'program'
````
**Note:** The ``pipe`` redirects the output of the *left-hand* command (``cmd``) to the **input** of the ``awk`` command[^2].


* When the program is **long**, it is usually more convenient to put it in a file and run it with a command like this:

```` shell
awk -f program-file input-file1 input-file2

cmd | awk -f program-file
````

Or just make it executable


```` shell
#!/bin/awk -f

BEGIN { print "hello world!!" }
````

#### Other useful flags

`-F fs` Set the `FS` variable to `fs`.

`-v var=val` Set the variable `var` to the value `val` before execution of the program begins. *Note*: it can be used more than once, setting another variable each time.


#### BEGIN and END

`BEGIN` and `END` special patterns or blocks that supply *startup* and *cleanup* actions for `awk` programs. 


```` shell
BEGIN{
    // initialize variables
}
{
    /pattern/ { action }
}
END{
    // cleanup
}
````

Both rules are executed **once only**, `BEGIN` before the first input record is read, `END` after all the input is consumed.

```` shell
$ echo "hello"| awk 'BEGIN{print "BEGIN";f=1}{print $f}END{print "END"}'
BEGIN
hello
END
````


<hr>

#### Why ``grep``if you have ``awk``??


```` shell
$ cat lorem_ipsum.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci, euismod id nisi eget, interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

```` shell
$ grep dolor lorem_ipsum.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
````

```` shell
$ awk '/dolor/' lorem_ipsum.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
````
**Note:** If the action is not given the default action is to print all that lines that matches the given pattern.

But... how can we find out the *first* and *last* word of each line?

Of course `grep` can do it, but needs two steps:

```` shell 
$ grep -Eo '^[^ ]+' lorem_ipsum.dat 
Lorem
Maecenas
Nunc
Curabitur
Lorem
Aliquam
````

```` shell
$ grep -Eo '[^ ]+$' lorem_ipsum.dat 
elit.
condimentum.
ex.
tellus.
elit.
ultrices.
````

Let\'s see `awk` in action here:

```` shell
$ awk '{print $1,$NF}' lorem_ipsum.dat 
Lorem elit.
Maecenas condimentum.
Nunc ex.
Curabitur tellus.
Lorem elit.
Aliquam ultrices.
````

### Isn't this better :sunglasses: ?  , but ... HTF does this works ?

`awk` divides the input for your program into *records* and *fields*.

#### Records

Records are separated by a character called the *record separator* `RS`. 
By default, the record separator is the newline character `\n`. This is why records are, **by default**, *single lines*.

Additionally `awk` has the `ORS` *Output Record Separator* to control the way the records are given to the `stdout`.

`RS` and `ORS` should be enclosed in **quotation marks**, which indicate a string constant. 

To use a different character or a *regex* simply assign it to the `RS` variable:

- Often, the right time to do this is at the beginning of execution `BEGIN`, before any input is processed, so that the very first record is read with the proper separator. 
- Another way to change the record separator is on the command line, using the variable-assignment feature.

Examples:


```` shell
$ awk 'BEGIN{RS=" *, *";ORS="<<<---\n"}{print $0}' lorem_ipsum.dat 
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci<<<---
euismod id nisi eget<<<---
interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat<<<---
et facilisis neque ultrices.
<<<---
````

```` shell
$ awk '{print $0}' RS=" *, *" ORS="<<<---\n" lorem_ipsum.dat 
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci<<<---
euismod id nisi eget<<<---
interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat<<<---
et facilisis neque ultrices.
<<<---
````


#### Fields

`awk` records are automatically parsed or separated into *chunks* called **fields**. 

By default, fields are separated by **whitespace** (any string of one or more spaces, TABs, or newlines), like words in a line.

To refer to a field in an `awk` program, You use a dollar `$` sign followed by the number of the field you want. Thus, `$1` refers to the first field, `$2` to the second, and so on. 

```` shell
$ awk '{print $3}' lorem_ipsum.dat 
dolor
erat
orci,
dapibus
dolor
mauris
````

`NF` is a predefined variable whose value is the **number of fields in the current record**. So, `$NF` will be allways the last field of the record.

```` shell
$ awk '{print NF}' lorem_ipsum.dat 
8
7
10
4
8
10
````

`FS` holds the valued of the *field separator*, this value is a single-character string or a `regex` that matches the separations between fields in an input record. 

The default value is `"  "`, a string consisting of a single space. As a special exception, this value means that any sequence of spaces, TABs, and/or newlines is a single separator.

In the same fashion that `ORS` we have a `OFS` variable to manage how our fields are going to be send to the output stream.

```` shell
$ cat /etc/group
nobody:*:-2:
nogroup:*:-1:
wheel:*:0:root
daemon:*:1:root
kmem:*:2:root
sys:*:3:root
tty:*:4:root
````


```` shell
$ awk '!/^(_|#)/&&$1=$1' FS=":" OFS="<->" /etc/group
nobody<->*<->-2<->
nogroup<->*<->-1<->
wheel<->*<->0<->root
daemon<->*<->1<->root
kmem<->*<->2<->root
sys<->*<->3<->root
tty<->*<->4<->root
````

**Note**: Ummm ... `$1=$1` ?[^3]

<hr>

Keeping *records* an *fields* in mind, were now ready to understand our previous code:

```` shell
$ awk '{print $1,$NF}' lorem_ipsum.dat 
Lorem elit.
Maecenas condimentum.
Nunc ex.
Curabitur tellus.
Lorem elit.
Aliquam ultrices.
````


#### NR and FNR

These are two useful built-in variables:

`NR` : number of input records `awk` has processed since the beginning of the program’s execution.

`FNR` : current record number in the current file, `awk` resets `FNR` to zero each time it starts a new input file.

```` shell
$ cat n1.dat 
one
two
````

```` shell
$ cat n2.dat 
three
four
````

```` shell
$ awk '{print NR,FNR,$0}' n1.dat n2.dat 
1 1 one
2 2 two
3 1 three
4 2 four
````




#### Fancier Printing

The format string is very similar to that in the ISO C.

Syntax:

`printf format, item1, item2, …`

```` shell
$ awk '{printf "%20s <-> %s\n",$1,$NF}' lorem_ipsum.dat 
               Lorem <-> elit.
            Maecenas <-> condimentum.
                Nunc <-> ex.
           Curabitur <-> tellus.
               Lorem <-> elit.
             Aliquam <-> ultrices.
````


#### Redirecting Output

Output from `print` and `printf` is directed to the standard output by default but we can use redirection to change the destination.

Redirections in `awk` are written just like redirections in shell commands, except that they are written inside the `awk` program.

```` shell
$ awk 'BEGIN{print "hello">"hello.dat"}'
$ awk 'BEGIN{print "world!">>"hello.dat"}'
$ cat hello.dat 
hello
world!
````

It is also possible to send output to another program through a pipe:

```` shell
$ awk 'BEGIN{sh="/bin/sh";print "date"|sh;close(sh)}'
dom nov 13 18:36:25 CET 2016
````

<hr>

## Now let\'s have some fun :godmode:


Challenges:

1. Print the penultimate field of a file.

2. Add a semicolon at the end of each line.

3. Add a colon between words.

4. All together??

5. Print odd lines to `odd.dat` and even lines to `even.dat`.


Solved:

1. 
```` shell
$ awk '{print $(NF-1)}' lorem_ipsum.dat 
adipiscing
consectetur
cursus
dapibus
adipiscing
neque
````

2. 
```` shell
$ awk '1' ORS=";\n" lorem_ipsum.dat 
Lorem ipsum dolor sit amet, consectetur adipiscing elit.;
Maecenas pellentesque erat vel tortor consectetur condimentum.;
Nunc enim orci, euismod id nisi eget, interdum cursus ex.;
Curabitur a dapibus tellus.;
Lorem ipsum dolor sit amet, consectetur adipiscing elit.;
Aliquam interdum mauris volutpat nisl placerat, et facilisis neque ultrices.;
````
     **Note**: What does that `1` means[^4]?

3. 
```` shell
$ awk '{$1=$1}1' OFS=',' lorem_ipsum.dat 
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.
Maecenas,pellentesque,erat,vel,tortor,consectetur,condimentum.
Nunc,enim,orci,,euismod,id,nisi,eget,,interdum,cursus,ex.
Curabitur,a,dapibus,tellus.
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.
Aliquam,interdum,mauris,volutpat,nisl,placerat,,et,facilisis,neque,ultrices.
````

4. 
```` shell
$ awk '{$1=$1}1' OFS=',' ORS=';\n' lorem_ipsum.dat 
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.;
Maecenas,pellentesque,erat,vel,tortor,consectetur,condimentum.;
Nunc,enim,orci,,euismod,id,nisi,eget,,interdum,cursus,ex.;
Curabitur,a,dapibus,tellus.;
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.;
Aliquam,interdum,mauris,volutpat,nisl,placerat,,et,facilisis,neque,ultrices.;
````


[^1]: [A Guide to Unix Shell Quoting][quoting-guide].

[^2]: [Wikipedia on pipelines][pipes].

[^3]: There are times when it is convenient to force `awk` to rebuild the entire record, using the current values of the `FS` and `OFS`. 
      
      To do this, we use the seemingly innocuous assignment: `$1 = $1`

[^4]: Quick answer, It\'s just a *shortcut* to avoid using the print statement.
      
      In `awk` when a condition gets matched the *default action* is to print the input line.

      `$ echo "test" |awk '1'`

      Is equivalent to:

      `echo "test"|awk '1==1'`
      
      `echo "test"|awk '{if (1==1){print}}'`

      That\'s because `1` will be always [true].


[quoting-guide]: http://resources.mpi-inf.mpg.de/departments/rg1/teaching/unixffb-ss98/quoting-guide.html "A Guide to Unix Shell Quoting"
[pipes]: https://en.wikipedia.org/wiki/Pipeline_(Unix) "Pipeline (Unix)"
[true]:  https://www.gnu.org/software/gawk/manual/gawk.html#Truth-Values-and-Conditions "True and False in awk"

