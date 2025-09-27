
# Escaneo de puertos 

usaremos el siguinete comando con nmap:

```ruby
nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
```
<img width="535" height="170" alt="image" src="https://github.com/user-attachments/assets/02308f54-d7ed-4f3d-bf54-b539a75b3d47" />

estos nos mostrara los servcios y puertos disponiles 

Nos iremos al puerto 80 como nos indica el nmap veremos lo siguiente:

<img width="1271" height="826" alt="image" src="https://github.com/user-attachments/assets/03052344-ff50-4a68-b4f8-bd0271a47816" />

al darle click a forum nos saltara no estamos autorizados, lo agregaremos a /etc/host ```echo '172.17.0.2 HackZones.hl' >> /etc/hosts ```

<img width="1274" height="751" alt="image" src="https://github.com/user-attachments/assets/2cdfbf55-e986-4ddb-8eaf-7fb95b517f30" />

veremos lo siguiente

<img width="1274" height="751" alt="image" src="https://github.com/user-attachments/assets/52e3af2c-fe38-465c-96db-f5f4bbbbbe51" />

