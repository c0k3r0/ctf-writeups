---
layout: default
title: THLPWN (THL)
---

# 🧠 Write-up: THLPWN (THL)

[portada](portada.png)

---

## 📸 Evidencias del proceso

### Fase de Reconocimiento

Realizamos un escaneo con Nmap para identificar puertos abiertos y servicios activos.

[1](1.png)

El escaneo revela que los puertos `22 (SSH)` y `80 (HTTP)` están abiertos.

### Acceso denegado vía HTTP

[2](2.png)

Al intentar acceder al sitio web desde el navegador mediante la IP, recibimos un error 403 Forbidden. Se menciona que se usa virtual hosting y que es necesario encontrar el nombre de host correcto.

### Enumeración con curl

[3](3.png)

Utilizamos curl para inspeccionar el contenido HTML directamente. Encontramos un comentario oculto en el código fuente: <!--thlpwn.thl-->, lo que sugiere el nombre del host.

### Edición del archivo hosts

[4](4.png)

Editamos el archivo /etc/hosts para redirigir thlpwn.thl a la IP del objetivo.

[5](5.png)

Agregamos la entrada 10.0.2.18 thlpwn.thl.

### Acceso correcto al sitio

[6](6.png)

Al acceder a http://thlpwn.thl, ahora se muestra una página de login con múltiples enlaces de interés.

### Fuerza bruta de rutas con FFUF

[7](7.png)

Utilizamos ffuf con la wordlist common.txt para descubrir rutas ocultas. Encontramos, entre otras, robots.txt.

### Inspección de robots.txt

[8](8.png)

Accedemos a robots.txt donde encontramos referencias a directorios ocultos y pistas, entre ellas: /downloads, /admin/, /backup/, además de información útil sobre posibles vectores de ataque.

### Sección de descargas

[9](9.png)

Accedemos a la ruta /downloads, donde encontramos un binario llamado auth_checker. La descripción sugiere que podría revelar credenciales SSH.

### Análisis del binario

[10](10.png)

Utilizamos strings sobre el binario y extraemos credenciales válidas:

Username: thluser  
Password: 9Kx7mP2wQ5nL8vT4bR6zY

### Conexión SSH

[11](11.png)

Accedemos vía SSH al sistema usando las credenciales obtenidas y confirmamos que estamos dentro como thluser.

### Primera flag

[12](12.png)

Dentro del sistema, listamos los archivos del usuario y encontramos flag.txt.

### Verificación de privilegios sudo

[13](13.png)

Revisamos permisos sudo y descubrimos que el usuario puede ejecutar /bin/bash con sudo sin contraseña.

### Escalada de privilegios

[14](14.png)

Consultamos GTFObins para ver cómo escalar privilegios usando sudo bash.

### Escalada a root

[15](15.png)

Ejecutamos sudo bash y confirmamos que ahora somos el usuario root.

15. Flag de root

[16](16.png)

Accedemos al directorio /root y extraemos la flag final de root.txt.

---

### 🏁 Conclusión

Este reto fue un excelente ejercicio práctico que cubre múltiples aspectos del hacking ético:

- Enumeración de servicios

- Bypass de restricciones por virtual host

- Enumeración de rutas ocultas con FFUF

- Análisis de binarios

- Uso de credenciales SSH

- Escalada de privilegios local con GTFObins

Una muestra clara de cómo pequeños indicios pueden conducir a una escalada total del sistema.

### Write-up realizado por **c0k3r0** — El Sótano de c0k3r0 �

[⬅️ Volver al inicio](https://c0k3r0.github.io/ctf-writeups/)
