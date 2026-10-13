# Playbook de Despliegue Técnico: OpenLDAP en Ubuntu

## A. Ficha Técnica y Prerrequisitos del Entorno

### A.1 Tabla resumen

| | |
|---|---|
| Topología de red | Red NAT `NatJoel` (20.0.0.0/24) con DHCP deshabilitado |
| Sistema operativo | Ubuntu Desktop 24.04 LTS |
| Dirección IP | 20.0.0.5/24 (estática, Netplan) |
| FQDN | ldapserver.planetafp.local |
| Dominio del directorio (DIT) | planetafp.local (`dc=planetafp,dc=local`) |
| Organización | planetafp |
| Paquetes a instalar | slapd, ldap-utils |

Red NAT creada en VirtualBox, sin DHCP:

![Red NAT en VirtualBox](./img/Playbook/crear_nat.png)

Versión del sistema operativo:

![Versión del SO](lsb_release.png)

### A.2 Descripción del servicio

LDAP es un servicio de directorio. Guarda la información (usuarios, grupos, equipos...) ordenada en forma de árbol, el DIT (Directory Information Tree), de forma parecida a las antiguas páginas amarillas. Está pensado para consultar datos muy rápido y para que muchas aplicaciones usen la misma fuente de información, por ejemplo para autenticar usuarios.

Cada entrada del árbol se identifica con un DN (nombre distinguido), que es su ruta completa. Por ejemplo: `uid=jmartinez,ou=DireccionAcademica,dc=planetafp,dc=local`.

Componentes principales:

- **slapd:** el demonio del servidor LDAP. Es el programa que se queda funcionando y responde a las consultas.
- **ldap-utils:** los programas cliente para trabajar con el directorio, como `ldapsearch`, `ldapadd`, `ldapdelete` o `ldapmodify`.
- **libldap:** las bibliotecas que implementan el protocolo LDAP (junto con liblber).

---

## B. Procedimiento de Despliegue Paso a Paso

### Paso 1. Configuración de red estática

El servidor necesita siempre la misma IP para que los clientes lo encuentren, así que se desactiva DHCP y se define la IP a mano en Netplan.

```
sudo nano /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 20.0.0.5/24
      routes:
        - to: default
          via: 20.0.0.1
      nameservers:
        addresses:
          - 8.8.8.8
```

```
sudo chmod 600 /etc/netplan/*.yaml
sudo netplan apply
ip a
```

Fichero de Netplan:

![Fichero Netplan](netplan.png)

Aplicación y comprobación de la IP:

![netplan apply e ip a](netplan_ip.png)

### Paso 2. Resolución de nombres local

Se añade una línea en `/etc/hosts` que une la IP con el FQDN y el alias. Es necesaria para poder llamar al servidor por su nombre sin depender de un servidor DNS.

```
sudo nano /etc/hosts
```

Línea añadida:

```
20.0.0.5    ldapserver.planetafp.local    ldapserver
```

![Fichero hosts](hosts.png)

Comprobación de que el nombre resuelve:

```
ping -c 4 ldapserver.planetafp.local
```

![Ping al FQDN](ping.png)

### Paso 3. Instalación de software

```
sudo apt update
sudo apt install slapd ldap-utils -y
```

### Paso 4. Reconfiguración del servicio

Al instalar, slapd se configura con valores por defecto. Con `dpkg-reconfigure` se vuelve a configurar con los datos de la práctica.

```
sudo dpkg-reconfigure slapd
```

| Pregunta | Respuesta |
|---|---|
| ¿Omitir la configuración del servidor OpenLDAP? | No |
| Nombre de dominio DNS | planetafp.local |
| Nombre de la organización | planetafp |
| Contraseña de administrador | La elegida (no se muestra) |
| ¿Borrar la base de datos al purgar slapd? | No |
| ¿Mover la base de datos antigua? | Yes |

Nombre de dominio:

![Dominio](dominio.png)

Organización:

![Organización](organizacion.png)

Borrar la base de datos al purgar:

![Borrar BD](borrar_bd.png)

Mover la base de datos antigua:

![Mover BD](mover_bd.png)

---

## C. Verificación y Comprobación

### C.1 Estado del demonio

```
sudo systemctl status slapd
```

El servicio aparece como `active (running)`:

![Estado de slapd](status_slapd.png)

### C.2 Esquemas del sistema

No hay que configurar ningún esquema, ya vienen con slapd. Se comprueba que están presentes:

```
ls /etc/ldap/schema
```

Se ven, entre otros, `core.schema`, `cosine.schema`, `inetorgperson.schema` y `nis.schema`:

![Esquemas del sistema](esquemas.png)

---

## D. Población del Directorio y Operaciones Básicas

### Estructura prevista del directorio

```
dc=planetafp,dc=local
├── ou=DireccionAcademica
│   ├── uid=jmartinez (Director Pedagógico)
│   └── uid=mgarcia (Jefa de Estudios)
├── ou=Departamentos
│   ├── ou=Adisiones
│   │   ├── uid=lperez
│   │   └── uid=asanchez
│   ├── ou=CampusVirtual
│   │   ├── uid=cmoreno
│   │   └── uid=nfernandez
│   ├── ou=Secretaria
│   │   ├── uid=rjimenez
│   │   └── uid=pvidal
│   └── ou=SoporteTecnico
│       ├── uid=jroca
│       └── uid=anavarro
├── ou=AreasFormativas
│   ├── ou=Informatica
│   │   ├── uid=aruiz (Tutor DAM)
│   │   └── uid=dortega (Tutor DAW)
│   ├── ou=Empresa
│   │   ├── uid=jmolina (Tutor Admin y Finanzas)
│   │   └── uid=mvazquez (Tutor Marketing)
│   └── ou=Sanidad
│       ├── uid=sramos (Tutor Higiene Bucodental)
│       └── uid=cgil (Tutor TCAE)
└── ou=GestionFCT
    ├── ou=RelacionesEmpresa
    │   ├── uid=ibenitez
    │   └── uid=jflores
    └── ou=TutoresFCT
        ├── uid=kpuig
        └── uid=lmora
```

El LDIF de prueba de este playbook crea la unidad `DireccionAcademica` y el usuario `jmartinez`, que es la primera rama del árbol.

### D.1 Fichero LDIF

Un LDIF es un fichero de texto con las entradas del directorio: primero el `dn`, después sus atributos, y una línea en blanco entre una entrada y otra.

La entrada del dominio `dc=planetafp,dc=local` ya la creó `dpkg-reconfigure`, por eso está comentada en el fichero. El LDIF crea una unidad organizativa y un usuario:

```
nano ldif/usuarios.ldif
```

```
# Crear OUs y usuarios

# Definir dominio

#dn: dc=planetafp,dc=local
#objectclass: top
#objectclass: domain
#dc: planetafp

# Direccion Academica

dn: ou=DireccionAcademica,dc=planetafp,dc=local
objectClass: organizationalUnit
objectClass: top
ou: DireccionAcademica

dn: uid=jmartinez,ou=DireccionAcademica,dc=planetafp,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
cn: Javier Martinez
sn: Martinez
uid: jmartinez
uidNumber: 10001
gidNumber: 5000
homeDirectory: /home/jmartinez
```

![Fichero LDIF](ldif.png)

### D.2 Inserción con ldapadd

```
sudo ldapadd -x -D "cn=admin,dc=planetafp,dc=local" -W -f ldif/usuarios.ldif
```

- `-x`: autenticación simple
- `-D`: DN del usuario con el que se hace el bind (el administrador)
- `-W`: pide la contraseña
- `-f`: carga las entradas desde el fichero

![ldapadd](ldapadd.png)

### D.3 Consulta con ldapsearch

```
ldapsearch -x -b "dc=planetafp,dc=local" -s sub "(uid=jmartinez)"
```

- `-b`: base desde donde empieza la búsqueda
- `-s sub`: ámbito, busca en la base y en todo lo que cuelga de ella
- `(uid=jmartinez)`: filtro, la entrada que se busca

![ldapsearch](ldapsearch.png)

Árbol del directorio:

![Árbol LDAP](arbol_ldap.png)

---

## E. Control de Errores y Troubleshooting

### Error 1. ldapadd: la entrada ya existe

Al cargar el LDIF por primera vez, con la entrada del dominio sin comentar:

```
adding new entry "dc=planetafp,dc=local"
ldap_add: Already exists (68)
```

![Error Already exists](error1_ldap.png)

**Causa:** el LDIF intentaba crear `dc=planetafp,dc=local`, pero esa entrada ya la había creado `dpkg-reconfigure slapd`. No se puede añadir dos veces un mismo DN, y `ldapadd` se para en el primer fallo.

**Solución:** comentar con `#` las líneas de la entrada del dominio en el LDIF y volver a ejecutar el mismo `ldapadd`. Así solo se crean la unidad organizativa y el usuario.

### Error 2. Netplan: la ruta no tiene `via`

Al aplicar la configuración de red:

```
Error in network definition: global unicast route must include both a 'to' and 'via' IP
- to: default
```

**Causa:** la ruta por defecto tenía `to: default` pero le faltaba la línea `via:` con la puerta de enlace, o estaba mal alineada.

**Solución:** añadir `via: 20.0.0.1` justo debajo de `to: default`, con la misma indentación, y volver a aplicar:

```
sudo netplan apply
```
