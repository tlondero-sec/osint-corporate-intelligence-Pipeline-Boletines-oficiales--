## ⚙️ Arquitectura del Pipeline (Proof of Concept)

El sistema está diseñado bajo un modelo modular de procesamiento local (*Air-Gapped Processing*):

1. **Ingesta Local (`parser_pdf.py`):** Toma un directorio local provisto por el usuario conteniendo documentos PDF, aplicando expresiones regulares (Regex) para aislar metadatos estructurales (CUITs, fechas, tipos societarios) sin alterar los archivos originales.
2. **Persistencia Ligera (`indexer_db.py`):** Almacena de forma estructurada los metadatos procesados en una base de datos local SQLite, optimizada para búsquedas indexadas de baja latencia.
3. **Motor CLI de Consultas (`cli_search.py`):** Interfaz de línea de comandos que permite ejecutar filtros por entidades corporativas y simular un monitoreo periódico mediante alertas basadas en palabras clave.
