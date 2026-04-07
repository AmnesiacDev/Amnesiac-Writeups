
# OverTheWire Bandit Writeups 33 Levels

Here I will show and explain how to solve each level in detail
## Table of contents

0. [Level 0](#Level-0)

1. [Level 1](#Level-1)

2. [Level 2](#Level-2)

3. [Level 3](#Level-3)

## Level 0

Just view contents of file

```bash
 ~$ ls
 readme
 ~$ cat readme
```


## Level 1

Use explicit relative path

```bash
 ~$ ls
 -
 ~$ cat ./-
```

## level 2

Use quotation marks around file name
```bash
 ~$ ls
 --spaces in this filename--
 ~$ cat ./"--spaces in this filename--"
```
## level 3

Use **ls -a** or **ls -la**

l -> list in directory in the list format

a -> show all items
```bash
 ~$ ls
 inhere
 ~$ ls -la inhere
 drwxr-xr-x 2 root    root    4096 Apr  3 15:18 .
 drwxr-xr-x 3 root    root    4096 Apr  3 15:18 ..
 -rw-r----- 1 bandit4 bandit3   33 Apr  3 15:18 ...Hiding-From-You
 ~$ cat inhere/...Hiding-From-You
```

## level 4

Use wildcard operator * to show all files and pipe operator | to feed commands directly

strings command show only human readable text

```bash
 ~$ ls
 inhere
 ~$ ls inhere
 -file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09
 ~$ cat inhere/* | strings
```

## level 5

Here we know the file is 1033 Bytes, human-readable, and not executable

```bash
 ~$ ls inhere
 <maybehere00-19>
 ~$ find ./inhere -size 1033c ! -executable
 ./inhere/maybehere07/.file2
 ~$ cat ./inhere/maybehere07/.file2
```

## level 6

Here we know the file has the following properties

Owned by user bandit7
Owned by group bandit6
```bash

```
