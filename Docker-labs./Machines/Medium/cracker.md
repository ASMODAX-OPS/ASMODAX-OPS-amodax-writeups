<img width="474" height="333" alt="image" src="https://github.com/user-attachments/assets/e840739e-371a-4e0e-9d0d-e05956b6899d" />


# Enumeracion de Puertos,Servicios y versiones 

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-08 20:00 EST
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 20:c2:39:55:4e:1e:5f:4e:1d:e7:f1:17:39:1a:4e:7a (ECDSA)
|_  256 a5:b5:c0:f2:ce:3e:1a:e4:eb:f8:c3:dd:b5:c6:f3:b3 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: P\xC3\xA1gina Web Profesional
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.67 seconds
```

Este escaneo nos nuestra dos servicios el htttp  y el ssh lo que vamos a ir al servicio web 

<img width="1917" height="884" alt="image" src="https://github.com/user-attachments/assets/79a61828-0338-4ebe-802d-d9aab94db3c9" />

<img width="524" height="179" alt="image" src="https://github.com/user-attachments/assets/11ced599-1e47-4b96-8618-3be7b260efd1" />

veremos esta pagina al principio no veremos algo relevante, pero al inspecionar el codigo frond end, veremos que hay posible dominio llamado cracker.dl lo agregaremos al `/etc/hosts`

```ruby
echo '172.17.0.2  cracker.dl' >> /etc/hosts
```
Con esto echo iremos al domiinio y encontreremos otro sitio web 

<img width="1914" height="935" alt="image" src="https://github.com/user-attachments/assets/705e7b6c-4092-4d21-ad50-299ad57e8342" />

Esta pagina parece que esta mal configurada por lo que no funciona nada, con esto dicho iremos hacer un fuzzing web que nos podremos encontrar.

```ruby
┌──(kali㉿kali)-[~/Downloads]
└─$ feroxbuster -u 'http://172.17.0.2/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 --random-agent --no-state -d 5
                                                                                                                                                                                                                                            
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
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET      162l      319w     4029c http://172.17.0.2/
200      GET      162l      319w     4029c http://172.17.0.2/index.html
[###>----------------] - 78s   357735/2283919 7m      found:2       errors:530    
[###>----------------] - 78s   359249/2283919 4585/s  http://172.17.0.2/                      
```
No vemos nada interesante con directorios con esto iremos a buscar subdominios.
```ruby
$  wfuzz -c -u cracker.dl -H "HOST: FUZZ.cracker.dl" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt  --hl=162
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://cracker.dl/
Total requests: 114442

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000002128:   200        91 L     224 W      3199 Ch     "japan - japan"                                                                                                                                                            
000009532:   400        10 L     35 W       302 Ch      "#www - #www"                                                                                                                                                              
000010581:   400        10 L     35 W       302 Ch      "#mail - #mail"                                                                                                                                                            
^C000021342:   200        162 L    319 W      4029 Ch     "boca - boca"                                                                                                                                                              

Total time: 0
Processed Requests: 21355
Filtered Requests: 21352
Requests/sec.: 0
```
vemos que hay un subdominio llamdo japan, con esta informacion lo agregaremos al `/etc/hosts`

```ruby
172.17.0.2  cracker.dl japan.cracker.dl
```
iremos a revisar que tiene este subdominio, este subdominio tiene algo interesante y es que tiene un software lo descargaremnos y miraremos que tiene adentro 

<img width="1915" height="917" alt="image" src="https://github.com/user-attachments/assets/353e943b-4192-47c2-bac3-e00780db1317" />

```ruby


┌──(kali㉿kali)-[~/Downloads]
└─$ file PanelAdmin 
PanelAdmin: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=1a373bf2779766088c7e35eeb06784b9156e2add, for GNU/Linux 3.2.0, not stripped

le daremos permisos
chmod +x PanelAdmin                                                                                                                                                                                                  
```
lo ejecutamos y veremos lo siguiente 

<img width="443" height="273" alt="image" src="https://github.com/user-attachments/assets/8d0b75f4-2852-457a-88ff-6992203014af" />
s

solicita un serial para el acceso asi que vere que saco de hacerle `reversing al binario` asi que ejecuto ghidra

<img width="504" height="834" alt="image" src="https://github.com/user-attachments/assets/3a6f541d-9a4b-4ce3-8a8e-7b53128111ac" />

despues de habrilo agregamos un active project y agregaremos el archivo 'PanelAdmin'

<img width="794" height="588" alt="image" src="https://github.com/user-attachments/assets/5b2b294c-4b9a-4338-8759-178a88600701" />

con esto lo selecionamos y daremos click al icono del dragon despues eso nos aparecera una pantalla que nos dira hacer un analizys y le diremos que si , tranquilos si no entienden nada, veremos lo siguiente 

<img width="1616" height="917" alt="image" src="https://github.com/user-attachments/assets/0cf976f3-9456-46e6-a0c3-296c38a887e0" />

ya con esta parte con el codigo decopilado buscaremos en la parte de `symbol Tree` y en la carpeta fuctions buscaremos hasta encontrar la funcion con el nombre `validate_serial` veremos lo siguientte


<img width="1887" height="826" alt="image" src="https://github.com/user-attachments/assets/c203c217-ae1f-4765-b9c5-1ecb729ce372" />

estos numeros que vemos en esa parte puede que sea el numero serial que nos pide en la verificacion del serial las comas las quitaremos y agregaremos guiones

<img width="952" height="462" alt="image" src="https://github.com/user-attachments/assets/d1ab1f86-2fbc-436b-bb0d-4a8afb812fc5" />

<img width="1574" height="523" alt="image" src="https://github.com/user-attachments/assets/65ce5c3d-a199-46da-8ebb-74395566def3" />

<img width="1461" height="429" alt="image" src="https://github.com/user-attachments/assets/4561fd62-f357-4d42-a2c2-81525dfbf520" />


obtenemos la password, sin embago no tengo un user con el cual loguearme via `ssh` por lo que aplicare un ataque de diccionario contra el servicio

```ruby
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p '#P@$$w0rd!%#S€c7T' ssh://172.17.0.2

──(kali㉿kali)-[~/Downloads]
└─$ hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p '#P@$$w0rd!%#S€c7T' ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-12-08 20:37:24
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 8295455 login tries (l:8295455/p:1), ~518466 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 246.00 tries/min, 246 tries in 00:01h, 8295212 to do in 562:01h, 13 active
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.

```
este ataque nos da un error o no inicia, debe ser por algo asi que usaremos `crackmapexec`

```ruby
cracksapexec ssh 172.17.0.2/usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p '#P@$$w0rd!%#S€c7T' |grep -v 'Authentication failed.'
SSH              172.17.0.2 22         22   172.17.0.2      [*] SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.5
SSH              172.17.0.2            22   172.17.0.2      *[+] cracker:#P@Sswürdi#SEC7T
```
entramos con las password via `ssh` 

```ruby
ssh-keygen -R 172.17.0.2 && ssh cracker@172.17.0.2
# Host 172.17.0.2 found: line 4
/home/kali/.ssh/known_hosts updated.
Original contents retained as /home/kali/.ssh/known_hosts.old
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is: SHA256:CEJTnHaAqyTet95lZFlz0g4E8/2iNHT1Bh5+5RM5yvc
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
cracker@172.17.0.2's password: 
Permission denied, please try again.
cracker@172.17.0.2's password: 
Permission denied, please try again.
cracker@172.17.0.2's password: 
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.12.38+kali-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

cracker@885c9b5295ae:~$ 
```
con el acceso haremos iremos a directorio `/tmp/`, buscando por bucho tiempo la clave de root temina siendo la clave de serial osea `47378-10239-84236-54367-83291-78354`

```ruby
cracker@885c9b5295ae:/tmp$ su
Password: 
root@885c9b5295ae:/tmp# whoami
root
```
con esto ya tendremos la maquina comprometida y resuelta
