  
We start by launching a scan, to see the ports and services running, with nmap
```JavaScript
sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.10.79 > escaneo.txt
```
![[Pasted image 20240802040632.png]]
We observe port 22 open running an ssh service, port 80 with an http service and port 443 with an https service.
Seeing that we have no way to access via ssh (we do not have credentials) we are going to scan the web using the whatweb tool
`whatweb http://10.10.10.79/
`whatweb https://10.10.10.79/
![[Pasted image 20240802041032.png]]
We see that both cases give the same information:
- An Apache server with version 2.2.22

Also the ssh version is OpenSSH 5.9p1.

Let's visit the websites

- hhtp
![[Pasted image 20240802042927.png]]
- https
![[Pasted image 20240802042945.png]]
We can see that they are the same web, we could deduce it with the previous scan.
Let's take a look at the metadata of both images
`exiftool omgHTTP.jpg`
![[Pasted image 20240802043305.png]]
`exiftool omgHTTPS.jpg`
![[Pasted image 20240802043344.png]]
After seeing nothing interesting, we proceed to a directory scan using the Wfuzz tool. 
`wfuzz -c --hc=404 -t 200 -w ../Editorial/../../../diccionario/Directorios/directory-list-2.3-medium.txt http://10.10.10.79/FUZZ`

![[Pasted image 20240802044022.png]]
The directory that most attracts my attention is /dev
![[Pasted image 20240802044138.png]]
- hype_key
![[Pasted image 20240802044333.png]]
- notes.txt
![[Pasted image 20240802044342.png]]

We can see that the content of hupe_key are bytes, we pass it through a bytes to string converter and we obtain what seems to be a private key of a user to access through ssh
- id_rsa 
![[Pasted image 20240802044848.png]]
 We tried to access through the root user in case it was the root key but I don't think so.

`chmod 600 id_rsa
`ssh -i id_rsa root@10.10.10.79

![[Pasted image 20240802045322.png]]

We see that it asks us for a passphrase, let's say it is like the password of the private key, so we will need to find it out.
Let's see what we get back this second scan with the nmap tool and one of its scripts

- Script vuln  Search for vulnerabilities

![[Pasted image 20240802045659.png]]
![[Pasted image 20240802045741.png]]
![[Pasted image 20240802045807.png]]
Podemos observar varias vulneravilidades pero la que mas interesante me parece es la siguiente
![[Pasted image 20240802045854.png]]

Ya que haciendo una busqueda en internet de que se trataba di con lo siguiente:

Heartbleed es un agujero de seguridad de software en la biblioteca de código abierto OpenSSL, solo vulnerable en su versión 1.0.1f, que permite a un atacante leer la memoria de un servidor o un cliente, permitiéndole por ejemplo, conseguir las claves privadas SSL de un servidor​.

![[Pasted image 20240802191404.png]]
https://github.com/mpgn/heartbleed-PoC
`git clone https://github.com/mpgn/heartbleed-PoC`
`chmod +x heartbleed-exploit.py`
`python2 heartbleed-exploit.py 10.10.10.79`
![[Pasted image 20240802191618.png]]
Podemos observar que nos devuelve mas informacion de la necesaria, vamos verla en el txt que se genera despues de ejecutar el exploit 

![[Pasted image 20240802191728.png]]
`cat out.txt`
A primera vita podemos observar lo siguiente 
![[Pasted image 20240802191853.png]]

aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==

Parece ser base 64 y ademas parce que es el resultado del codificador de la web que encontramos en el fuzzing
- /encode

![[Pasted image 20240802192151.png]]

Vamos a comprobarlo en el decodificador para ver el resultado 
- /decode
![[Pasted image 20240802192210.png]]
Ahora vamos a pasarlo por un decodificador de base 64:
![[Pasted image 20240802192331.png]]
heartbleedbelievethehype

Esto a primera vista puede ser la contraseña de la clave privada, vamos a comprobarlo de la siguiente manera
 En la siguiente imagen no pondremos la contraseña, kle daremos a enter, y observamos que nos pide la contraseña del usuario root


Ahora pondremos en las 2 primeras ocasiones dos contraseñas aleatorias y obsevamos que nos sigue piediendo la contraseña de la clave, y en la tercera opcion le indicamos la contraseña decosificada de base 64 y vemos que ya nos pide la contraseña del usuario que le hemos puesto.


Podemos suponder que es la contraseña de la clave pero no es el usuario correcto, vamos a ver las vulneravilidades de ssh para ver si podemos aprovecharnos de alguna.


`searchsploit SSH 5.9p1`
![[Pasted image 20240802201227.png]]
Usaremos la señalada ya que es de la version 7.7 para atras.(nuestra version es la 5.9)
`searchsploit -m linux/remote/45939.py`

Vemos el contenido del exploit para ver el funcionamiento y encontramos lo siguiente
![[Pasted image 20240802201630.png]]
Lanzamos el esploit  y obtenemos un error
`python2 45939.py 10.10.10.79 username`
Redirgirtemos al error a null para ver si podemos ejecutar el esploit
`python2 45939.py 10.10.10.79 username 2>/dev/null`
![[Pasted image 20240802202619.png]]
Vemos que funciona y el usuario indicado no existe, probemos con root
![[Pasted image 20240802202854.png]]
Vemos que la enumeracion funciona correctamente, suponemos que el usuario que podemos verificar es hype por lo siguiente.
![[Pasted image 20240802202732.png]]
La clave encontrada es de hype se supone
Probemoslo

![[Pasted image 20240802202902.png]]
Parece que ya lo tenemos, veamos si funciona el acceso ssh con las siguientes credenciales

User: hype
Key: id_rsa
Passphrase: heartbleedbelievethehype
`ssh -i id_rsa hype@10.10.10.79`
![[Pasted image 20240808210630.png]]![[Pasted image 20240808210700.png]]
openssl rsa -in < > -out < >
![[Pasted image 20240808210832.png]]