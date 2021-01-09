---
title: Como encontré una vulnerabilidad crítica en ClaveUnica
date: 2021-01-08 11:58:47 +07:00
---

Antes de todo, agradezco el apoyo que entregó el [CSIRT](https://twitter.com/csirtgob/) en donde trabajamos en conjunto en un informe para públicar, pero no pudo salir a la luz debido a la antigua cultura que tiene [Gobierno Digital](https://twitter.com/GobDigitalCL) en donde demostrar que tus sistemas tienen vulnerabilidades es una debilidad. Finalmente logran que la gente que les detecte vulnerabilidades no les reporten sus brechas, lo ideal sería que cambiaramos ese switch como cultura y se premien estas actividades como en el resto del mundo en donde se crea una cultura proactiva del hacking por el mejorar internet. 


Bueno a lo que vamos ... 

Como la pandemia nos ha hecho depender de muchas cosas a traves de internet, en Chile tenemos ClaveUnica en donde se pueden hacer aproximadamente 1.017 tramites en linea. Dije bueno quizas el sistema auge de la pandemia no creo que sea perfecto y me puse a revisar la plataforma.

Observando el método de recuperación de la clave del sistema ClaveÚnica, me encontré con un formulario que solicitaba únicamente el RUT para iniciar el proceso de recuperación.

![claveunica](/assets/img/claveunica.png)

Al solicitar la recuperación de la clave al sistema, este solicita un medio (correo electrónico o SMS) para entregar un código que es necesario para el proceso. Esto despertó mi curiosidad y quise saber que era lo que respondería el servidor. Entonces, ocupando Burpsuite, revisé como era la comunicación cliente servidor, lo que me permitió encontrar una solicitud con lo siguiente:

![claveunica1](/assets/img/claveunica1.png)

La búsqueda me llevó a encontrar el host de la API de autenticación de ClaveÚnica que es iambackend.claveunica.gob.cl, en donde si accedíamos al sitio de la API obteníamos todos los recursos que se podían consumir dentro de esta.

![claveunica2](/assets/img/claveunica2.png)

Con en esta información, ya tenía dos certezas:

- Para solicitar la recuperación de ClaveÚnica hay una solicitud donde solo pasa el RUT sin el digito verificador
- La autenticación pasa por la API

Continué analizando el flujo de la recuperación de clave y viendo la respuesta por parte del servidor hacia el cliente, encontré lo siguiente:

![claveunica3](/assets/img/claveunica3.png)

En la información devuelta observé un Token de sesión. De inmediato surgió la inquietud sobre la factibilidad de que una tercera parte pudiera ocupar este Token para explotar la vulnerabilidad y realizar acciones no permitidas en el sistema, como por ejemplo, tomar el control de otras cuentas. Al analizar ClaveÚnica encontré una opción para actualizar los datos en donde está el correo electrónico y el número de teléfono.

![claveunica4](/assets/img/claveunica4.png)

Para comprobar lo anterior, repetí el paso de la recuperación para obtener el tráfico que pasa entre cliente servidor, en el cual encontramos la siguiente solicitud:

![claveunica5](/assets/img/claveunica5.png)

Observé que para la actualización de datos se necesitaban los siguientes requisitos:

- La cabecera Token debe tener un Token de sesión
- En el cuerpo de la solicitud se observa que pasa el RUT sin dígito verificador en la variable numero
- En la variable email se entrega el correo electrónico con el que se actualizará la cuenta
- En la variable number se encuentra el número de teléfono con el que se actualizará la cuenta

Con esta información se configura una idea de qué hacer para poder explotar la vulnerabilidad, que es un IDOR, y esto conllevaría finalmente a tomar la cuenta de otra persona cambiando esta información.

PoC:

Siguiendo el procedimiento señalado anteriormente, el primer paso fue obtener el Token de sesión de otra cuenta, replicando la solicitud original para la recuperación de la contraseña.

![claveunica6](/assets/img/claveunica6.png)

Al obtener el Token de una tercera persona, fue posible ocupar la solicitud dónde se actualizaba la información de la cuenta y suministrar nueva información para actualizar la cuenta 8XXXXXX4.

![claveunica7](/assets/img/claveunica7.png)

En la respuesta entregada por el servidor se puede apreciar que la información suministrada se actualizó correctamente.

![claveunica8](/assets/img/claveunica8.png)

Teniendo en cuenta esto, revisamos la cuenta en la que se tomará el control para validar si se cambiaron los datos. Esto fue posible porque había conocimiento de las credenciales de las 2 cuentas.

![claveunica9](/assets/img/claveunica9.png)

La evidencia muestra que la cuenta ya contiene la información modificada por la tercera parte. El paso siguiente es solicitar la recuperación de la clave de la cuenta.

![claveunica10](/assets/img/claveunica10.png)

El RUT se ingresa con dígito verificador dentro del flujo del sitio web y se constata que se puede continuar con el procedimiento sin problemas.

![claveunica11](/assets/img/claveunica11.png)

Luego se realiza la solicitud para que el código sea enviado por correo electrónico, el cual ya fue modificado, para recibir el mensaje en la nueva casilla.

![claveunica12](/assets/img/claveunica12.png)

En este paso se observa cómo el código fue enviado a la casilla modificada. Con el mensaje, seguimos con el flujo de recuperación de la cuenta.

![claveunica13](/assets/img/claveunica13.png)

El siguiente paso es hacer clic en “Recuperar ClaveÚnica” para que nos redirija al sitio web oficial.

![claveunica14](/assets/img/claveunica14.png)

Posteriormente utilizamos el código recibido en el correo electrónico para poder hacer el cambio de clave de la cuenta comprometida.

![claveunica15](/assets/img/claveunica15.png)

El sistema nos solicita el cambio de contraseña de la cuenta comprometida, que nos permitió apropiarnos de la cuenta.

Esto a día de hoy esta mitigado.

Vida del reporte:

![claveunica16](/assets/img/claveunica16.png)

El mensaje que les quiero dejar al final es que si van a reportar haganlo por los medios que hay para que cambiemos esta cultura del miedo y del ego de los sistemas 'invulnerables'.




