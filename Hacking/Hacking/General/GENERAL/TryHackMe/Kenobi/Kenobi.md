  
Iniciamos la VPN
![[/Untitled 49.png|Untitled 49.png]]
![[/Untitled 1 8.png|Untitled 1 8.png]]
Iniciamos la maquina
![[/Untitled 2 9.png|Untitled 2 9.png]]
![[/Untitled 3 9.png|Untitled 3 9.png]]
![[/Untitled 4 8.png|Untitled 4 8.png]]
Confirmamos la conexión con la ip victima
Comenzamos haciendo un escaneo de puertos y servicios que están corriendo
sudo nmap -p- --open -sS --min-rate 5000 -v -n -Pn <IP_VICTIMA> > escaneo
![[/Untitled 5 8.png|Untitled 5 8.png]]
cat para ver el escaneo
![[/Untitled 6 8.png|Untitled 6 8.png]]
Contestamos las primeras preguntas de la maquina
![[/Untitled 7 8.png|Untitled 7 8.png]]
Lo siguiente es hacer una enumeración de las comparticiones Samba con nmap
NOTA:SAMBA
Samba es el conjunto estándar de programas de interoperabilidad con Windows para Linux y Unix. Permite a los usuarios finales acceder y usar archivos, impresoras y otros recursos compartidos comúnmente en la intranet o internet de una empresa. A menudo se le conoce como un sistema de archivos de red.
Samba se basa en el protocolo común cliente/servidor de Server Message Block (SMB). SMB fue desarrollado únicamente para Windows; sin Samba, otras plataformas informáticas estarían aisladas de las máquinas Windows, incluso si fueran parte de la misma red.
![[/Untitled 8 8.png|Untitled 8 8.png]]
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse <IP_VICTIMA>
  
  
- `-p 445`: Especifica que quieres escanear el puerto 445. Este puerto es comúnmente utilizado por el protocolo SMB, que se utiliza para compartir archivos, servicios de impresión y otras comunicaciones entre computadoras en una red.
- `-script=smb-enum-shares.nse,smb-enum-users.nse`: Esta opción indica a `nmap` que utilice dos scripts específicos del Motor de Scripts de Nmap (NSE):
- `smb-enum-shares.nse`: Este script se utiliza para enumerar (listar) los recursos compartidos SMB disponibles en el sistema de destino. Los recursos compartidos SMB son directorios o recursos que se comparten en la red.
- `smb-enum-users.nse`: Este script se utiliza para enumerar (listar) los usuarios que están autenticados en el servicio SMB que se ejecuta en el sistema de destino.
![[/Untitled 9 8.png|Untitled 9 8.png]]
cat para ver el documento
![[/Untitled 10 8.png|Untitled 10 8.png]]
vamos contestando las preguntas de la maquina
![[/Untitled 11 8.png|Untitled 11 8.png]]
Nos conectamos a la maquina mediante
No ingresamos nada en la contraseña
smbclient //10.10.132.120/anonymous
![[/Untitled 12 7.png|Untitled 12 7.png]]
Contestamos las preguntas de la maquina
![[/Untitled 13 7.png|Untitled 13 7.png]]
  
![[/Untitled 14 6.png|Untitled 14 6.png]]
  
Nuestro escaneo anterior con nmap, mostro que en el puerto 111 esta ejecutandose el sevicio rpcbind
  
NOTA:Este es simplemente un servidor que convierte el número de programa de llamada a procedimiento remoto (RPC) en direcciones universales. Cuando se inicia un servicio RPC, le dice a rpcbind la dirección en la que está escuchando y el número de programa RPC que está preparado para servir.
  
El puerto 111 da acceso a un sistema de archivos en red, vamos a enumerarlo
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount <IP_VICTIMA>
![[/Untitled 15 5.png|Untitled 15 5.png]]
Contestamos las preguntas de la maquina
![[/Untitled 16 5.png|Untitled 16 5.png]]
Vamos a obtener la versión de ProFtpd, para ello nos conectaremos al puerto donde esta corriendo el servicio mediante netcat y nos debería dar la versión
![[/Untitled 17 5.png|Untitled 17 5.png]]
Contestamos las preguntas de la maquina
![[/Untitled 18 5.png|Untitled 18 5.png]]
Usaremos searchsploit para buscar con la version y el nombre del servidor ftp
![[/Untitled 19 5.png|Untitled 19 5.png]]
Contestamos las preguntas de la maquina
![[/Untitled 20 5.png|Untitled 20 5.png]]
  
NOTA:  
  
Los comandos `SITE CPFR` y `SITE CPTO` son comandos específicos del módulo `mod_copy` de ProFTPD, un servidor FTP. Estos comandos permiten copiar archivos o directorios dentro del sistema de archivos del servidor FTP.
- `**SITE CPFR**` **(Copy From):** Este comando se utiliza para especificar la ruta del archivo o directorio que se desea copiar. Es el punto de origen.
- `**SITE CPTO**` **(Copy To):** Este comando se utiliza para especificar la ruta de destino a donde se desea copiar el archivo o directorio previamente especificado con `SITE CPFR`.
  
Vamos a conectarnos al servidor y vamos a copiar la clave privada del usuario sabiendo que es kenobi
![[/Untitled 21 5.png|Untitled 21 5.png]]
El directorio /var es un montaje que podemos ver, por lo que hemos movido la clave privada de kenobi al directorio /var/tmp
  
Vamos a montar el directorio /var/tmp en nuestra máquina
![[/Untitled 22 5.png|Untitled 22 5.png]]
![[/Untitled 23 5.png|Untitled 23 5.png]]
Tenemos el montaje en nuestra maquina, ahora podemos acceder a /var/tmp y obtener la clave pribada del usuario kenobi
Accedemos mediantre ssh con el usuario y la clave priada
![[/Untitled 24 5.png|Untitled 24 5.png]]
Verificamos los permisos
![[/Untitled 25 5.png|Untitled 25 5.png]]
Al no tener permisos de escritura no podemos cambiar los permisos.
![[/Untitled 26 5.png|Untitled 26 5.png]]
Tenemos la clave privada en nuestro escritorio
![[/Untitled 27 5.png|Untitled 27 5.png]]
Intentamos el acceso mediante ssh
![[/Untitled 28 5.png|Untitled 28 5.png]]
Ya tenemos el acceso mediante ssh
![[/Untitled 29 5.png|Untitled 29 5.png]]
Respondemos las preguntas de la maquina
![[/Untitled 30 5.png|Untitled 30 5.png]]
  
Escalada de privilegios
![[/Untitled 31 5.png|Untitled 31 5.png]]
Buscamos binarios con permisos SUID en el sistema
1ª Forma → find / -perm -4000 2>/dev/null
2ª Forma→ find / -perm -u=s -type f 2>/dev/null
![[/Untitled 32 5.png|Untitled 32 5.png]]
Ejecutamos el binario
![[/Untitled 33 5.png|Untitled 33 5.png]]
Contestamos las preguntas de la maquina
![[/Untitled 34 5.png|Untitled 34 5.png]]
![[/Untitled 35 5.png|Untitled 35 5.png]]
Dado que este archivo se ejecuta con los privilegios del usuario root, podemos manipular nuestra variable de entorno PATH para obtener una shell como root.
Copiamos la shell /bin/sh, lo renombramos como curl, le dimos los permisos correctos y luego agregamos su ubicación a nuestra variable PATH. Esto significa que cuando se ejecuta el binario /usr/bin/menu, utiliza nuestra variable PATH para encontrar el binario "curl" que en realidad es una versión de /bin/sh. Además, dado que este archivo se ejecuta como root, nuestra shell también se ejecuta como root.
![[/Untitled 36 4.png|Untitled 36 4.png]]