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

<!-- AutoExecPlugin.txt --> 
Veremos lo que parece ser un archivo .txt el cual vamos a probar si estuviera subido en el servidor.


Copy
URL = http://<IP>/AutoExecPlugin.txt
Info:


Copy
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
Por lo que vemos si existe, esto es una clase de JAVA la cual nos esta dando una pista de que si esto estuviera en el servidor de minecraft como un plugin puede ser muy vulnerable ya que puede ejecutar comandos del sistema dentro del propio servidor del juego, por lo que vamos a descargarnos minecraft para meternos en dicho servidor.

Instalación Minecraft (Gratis)
Vamos a ir al siguiente enlace en la pagina del TLauncher de Minecraft, es para jugar Minecraft gratis.
