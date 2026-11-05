# Copias incrementales con `rsync`

En esta actividad hago una copia de unos archivos con `rsync`. Después cambio un archivo y vuelvo a copiar para comprobar que el destino se actualiza.

Los comandos que pongo son ejemplos. No puedo confirmar que sean los mismos que salen en las capturas, así que revisa las rutas y los nombres antes de entregarlo.

## Crear un archivo de prueba

En esta captura creo un archivo vacío con `touch`. Lo usaré para probar la copia.

![Creación del archivo de prueba](../img/touch.png)

```bash
mkdir -p origen destino
touch origen/archivo_prueba.txt
```

## Revisar el origen

Aquí miro qué archivos hay en el origen antes de copiarlos.

![Archivos del origen](../img/origen.png)

```bash
ls -la origen/
```

## Revisar el destino

En esta captura compruebo qué hay en el destino antes de hacer la copia.

![Archivos del destino](../img/destino.png)

```bash
ls -la destino/
```

## Hacer la copia

Ahora copio los archivos del origen al destino. Con `-a` se conservan permisos y fechas, `-v` muestra los archivos que procesa y `-h` muestra los tamaños de forma más clara.

![Primera copia incremental](../img/copia_incremental.png)

```bash
rsync -avh origen/ destino/
```

En esta otra captura se ve una segunda ejecución o la comprobación de la copia.

![Segunda captura de la copia incremental](../img/copia_incremental2.png)

```bash
rsync -avh --itemize-changes origen/ destino/
```

## Comprobar los cambios

Cambio el contenido de un archivo y vuelvo a ejecutar `rsync`. Así puedo comprobar que el destino recibe la modificación.

![Cambios en la copia incremental](../img/cambios_incremental.png)

```bash
echo "Cambio de prueba" >> origen/archivo_prueba.txt
rsync -avh --itemize-changes origen/ destino/
cat destino/archivo_prueba.txt
```

## Comprobar los permisos

En esta captura reviso los permisos del archivo en el origen y en el destino. Uso `stat` para verlos.

![Comprobación de permisos](../img/comprobar_permisos.png)

```bash
stat -c '%A (%a) %n' origen/archivo_prueba.txt destino/archivo_prueba.txt
```

## Recuperar el archivo

Por último, copio el archivo del destino a una carpeta de recuperación y miro su contenido.

![Recuperación del contenido](../img/recuperar_contenido.png)

```bash
mkdir -p recuperacion
cp -a destino/archivo_prueba.txt recuperacion/
cat recuperacion/archivo_prueba.txt
```

Con esta práctica compruebo que `rsync` copia los archivos y actualiza los cambios. También reviso los permisos y recupero un archivo desde el destino.
