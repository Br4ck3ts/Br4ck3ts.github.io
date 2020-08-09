---
title: Hackaton-Telefonica-Bury-A-Fr1end
date: 2019-07-30 11:58:47 +07:000
---

Primero para empezar ocuparemos Nmap para revisar que puertos tiene abiertos la máquina.

![hackaton](/assets/img/hackaton.png)

En el puerto 80 se encuentra lo siguiente

![hackaton2](/assets/img/hackaton2.png)

En donde si hacen ataque por fuerza bruta para listar directorios no encontraran nada, por lo que tendremos que revisar el siguiente puerto. En donde se observa FTP anónimo al cual nos conectaremos para revisar hint lo que nos indica el mismo reconocimiento de Nmap, pero además revisaremos el directorio del FTP para revisar si hay archivos ocultos.

![hackaton3](/assets/img/hackaton3.png)

Al descargar hint obtenemos lo siguiente

![hackaton4](/assets/img/hackaton4.png)

Un mensaje en texto plano y un mensaje cifrado en sha256 el cual corresponde a lo siguiente

![hackaton5](/assets/img/hackaton5.png)

Además, encontramos la primera flag de la maquina como archivo oculto.

![hackaton6](/assets/img/hackaton6.png)

Ya que tenemos esa palabra debemos entender que hay que ocuparlo en el servicio web el cual nos arrojara lo siguiente.

![hackaton7](/assets/img/hackaton7.png)

Ya que encontramos algo, aplicaremos herramientas para buscar directorios y archivos en la web encontrando lo siguiente.

![hackaton8](/assets/img/hackaton8.png)

Como el mensaje del FTP decía “Find the secrets” revisaremos secrets.txt, el cual tiene un mensaje cifrado en base64.


RnIxM05kOjZ1akpNRg==
El cual nos arroja las siguientes credenciales:

Fr13Nd
6ujJMF

Revisamos upload.php el cual nos pide autenticación, ocupamos las credenciales obtenidas y entramos.

![hackaton9](/assets/img/hackaton9.png)

Teniendo en cuenta que hay un directorio llamado files vacío, entendemos que los archivos se subirán en ese directorio. Por lo que procederemos a subir la Shell reversa en donde nos encontraremos con una validación de Content-Type que debe ser image.

![hackaton10](/assets/img/hackaton10.png)

Después de obtener nuestra Shell reversa en el direcotrio /var/www encontraremos nuestra segunda flag oculta.

![hackaton11](/assets/img/hackaton11.png)

Y dos archivos importantes.

![hackaton12](/assets/img/hackaton12.png)

En donde la imagen oculta contiene un archivo oculto, así que deberemos ocupar herramientas de esteganografía para encontrar el archivo en donde la contraseña es el nombre de la imagen que no está oculta.

![hackaton13](/assets/img/hackaton13.png)

Revisamos el archivo secrets2 el cual contiene credenciales del usuario Billie

![hackaton14](/assets/img/hackaton14.png)

Nos logeamos como billie y encontraremos nuestra tercera flag en la carpeta en el escritorio

![hackaton15](/assets/img/hackaton15.png)

Para elevar privilegios revisaremos si tenemos permisos de sudo y encontramos lo siguiente

![hackaton16](/assets/img/hackaton16.png)

Buscamos documentación como escalar con Nmap no interactivo y encontramos la siguiente forma para elevar privilegios.

![hackaton17](/assets/img/hackaton17.png)

Posterior a esto encontraremos nuestra última flag en /root/

![hackaton18](/assets/img/hackaton18.png)
