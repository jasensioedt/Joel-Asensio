# Actividad: copias de seguridad incrementales con `rsync`

## Introducción

En esta actividad utilizo `rsync` para copiar archivos de un directorio de origen a otro de destino. Después hago cambios en el origen y vuelvo a ejecutar la copia para comprobar que el destino se actualiza. También reviso los permisos y compruebo que puedo recuperar un archivo.

> **Importante:** Los comandos de abajo son ejemplos coherentes con el nombre y el propósito de cada captura. Cambia las rutas, los nombres de archivo y el usuario por los que aparecen en tu práctica. En los comandos con `origen/` y `destino/`, se supone que ambos directorios están en la carpeta actual.

## 1. Crear un archivo de prueba

En esta captura se muestra la creación de un archivo vacío con `touch`. Sirve para tener un archivo de prueba que después se pueda copiar y localizar en el destino.

```bash
mkdir -p origen destino
cd origen
touch archivo_prueba.txt
cd ..
```

## 2. Comprobar el directorio de origen

Aquí reviso los archivos que hay en el origen antes de hacer la copia. `ls -la` muestra también los archivos ocultos y los permisos.

```bash
ls -la origen/
```

## 3. Comprobar el directorio de destino

Esta captura permite ver el estado inicial del destino. Si está vacío, todavía no contiene los archivos del origen.

```bash
ls -la destino/
```

## 4. Hacer la primera copia con `rsync`

En esta parte copio el contenido del directorio de origen al destino. Las opciones `-a` conservan atributos como permisos y fechas, `-v` muestra los archivos procesados y `-h` presenta los tamaños de forma legible.

```bash
rsync -avh origen/ destino/
```

La barra final de `origen/` indica que se copia el contenido del directorio. Después compruebo que el archivo está en el destino:

```bash
ls -la destino/
```

## 5. Volver a ejecutar la copia incremental

La captura documenta otra ejecución de `rsync`. Si no hay cambios, el programa no necesita volver a transferir los archivos que ya están actualizados. La opción `--itemize-changes` ayuda a mostrar qué cambios detecta.

```bash
rsync -avh --itemize-changes origen/ destino/
```

## 6. Modificar un archivo y sincronizar los cambios

Para comprobar la actualización incremental, añado una línea al archivo del origen y vuelvo a ejecutar `rsync`. En la salida se puede observar que detecta el archivo modificado.

```bash
echo "Cambio realizado para probar la copia incremental" >> origen/archivo_prueba.txt
rsync -avh --itemize-changes origen/ destino/
```

Después puedo comprobar que el cambio también aparece en la copia:

```bash
cat destino/archivo_prueba.txt
```

## 7. Comprobar los permisos

En esta captura reviso los permisos del archivo en el origen y en el destino. `stat` muestra el modo numérico y simbólico, además del nombre del archivo. Como `rsync` se ejecutó con `-a`, se espera que conserve los permisos, siempre que el sistema de archivos y los permisos del usuario lo permitan.

```bash
stat -c '%A (%a) %n' origen/archivo_prueba.txt destino/archivo_prueba.txt
```

## 8. Recuperar el contenido de la copia

La última captura representa la recuperación de un archivo guardado en el destino. En este ejemplo lo copio a un directorio de recuperación y luego leo su contenido.

```bash
mkdir -p recuperacion
cp -a destino/archivo_prueba.txt recuperacion/
cat recuperacion/archivo_prueba.txt
```

## Conclusión

Con esta práctica comprobé que `rsync` puede copiar el contenido de un directorio y actualizar el destino cuando cambia un archivo. También revisé el resultado, comparé los permisos y recuperé un archivo desde la copia. Para repetir la actividad, hay que adaptar las rutas y los nombres a los utilizados en el equipo.
