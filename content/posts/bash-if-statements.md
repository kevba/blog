---
title: "I want to be the very Bash-t"
date: 2020-03-16T10:55:07+01:00
draft: true
---

# I want to be the very Bash-t

One of the things in bash that always confuses me are if statements. I never know how to write one without googling it.
For example: Do i need two square brackets `[[ ` or just one `[ `. When should I use `-eq ` and when `==`?. Why are there even two options for all those things?.

With this post I will attempt to anwser those questions to force myself to learn about them.

# The square bracket situation
In bash an if statement can be written as follows:

```bash
if [ "$1" == "" ]; then echo "something"; fi
```

but something like this is also allowed:
```bash
if [[ "$1" == "" ]]; then echo "something"; fi
```

### Whats the meaning of the extra square brackets?

The single bracket `[ ]` means it is POSIX compliant. This means that a singe bracket is compatible with all POSIX compliant systems.
This includes most linux distributions so it runs pretty much on any system.

The double bracket `[[ ]]`means it uses the extended bash specification. This means it can only run on systems which use bash. 

### Which one should I use.
In general it is better to use the singe bracket, because it is compatible with most systems.
If the bash specific features are needed the double brackets can be used. Be aware your script might not work for everyone if you do this.
If you use the double brackets be sure to add the `#!/bin/bash` statement to the top of your script.

###  the `test` command
The single single bracket syntax really is just a fancy way of calling the `test` command. The statement above is equivalent to:

```bash
if test "$1" == "" ; then echo "something"; fi
```

Basically everything between the brackets gets passed as an argument to `test`. You can call the test command in your terminal like any other program on your system. Executing something like `which test` will show you where the command can eb found on your system.