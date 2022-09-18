## Link to tryhackme room

[Intermediate Nmap](https://tryhackme.com/room/intermediatenmap)

## Target IP (This IP will be different on your machine)

┌──(edward㉿repoman)-[~]

└─$ export target="10.10.12.222"

## Scan for open ports

nmap options explained:

- -Pn: Do not ping host(Assume it is alive)
- -p-: Scan all TCP ports(1-65535)
- -sS: SYN scan; Do not complete the TCP 3-Way handshake, only send SYN packet
- -T4: Agressive scan

┌──(edward㉿repoman)-[~]
└─$ sudo nmap -Pn -p- -sS -T4 ${target}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-17 23:43 EDT
Nmap scan report for 10.10.12.222
Host is up (0.084s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
**22/tcp    open  ssh**
**2222/tcp  open  EtherNetIP-1**
**31337/tcp open  Elite**

Nmap done: 1 IP address (1 host up) scanned in 17.40 seconds

## Enumerate the discovered open ports

nmap options explained:

- -Pn: Do not ping host(Assume it is alive)
- -p:  List of ports to scan
- -sC: Run **C**ommand scripts against the selected ports
- -sV: Display service **V**ersion
- -oA: Save scan results in **A**ll formats(nmap, xml, greppable)

┌──(edward㉿repoman)-[~]
└─$ nmap -Pn -p22,2222,31337 -sC -sV ${target} -oA ${target}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-17 23:45 EDT
Nmap scan report for 10.10.12.222
Host is up (0.084s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7d:dc:eb:90:e4:af:33:d9:9f:0b:21:9a:fc:d5:77:f2 (RSA)
|   256 83:a7:4a:61:ef:93:a3:57:1a:57:38:5c:48:2a:eb:16 (ECDSA)
|_  256 30:bf:ef:94:08:86:07:00:f7:fc:df:e8:ed:fe:07:af (ED25519)
2222/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ce:e3:85:21:c1:8d:7e:47:dd:f8:60:91:f4:91:80:b3 (RSA)
|   256 85:e9:6e:dd:6e:37:2b:2c:cb:a1:1e:1b:6e:61:b2:04 (ECDSA)
|_  256 d9:d4:18:75:be:8c:b1:8c:42:90:04:31:33:94:34:fc (ED25519)
31337/tcp open  Elite?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     `REDACTED`
|_    `REDACTED`

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.42 seconds


## Take a close at the nmap output.  The ssh login credentials is somewhere inside that mess. Once you found it, log in

┌──(edward㉿repoman)-[~]
└─$ ssh -l **REDACTED** ${target}
The authenticity of host '10.10.12.222 (10.10.12.222)' can't be established.
ED25519 key fingerprint is SHA256:8VuYGtc5lO2sXK+MVsdbgQV9nF+EVHf8wJcrMAEWg10.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.12.222' (ED25519) to the list of known hosts.
**REDACTED**@10.10.12.222's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.13.0-1014-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ whoami
**REDACTED**


## Find the flag

find options explained

- /: Search starting from the root filesystem
- -type f: Search for files (We don't care for directories at this time)
- iname: Search for a file with this name, case insensitive
- 2> /dev/null: Don't display stderr to the screen

$ find / -type f -iname *flag* 2> /dev/null
/sys/devices/pnp0/00:04/tty/ttyS0/flags
/sys/devices/platform/serial8250/tty/ttyS2/flags
/sys/devices/platform/serial8250/tty/ttyS3/flags
/sys/devices/platform/serial8250/tty/ttyS1/flags
/sys/devices/virtual/net/eth0/flags
/sys/devices/virtual/net/lo/flags
/sys/module/scsi_mod/parameters/default_dev_flags
/proc/sys/kernel/acpi_video_flags
/proc/sys/net/ipv4/fib_notify_on_flag_change
/proc/sys/net/ipv6/fib_notify_on_flag_change
/proc/kpageflags
/usr/lib/x86_64-linux-gnu/perl/5.30.0/bits/waitflags.ph
/usr/lib/x86_64-linux-gnu/perl/5.30.0/bits/ss_flags.ph
/usr/bin/dpkg-buildflags
/usr/include/x86_64-linux-gnu/asm/processor-flags.h
/usr/include/x86_64-linux-gnu/bits/termios-c_lflag.h
/usr/include/x86_64-linux-gnu/bits/termios-c_oflag.h
/usr/include/x86_64-linux-gnu/bits/ss_flags.h
/usr/include/x86_64-linux-gnu/bits/termios-c_cflag.h
/usr/include/x86_64-linux-gnu/bits/waitflags.h
/usr/include/x86_64-linux-gnu/bits/mman-map-flags-generic.h
/usr/include/x86_64-linux-gnu/bits/termios-c_iflag.h
/usr/include/linux/kernel-page-flags.h
/usr/include/linux/tty_flags.h
/usr/share/perl5/Dpkg/BuildFlags.pm
/usr/share/dpkg/buildflags.mk
**REDACTED**

## Display flag output

$ cat **REDACTED**
flag{**REDACTED**}
