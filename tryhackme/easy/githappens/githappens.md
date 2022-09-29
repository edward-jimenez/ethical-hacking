## Link to tryhackme room

[Git Happens](https://tryhackme.com/room/githappens)

## Save target IP to a variable (NOTE: Your IP will be different)

`export target="10.10.34.17"`

## Scan for open ports

nmap options explained:    

- -Pn: Do not ping host(Assume it is alive)  
- -p-: Scan all TCP ports(1-65535)  
- -sS: SYN scan; Do not complete the TCP 3-Way handshake, only send SYN packet  
- -T4: Agressive scan  

```
sudo nmap -Pn -p- -sS -T4 ${target}                                                                                                                                                                    1 ⚙
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-28 22:25 EDT
Stats: 0:01:17 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 87.25% done; ETC: 22:26 (0:00:11 remaining)
Nmap scan report for 10.10.34.17
Host is up (0.088s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 88.22 seconds
``` 

## Enumerate open ports

nmap options explained:  

- -Pn: Do not ping host(Assume it is alive)  
- -p:  List of ports to scan  
- -sC: Run **C**ommand scripts against the selected ports  
- -sV: Display service **V**ersion  
- -oA: Save scan results in **A**ll formats(nmap, xml, greppable)  

```
nmap -Pn -p80 -sC -sV ${target} -oA ${target}                                                                                                                                                          1 ⚙
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-28 22:26 EDT
Nmap scan report for 10.10.34.17
Host is up (0.088s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Super Awesome Site!
| http-git: 
|   10.10.34.17:80/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.69 seconds
```

## Vist the site

I visited the site and was presented with a login screen.  I viewed the source code and noticed obfuscated javascript code. I submitted the code to an online javascript de-obfuscator, but it didn't clear things up as to what the code was doing.  I decided to stick with the /.git directory found by nmap instead of chasing this rabbit hole.  

## Download git repo  

```
wget -r http://${target}/.git/  
```

## Check the git logs

```
git log                                                                                                                                                                                                1 ⚙
commit d0b3578a628889f38c0affb1b75457146a4678e5 (HEAD -> master, tag: v1.0)
Author: Adam Bertrand <hydragyrum@gmail.com>
Date:   Thu Jul 23 22:22:16 2020 +0000

    Update .gitlab-ci.yml

commit 77aab78e2624ec9400f9ed3f43a6f0c942eeb82d
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Fri Jul 24 00:21:25 2020 +0200

    add gitlab-ci config to build docker file.

commit 2eb93ac3534155069a8ef59cb25b9c1971d5d199
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Fri Jul 24 00:08:38 2020 +0200

    setup dockerfile and setup defaults.

commit d6df4000639981d032f628af2b4d03b8eff31213
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:42:30 2020 +0200

    Make sure the css is standard-ish!

commit d954a99b96ff11c37a558a5d93ce52d0f3702a7d
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:41:12 2020 +0200

    re-obfuscating the code to be really secure!

commit bc8054d9d95854d278359a432b6d97c27e24061d
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:37:32 2020 +0200

    Security says obfuscation isn't enough.
    
    They want me to use something called 'SHA-512'

commit e56eaa8e29b589976f33d76bc58a0c4dfb9315b1
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:25:52 2020 +0200

    Obfuscated the source code.
    
    Hopefully security will be happy!

commit 395e087334d613d5e423cdf8f7be27196a360459
Author: Hydragyrum <hydragyrum@gmail.com>
Date:   Thu Jul 23 23:17:43 2020 +0200

    Made the login page, boss!

commit 2f423697bf81fe5956684f66fb6fc6596a1903cc
Author: Adam Bertrand <hydragyrum@gmail.com>
Date:   Mon Jul 20 20:46:28 2020 +0000

    Initial commit
```

### diff the git logs

I compared each commit with the previous commit starting with the initial commit.  
  
`git diff 2f423697bf81fe5956684f66fb6fc6596a1903cc 395e087334d613d5e423cdf8f7be27196a360459`  

The output from the above command reveals css style rules were added.  I saw a base64 encoded image file and decided to investigate.  I converted the base64 string to an image file and ran exiftool and binwalk against it.  Unfortunately, it was a time waster as there was nothing out of the ordinary  in that file.  

Next, I compared the next commit in line with the previous commit.  

`git diff 395e087334d613d5e423cdf8f7be27196a360459 e56eaa8e29b589976f33d76bc58a0c4dfb9315b1`  

This was definitely interesting as this gives us a vector for bypassing the login screen:  

```
-      function login() {
-        let form = document.getElementById("login-form");
-        console.log(form.elements);
-        let username = form.elements["username"].value;
-        let password = form.elements["password"].value;
-        if (
-          username === "admin" &&
-          password === "**REDACTED**"
-        ) {
-          **document.cookie = "login=1";**
-          window.location.href = "/dashboard.html";
-        } else {
-          document.getElementById("error").innerHTML =
-            "INVALID USERNAME OR PASSWORD!";
-        }
-      }
```

Looks like we can just create a cookie named "login" with a value of "1" and visit /dashboard.html to gain access.  Sure enough, it worked!  The dashboard gives us a clue for submitting the flag.   
