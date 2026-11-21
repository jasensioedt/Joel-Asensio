Joel Asensio
# Práctica: Nginx confinado con AppArmor

## Objetivo

Instalar Nginx, crear un perfil de AppArmor para confinarlo, probarlo primero en modo de aprendizaje (complain) y después activarlo de verdad (enforce), comprobando que el servicio sigue funcionando.

## Entorno

- Ubuntu Server (máquina `joelubuntu`)
- Nginx instalado desde los repos de Ubuntu
- Paquetes `apparmor` y `apparmor-utils`

## 1. Instalación de Nginx

```bash
sudo apt update
sudo apt install -y nginx apparmor apparmor-utils apparmor-profiles
sudo systemctl enable --now nginx
```

Al arrancar dio este error:

```
nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
```

El puerto 80 ya lo tenía ocupado Apache2, que venía instalado de antes:

```bash
sudo ss -tlnp | grep :80
sudo systemctl stop apache2
sudo systemctl disable apache2
sudo systemctl restart nginx
curl -I http://localhost/
```

Después de parar Apache2, Nginx arrancó bien y respondió `200 OK`.

## 2. Generar el perfil con aa-genprof (modo complain)

```bash
sudo aa-genprof /usr/sbin/nginx
```

Mientras se quedaba esperando, en otra terminal se le dio tráfico de prueba para que detectara todo lo que usa:

```bash
sudo systemctl reload nginx
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/no-existe
sudo nginx -s reopen
```

Con eso, `aa-genprof` fue detectando los accesos que hace Nginx (su fichero de configuración, el fichero de passwd para buscar el usuario www-data, el módulo de nginx, las señales HUP y USR1 que usa para recargar y reabrir logs...) y se fueron aceptando uno por uno con "Allow", hasta guardar el perfil.

**Cosa importante que pasó:** cuando terminas con `aa-genprof`, el perfil se queda guardado directamente en modo **enforce**, no en complain como cabría esperar. Hubo que forzarlo a mano:

```bash
sudo aa-complain /usr/sbin/nginx
sudo systemctl restart nginx
sudo aa-status | grep -B2 -A5 "complain mode"
```

Y ahí ya salía bien, con nginx en la lista de perfiles y procesos en modo complain.

## 3. Logs en modo complain

```bash
sudo journalctl -k --no-pager | grep -i apparmor | grep -i nginx | tail -20
```

Salen muchas líneas tipo `apparmor="ALLOWED"`, porque en complain no se bloquea nada, solo se apunta todo lo que el programa intenta hacer. Un par de ejemplos reales:

```
apparmor="ALLOWED" operation="accept" class="net" profile="/usr/sbin/nginx" lport=80 family="inet6" requested="accept" denied="accept"
apparmor="ALLOWED" operation="file_perm" class="file" profile="/usr/sbin/nginx" name="/var/log/nginx/access.log" requested_mask="w" denied_mask="w"
```

Fijándose bien, en esas líneas `denied_mask` es igual que `requested_mask`: eso quiere decir que si lo hubiéramos tenido en enforce, esas acciones se habrían bloqueado de verdad. El motivo era que el perfil no cubría bien las conexiones por IPv6 (`::1`, que es a donde apunta `localhost`). Para no complicarlo, se quitó directamente el listener IPv6 de Nginx:

```bash
sudo sed -i 's/^\tlisten \[::\]:80 default_server;/\t# listen [::]:80 default_server;/' /etc/nginx/sites-available/default
sudo nginx -t
sudo systemctl restart nginx
```

## 4. Pasar a enforce

```bash
sudo aa-enforce /usr/sbin/nginx
sudo systemctl restart nginx
```

Al activar enforce con el perfil tal cual lo había dejado `aa-genprof`, Nginx dejó de arrancar. Hubo que ir corrigiendo el perfil poco a poco, cada vez que fallaba por un motivo distinto:

1. `open() "/run/nginx.pid" failed (Permission denied)` → el perfil solo tenía permiso de lectura sobre el pid, hacía falta también escritura: `owner /run/nginx.pid rw,`
2. `open() "/var/log/nginx/error.log" failed` + un DENIED de capability `dac_override` → faltaba darle esa capability y permiso para escribir en los logs: `capability dac_override,` y `/var/log/nginx/*.log rw,`
3. `socket() 0.0.0.0:80 failed (Permission denied)` → faltaba la capability para poder escuchar en un puerto por debajo de 1024: `capability net_bind_service,`

Cada vez que se tocaba el perfil:

```bash
sudo nano /etc/apparmor.d/usr.sbin.nginx
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
sudo systemctl restart nginx
curl -I http://localhost/
```

## 5. Perfil final

```
abi <abi/3.0>,

include <tunables/global>

/usr/sbin/nginx {
  include <abstractions/base>

  capability dac_override,
  capability net_bind_service,

  signal send set=hup peer=unconfined,
  signal send set=usr1 peer=unconfined,

  /usr/sbin/nginx mr,
  owner /etc/group r,
  owner /etc/nginx/conf.d/ r,
  owner /etc/nginx/mime.types r,
  owner /etc/nginx/modules-enabled/ r,
  owner /etc/nginx/nginx.conf r,
  owner /etc/nginx/sites-available/default r,
  owner /etc/nginx/sites-enabled/ r,
  owner /etc/nsswitch.conf r,
  owner /etc/passwd r,
  owner /run/nginx.pid rw,
  /var/log/nginx/ r,
  /var/log/nginx/*.log rw,
}
```

## 6. Comprobación final

```bash
sudo aa-status | grep -B2 -A2 nginx
systemctl is-active nginx
curl -I http://localhost/
```


## Complain vs enforce, la diferencia en la práctica

En complain no se bloquea nada, solo se registra. Sirve para ver qué necesita el programa sin arriesgarse a romperlo. En enforce, en cambio, todo lo que no esté en el perfil se bloquea de verdad. Esto se vio clarísimo en esta práctica: el mismo perfil que en complain funcionaba sin ningún aviso (todo salía como ALLOWED), al pasar a enforce hizo que Nginx no arrancara, porque esos accesos que antes solo se registraban pasaron a denegarse.

## Qué accesos hubo que permitir y por qué

- **Pid con lectura y escritura**: Nginx tiene que escribir su propio PID cada vez que arranca, no solo leerlo.
- **Capability dac_override**: Nginx arranca como root y necesita poder escribir en ficheros que no son suyos (como los logs, que pertenecen a www-data) antes de bajar privilegios.
- **Capability net_bind_service**: para poder abrir el puerto 80, que está por debajo de 1024 y requiere permiso especial.
- **Lectura y escritura en /var/log/nginx**: sin esto no puede generar access.log ni error.log.
- **El lío de Apache2**: no tiene que ver con AppArmor, pero fue el primer problema real: dos servicios no pueden usar el mismo puerto a la vez.
- **IPv6**: el perfil generado automáticamente no cubría bien las conexiones por `::1`, así que se optó por quitar ese listener en vez de complicar el perfil.
