<img width="545" height="299" alt="Captura de pantalla 2025-09-23 170736" src="https://github.com/user-attachments/assets/2f8b1305-76a5-42d9-8122-fb5f55aab496" />



Enumeracion de Puertos,Servicios y Versiones

```bash
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
