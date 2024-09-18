  
We start by performing a scan with Nmap to see which ports are open and what services are running
sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.11.20 > escaneo.txt
![[Untitled 3.png]]
![[Untitled 1.png]]
We found ports 22 (SSH) and 80 (HTTP) open, so we proceed to scan the website using WhatWeb
![[Untitled 22.png]]
![[Untitled 32.png]]
Now, to visit the website, we need to modify the /etc/hosts file so that the address editorial.htb points to the IP address
sudo nano /etc/hosts
![[Untitled 4.png]]
On the website, we find the home page
![[Untitled 5.png]]
/upload
![[Untitled 6.png]]
And /about
![[Untitled 7.png]]
We will focus on the /upload view, as it has a URL input and allows file uploads, meaning it opens several options to be vulnerable.
Let's check the responses of the requests using Burp Suite
![[Untitled 8.png]]
  
FoxyProxy is an extension used to raise up a proxy between the host and the target. It can be used to intercept the traffic in the port 8080 and use Burpsuite as interceptor.
  
In this case we’ll be using “Open Browser” Burpsuite feature.
  
Make the request to the web page and navigate to “HTTP History” tab in Burpsuite.
  
In the input, we enter something random and upload a txt file with some text, then click preview
![[Untitled 9.png]]
  
El placeholder del input del usuario incluye un texto diciendo que indiquemos una URL.
Podemos intentar incluir una URL que apunte a nuestro host para comprobar si tenemos un SSRF. Este caso de SSRF consistiria en que el backend donde enviamos la URL alojado en el serivdor, usara nuestra URL como parametro para hacer otra peticion en el lado del serivdor. De esta manera tendriamos control sobre las peticiones que hace el servidor.
  
Note:Server-Side Request Forgery (SSRF) is a type of web application vulnerability that allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain of the attacker's choosing.
  
Ahora vamos a levantar un servidor HTTP con python, con el objetivo de comprobar si el servidor nos realiza alguna peticion cuando le pasamos nuestra IP como URL.
  
  
```JavaScript
sudo python3 -m http.server 8081
```
  
A continuacion le incluimos a la request la url http://<OwnHTB_IP>:8081.
  
Mandamos la request y observamos que recibimos una perticion GET que proviene del servidor.
  
Lo hemos conseguido! Sabemos que es vulnerable a SSRF.
  
A continuacion incluimos la URL [http://127.0.0.1/](http://127.0.0.1/) para ver como se comporta el servidor al hacerse una peticion a el mismo.
  
![[Untitled 10.png]]
It seems to be the same, so we will use Intruder to brute force the port
![[Untitled 11.png]]
Payload Configuration
![[Untitled 12.png]]
Start attack
![[Untitled 13.png]]
The Intruder was taking too long, so I checked the response.
PORT: 5000
![[Untitled 14.png]]
![[Untitled 15.png]]
wget [http://editorial.htb](http://editorial.htb/static/uploads/9bdd2b6b-e1b6-4821-a4f9-e603220f497b)/static/uploads/1c4f2f18-a4cd-4069-a1fa-14864efd5de2
![[Untitled 16.png]]
cat 765abd5b-3dc5-46cf-a898-c07c20c5f33e | jq
![[Untitled 17.png]]
![[Untitled 18.png]]
  
Nos resulta interesante este endpoint "endpoint": "/api/latest/metadata/messages/authors",
![[Untitled 19.png]]
Intentamos acceder mediante la petición
[http://editorial.htb:5000/api/latest/metadata/messages/authors](http://127.0.0.1:5000/api/latest/metadata/messages/authors)
![[Untitled 20.png]]
Username: dev
Password: dev080217_devAPI!@
Ya que no tenemos ningun panel de login por ningun sitio de la web intentaremos acceder mediante ssh con estas credenciales
![[Untitled 21.png]]
Encontramos la flag de usuario
![[Untitled 22.png]]
En el directorio de apps encontramos un .git
![[Untitled 23.png]]
Accedemos
![[Untitled 24.png]]
Nos dirigimos logs y encontramos
![[Untitled 25.png]]
Hacemos un cat a HEAD y encontramos los commits, de los cuales destacamos el siguiente
![[Untitled 26.png]]
Accedemos al commit con git show b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae
![[Untitled 27.png]]
Cambiamos de usuario con las credenciales encontradas.
Username: prod
Password: 080217_Producti0n_2023!@
![[Untitled 28.png]]
Intentamos sudo -l, ya que con el usuario dev no pudimos acceder
![[Untitled 29.png]]
USER prod
![[Untitled 30.png]]
Hacemos un cat al binario que se ejecuta con python
![[Untitled 31.png]]
Hacemos una busqueda rapida y encontramos lo siguiente
![[Untitled 32.png]]
![[Untitled 33.png]]
Parece ser que podemos lanzar un comando cualquiera de la siguiente manera
`**<gitpython::clone> 'ext::sh -c COMANDO'**`
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c cat% /root/root.txt% >% /tmp/root’
![[Untitled 34.png]]
Obtenemos la flag de root
![[Untitled 35.png]]