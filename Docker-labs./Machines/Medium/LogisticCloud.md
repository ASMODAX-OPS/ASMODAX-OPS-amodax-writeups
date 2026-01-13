
# 📑 Writeup: Máquina DockHackLab 🚀

![OS: Linux](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Platform: DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)
![Dificultad: Media](https://img.shields.io/badge/Dificultad-Media-yellow?style=for-the-badge)


# Reconocomiento de Puertos Servicios 

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-09 05:04 EDT
Nmap scan report for bicho.dl (172.17.0.2)
Host is up (0.00016s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e9:59:86:db:ea:af:ff:09:ee:8f:ab:c6:0d:b8:b5:82 (ECDSA)
|_  256 ff:8d:9f:f8:e7:a5:f4:ce:6a:2d:e4:30:ac:77:18:fc (ED25519)
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Login - HLG Logistics
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.58 (Ubuntu)
9000/tcp open  http    Golang net/http server
|_http-title: Did not follow redirect to http://bicho.dl:9001
|_http-server-header: MinIO
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 183DD07DC90B2EC0
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Fri, 09 May 2025 09:04:29 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice 
ports,/Trinity.txt.bak</Resource><RequestId>183DD07DC90B2EC0</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Er
ror>
|   GenericLines, Help, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 276
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 183DD07A46C8B7BE
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Fri, 09 May 2025 09:04:14 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid 
argument)</Message><Resource>/</Resource><RequestId>183DD07A46C8B7BE</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</Hos
tId></Error>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Vary: Origin
|     Date: Fri, 09 May 2025 09:04:14 GMT
|_    Content-Length: 0
9001/tcp open  http    Golang net/http server
|_http-title: MinIO Console
|_http-server-header: MinIO Console
| fingerprint-strings: 
|   GenericLines, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Content-Length: 1309
|     Content-Security-Policy: default-src 'self' 'unsafe-eval' 'unsafe-inline'; script-src 'self' https://unpkg.com; connect-src 'self' https://unpkg.com;
|     Content-Type: text/html
|     Last-Modified: Fri, 09 May 2025 09:04:14 GMT
|     Referrer-Policy: strict-origin-when-cross-origin
|     Server: MinIO Console
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: DENY
|     X-Xss-Protection: 1; mode=block
|     Date: Fri, 09 May 2025 09:04:14 GMT
|_    <!doctype html><html lang="en"><head><meta charset="utf-8"/><base href="/"/><meta content="width=device-width,initial-scale=1" name="viewport"/><meta 
content="#081C42" media="(prefers-color-scheme: light)" name="theme-color"/><meta content="#081C42" media="(prefers-color-scheme: dark)" 
name="theme-color"/><meta content="MinIO Console" name="description"/><meta name="minio-license" content="agpl"/><link href="./s
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at 
https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9000-TCP:V=7.95%I=7%D=5/9%Time=681DC50E%P=x86_64-pc-linux-gnu%r(Gen
SF:ericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20te
SF:xt/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x2
SF:0Request")%r(GetRequest,2B0,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nAcce
SF:pt-Ranges:\x20bytes\r\nContent-Length:\x20276\r\nContent-Type:\x20appli
SF:cation/xml\r\nServer:\x20MinIO\r\nStrict-Transport-Security:\x20max-age
SF:=31536000;\x20includeSubDomains\r\nVary:\x20Origin\r\nX-Amz-Id-2:\x20dd
SF:9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8\r\nX-Amz
SF:-Request-Id:\x20183DD07A46C8B7BE\r\nX-Content-Type-Options:\x20nosniff\
SF:r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Fri,\x2009\x20May\
SF:x202025\x2009:04:14\x20GMT\r\n\r\n<\?xml\x20version=\"1\.0\"\x20encodin
SF:g=\"UTF-8\"\?>\n<Error><Code>InvalidRequest</Code><Message>Invalid\x20R
SF:equest\x20\(invalid\x20argument\)</Message><Resource>/</Resource><Reque
SF:stId>183DD07A46C8B7BE</RequestId><HostId>dd9025bab4ad464b049177c95eb6eb
SF:f374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>")%r(HTTPOptions,59,
SF:"HTTP/1\.0\x20200\x20OK\r\nVary:\x20Origin\r\nDate:\x20Fri,\x2009\x20Ma
SF:y\x202025\x2009:04:14\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRe
SF:quest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/p
SF:lain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Req
SF:uest")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r
SF:\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close
SF:\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,2CB,"HTTP/1\.0\x204
SF:00\x20Bad\x20Request\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x203
SF:03\r\nContent-Type:\x20application/xml\r\nServer:\x20MinIO\r\nStrict-Tr
SF:ansport-Security:\x20max-age=31536000;\x20includeSubDomains\r\nVary:\x2
SF:0Origin\r\nX-Amz-Id-2:\x20dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af92
SF:51148b658df7ac2e3e8\r\nX-Amz-Request-Id:\x20183DD07DC90B2EC0\r\nX-Conte
SF:nt-Type-Options:\x20nosniff\r\nX-Xss-Protection:\x201;\x20mode=block\r\
SF:nDate:\x20Fri,\x2009\x20May\x202025\x2009:04:29\x20GMT\r\n\r\n<\?xml\x2
SF:0version=\"1\.0\"\x20encoding=\"UTF-8\"\?>\n<Error><Code>InvalidRequest
SF:</Code><Message>Invalid\x20Request\x20\(invalid\x20argument\)</Message>
SF:<Resource>/nice\x20ports,/Trinity\.txt\.bak</Resource><RequestId>183DD0
SF:7DC90B2EC0</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1
SF:af9251148b658df7ac2e3e8</HostId></Error>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9001-TCP:V=7.95%I=7%D=5/9%Time=681DC50E%P=x86_64-pc-linux-gnu%r(Gen
SF:ericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20te
SF:xt/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x2
SF:0Request")%r(GetRequest,702,"HTTP/1\.0\x20200\x20OK\r\nAccept-Ranges:\x
SF:20bytes\r\nContent-Length:\x201309\r\nContent-Security-Policy:\x20defau
SF:lt-src\x20'self'\x20'unsafe-eval'\x20'unsafe-inline';\x20script-src\x20
SF:'self'\x20https://unpkg\.com;\x20\x20connect-src\x20'self'\x20https://u
SF:npkg\.com;\r\nContent-Type:\x20text/html\r\nLast-Modified:\x20Fri,\x200
SF:9\x20May\x202025\x2009:04:14\x20GMT\r\nReferrer-Policy:\x20strict-origi
SF:n-when-cross-origin\r\nServer:\x20MinIO\x20Console\r\nX-Content-Type-Op
SF:tions:\x20nosniff\r\nX-Frame-Options:\x20DENY\r\nX-Xss-Protection:\x201
SF:;\x20mode=block\r\nDate:\x20Fri,\x2009\x20May\x202025\x2009:04:14\x20GM
SF:T\r\n\r\n<!doctype\x20html><html\x20lang=\"en\"><head><meta\x20charset=
SF:\"utf-8\"/><base\x20href=\"/\"/><meta\x20content=\"width=device-width,i
SF:nitial-scale=1\"\x20name=\"viewport\"/><meta\x20content=\"#081C42\"\x20
SF:media=\"\(prefers-color-scheme:\x20light\)\"\x20name=\"theme-color\"/><
SF:meta\x20content=\"#081C42\"\x20media=\"\(prefers-color-scheme:\x20dark\
SF:)\"\x20name=\"theme-color\"/><meta\x20content=\"MinIO\x20Console\"\x20n
SF:ame=\"description\"/><meta\x20name=\"minio-license\"\x20content=\"agpl\
SF:"/><link\x20href=\"\./s")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnectio
SF:n:\x20close\r\n\r\n400\x20Bad\x20Request")%r(HTTPOptions,702,"HTTP/1\.0
SF:\x20200\x20OK\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x201309\r\n
SF:Content-Security-Policy:\x20default-src\x20'self'\x20'unsafe-eval'\x20'
SF:unsafe-inline';\x20script-src\x20'self'\x20https://unpkg\.com;\x20\x20c
SF:onnect-src\x20'self'\x20https://unpkg\.com;\r\nContent-Type:\x20text/ht
SF:ml\r\nLast-Modified:\x20Fri,\x2009\x20May\x202025\x2009:04:14\x20GMT\r\
SF:nReferrer-Policy:\x20strict-origin-when-cross-origin\r\nServer:\x20MinI
SF:O\x20Console\r\nX-Content-Type-Options:\x20nosniff\r\nX-Frame-Options:\
SF:x20DENY\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Fri,\x2009
SF:\x20May\x202025\x2009:04:14\x20GMT\r\n\r\n<!doctype\x20html><html\x20la
SF:ng=\"en\"><head><meta\x20charset=\"utf-8\"/><base\x20href=\"/\"/><meta\
SF:x20content=\"width=device-width,initial-scale=1\"\x20name=\"viewport\"/
SF:><meta\x20content=\"#081C42\"\x20media=\"\(prefers-color-scheme:\x20lig
SF:ht\)\"\x20name=\"theme-color\"/><meta\x20content=\"#081C42\"\x20media=\
SF:"\(prefers-color-scheme:\x20dark\)\"\x20name=\"theme-color\"/><meta\x20
SF:content=\"MinIO\x20Console\"\x20name=\"description\"/><meta\x20name=\"m
SF:inio-license\"\x20content=\"agpl\"/><link\x20href=\"\./s");
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Vemos muchas cosas interesantes, entre ellas el puerto `80` que aloja una pagina web, si nos metemos dentro veremos un login como de una empresa de logistica.

<img width="1902" height="914" alt="image" src="https://github.com/user-attachments/assets/4a258fce-3987-4559-a646-3b2676f56a53" />

Si probamos credenciales por defecto no funcionaran, tampoco si probamos a realizar un SQLi, pero si inspeccionamos el codigo veremos lo siguiente
```ruby
<input hidden="huguelogistics-data" name="bucket">
```
Veremos esta parte del codigo en el formulario, por lo que vemos esta hablando de bucket y despues en el hidden vemos el siguiente nombre:
```ruby
huguelogistics-data
```
Podemos creer que es un servidor en una nube y ese es el endpoint de dicho servidor, basicamente el nombre del bucket.

Vamos a probar a intentar enumerar las politicas del bucket con dicho nombre a ver si se pudiera enumerar de forma anonima.

# AWS

```ruby
aws --no-sign-request --endpoint-url http://<IP>:9000 s3api get-bucket-policy --bucket huguelogistics-data

{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"AllowPublicReadPolicy\",\"Effect\":\"Allow\",\"Principal\":{\"AWS\":[\"*\"]},\"Action\":[\"s3:GetBucketPolicy\",\"s3:GetObject\",\"s3:ListBucket\"],\"Resource\":[\"arn:aws:s3:::huguelogistics-data\",\"arn:aws:s3:::huguelogistics-data/*\"]}]}"
}
```
Vemos que si nos deja, pero vamos a enumerarlo de forma mas clara y que se vea mejor.
```ruby
aws --no-sign-request --endpoint-url http://<IP>:9000 s3api get-bucket-policy --bucket huguelogistics-data | jq -r '.Policy | fromjson | .Statement[] | {Effect, Principal, Action, Resource, Sid}'
{
  "Effect": "Allow",
  "Principal": {
    "AWS": [
      "*"
    ]
  },
  "Action": [
    "s3:ListBucket",
    "s3:GetBucketPolicy",
    "s3:GetObject"
  ],
  "Resource": [
    "arn:aws:s3:::huguelogistics-data",
    "arn:aws:s3:::huguelogistics-data/*"
  ],
  "Sid": "AllowPublicReadPolicy"
}
```

Vemos que efectivamente todos los usuarios de la nube pueden enumerar las politicas del bucket, tambien veremos que podemos descargarnos cualquier cosa que este subida al bucket con el *, por lo que vamos a realizar lo siguiente:

Vamos a intentar enumerar los archivos o directorios que pueda tener la web.

#  Gobuster
```ruby
gobuster dir -u http://<IP>/ -w <WORDLIST> -x html,php,txt -t 50 -k -r
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,php,txt
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin.php            (Status: 200) [Size: 1953]
/index.php            (Status: 200) [Size: 1953]
/javascript           (Status: 403) [Size: 275]
/logout.php           (Status: 200) [Size: 1953]
/vendor               (Status: 200) [Size: 3104]
/note.txt             (Status: 200) [Size: 13]
/server-status        (Status: 403) [Size: 275]
```
Vemos que hay un archivo llamado note.txt que si entramos dentro veremos lo siguiente:
```ruby
/backup.xlsx
```
Por lo que vamos a intentar descargarnos el archivo en tal caso de que este subido en el bucket.

```ruby
aws --no-sign-request --endpoint-url http://<IP>:9000 s3 cp s3://huguelogistics-data/backup.xlsx .
download: s3://huguelogistics-data/backup.xlsx to ./backup.xlsx
```

Veremos que ha funcionado, por lo que vamos a intentar leer el archivo .xlsx, pero antes para leerlo instalaremos las herramientas necesarias en kali.

```ruby
sudo apt update
sudo apt install --reinstall libreoffice
```
Una vez echo esto, si intentamos abrir el archivo veremos lo siguiente:
```ruby
libreoffice backup.xlsx

<img width="353" height="162" alt="image" src="https://github.com/user-attachments/assets/e2351ef6-9a12-4f4a-ae8f-14d4985e02f8" />

```
