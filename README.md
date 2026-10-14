# OSINT Corporate Intelligence: Pipeline de Ingesta y Procesamiento de Boletines Oficiales

Este repositorio contiene un motor modular y un pipeline de ingeniería de datos diseñado para la extracción automatizada, sanitización, indexación y consulta local de documentos históricos en portales gubernamentales y repositorios públicos.

Este desarrollo complementa la investigación de debida diligencia societaria documentada en el [Repositorio de Link Analysis & Maltego](https://github.com/tlondero-sec/osint-corporate-intelligence-/tree/main).

---

## ⚠️ Aviso Legal y Términos de Uso (Disclaimer)

El software, scripts y metodologías contenidos en este repositorio se proporcionan **únicamente con fines educativos, de investigación académica, auditoría defensiva y análisis de fuentes abiertas (OSINT)**. 

1. **Responsabilidad del Usuario:** El uso de estas herramientas para interactuar con sistemas de terceros, portales gubernamentales o bases de datos de acceso público recae enteramente bajo la responsabilidad del usuario final. El autor no asume ninguna responsabilidad por el uso indebido, la recolección masiva de datos no autorizada o el incumplimiento de las normativas de protección de datos locales (tales como la Ley 25.326 o equivalentes internacionales).
2. **Naturaleza del Software:** Este repositorio funciona exclusivamente como un *framework* y motor lógico local. No incluye, distribuye, aloja ni comercializa bases de datos conteniendo Información de Identificación Personal (PII), documentos oficiales ni archivos protegidos. Cualquier corpus de datos analizado debe ser provisto legítimamente por el operador bajo su propia jurisdicción.
3. **Principio de No Afiliación:** Este proyecto es una prueba de concepto (PoC) independiente de código abierto y no guarda relación, patrocinio ni vínculo oficial con ningún organismo estatal, municipal o dependencia gubernamental.

---

## ⚙️ Arquitectura del Pipeline (Proof of Concept)

El sistema está diseñado bajo un modelo modular de procesamiento local (*Air-Gapped Processing*):

1. **Ingesta Local (`descargador_historico.py` / `parser_pdf.py`):** Automatiza la consulta y recuperación estructurada de documentos institucionales mediante patrones temporales y rutas parametrizadas.
2. **Persistencia Ligera (`indexer_db.py`):** Almacena de forma estructurada los metadatos procesados en una base de datos local SQLite, optimizada para búsquedas indexadas de baja latencia.
3. **Motor CLI de Consultas (`cli_search.py`):** Interfaz de línea de comandos que permite ejecutar filtros por entidades corporativas y simular un monitoreo periódico mediante alertas basadas en palabras clave.

---

## 🛠️ Script de Recolección (Módulo Base)

```python
import os
import requests
from datetime import datetime, timedelta

# --- CONFIGURACIÓN DE RASTREO HISTÓRICO ---
ANIO_INICIO = 2026
ANIO_FIN = 2020

START_DATE = datetime(ANIO_INICIO, 10, 14)
STOP_DATE = datetime(ANIO_FIN, 1, 1)

OUTPUT_DIR = "documentos_historicos"
os.makedirs(OUTPUT_DIR, exist_ok=True)

meses_espanol = {
    1: "Enero", 2: "Febrero", 3: "Marzo", 4: "Abril",
    5: "Mayo", 6: "Junio", 7: "Julio", 8: "Agosto",
    9: "Septiembre", 10: "Octubre", 11: "Noviembre", 12: "Diciembre"
}

print(f"=== Iniciando recolección histórica ({ANIO_INICIO} a {ANIO_FIN}) ===")
print(f"Directorio de destino: ./{OUTPUT_DIR}/\n")

current_date = START_DATE
descargados = 0
omitidos = 0
consecutivos_404 = 0

# Plantilla genérica parametrizada para análisis local de repositorios públicos
BASE_URL_TEMPLATE = "[https://www.ejemplo-gobierno.gov.ar/repositorio/calendario/Documento/](https://www.ejemplo-gobierno.gov.ar/repositorio/calendario/Documento/){anio}/{mes_nombre}/{fecha_str}.pdf"

while current_date >= STOP_DATE:
    # Omitir fines de semana (Sábados=5, Domingos=6)
    if current_date.weekday() < 5:
        anio = current_date.strftime("%Y")
        mes_nombre = meses_espanol[current_date.month]
        fecha_str = current_date.strftime("%d-%m-%y")

        url = BASE_URL_TEMPLATE.format(anio=anio, mes_nombre=mes_nombre, fecha_str=fecha_str)
        pdf_path = os.path.join(OUTPUT_DIR, f"{current_date.strftime('%Y-%m-%d')}.pdf")

        if os.path.exists(pdf_path):
            omitidos += 1
            consecutivos_404 = 0
        else:
            try:
                headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ResearchBot/1.0'}
                response = requests.get(url, headers=headers, timeout=8)
                
                if response.status_code == 200 and len(response.content) > 1000:
                    with open(pdf_path, 'wb') as f:
                        f.write(response.content)
                    print(f"[OK] Descargado: {current_date.strftime('%Y-%m-%d')}")
                    descargados += 1
                    consecutivos_404 = 0
                else:
                    consecutivos_404 += 1
            except Exception as e:
                consecutivos_404 += 1

    current_date -= timedelta(days=1)

print(f"\n=== Proceso Finalizado ===")
print(f"Archivos nuevos descargados: {descargados}")
print(f"Archivos omitidos (existentes): {omitidos}")
