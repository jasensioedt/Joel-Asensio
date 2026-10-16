##  Instalación
 
### 1. Clonar el repositorio
 
```bash
git clone https://github.com/tu-usuario/LinkSight.git
cd LinkSight
```
 
### 2. Instalar dependencias Python
 
```bash
pip install -r requirements.txt
```
 
### 3. Levantar el entorno de laboratorio
 
```bash
docker compose up -d
```
 
Targets disponibles:
- DVWA → `http://localhost:8080`
- Juice Shop → `http://localhost:3000`
- WebGoat → `http://localhost:8888`
### 4. Crear el fichero de autorización
 
```bash
cp auth.example.json auth.json
# Editar auth.json con el target y los datos del responsable
```
 
---
 
##  Uso
 
```bash
# Solo reconocimiento pasivo
python3 auditor/cli.py --target http://localhost:8080 --mode recon
 
# Análisis dinámico completo
python3 auditor/cli.py --target http://localhost:8080 --mode dynamic
 
# Análisis estático de código fuente
python3 auditor/cli.py --mode static --source ./codigo-fuente/
 
# Análisis completo
python3 auditor/cli.py --target http://localhost:8080 --mode full
 
# Especificar carpeta de salida para el informe
python3 auditor/cli.py --target http://localhost:8080 --mode full --output ./reports/
```
 
---
 
##  Formato de hallazgo
 
Todos los módulos generan hallazgos con esta estructura estándar:
 
```json
{
  "id": "DYN-001",
  "tipo": "SQLi",
  "severidad": "alta",
  "url": "http://localhost:8080/login",
  "parametro": "username",
  "payload": "' OR '1'='1",
  "evidencia": "La respuesta contiene contenido autenticado sin credenciales válidas",
  "modulo": "dynamic",
  "fix_sugerido": null
}
```
 
---
 
##  Tests
 
```bash
# Ejecutar todos los tests
python3 -m pytest tests/
 
# Test de un módulo concreto
python3 -m pytest tests/test_dynamic.py -v
```
 
---
