# Enumeracion de Puertos,Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 <IP> -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-17 11:35 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 8e:a2:56:38:e1:85:2f:21:2b:55:ec:29:5b:f8:63:d9 (ECDSA)
|_  256 0f:4b:38:fa:04:33:c7:01:5a:98:12:05:2d:42:cf:1a (ED25519)
53/tcp open  domain  ISC BIND 9.18.28-0ubuntu0.24.04.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.18.28-0ubuntu0.24.04.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: HackZones.hl - Seguridad para tu Empresa
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.84 seconds
```
Se observa que el dominio 'HackZones.hl' en el reporte que nos da 'nmap', lo podemos agregar al archivo '/etc/host' en mi caso es 
```ruby
echo '172.17.0.2 HackZones.hl' >> /etc/hosts
```

