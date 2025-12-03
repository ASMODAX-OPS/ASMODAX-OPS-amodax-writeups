# Enumeracion de puertos ,servios y Versiones

```ruby
nmap -sSCV --min-rate 5000 -Pn -vvv 172.17.0.2 > nmap.txt 
shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 17:19
Completed NSE at 17:19, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 17:19
Completed NSE at 17:19, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 17:19
Completed NSE at 17:19, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
           Raw packets sent: 1001 (44.028KB) | Rcvd: 1001 (40.032KB)
```

`nmap` nos indica que hay un solo puerto  y este es de el servicio (HTTP), por que irermos a mirar que hay en el.

<img width="1013" height="550" alt="image" src="https://github.com/user-attachments/assets/a8757169-49ff-4eb2-a73e-4a024fb2d8fc" />

Nos encontramos  un servicio apache mirando detalladamente vemos una ruta un tanto modificada, `/var/www/5eEk3r`. lo primero que haremos sera agregar la `IP` a nuestro `/etc/hosts/` con el nombre `5eEKr` como dominio 

De esta manera lo agregaremos y volveremos a visitar el dominio  si algo cambia..

```ruby
echo '172.17.0.2 5eEk3r.dl' >> /etc/hosts
```

volviendo a servicio web nuevamente nos encontraremos lo mismo, nos dispondremos a realizarle un fuzzing Web con la herramienta que prefieran, en mi caso usare feroxbuster

```ruby
feroxbuster -u http://5eek3r.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 5

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://5eek3r.dl/
 🚩  In-Scope Url          │ 5eek3r.dl
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
200      GET      368l      933w    10705c http://5eek3r.dl/index.html
301      GET        9l       28w      311c http://5eek3r.dl/javascript => http://5eek3r.dl/javascript/
200      GET       25l      127w    10359c http://5eek3r.dl/icons/openlogo-75.png
200      GET      368l      933w    10705c http://5eek3r.dl/
[##>-----------------] - 85s   465261/4567915 13m     found:4       errors:0      
[##>-----------------] - 85s   255585/2283919 3017/s  http://5eek3r.dl/ 
[#>------------------] - 83s   206998/2283919 2506/s  http://5eek3r.dl/javascript/                                                                                                                                                          ^C

```

En este resultado no vemos directorios con algo importante asi que haremos una busqueda de subdominos con wfuzz 

```ruby
wfuzz -H "Host: FUZZ.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl/

```

<img width="1001" height="336" alt="image" src="https://github.com/user-attachments/assets/0c308c58-b141-4f96-a262-600922fc36e1" />
