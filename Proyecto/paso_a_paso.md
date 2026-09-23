# El proyecto por etapas

## PASO A PASO

### 1. BRIEFING - IDEAS - GRUPOS (16 de septiembre al 30 de septiembre)

a. Dinámica de briefing para la elección de proyecto
   i. Problemas de infraestructura y pitches técnicos
   ii. Formación de equipos y briefing de infraestructura
   iii. Qué aporto, qué busco para mi proyecto

b. Constituir los equipos: firmar acuerdo de grupo ???????

c. ¿TRELLO, EXCEL o nos vamos directo a GitBook o GitHub? ?????????

### 2. PLANIFICACIÓN

d. **Arquitectura del sistema** (01 de octubre al 11 de octubre)
   iv. Definir las piezas clave o módulos que compondrán una aplicación web. Cada componente realiza funciones específicas y se integra con otros para lograr el conjunto completo de características y servicios requeridos por la aplicación.
   v. Hardware - características
   vi. Sistema Operativo
   vii. Lógica de negocio. Backend

e. **Diseño de la aplicación web** (01 de octubre al 11 de octubre)
   viii. Mapa del sitio. Incluir las páginas principales y sus relaciones. Utiliza un diagrama de árbol o un diagrama de flujo para visualizar la jerarquía de las páginas y cómo se conectan entre sí.
   ix. Mockups. Mostrar la disposición de los elementos en la interfaz de usuario y la paleta de colores. Incluir botones, menús, enlaces y otros elementos de navegación. Describir las funcionalidades de todos los elementos, o sea, cómo funcionarán los elementos de navegación y cómo se relacionan con otras páginas o funciones.
   x. Objetivos o funcionalidades. Definir las funcionalidades u objetivos de la aplicación respondiendo a las preguntas: ¿QUÉ? ¿CUÁNDO? ¿POR QUÉ?
      1. Objetivo
      2. Prioridades
      3. Funcionalidad
      4. Disparador
      5. Fecha de entrega
      6. Estado

### 3. MEMORIA (16 de septiembre al 20 de noviembre)

f. Gitbook / Github. Crear un espacio de trabajo donde irá la memoria del proyecto, código y base de datos. Todas las actividades que se orienten se redactan y entregan como parte de esa memoria que iremos revisando cada semana.

g. **Diagrama de Gantt**
   xi. Definir un calendario detallado con los objetivos, resultados a alcanzar, requisitos técnicos (tanto de hardware como de software).

### 4. EJECUCIÓN (15 de octubre al 15 de noviembre)

h. Diseño de la base de datos (12 de octubre)
i. Instalación, configuración y puesta a punto de Proxmox
j. Instalación y configuración de los servidores: nginx, dns, dhcp, mysql
k. Configuración de IPTables
l. Puesta a punto de la DB de MySQL
m. Programación de las funcionalidades de la web
n. Actualizar la aplicación web dentro de Proxmox

### 5. CONTROL (17 de noviembre al 20 de noviembre)

o. Cloudflare
p. Revisiones
q. Preparación de la presentación

### 6. Presentación (03 de diciembre)

r. Presentaciones en el auditorio si es posible.

---

## Ejemplo de tabla para definir la arquitectura de sistema

| Componente de sistema | Tecnología o framework | Versión | Puerto | Descripción de uso o requisitos | Enlace a documentación o información adicional |
|---|---|---|---|---|---|
| Hardware | Modelo, procesador, RAM, espacio en disco, adaptadores de red | | | no | |
| Sistema operativo | ¿Qué SO? ¿Libre o propietario? | | | no | |
| Interfaz de usuario (Frontend) | HTML, CSS, JavaScript, bibliotecas y frameworks como React, Angular o Vue.js | | | | |
| Lógica de negocio (Backend) | Lenguajes de programación como PHP, Python, Java, Node.js; frameworks como Django, Flask, Spring (Java), Express (Node.js) | | | | |
| Servidor web | Apache, Nginx, Microsoft IIS | | | | |
| Base de datos | MySQL, PostgreSQL, MongoDB, Firebase, SQLite | | | | |
| Sistema gestor de base de datos | MySQL Workbench, PHPMyAdmin, Access, Oracle SQL, MongoDB Atlas | | | | |
| Servicios de APIs | REST, GraphQL, SOAP o APIs concretas | | | | |

---

## Ejemplo de tabla para definir los objetivos, prioridades, fechas de entrega, etc.

| ID | Prioridad | Objetivo | Funcionalidad | Disparador | Fecha Entrega | Estado |
|---|---|---|---|---|---|---|
| ID1 | Alta | Registrar usuarios en la página | Deberá crearse un sistema de registro de usuarios y posterior login que guarde información individual del usuario | Un botón en la esquina superior te llevará a la pantalla de registro/login | 13/10/24 | Pendiente |
