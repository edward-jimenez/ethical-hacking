## Link to tryhackme room

[<room>](https://tryhackme.com/room/<room>)

## Save target IP to a variable (NOTE: Your IP will be different)

`export target="<target ip>"`

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
