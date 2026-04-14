
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
2. **Iniciar Túnel:** Escribe `start` para comenzar el intercambio de paquetes.



> **Tip Técnico:** Verifica siempre con `ip route` que la ruta hacia la red `20.20.20.0/24` apunta correctamente a la interfaz `ligolo`. Si no lo haces, intentarás salir por tu puerta de enlace predeterminada (tu router) y no llegarás a Poseidón.
