<img width="542" height="300" alt="image" src="https://github.com/user-attachments/assets/5f17e336-57b6-4c19-b353-a5fc0ff79e98" />

# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)


# INTRODCUCION 

1- Descargamos el zip de la plataforma. Con unzip descomprimimos

 unzip 404-not-found.zip

  Archive:  404-not-found.zip
  inflating: auto_deploy.sh     	 
  inflating: 404-not-found.tar

 2- Y ahora desplegamos la máquina

sudo bash auto_deploy.sh 404-not-found.tar

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla

# conectividad
 ping -c1 172.17.0.2
veremos que funciona e manda maquetes 

#Escaneo de Puetos,servicios 

haremos un nmap que nos de los puertos,sevicos,OS

```ruby
nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
```
``` bash
(-p) = Escanea todos los puertos 65535 puertos TCP
(-Pn) = No hace ping,Como ya asumimos que la maquian esta activa no es necesario
(-sVCS) = este combina 3 tecnicas
  (-sV) = Este nos muestra la version de los sevicios 
  (-sC) = Usa los script NSE por defecto (Detecion basica de vulnerabilidades)
  (-sS) = Escaneo SYN stealth, mas sigiloso que el TCP connect
```
```bash

 nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 21:46 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000016s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 59:4e:10:e2:31:bf:13:43:c9:69:9e:4f:3f:a2:95:a6 (ECDSA)
|_  256 fb:dc:ca:6e:f5:d6:5a:41:25:2b:b2:21:f1:71:16:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://404-not-found.hl/
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.64 seconds

```
Este escaneo nos muestra el puerto ssh y http, iremos al servidor web no lleva a  "http://404-not-found.h", al irnos nos dara una conexion de no encontradaa por lo cual lo agregaremos al /etc/hosts

<img width="598" height="306" alt="image" src="https://github.com/user-attachments/assets/2c00ade2-d6d6-44c3-aa15-147f23aa25cf" />

<img width="1274" height="761" alt="image" src="https://github.com/user-attachments/assets/2b58e26a-b351-4301-9cf7-996d860677e9" />

le daremos a paraticipar nos llevara a al siguiente directorio 

<img width="1267" height="752" alt="image" src="https://github.com/user-attachments/assets/1df48ceb-75cb-4ef4-a2c0-706166c99dc9" />

vermeos un hash en base 64 lo decodificaremos y veremos lo siguiente  

<img width="1017" height="727" alt="image" src="https://github.com/user-attachments/assets/5628fb33-f44c-4cee-947d-03382033fc20" />

nos indica que debemos ver la url por lo que lo podemos ver como una busqueda de posibles subdominios usaremos la herramienta wfuzz
# Que hace esta herramienta

- Realiza un ataque de fuerza bruta de subdominios contra el dominio que especifiques.
- Usa una wordlist masiva para aumentar la probabilidad de encontrar subdominios activos.
- Filtra respuestas repetitivas con --hw 28, mostrando solo las que difieren en contenido.
- Ideal para entornos de CTF, pentesting o reconocimiento ofensivo.

```ruby
wfuzz -c --hw 28 -w /root/tools/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u 
http://404-not-found.hl -H "Host: FUZZ.404-not-found.hl"
```

```bash
wfuzz : herramienta de fuzzing para descubrir recursos ocultos (subdominios, directorios, parámetros).
  -c	 : colorea la salida en consola para facilitar la lectura.
 --hw 28 : filtra respuestas que tengan exactamente 28 palabras en el cuerpo. Útil para ignorar respuestas genéricas como errores 404 o redirecciones.
 -w 	: especifica la wordlist con los subdominios más comunes (110,000 entradas). Puedes cambiar la ruta si usas otra ubicación.
 -u 	: URL objetivo. Wfuzz reemplaza  por cada palabra de la wordlist para probar subdominios como: http://admin.tudominio
```
Nos encontrara un subdomio llamado  info.404-not-found.hl lo agregaremos a nuestro /etc/hosts vermeos un login 

<img width="1272" height="762" alt="image" src="https://github.com/user-attachments/assets/a717bb68-c046-498c-8b70-190ce43029ba" />

Tenemos un panel de login en el que pruebo manualmente varias credenciales y no encuentraremosnada.

<img width="594" height="331" alt="image" src="https://github.com/user-attachments/assets/fc691f8d-becf-4633-bd47-a025d5120858" />

Investigaremos el código fuente

<img width="601" height="423" alt="image" src="https://github.com/user-attachments/assets/56004be2-f482-493d-be2b-739b83e56795" />

vermeos un HTML, al pulsar en el veremos lo siguiente

<img width="608" height="482" alt="image" src="https://github.com/user-attachments/assets/2204032e-d1c3-42ca-b692-1d93e829d0e5" />

```<!-- I believe this login works with LDAP → ```

tendremos que explotar una vulnerabilidad llamada LDAP

# ¿Qué es LDAP?
LDAP es un protocolo ligero para acceder y administrar servicios de directorio, como Active Directory. Se usa para autenticar usuarios, buscar información (nombres, correos, grupos), y controlar acceso en redes corporativas.

Ejemplo de consulta LDAP: 

```(&(uid=user)(userPassword=123456))```

# ¿Qué es una inyección LDAP?
La inyección LDAP ocurre cuando una aplicación web no valida correctamente la entrada del usuario y la inserta directamente en una consulta LDAP. Esto permite al atacante manipular la lógica de autenticación o búsqueda, similar a una inyección SQL.

# ¿Cómo funciona?
Supón que una aplicación construye esta consulta: 

```(&(uid={usuario})(userPassword={contraseña}))```

Si nosotros como el atacante ingresa:
- Usuario: *)(uid=*))(|(uid=*
- Contraseña: cualquiercosa
La consulta final sería:
```(&(uid=*)(uid=*))(|(uid=*)(userPassword=cualquiercosa))```

Esto rompe la lógica original y puede devolver todos los usuarios del directorio, bypasseando la autenticación.


# Prevencion de este 

- Escapar caracteres especiales: *, (, ), |, &, !
- Validar y sanitizar entradas del usuario.
- Usar consultas parametrizadas si el lenguaje lo permite.
- Limitar errores detallados que revelen estructura LDAP.

# Explotacion 

Vamos probando las combinaciones hasta que bingo:

user=*)(|(&
pass=pwd)


Credenciales de Admin

Nombre de usuario: 404-page

Contraseña: not-found-page-secret


<img width="960" height="699" alt="image" src="https://github.com/user-attachments/assets/886874bd-d4c7-4856-bdad-16569a16d72c" />


Como tenemos el puerto 22 abierto vamos a probar

```ssh 404-page@172.17.0.2 ```

<img width="589" height="337" alt="image" src="https://github.com/user-attachments/assets/e9025e06-435a-4ad1-aaff-df27762370ef" />

# Escalacion de privilecios 

buscamos los permisos de sudo 

<img width="593" height="92" alt="image" src="https://github.com/user-attachments/assets/c9351a6b-ec90-4696-9ae3-c8cd40133d43" />


Utilizamos la función __import__() para importar dinámicamente el módulo

'os' y luego ejecutar comandos del sistema a través de os.system().

Nos hacemos 200-ok


Listamos directorios
```bash
200-ok@1eca5f7152e7:~$ cat boss.txt

What is rooteable

```

Nos aremos root 

<img width="593" height="92" alt="image" src="https://github.com/user-attachments/assets/f57aa704-7922-4928-8c55-ee4e14e2098e" />


con esto tendremos permisos root y hemos completado la maquina, Gracias 
