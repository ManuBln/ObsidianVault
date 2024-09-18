
Comenzamos con un escaneo de puertos y servicios mediante la herrmienta nmap

![[Pasted image 20240917040919.png]]

Al ver el puerto 80 abierto decido escanear la web mediante la herrmaienta whatweb.

![[Pasted image 20240917040715.png]]

Visitamos la web
![[Pasted image 20240917040949.png]]
![[Pasted image 20240917040958.png]]

Al no encontrar nada decido fijarme en el header de la respuesta del escaneo realizado por whatweb y buscar el contenido en internet.
![[Pasted image 20240917041041.png]]
![[Pasted image 20240917040749.png]]
Al ver que es una raspberry y el puerto 22 esta abierto decido comprobar las credenciales por defecto de la raspberry para acceder mediante ssh
![[Pasted image 20240916061449.png]]

`pi`
`raspberry`
`ssh pi@10.10.10.48`
![[Pasted image 20240917041145.png]]

Ya estamos dentro 

User flag
![[Pasted image 20240917041207.png]]
Escalada de privilegios 
`sudo -l`
![[Pasted image 20240917041315.png]]
Al no tener contraseña alguna podemos escalar privilegios directamente mediante sudo su
![[Pasted image 20240917041412.png]]
Ya tenemos acceso como root por lo que vamos a por la root flag
Root flag
![[Pasted image 20240917041502.png]]
Vaya al parecer la flag esta en un usb conectado al equipo, por lo que decido lanzar `df -h` para localizar la direccion de dicho usb
Nota:El comando `df -h` se utiliza  para mostrar el uso del espacio en disco en un formato legible.
![[Pasted image 20240917041556.png]]
Vemos la dirección del usb que buscamos, vamos a mostrar su contenido
`strings /dev/sdb `
![[Pasted image 20240917041637.png]]

Si en vez de usar strings para mostrar el contenido nos dirigimos al directorio no encontraremos nada mas que el siguiente mensaje
![[Pasted image 20240917041818.png]]
Por lo que necesitaremos una herramienta para recuperar contenido eliminado.
