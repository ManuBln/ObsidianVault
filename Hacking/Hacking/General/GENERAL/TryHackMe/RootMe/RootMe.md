  
Iniciamos la VPN
![[/Untitled 48.png|Untitled 48.png]]
![[/Untitled 1 7.png|Untitled 1 7.png]]
![[/Untitled 2 8.png|Untitled 2 8.png]]
Iniciamos la maquina
![[/Untitled 3 8.png|Untitled 3 8.png]]
Confirmamos la conexión con la ip victima
![[/Untitled 4 7.png|Untitled 4 7.png]]
  
Comenzamos haciendo un escaneo para ver los puertos y servicios que están corriendo
![[/Untitled 5 7.png|Untitled 5 7.png]]
Vemos que tenemos dos puertos abiertos, el 80 (http) y el 22 (ssh), vamos respondiendo las preguntas de la maquina
![[/Untitled 6 7.png|Untitled 6 7.png]]
Hacemos uso de la herramienta whatweb para obtener las tecnologías usadas en el servicio web
whatweb -v http://10.10.160.80/
![[/Untitled 7 7.png|Untitled 7 7.png]]
![[/Untitled 8 7.png|Untitled 8 7.png]]
Obtenemos información importante como la versión del servidor apache y el sistema operativo donde esta corriendo el servidor web (LINUX).
Respondemos las preguntas de la maquina
![[/Untitled 9 7.png|Untitled 9 7.png]]
Nos dirigimos al navegador e introducimos la ip victima
![[/Untitled 10 7.png|Untitled 10 7.png]]
Ctrl u
![[/Untitled 11 7.png|Untitled 11 7.png]]
Al no encontrar nada ni en la web ni en su código intentaremos hacerle fuzzing a los directorios y extensiones
ffuf -w directory-list-2.3-medium.txt:/D -w /home/user/wordlists/extensions.txt:/E -u [http://10.10.160.80/](http://10.10.160.80/)[DIRFUZZ](http://example.com/DIRFUZZ) -e E -c
otra manera →
ffuf -w directory-list-lowercase-2.3-big.txt -u [http://10.10.56.243/](http://10.10.160.80/)[FUZZ](http://example.com/FUZZ) -e php,html,js,css -c
![[/Untitled 12 6.png|Untitled 12 6.png]]
![[/Untitled 13 6.png|Untitled 13 6.png]]
![[/Untitled 14 5.png|Untitled 14 5.png]]
![[/Untitled 15 4.png|Untitled 15 4.png]]
Encontramos estas direcciones, vamos a ver que contienen
  
Las direcciones mas interesantes serian las siguientes
/panel
![[/Untitled 16 4.png|Untitled 16 4.png]]
/uploads
![[/Untitled 17 4.png|Untitled 17 4.png]]
A primera vista creo que podemos subir una reverse shell por la dirección panel y después ejecutarla en el directorio uploads
  
Probaremos con una reverse shell en php
![[/Untitled 18 4.png|Untitled 18 4.png]]
Creamos el archivo
![[/Untitled 19 4.png|Untitled 19 4.png]]
Subimos el archivo
![[/Untitled 20 4.png|Untitled 20 4.png]]
No nos deja subir un .php, probaremos con un .php5
![[/Untitled 21 4.png|Untitled 21 4.png]]
El .php5 si lo ha aceptado
![[/Untitled 22 4.png|Untitled 22 4.png]]
Nos ponemos en escucha en el puerto 5555 y nos dirigimos al directorio uploads a ejecutar la shell
  
![[/Untitled 23 4.png|Untitled 23 4.png]]
Ejecutamos la reverse shell pinchando en el archivo desde el directorio o indicandolo en la url
Tras ejecutar la reverse tenemos el acceso
![[/Untitled 24 4.png|Untitled 24 4.png]]
  
Contestamos las preguntas de la máquina
![[/Untitled 25 4.png|Untitled 25 4.png]]
  
Ya que tenemos el nombre del archivo donde se encuentra la flag del usuario (user.txt), buscaremos donde se encuentra dicho archivo con find
![[/Untitled 26 4.png|Untitled 26 4.png]]
![[/Untitled 27 4.png|Untitled 27 4.png]]
Le hacemos un cat para ver el contenido de la flag y entregarla
![[/Untitled 28 4.png|Untitled 28 4.png]]
![[/Untitled 29 4.png|Untitled 29 4.png]]
Comenzamos con la escalada de privilegios
![[/Untitled 30 4.png|Untitled 30 4.png]]
Tenemos que buscar un archivo con permisos de SUID para escalar privilegios
Ademas si hacemos sudo -l nos aparece eso
  
![[/Untitled 31 4.png|Untitled 31 4.png]]
Comenzamos haciendo una búsqueda de binarios con los permisos de SUID
[https://diegoaltf4.com/privesc01/](https://diegoaltf4.com/privesc01/)
find / -perm -4000 2>/dev/null
![[/Untitled 32 4.png|Untitled 32 4.png]]
Encontramos que python tiene permisos SUID
Nos dirigimos a GTFOBins→[https://gtfobins.github.io/#python](https://gtfobins.github.io/#python)
![[/Untitled 33 4.png|Untitled 33 4.png]]
![[/Untitled 34 4.png|Untitled 34 4.png]]
![[/Untitled 35 4.png|Untitled 35 4.png]]
Probamos accediendo a python mediante /usr/bin/python
![[/Untitled 36 3.png|Untitled 36 3.png]]
Ya escalamos los privilegio, ahora a por la flag de root
![[/Untitled 37 3.png|Untitled 37 3.png]]
![[/Untitled 38 3.png|Untitled 38 3.png]]