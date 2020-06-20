---
title: "For Loops in the Shell"
date: 2020-05-08T09:11:57+02:00
draft: true
---

## For loops
A `for` loop can iterate over pretty much everything. A for loop iterating over every character in a string looks like this. 
``` bash
for i in "hello world"; do
  echo $i
done
```

If you want to iterate over every file or directory ibn the currect directory replace the string with a `*`. This will print every file and directory in the current directory.

``` bash
for file in *; do
  echo $i
done
```

Both examples are POSIX ... This enas they shoudl work in all POSIX compliant shells. In a bash shell it is possible to use an `arithmetic for loop`. Such a loop can use an arithmetic expression as a contion for the loop. This is how for loops are often used in other progrtamming languages. An arithmetic for loop in bash looks like this:

```bash
for (( i=1; i < 10; i++ )); do
    echo "$i"
done
```

In a POSIX shell such an expression is not possible. If you want to loop untill a certain contirion is met, the `while` loop is needed.



## While loops