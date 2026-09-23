Joel Asensio y Aleix Nicolás

# IDEAS DE PROYECTO

### Idea 1:
* **Programa para detectar vulnerabilidades de webs**<br>
  * La idea seria hacer un programa que analice (con autorizacion previa) el codigo de una web y
      detecte vulnerabilidades que puedan ser atacadas por cualquiera para generar un informe, pasar el informe a alguna ia
      y que proponga soluciones tecnicas para agilizar el trabajo.<br>

* **¿Como lo hariamos? (resumen con IA)**
   * Para el análisis estático, parsear el código a un AST (árbol de sintaxis abstracta) usando librerías como `ast` en Python o `tree-sitter`/`esprima` para JavaScript, y recorrer ese árbol buscando patrones                     concretos: funciones peligrosas (`eval`, `innerHTML`, concatenación directa en consultas SQL sin parametrizar), variables con input de usuario que llegan sin sanitizar a una función sensible (taint analysis), y              dependencias en `package.json` o `requirements.txt` cruzadas contra bases de CVEs como OSV.dev o NVD.
   * Para el análisis dinámico, lanzar peticiones HTTP automatizadas con `requests` o `httpx`, probando payloads de fuzzing típicos (SQLi, XSS reflejado, path traversal) y comparando respuestas, tiempos y códigos de               estado.
   * Guardar cada hallazgo en una estructura JSON con archivo, línea, tipo, severidad y fragmento de código relevante, evitando pasar el código completo a la IA.
   * Enviar esa estructura como prompt a un LLM para que valide la severidad, explique el vector de ataque y proponga el fix técnico, devolviendo la respuesta en JSON.
   * Generar el informe final en Markdown o PDF con los hallazgos y soluciones. <br>
  
### Idea 2:
* **Anti-cheat para Valorant**<br>
  * La idea seria hacer un programa que analice el trafico de paquetes del servidor durante una partida propia y
      detecte patrones anomalos (aimbot, wallhack, movimientos imposibles) para generar un informe con evidencias
      y facilitar el reporte a Riot Games.<br><br>
* **¿Como lo hariamos? (resumen con IA)** <br>
   * Capturar el tráfico de red local durante la partida con librerías como `scapy` o `pyshark`, filtrando únicamente los paquetes relacionados con la sesión del propio cliente (nunca interceptando ni modificando el              juego, solo observando el tráfico como un sniffer pasivo).
   * Extraer de esos paquetes datos estadísticos disponibles públicamente en el cliente/API (posiciones, ángulos de cámara, timings de disparo, precisión) y construir series temporales por jugador a partir de esa                 información.
   * Aplicar análisis estadístico/heurístico sobre esas series: detección de outliers en velocidad de giro (snap aim), tasa de headshots anormalmente alta, tiempos de reacción por debajo del límite humano, o visión a             través de paredes inferida por cambios de comportamiento antes de ver al rival.
   * Estructurar cada anomalía detectada en un JSON con timestamp, tipo de patrón, jugador implicado y el valor estadístico que lo justifica (por ejemplo, desviación respecto a la media de la partida).
   * Pasar esa estructura a un LLM para que redacte el informe en lenguaje claro, resuma la evidencia y la ordene según el formato que pide el sistema de reportes de Riot (Vanguard/Player Support), generando un                   documento final en PDF o Markdown listo para adjuntar al ticket de soporte.
