Joel Asensio Chavarria 
---
# Informe: configuración de un servidor DNS esclavo (secundario) con BIND9

## 1. Esquema de la arquitectura

```
                    Transferencia de zona (AXFR)
                 192.168.1.110  ──────────────▶  192.168.1.111
        ┌───────────────────────┐          ┌───────────────────────┐
        │   MASTER               │          │   ESCLAVO              │
        │   joelubuntuserver      │          │   joeldnsesclavo        │
        │   named.conf.local:     │          │   named.conf.local:     │
        │     type master;        │          │     type slave;         │
        │     allow-transfer      │          │     masters {           │
        │       { 192.168.1.111;} │          │       192.168.1.110; }  │
        │                         │          │                         │
        │   Zonas:                │          │   Zonas (copia local):  │
        │   myguest.virtualbox.org│          │   /etc/bind/zones/slave/│
        │   1.168.192.in-addr.arpa│          │   db.myguest.virtualbox │
        └───────────────────────┘          │   db.1.168.192          │
                                              └───────────────────────┘
                     ▲                                  ▲
                     │                                  │
                     └──────────── clientes DNS ─────────┘
```

El master es la única fuente editable de las zonas (`type master`); el esclavo solo guarda una copia de solo lectura que actualiza automáticamente cada vez que el master le notifica un cambio (`sending notifies`).

## 2. Configuración en el servidor master (`joelubuntuserver`)

### 2.1 Permitir la transferencia de zona hacia el esclavo

Fichero: `/etc/bind/named.conf.local`

```
// Directas
zone "myguest.virtualbox.org" in {
  type master;
  file "/etc/bind/zones/db.myguest.virtualbox.org";
  allow-transfer { 192.168.1.111; };
};

// Inversas
zone "1.168.192.in-addr.arpa" in {
  type master;
  file "/etc/bind/zones/db.1.168.192";
  allow-transfer { 192.168.1.111; };
};
```

`allow-transfer` restringe la transferencia de zona únicamente a la IP del esclavo (`192.168.1.111`), en lugar de dejarla abierta a cualquier cliente.

### 2.2 Añadir los registros del esclavo a las zonas

Fichero: `/etc/bind/zones/db.myguest.virtualbox.org`

```
$TTL    604800
@       IN      SOA     myguest.virtualbox.org. joel.myguest.virtualbox.org. (
                        2               ; se = Serial
                        12h             ; ref = Refresh
                        15m             ; ret = Retry
                        3w              ; ex = Expire
                        2h              ; nx = nxdomain ttl
                        )

;name servers - NS records
@       IN      NS      joel.myguest.virtualbox.org.
@       IN      NS      joelslave.myguest.virtualbox.org.

;name servers - A records
myguest   IN      A       192.168.1.110
joel      IN      A       192.168.1.110
joelslave IN      A       192.168.1.111
```

Fichero: `/etc/bind/zones/db.1.168.192`

```
$TTL    604800
@       IN      SOA     myguest.virtualbox.org. joel.myguest.virtualbox.org. (
                        2               ; se = Serial
                        12h             ; ref = Refresh
                        15m             ; ret = Retry
                        3w              ; ex = Expire
                        2h              ; nx = nxdomain TTL
                        )

; name servers
@       IN      NS      joel.myguest.virtualbox.org.

; PTR Records
110     IN      PTR     joel.myguest.virtualbox.org.
111     IN      PTR     joelslave.myguest.virtualbox.org.
```

Se añade el registro NS y A/PTR del esclavo (`joelslave`, `192.168.1.111`) y se incrementa el `Serial` para que el master detecte el cambio y lo transfiera.

### 2.3 Reiniciar BIND9 en el master

```bash
sudo systemctl restart bind9
```

## 3. Configuración en el servidor esclavo (`joeldnsesclavo`)

### 3.1 Hostname y resolución local

Fichero: `/etc/hosts`

```
127.0.0.1 joel.myguest.virtualbox.org joeldnsesclavo
```

### 3.2 Opciones generales de BIND9

Fichero: `/etc/bind/named.conf.options`

```
acl "safeclients" {
        localhost;
        192.168.1.0;
        localnets;
};

options {
        directory "/var/cache/bind";

        recursion yes;
        allow-recursion { safeclients; };
        listen-on { 192.168.1.111; };
        allow-transfer { none; };

        allow-query { safeclients; };
        allow-query-cache { safeclients; };

        forwarders {
                1.1.1.1;
                8.8.8.8;
        };
};
```

### 3.3 Declaración de las zonas como esclavas

Fichero: `/etc/bind/named.conf.local`

```
// Directas
zone "myguest.virtualbox.org" in {
  type slave;
  masters { 192.168.1.110; };
  file "/etc/bind/zones/slave/db.myguest.virtualbox.org";
};

// Inversas
zone "1.168.192.in-addr.arpa" in {
  type slave;
  masters { 192.168.1.110; };
  file "/etc/bind/zones/slave/db.1.168.192";
};
```

A diferencia del master, el esclavo no edita directamente el fichero de zona: `type slave` y `masters { 192.168.1.110; }` indican que debe obtener (transferir) el contenido desde el master, y guardarlo en `/etc/bind/zones/slave/`.

```bash
sudo mkdir -p /etc/bind/zones/slave
sudo chown bind:bind /etc/bind/zones/slave
```

### 3.4 Reiniciar BIND9 en el esclavo

```bash
sudo systemctl restart bind9
```

## 4. Verificación de la transferencia de zona

```bash
sudo journalctl -u named -f
```

Salida obtenida en el esclavo:

```
oct 08 15:16:39 joeldnsesclavo named[981]: zone 127.in-addr.arpa/IN: loaded serial 1
oct 08 15:16:39 joeldnsesclavo named[981]: zone 1.168.192.in-addr.arpa/IN: loaded serial 2
oct 08 15:16:39 joeldnsesclavo named[981]: zone localhost/IN: loaded serial 2
oct 08 15:16:39 joeldnsesclavo named[981]: all zones loaded
oct 08 15:16:39 joeldnsesclavo named[981]: systemd[1]: Started named.service - BIND Domain Name Server.
oct 08 15:16:39 joeldnsesclavo named[981]: running
oct 08 15:16:39 joeldnsesclavo named[981]: zone 1.168.192.in-addr.arpa/IN: sending notifies (serial 2)
oct 08 15:16:39 joeldnsesclavo named[981]: zone myguest.virtualbox.org/IN: sending notifies (serial 2)
oct 08 15:16:39 joeldnsesclavo named[981]: managed-keys-zone: Key 20326 for zone . is now trusted (accepted)
oct 08 15:16:39 joeldnsesclavo named[981]: managed-keys-zone: Key 38696 for zone . is now trusted (accepted)
```

El esclavo carga las zonas `1.168.192.in-addr.arpa` y `myguest.virtualbox.org` con el `serial 2` recibido del master (`all zones loaded`) y confirma el servicio en marcha (`running`), por lo que la **transferencia de zona desde el master funciona correctamente**.

## 5. Capturas

### 5.1 Zona inversa en el master (con el registro del esclavo)

![Zona inversa en el master](../img/dns/master/zona_inversa.png)

### 5.2 Zona directa en el master (con el registro del esclavo)

![Zona directa en el master](../img/dns/master/zona_directa.png)

### 5.3 named.conf.local del master (allow-transfer)

![named.conf.local del master](../img/dns/master/named_conf_local.png)

### 5.4 named.conf.local del esclavo (zonas tipo slave)

![named.conf.local del esclavo](../img/dns/slave/named_conf_local.png)

### 5.5 named.conf.options del esclavo

![named.conf.options del esclavo](../img/dns/slave/named_conf_options.png)

### 5.6 /etc/hosts del esclavo

![hosts del esclavo](../img/dns/slave/hosts.png)

### 5.7 Verificación con journalctl en el esclavo

![journalctl del esclavo](../img/dns/slave/journalctl.png)

## 6. Conclusión

El servidor esclavo `joeldnsesclavo` (`192.168.1.111`) queda configurado como secundario de `joelubuntuserver`, replicando por transferencia de zona (AXFR) tanto la zona directa `myguest.virtualbox.org` como la inversa `1.168.192.in-addr.arpa`. El master restringe la transferencia solo a la IP del esclavo (`allow-transfer`), y el log de `journalctl` confirma que el esclavo recibió y cargó ambas zonas con el serial correcto, quedando operativo como servidor DNS de respaldo.
