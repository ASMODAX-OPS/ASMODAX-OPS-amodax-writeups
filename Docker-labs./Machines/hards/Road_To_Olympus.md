
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
