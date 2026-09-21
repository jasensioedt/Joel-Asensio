Joel Asensio Chavarria

# A1: Análisis y Resolución del Sistema DNS 
![Imagen DNS](../img/dns.png)
<br>
<br>
<br>
<br>
## El ecosistema DNS (OSINT y Web) [2p]
El sistema DNS es una estructura jerárquica a nivel mundial. Para administrar redes, primero debemos
entender quién gestiona cada parte del pastel.
<br>
### 1. Investigación de Jerarquía:
* <strong>¿Qué organismo internacional coordina y asigna los parámetros a nivel global del sistema de nombres de dominio e IPs?</strong>   https://es.wikipedia.org/wiki/Corporaci%C3%B3n_de_Internet_para_la_Asignaci%C3%B3n_de_Nombres_y_N%C3%BAmeros
    * La Corporación de Internet para la Asignación de Nombres y Números (ICANN) es el organismo internacional que coordina y asigna los parámetros a nivel global del sistema de nombres de dominio y las direcciones IP.
* ¿Qué empresa u organismo gestiona (Registry) cada uno de los siguientes dominios de nivel
superior (TLD)?
 1. `.es` Esta gestionado por la organicacion **red.es**  https://helpdesk.cdmon.com/portal/es/kb/articles/informaci%C3%B3n-de-los-dominios-es
 2. `.cat` Esta gestionado por **Accent Obert** (antes era **Fundació puntCAT**)
 3. `.edu` Esta gestionado por **Educause**
 4. `.ifp.es` Esta gestionado por **Grupo Planeta**

### 2. Herramientas OSINT (Whois y DNS Lookup):
* **Utiliza herramientas online (como Dominios.es, whois.com, nslookup.io) para responder a lo siguiente:**
  a. ¿Qué información te brinda una consulta Whois sobre un dominio?
  b. Define brevemente la diferencia entre el Registry de la base de datos y el Registrar (Registrador)
del dominio.
  c. Investiga: ¿Qué es **DNSSEC** y qué problema de seguridad intenta resolver en las resoluciones
**DNS**?
