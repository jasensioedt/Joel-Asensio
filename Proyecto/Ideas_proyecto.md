Joel Asensio y Aleix Nicolás

# IDEAS DE PROYECTO

### Idea 1:
* **Programa para detectar vulnerabilidades de webs**
    * La idea seria hacer un programa que analice (con autorizacion previa) el codigo de una web y
      detecte vulnerabilidades que puedan ser atacadas por cualquiera para generar un informe, pasar el informe a alguna ia
      y que proponga soluciones tecnicas para agilizar el trabajo.<br>

    * **¿Como lo hariamos? (resumen con IA)**
        * Para el análisis estático, parsear el código a un AST (árbol de sintaxis abstracta) usando librerías como `ast` en Python o `tree-sitter`/`esprima` para JavaScript, y recorrer ese árbol buscando patrones                     concretos: funciones peligrosas (`eval`, `innerHTML`, concatenación directa en consultas SQL sin parametrizar), variables con input de usuario que llegan sin sanitizar a una función sensible (taint analysis), y              dependencias en `package.json` o `requirements.txt` cruzadas contra bases de CVEs como OSV.dev o NVD.
        * Para el análisis dinámico, lanzar peticiones HTTP automatizadas con `requests` o `httpx`, probando payloads de fuzzing típicos (SQLi, XSS reflejado, path traversal) y comparando respuestas, tiempos y códigos de               estado.
        * Guardar cada hallazgo en una estructura JSON con archivo, línea, tipo, severidad y fragmento de código relevante, evitando pasar el código completo a la IA.
        * Enviar esa estructura como prompt a un LLM para que valide la severidad, explique el vector de ataque y proponga el fix técnico, devolviendo la respuesta en JSON.
        * Generar el informe final en Markdown o PDF con los hallazgos y soluciones.
  
