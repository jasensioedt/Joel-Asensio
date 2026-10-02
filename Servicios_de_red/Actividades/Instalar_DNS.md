Joel Asensio Chavarria
# Informe de instalación de DNS en Ubuntu server


## 1. Datos del servidor

| Elemento | Valor |
|---|---|
| Hostname | `joelasensio` |
| Interfaz NAT | `enp0s3` |
| Interfaz red interna | `enp0s8` |
| IP estática del servidor | `192.168.1.110/24` |
| Dominio | `joel.myguest.virtualbox.org` |

## 2. Instalación del paquete

```bash
sudo apt update
sudo apt install bind9 
```

## 3. Configuración de red (Netplan)

Fichero: `/etc/netplan/50-cloud-init.yaml`

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.1.110/24
```

Aplicar los cambios:

```bash
sudo netplan apply
```

## 4. Opciones generales de BIND9

Fichero: `/etc/bind/named.conf.options`

```
acl "safeclients" {
        localhost;
        192.168.1.110;
        localnets;
};

options {
        directory "/var/cache/bind";

        recursion yes;
        allow-recursion { safeclients; };
        listen-on { 192.168.1.110; };
        allow-transfer { none; };

        allow-query { safeclients; };
        allow-query-cache { safeclients; };

        forwarders {
                1.1.1.1;
                8.8.8.8;
        };
};
```

## 5. Declaración de zonas

Fichero: `/etc/bind/named.conf.local`

```
// Directas
zone "myguest.virtualbox.org" in {
  type master;
  file "/etc/bind/zones/db.myguest.virtualbox.org";
};

// Inversas
zone "1.168.192.in-addr.arpa" in {
  type master;
  file "/etc/bind/zones/db.1.168.192";
};
```

## 6. Fichero de zona directa

Fichero: `/etc/bind/zones/db.myguest.virtualbox.org`

```
$TTL    604800
@       IN      SOA     myguest.virtualbox.org. joel.myguest.virtualbox.org. (
                        2               ; Serial
                        12h             ; Refresh
                        15m             ; Retry
                        3w              ; Expire
                        2h              ; Negative TTL
                        )

; Name servers - NS records
@       IN      NS      joel.myguest.virtualbox.org.

; A records
myguest IN      A       192.168.1.110
joel    IN      A       192.168.1.110
```

## 7. Fichero de zona inversa

Fichero: `/etc/bind/zones/db.1.168.192`

```
$TTL    604800
@       IN      SOA     myguest.virtualbox.org. joel.myguest.virtualbox.org. (
                        2               ; Serial
                        12h             ; Refresh
                        15m             ; Retry
                        3w              ; Expire
                        2h              ; Negative TTL
                        )

; Name servers
@       IN      NS      joel.myguest.virtualbox.org.

; PTR Records
110     IN      PTR     joel.myguest.virtualbox.org.
```

## 8. Configuración de resolv

Fichero: `/etc/resolv.conf`

```
nameserver 192.168.1.110
search .
```

## 9. Verificación de ficheros

Comprobar la sintaxis de la configuración y de las zonas:

```bash
sudo named-checkconf
sudo named-checkzone myguest.virtualbox.org /etc/bind/zones/db.myguest.virtualbox.org
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.1.168.192
```

Reiniciar y comprobar el estado de BIND9:

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

## 10. Pruebas de funcionamiento

### Resolución directa

```bash
nslookup joel.myguest.virtualbox.org
```

Resultado obtenido:

```
Server:         192.168.1.110
Address:        192.168.1.110#53

Name:   joel.myguest.virtualbox.org
Address: 192.168.1.110
```

El servidor responde desde `192.168.1.110#53` y resuelve `joel.myguest.virtualbox.org` a `192.168.1.110`, por lo que la **zona directa funciona correctamente**.

### Resolución inversa

```bash
nslookup 192.168.1.110
```

Resultado obtenido:

```
110.1.168.192.in-addr.arpa      name = joel.myguest.virtualbox.org.
```

La IP `192.168.1.110` se resuelve al nombre `joel.myguest.virtualbox.org`, por lo que la **zona inversa funciona correctamente**.

## 11. Problemas durante la practica

### Error NXDOMAIN en la resolución inversa

Al ejecutar `nslookup 192.168.1.110` apareció el siguiente error:

```
server can't find 110.1.168.192.in-addr.arpa: NXDOMAIN
```

**Causa:** el registro PTR de la zona inversa estaba definido como `100`, que corresponde a la IP `192.168.1.100`. Como la IP del servidor es `192.168.1.110`, no existía ningún registro PTR para ella.

**Solución:**

Cambiar el registro PTR de `100` a `110` en `/etc/bind/zones/db.1.168.192`.




