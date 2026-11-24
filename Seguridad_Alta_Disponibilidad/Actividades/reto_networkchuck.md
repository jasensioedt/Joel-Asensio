
Joel Asensio

## 1. Objetivo

Conseguir la contraseña y el usuario de una maquina en la red usando wordlists con **hydra**.

## 2. Wireshark

Teniamos un archivo pcap como pista aparte del video de networkchuck.
En wireshark tenemos que filtrar por ftp-data y seleccionar los paquetes que contienen password.txt y usuarios.txt
Para descargarlos tenemos que hacer click derecho en el paquete --> Seguir --> TCP stream y despues guardarlo con el nombre que quieres para el archivo.

---

## 3. Hydra

Para saber el usuario y la contraseña que tiene tenemos que usar hydra, el comando que hemos usado es: 
```bash
sudo hydra -L usuarios.txt -P password.txt 192.168.111.55 ssh
```
Esto hace que utilice el archivo usuarios.txt para adivinar el usuario y el archivo password.txt para adivinar la contraseña, esto daba error y he estado buscando y es porque con ssh por defecto lo hace con 16 hilos y da fallos de conexion, lo que se recomienda es bajarlo a 4 hilos con -t 4 al final del comando como este: 
```bash
sudo hydra -L usuarios.txt -P password.txt -t 4 192.168.111.55 ssh
```

