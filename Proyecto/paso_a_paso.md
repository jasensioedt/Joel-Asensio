# El proyecto por etapas

## Paso a paso

### 1. Briefing - Ideas - Grupos (16 de septiembre al 30 de septiembre)

- [Dinámica de briefing para la elección de proyecto](./Ideas_proyecto.md) <br>
  - Problemas de infraestructura y pitches técnicos
  - Formación de equipos y briefing de infraestructura
  - Qué aporto, qué busco para mi proyecto
- Constituir los equipos: firmar acuerdo de grupo
- ¿Trello, Excel o vamos directo a GitBook o GitHub?

### 2. Planificación

**Arquitectura del sistema** (01 de octubre al 11 de octubre)
- Definir las piezas clave o módulos que compondrán la aplicación web. Cada componente realiza funciones específicas y se integra con otros para lograr el conjunto completo de características y servicios requeridos
- Hardware - características
- Sistema operativo
- Lógica de negocio. Backend

**Diseño de la aplicación web** (01 de octubre al 11 de octubre)
- Mapa del sitio: incluir las páginas principales y sus relaciones, usando un diagrama de árbol o de flujo para visualizar la jerarquía y las conexiones entre páginas
- Mockups: mostrar la disposición de los elementos en la interfaz, la paleta de colores, botones, menús, enlaces y demás elementos de navegación, describiendo cómo funcionan y cómo se relacionan con otras páginas o funciones
- Objetivos o funcionalidades: definir qué, cuándo y por qué, especificando para cada uno:
  - Objetivo
  - Prioridades
  - Funcionalidad
  - Disparador
  - Fecha de entrega
  - Estado

### 3. Memoria (16 de septiembre al 20 de noviembre)

- Gitbook / GitHub: crear un espacio de trabajo donde irá la memoria del proyecto, el código y la base de datos. Todas las actividades se redactan y entregan como parte de esa memoria, revisada cada semana
- Diagrama de Gantt: definir un calendario detallado con los objetivos, resultados a alcanzar y requisitos técnicos (hardware y software)

### 4. Ejecución (15 de octubre al 15 de noviembre)

- Diseño de la base de datos (12 de octubre)
- Instalación, configuración y puesta a punto de Proxmox
- Instalación y configuración de los servidores: nginx, DNS, DHCP, MySQL
- Configuración de IPTables
- Puesta a punto de la base de datos MySQL
- Programación de las funcionalidades de la web
- Actualización de la aplicación web dentro de Proxmox

### 5. Control (17 de noviembre al 20 de noviembre)

- Cloudflare
- Revisiones
- Preparación de la presentación

### 6. Presentación (03 de diciembre)

- Presentaciones en el auditorio, si es posible

---

## Ejemplo de tabla para definir la arquitectura de sistema

| Componente de sistema | Tecnología o framework | Versión | Puerto | Descripción de uso o requisitos | Enlace a documentación |
|---|---|---|---|---|---|
| Hardware | Modelo, procesador, RAM, espacio en disco, adaptadores de red | | | no | |
| Sistema operativo | ¿Qué SO? ¿Libre o propietario? | | | no | |
| Interfaz de usuario (Frontend) | HTML, CSS, JavaScript, bibliotecas y frameworks como React, Angular o Vue.js | | | | |
| Lógica de negocio (Backend) | PHP, Python, Java, Node.js; frameworks como Django, Flask, Spring, Express | | | | |
| Servidor web | Apache, Nginx, Microsoft IIS | | | | |
| Base de datos | MySQL, PostgreSQL, MongoDB, Firebase, SQLite | | | | |
| Sistema gestor de base de datos | MySQL Workbench, PHPMyAdmin, Access, Oracle SQL, MongoDB Atlas | | | | |
| Servicios de APIs | REST, GraphQL, SOAP o APIs concretas | | | | |

---

## Ejemplo de tabla para definir objetivos, prioridades y fechas de entrega

| ID | Prioridad | Objetivo | Funcionalidad | Disparador | Fecha entrega | Estado |
|---|---|---|---|---|---|---|
| ID1 | Alta | Registrar usuarios en la página | Deberá crearse un sistema de registro de usuarios y posterior login que guarde información individual del usuario | Un botón en la esquina superior lleva a la pantalla de registro/login | 13/10/24 | Pendiente |
