
# OverTheWire Bandit Writeups 33 Levels

Here I will show and explain how to solve each level in detail
## Table of contents

|  Levels 0 - 10  | Writeup  |  Levels 11 - 21  | Writeup |
| ------------- |------------- | ------------- |------------- |
|  0 -> 1  |  [Level 0](#Level-0)  |  11 -> 12  |  [Level 11](#Level-11)  |
|  1 -> 2  |  [Level 1](#Level-1)  |  12 -> 13  |  [Level 12](#Level-12)  |
|  2 -> 3  |  [Level 2](#Level-2)  |  13 -> 14  |  [Level 13](#Level-13)  |
|  3 -> 4  |  [Level 3](#Level-3)  |  14 -> 15  |  [Level 14](#Level-14)  |
|  4 -> 5  |  [Level 4](#Level-4)  |  15 -> 16  |  [Level 15](#Level-15)  |
|  5 -> 6  |  [Level 5](#Level-5)  |  16 -> 17  |  [Level 16](#Level-16)  |
|  6 -> 7  |  [Level 6](#Level-6)  |  17 -> 18  |  [Level 17](#Level-17)  |
|  7 -> 8  |  [Level 7](#Level-7)  |  18 -> 19  |  [Level 18](#Level-18)  |
|  8 -> 9  |  [Level 8](#Level-8)  |  19 -> 20  |  [Level 19](#Level-19)  |
|  9 -> 10 |  [Level 9](#Level-9)  |  20 -> 21  |  [Level 20](#Level-20)  |
|  10 -> 11  |  [Level 10](#Level-10)  |  21 -> 22  |  [Level 21](#Level-21)  |



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

Owned by user bandit7, group bandit6 and size is 33 bytes

```bash
~$ find / -user "bandit7" -group "bandit6" -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
~$ cat /var/lib/dpkg/info/bandit7.password
```
The reason why we use 2>/dev/null is to ignore any error messages such as "permission denied"

## level 7

The password is stored next to the word "millionth" in the data.txt file

```bash
~$ grep "millionth" data.txt
millionth     **password**
```

## level 8

The password is the only line that occurs only once

```bash
~$ sort data.txt | uniq -u
```
The -u option in uniq command is used to show only unique lines, you can also use -c which show how many occurences each line has

## level 9

Password is in few human-readable lines that's preceeded by several '='
```bash
~$ strings data.txt | grep '='
========== password
={M\
========== is
=Dvq
=n/N
========== **password**
zX]%=
]\{=
```
## level 10

The password is base64 encoded in the data.txt file
```bash
~$ cat data.txt | base64 -d
```

## level 11

Password is rotated by 13 characters

```bash
~$ cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"
```
## level 12

This one is a bit repetitive, the password has been encoded multiple times and then hexdumped into data.txt

```bash
# create temporary directory
~$ mktemp -d
~$ cp data.txt /tmp/tmp.6A1TPbbqlD
~$ cd /tmp/tmp.6A1TPbbqlD

# decode hexdump
~$ xxd -r data.txt > recovered.bin
~$ file recovered.bin
gzip
~$ mv recovered.bin stage2.gz
~$ gzip -d stage2.gz
~$ file stage2
bzip2
~$ mv stage2 stage3.bz2
~$ bzip -d stage3.bz2
~$ file stage3
POSIX tar archive
~$ mv stage3 stage4.tar
~$ tar -xf stage4.tar
~$ file data5.bin
repeat
.
.
.
.
~$ file recovered
ASCII text
```
## level 13

Password is stored in /etc/bandit_pass/bandit14 but can only accessed by bandit14, however we have sshkey

```bash
~$ ls
sshkey.private
~$ cat sshkey.private
# Copy the text in sshkey.private
~$ exit
~kali@kali $ nano sshkey.private
# Paste text
~kali@kali $ chmod 600 sshkey.private
~kali@kali $ ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
~$ cat /etc/bandit_pass/bandit14
```

## level 14

Here we simply need to send the current level's password to port 30000 on localhost

```bash
~$ echo "password" | nc 127.0.0.1 30000
Correct!
```

## level 15

Here we need to send current level password to localhost on port 30001 Using TLS/SSL encryption
```bash
~$ openssl s_client -connect 127.0.0.1:30001
# Paste current level password
Correct! 
```

## level 16

Here we have a wide range of ports (31000 -> 32000) so we need to do some bash coding

```bash
~$ pass='kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx'

for port in {31000..32000}; do
    response=$(echo "$pass" |
        openssl s_client -connect 127.0.0.1:"$port" -quiet 2>/dev/null)

    if [ -n "$response" ] && [ "$response" != "$pass" ]; then
        echo "Port: $port $response"
    fi
done

Port: 31790 Correct!
# We get ssh certificate 
```

## level 17

```bash
```

## level 18

```bash
```

## level 19

```bash
```
## level 20

```bash
```

## level 21

```bash
```

## level 22

```bash
```

## level 23

```bash
```

## level 24

```bash
```

## level 25

```bash
```

## level 26

```bash
```

## level 27

```bash
```

## level 28

```bash
```

## level 29

```bash
```

## level 30

```bash
```

## level 31

```bash
```

## level 32

```bash
```

# THE END
