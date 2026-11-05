# Copias de seguridad con `rsync`

Una copia de seguridad (backup) es un duplicado de los datos que permite recuperarlos si se pierden, se borran o se estropean. Puede ser completa (full), que copia todo, o incremental, que solo copia lo que ha cambiado desde la última vez, con lo que ahorra tiempo y espacio.

`rsync` es una herramienta de Linux pensada para sincronizar y copiar ficheros, tanto en local como entre equipos por red a través de SSH. Es muy usada para backups porque solo transfiere las diferencias entre origen y destino y puede conservar permisos, usuarios y fechas de los ficheros.

## Crear los archivos de prueba

Creo las carpetas `origen` y `destino`, y dentro de origen creo 10 ficheros con `touch`.

![Creación de los archivos de prueba](../img/touch.png)

```bash
mkdir origen
mkdir destino
cd origen
touch file{1..10}
```

## Primera copia (backup full)

Copio el contenido de origen en destino. Con `-a` se conservan permisos, usuarios y fechas, `-v` muestra los archivos que procesa, `-z` comprime durante la transferencia y `-h` muestra los tamaños de forma más clara. Como es la primera vez, copia todos los ficheros.

![Primera copia](../img/copia_incremental.png)

```bash
rsync -avzh origen/ destino/
```

## Con barra y sin barra

No es lo mismo poner `origen/` que `origen`. Con barra copia solo el contenido de la carpeta. Sin barra copia la carpeta entera, y dentro de destino se crea una carpeta nueva llamada `origen`.

![Copia sin barra](../img/copia_incremental2.png)

```bash
rsync -avzh origen destino/
ls destino
```

## Copia incremental

Escribo algo en `file1` y creo el fichero `prueba`. Al volver a ejecutar `rsync`, solo transfiere esos dos ficheros, que son los que han cambiado. Rsync compara el tamaño y la fecha de modificación, y por eso funciona como copia incremental. Con `cat` compruebo que el cambio se ha copiado al destino.

![Copia incremental](../img/cambios_incremental.png)

```bash
echo "prueba backup" > origen/file1
rsync -avzh origen/ destino/
cat destino/file1
```

## Comprobar atributos y usuarios

Comparo origen y destino con `ls -l`. Los permisos (`-rw-rw-r--`), el tamaño, la fecha de modificación y el usuario (`joel joel`) son iguales en los dos, gracias a la opción `-a`. En destino también aparece la carpeta `origen` que se creó al copiar sin barra.

![Comprobación de atributos](../img/comprobar_permisos.png)

```bash
ls -l origen
ls -l destino
```

## Recuperar el contenido

Para recuperar el contenido invierto el origen y el destino. Con `ls` compruebo que vuelve todo a origen, incluidos `prueba` y la carpeta `origen` que había en destino.

![Recuperación del contenido](../img/recuperar_contenido.png)

```bash
rsync -avzh destino/ origen/
ls origen
```

## Copia por red a un equipo remoto

Envío los ficheros a otro equipo (IP `20.0.0.5`) usando SSH. La primera vez pregunta si confío en la clave del host (fingerprint) y la guarda en la lista de equipos conocidos. Después pide la contraseña del usuario remoto y hace la copia.


```bash
rsync -avz origen/ joel@20.0.0.5:/home/joel/destino
```

## Recuperación por red

Para recuperar los datos desde el equipo remoto tiene que estar instalado el servidor SSH en ese equipo. El comando es el mismo que en local, pero indicando el usuario y la IP del equipo remoto como origen.

```bash
sudo apt install openssh-server
rsync -avzh joel@20.0.0.5:/home/joel/destino/ origen/
```
