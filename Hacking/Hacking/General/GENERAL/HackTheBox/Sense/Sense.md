We start with a scan of ports and services using the nmap tool
```JavaScript
sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.60 > escaneo.txt
```
![[/Untitled 38.png|Untitled 38.png]]
We can see that ports 80 and 443 are open and running a lighttpd 1.4.35 http service.
We use the whatweb tool to scan the website before visiting it.
```JavaScript
whatweb -v https://10.10.10.60/
whatweb -v https://10.10.10.60:443/
```
![[/Untitled 1 4.png|Untitled 1 4.png]]
![[/Untitled 2 5.png|Untitled 2 5.png]]
Visit the website [https://10.10.10.60/](https://10.10.10.60/)
![[/Untitled 3 5.png|Untitled 3 5.png]]
We see that we only have one pfSense login.
We try to see if we detect any directory fuzzing using the ffuf tool.
```JavaScript
ffuf -w Desktop/diccionario/Directorios/directory-list-2.3-medium.txt -u https://10.10.10.60/FUZZ -e .txt,.php,.html,.js -c
```
We found the following
  
What strikes me most is this .txt file
![[/Untitled 4 4.png|Untitled 4 4.png]]
[https://10.10.10.60/system-users.txt](https://10.10.10.60/system-users.txt)
![[/Untitled 5 4.png|Untitled 5 4.png]]
It appears to be a credentials!!!!!!
User→rohit
We do a search for the default pfsense password
![[/Untitled 6 4.png|Untitled 6 4.png]]
We do the test
User→rohit
🔑→pfsense
![[/Untitled 7 4.png|Untitled 7 4.png]]
We are in!!!:)
![[/Untitled 8 4.png|Untitled 8 4.png]]
We can see that the version of PfSense is as follows
![[/Untitled 9 4.png|Untitled 9 4.png]]
We search for vulnerabilities or CVEs of the corresponding version.
We found the following
[https://www.exploit-db.com/exploits/39709](https://www.exploit-db.com/exploits/39709)
Thanks to this CVE we found the following exploit
https://github.com/lawrencevanlaere/pfsense-code-exec
We make the necessary modifications
![[/Untitled 10 4.png|Untitled 10 4.png]]
We listen on the port indicated in the exploit and launch it
![[/Untitled 11 4.png|Untitled 11 4.png]]
User flag
![[/Untitled 12 3.png|Untitled 12 3.png]]
Root flag
![[/Untitled 13 3.png|Untitled 13 3.png]]