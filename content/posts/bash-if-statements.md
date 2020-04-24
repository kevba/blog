---
title: "Bash if statements"
date: 2020-04-24T7:55:07+01:00
draft: true
---

One of the things in bash that always confuses me are if statements. I never know how to write one without googling it.
For example: Do I need two square brackets `[[ ` or just one `[ `. When should I use `-eq ` and when `==`?. Why are there even two options for all those things?

This post is mostly an excuse to figure these things out and learn about them. It is by no means comprehensive.

# The square bracket situation
In bash an if-statement can be written as follows:

```bash
if [ "$1" = "" ]; then echo "something"; fi
```

but something like this is also allowed:
```bash
if [[ "$1" = "" ]]; then echo "something"; fi
```

## Whats the meaning of the extra square brackets?

The single bracket `[ ]` is defined by POSIX. This means that a singe bracket is compatible with all POSIX compliant systems.
This includes most linux distributions so it runs pretty much on any system.

The double bracket `[[ ]]` means it uses the extended bash specification. This means it can only run on systems which use bash. 

## Which one should I use.
In general it is better to use the singe bracket, because it is compatible with most systems.
If the bash specific features are needed the double brackets can be used. Be aware your script might not work for everyone if you do this.
If you use the double brackets be sure to add the `#!/bin/bash` statement to the top of your script.

##  The "test" command
The single single bracket syntax really is just a fancy way of calling the `test` command. The statement above is equivalent to:

```bash
if test "$1" = "" ; then echo "something"; fi
```

Basically everything between the brackets gets passed as an argument to `test`. You can call the test command in your terminal like any other program on your system. Executing something like `which test` will show you where the command can be found on your system.

# Equality operators
In bash there seem to be multiple operators to comapre things. There is the `==`, the `-eq` and sometimes even a `=`.

## "=" and "=="
Both `=` and `==` are used for string comparison. The single `=` is defined by POSIX, so it is useable almost everywhere. The double `==` is part of bash, and is thus less portable. Since `=` and `==` do the exact same thing is is usually the best to use the single `=`.

## "-eq"
`-eq` is used to compare integers on equality. Beside `-eq` there are a other operators for comparing integers. The list below shows all operators for interger comparison.

- `-eq` equal
- `-ne` not equal
- `-lt` less than
- `-le` less than or equal
- `-gt` greater than
- `-ge` greater than or equal

A statement using these operators looks like this
```bash
if [ 1 -eq 2 ] ; then echo "something"; fi
```

## Arithmetic evaluation
It is is also possible to compare integers by using `<`, `>` and `==` using arithmetic evaluation. Within an arithmetic evaluation arithmetic can be done using the same syntax as in C. This emans the `<` and `>` can be used for comparing numbers and won't work as a redirect.

An example of this is
```bash
if ((1 > 9)); then echo "something"; fi
```
The bash-hackers wiki has a very good [article][0] describing how this works in detail. 



[0]: https://wiki.bash-hackers.org/syntax/arith_expr