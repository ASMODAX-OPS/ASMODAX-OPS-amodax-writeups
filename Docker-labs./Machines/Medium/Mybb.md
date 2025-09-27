
# Escaneo de puertos 

usaremos el siguinete comando con nmap:

```ruby
nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
```
<img width="535" height="170" alt="image" src="https://github.com/user-attachments/assets/02308f54-d7ed-4f3d-bf54-b539a75b3d47" />

estos nos mostrara los servcios y puertos disponiles 

Nos iremos al puerto 80 como nos indica el nmap veremos lo siguiente:

<img width="1334" height="757" alt="image" src="https://github.com/user-attachments/assets/dbc1e32d-e0ea-4a98-a2af-9e1e87563ff5" />

al darle click a forum nos saltara no estamos autorizados, lo agregaremos a /etc/host ```bash  echo '172.17.0.2 HackZones.hl' >> /etc/hosts ```
