

# Escaneo y reconocimiento de servicios 
```ruby

 nmap -p- --min-rate 5000 -sSVC -Pn -vvv 172.17.0.2
 
Discovered open port 139/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 445/tcp on 172.17.0.2 17s
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     REASON         VERSION
80/tcp  open  http        syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: \xC2\xBFQu\xC3\xA9 es Samba?
|_http-server-header: Apache/2.4.52 (Ubuntu)
139/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4
445/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 4
MAC Address: 02:42:AC:11:00:02 (Unknown)

Host script results:
|_clock-skew: 0s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 21783/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 9604/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 58197/udp): CLEAN (Failed to receive data)
|   Check 4 (port 18315/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-18T00:14:24
|_  start_date: N/A
```

vemos que tenemos el puerto 80 iremos a revisar que se encuentra en este puerto 

<img width="1195" height="701" alt="image" src="https://github.com/user-attachments/assets/c1038938-3f6c-4720-8269-5117198a296e" />

vemos que nos explica un poco de samba, mas informacion --> (https://hackers-arise.com/network-basics-for-hackers-server-message-block-smb-and-samba/)

Realizamos un fuzzeo en busca de directorios, subdirectorios y subdominios interesantes pero sin éxito. 

Nos centramos en el servidor smb, lanzamos enum4linux, para una posible enumeración de usuarios 

enum4linux 172.17.0.2
