<img width="542" height="300" alt="image" src="https://github.com/user-attachments/assets/5f17e336-57b6-4c19-b353-a5fc0ff79e98" />

#INTRODCUCION 

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
#conectividad
 ping -c1 172.17.0.2
veremos que funciona e manda maquetes 

#Escaneo de Puetos,servicios 

haremos un nmap que nos de los puertos,sevicos,OS

```ruby
nmap -p- -Pn -sVCS --min-rate 5000 172.17.0.2
```
(-p) 
