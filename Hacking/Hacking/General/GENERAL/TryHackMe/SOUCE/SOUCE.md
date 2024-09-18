  
We start the VPN
![[/Untitled 54.png|Untitled 54.png]]
![[/Untitled 1 10.png|Untitled 1 10.png]]
We start the machine
![[/Untitled 2 11.png|Untitled 2 11.png]]
We check the connectivity
  
![[/Untitled 3 11.png|Untitled 3 11.png]]
We perform a scan with Nmap to see the open ports and the services running
sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.174.253 > escaneo.txt
![[/Untitled 4 10.png|Untitled 4 10.png]]
We find ports 22 (SSH) and 10000 open, with the latter running the HTTP service, so we will scan the web using WhatWeb
![[/Untitled 5 10.png|Untitled 5 10.png]]
We find that the server is MiniServ, and its version is 1.890
We are going to visit the website
![[/Untitled 6 10.png|Untitled 6 10.png]]
We find a Webmin login page
NOTE:
MiniServ is a lightweight web server used as the backend for a system administration application called Webmin. Webmin is a web-based system administration tool for Unix and Linux systems. MiniServ provides the HTTP server functionality that allows administrators to access and manage their systems through a web interface.