Joel Asensio Chavarria
# Informe de instalación de DNS en Ubuntu server


## 1. Datos del servidor

| Elemento | Valor |
|---|---|
| Hostname | `joelubuntuserver` |
| Interfaz NAT | `enp0s3` |
| Interfaz red interna | `enp0s8` |
| IP estática del servidor | `192.168.1.110/24` |
| Dominio | `myguest.virtualbox.org` |
| Zona inversa | `1.168.192.in-addr.arpa` |
| Forwarders | `1.1.1.1`, `8.8.8.8` |

## 3. Instalación del paquete

```bash
sudo apt update
sudo apt install bind9 
```

## 4. Configuración de red (Netplan)

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

## 5. Opciones generales de BIND9

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

**Resumen de las directivas:**

| Directiva | Función |
|---|---|
| `acl "safeclients"` | Define los clientes de confianza (localhost, el propio servidor y la red local). |
| `recursion yes` | Permite resolver consultas recursivas. |
| `allow-recursion` / `allow-query` / `allow-query-cache` | Limitan el servicio a `safeclients`. |
| `listen-on` | El servidor escucha solo en `192.168.1.110`. |
| `allow-transfer { none; }` | Deshabilita las transferencias de zona. |
| `forwarders` | Reenvía las consultas externas a Cloudflare (`1.1.1.1`) y Google (`8.8.8.8`). |

## 6. Declaración de zonas

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

Crear el directorio de zonas:

```bash
sudo mkdir -p /etc/bind/zones
```

## 7. Fichero de zona directa

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

## 8. Fichero de zona inversa

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
100     IN      PTR     joel.myguest.virtualbox.org.
```

## 9. Configuración del resolvedor local

Fichero: `/etc/resolv.conf`

```
nameserver 192.168.1.110
search .
```

## 10. Verificación y puesta en marcha

Comprobar la sintaxis de la configuración y de las zonas:

```bash
sudo named-checkconf
sudo named-checkzone myguest.virtualbox.org /etc/bind/zones/db.myguest.virtualbox.org
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.1.168.192
```

Reiniciar y habilitar el servicio:

```bash
sudo systemctl restart bind9
sudo systemctl enable bind9
sudo systemctl status bind9
```

## 11. Pruebas de funcionamiento

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

## 12. Observaciones

- El registro PTR de la zona inversa está definido como `100`, es decir, `192.168.1.100`. Como la IP del servidor es `192.168.1.110`, para que la resolución inversa del servidor funcione el registro debería ser `110`:

```
  110     IN      PTR     joel.myguest.virtualbox.org.
```

  Tras modificarlo, incrementar el número de serie (`Serial`) y reiniciar con `sudo systemctl restart bind9`.

- Al modificar cualquier fichero de zona, hay que incrementar el `Serial` del SOA.

## 13. Conclusión

El servidor DNS BIND9 queda instalado en `192.168.1.110`, con zona directa e inversa para `myguest.virtualbox.org`, recursividad restringida a la red local y reenvío de consultas externas a `1.1.1.1` y `8.8.8.8`. La prueba con `nslookup` confirma que la resolución directa funciona.
