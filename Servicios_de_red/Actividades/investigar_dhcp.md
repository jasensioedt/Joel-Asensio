# ACT4 – Investigar Ataques DHCP

## Fundamentos y arquitectura del protocolo DHCP

**1. El proceso DORA y el broadcast**

DORA son los 4 pasos para que un dispositivo consiga IP por DHCP:
- **DISCOVER**: el cliente todavía no tiene IP, así que manda un mensaje en broadcast a toda la red preguntando si hay algún servidor DHCP disponible
- **OFFER**: el servidor le contesta ofreciéndole una IP
- **REQUEST**: el cliente dice en broadcast que acepta la ip
- **ACK**: el servidor confirma que ya es suya

El tema de seguridad es que todo esto se manda en broadcast, o sea que lo ve todo el mundo que está conectado al mismo switch/red. Cualquiera que esté ahí puede escuchar esos mensajes, hacerse pasar por servidor DHCP falso, o mandar peticiones falsas para saturar al servidor real. No hace falta ningún permiso especial, solo estar conectado a la misma red.

**2. Capa 7 vs Capa 2**

Dentro del mensaje DHCP hay un campo llamado `chaddr` donde el cliente escribe "esta es mi MAC". Pero esto es distinto de la MAC que realmente lleva la trama que viaja por la red (la de verdad, la física).

El problema es que el servidor DHCP se fía del campo `chaddr`, que lo escribe el propio cliente a mano, y eso se puede falsificar fácilmente sin tocar nada más. Un atacante puede pedir cientos de IPs distintas solo cambiando ese campo cada vez, aunque toda la trama venga con la misma MAC real.

**3. Carencia de autenticación**

El protocolo DHCP (RFC 2131) no pide ninguna contraseña ni verificación a nadie. Se fía de cualquiera que pregunte. Por eso no hay nada que impida que un dispositivo se haga pasar por servidor, ni que alguien pida mil IPs seguidas.

Existió una propuesta (RFC 3118) para añadir autenticación, pero casi nadie la usó. Principalmente porque era un lío montar y gestionar esas claves en todos los dispositivos de la red, y no compatibilizaba bien con los clientes que no lo soportaban. Al final resultaba más fácil protegerse desde el switch (con DHCP Snooping, que viene más abajo) que montar todo ese sistema de autenticación.

## Vectores de ataque – Starvation

**4. Mecánica del DHCP Starvation**

El atacante manda muchas peticiones DISCOVER seguidas, cada una con una MAC falsa distinta en el campo `chaddr`. El servidor va contestando y repartiendo IPs hasta que se le acaban todas las del pool. Cuando se agotan, ya no le queda nada que ofrecer a los dispositivos legítimos que se quieran conectar.

Hay dos formas de hacerlo:
- **Simple**: solo cambias el `chaddr` (el campo de dentro del mensaje), pero la trama sigue saliendo con tu MAC real
- **Complejo**: cambias también la MAC real de la trama, no solo el `chaddr`

El segundo es mucho más difícil de parar para un switch, porque defensas como Port Security (que limita cuántas MACs puede usar un puerto) dejan de servir si las MACs van cambiando de verdad: el switch ve tráfico de "muchos dispositivos distintos" por el mismo puerto y no sabe si es un ataque o varios equipos reales conectados ahí.

**5. Herramientas de explotación**

Yersinia y Scapy son herramientas que te dejan montar y mandar estos paquetes DHCP manipulados sin tener que construirlos a mano byte a byte. La ventaja de hacerlo con un script en vez de a mano es la velocidad: a mano no te daría tiempo a agotar un pool de 254 IPs, pero con un script lo haces en segundos.

Sobre el tiempo de concesión: si el atacante agota el pool y se va, el efecto dura lo que le quede a esas IPs robadas antes de expirar. Con un lease de 7 días, el servidor se queda sin IPs libres casi una semana entera. Con un lease de 5 minutos, en pocos minutos esas IPs se empiezan a liberar solas y el servidor vuelve a tener pool disponible.

**6. Consecuencias colaterales**

Cuando se agota el pool, los dispositivos nuevos que intenten conectarse no consiguen IP de ninguna manera.

En ese caso, Windows y Linux se auto-asignan una IP del rango **169.254.0.0/16** (APIPA), simplemente para poder hablar entre ellos dentro de la misma red local aunque no haya servidor DHCP. El problema es que esa IP solo sirve para eso: comunicarse con otros equipos del mismo segmento. No tiene ni gateway ni DNS, así que no hay internet ni acceso a otras redes.

## Hardening y mitigaciones

**A nivel de infraestructura (switch)**

**DHCP Snooping** hace que el switch filtre los mensajes DHCP según de qué puerto vengan:
- Un puerto de confianza (**trusted**), normalmente donde está el servidor DHCP real, puede mandar mensajes de "servidor" (OFFER, ACK)
- Un puerto normal (**untrusted**), donde están los clientes, solo puede mandar mensajes de "cliente" (DISCOVER, REQUEST). Si desde ahí llega un mensaje de servidor, el switch lo tira directamente

Así se evita que alguien enchufe un servidor DHCP falso a un puerto normal y empiece a repartir IPs falsas (rogue DHCP).

La **DHCP Snooping Binding Database** es básicamente una tabla donde el switch va apuntando, de cada IP que se concede, la MAC, la IP, cuánto dura el lease, la VLAN y el puerto. Le sirve de referencia para otras protecciones del switch.

**Port Security** limita cuántas MACs distintas puede usar un puerto físico a la vez. Si alguien intenta un starvation usando montones de MACs falsas por el mismo puerto, en cuanto se pasa del límite el switch lo bloquea o avisa, cortando el ataque.

**A nivel de host (Linux)**

En la práctica de VirtualBox no hay un switch de verdad gestionable, así que la protección tiene que montarla el propio servidor Linux.

El módulo **hashlimit** de iptables sirve para limitar cuántos paquetes aceptas por segundo según algún criterio (por ejemplo, por MAC o IP). Es útil aquí porque puedes limitar cuántos DISCOVER aceptas en poco tiempo, cortando esas ráfagas típicas de un ataque automatizado, sin bloquear el tráfico DHCP normal de la gente legítima.

**ebtables** es como iptables pero trabaja un nivel más abajo: con las tramas Ethernet directamente, no con IPs. En un bridge de Linux se podría usar para comparar la MAC real de la trama con el `chaddr` de dentro del mensaje DHCP, y si no coinciden, tirar el paquete. Eso justo pilla el patrón típico de un ataque de starvation.

**Políticas de lease como defensa**

El tiempo de lease es un equilibrio: si lo pones muy largo (días), un ataque de starvation te deja sin IPs mucho tiempo aunque el atacante pare rápido. Si lo pones muy corto (minutos), te recuperas rápido de un ataque, pero a cambio todos los clientes están renovando constantemente y eso genera más tráfico de fondo y más carga para el servidor. Hay que buscar un punto intermedio según el tipo de red.

La detección de conflictos de IP (el `ping-check` o `conflict-detection`) hace que, antes de dar una IP, el servidor compruebe con un ping que esa IP no la está usando ya otro equipo. Esto evita que, si el pool queda en mal estado (por ejemplo después de un ataque), se repartan IPs duplicadas que generarían más líos de conectividad.

## El entorno virtual y los límites legales

**10. Modo promiscuo en VirtualBox**

El modo promiscuo hace que la tarjeta de red reciba todo el tráfico que pasa por el cable, no solo lo que va dirigido a su propia MAC. Los hipervisores lo tienen desactivado por defecto porque si no, cualquier máquina virtual podría espiar el tráfico de las demás, lo cual es un riesgo de seguridad si no lo activas tú mismo a propósito.

Si Kali no estuviera en modo promiscuo: el script con Scapy sí podría mandar paquetes con MACs falsas sin problema (eso lo hace por software, no depende del modo promiscuo). El problema sería al recibir: las respuestas OFFER que llegan dirigidas a esas MACs inventadas no coinciden con la MAC real de Kali, así que su propia tarjeta las descartaría sin enterarse de nada.

**11. Aspectos éticos y legales**

El DHCP Starvation es un tipo de Denegación de Servicio (DoS). Solo está permitido hacer este tipo de pruebas si tienes autorización clara y por escrito del dueño de la red (como en esta práctica, que es tu propio entorno de laboratorio). Sin ese permiso, aunque sea solo para "aprender", es ilegal.

En el Código Penal español esto entra dentro de los delitos de daños informáticos / sabotaje informático (artículo 264 y siguientes), que castigan interrumpir u obstaculizar el funcionamiento de un sistema informático ajeno sin permiso. Las penas van desde multas hasta prisión, y se agravan si el ataque afecta a sistemas críticos o se hace de forma organizada.
