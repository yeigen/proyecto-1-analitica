# Bloque: Conversion CSV a Parquet y carga en Azure Blob Storage

Este documento describe el siguiente paso del pipeline luego de completada la
exploracion ligera de los 101 CSVs. El objetivo es convertir los archivos crudos
a formato Parquet particionado y subirlos a Azure Blob Storage para procesamiento
distribuido con Databricks/Spark.

---

## Estado

- [ ] Pendiente de ejecucion en Google Colab

---

## Insumos del bloque anterior

Del documento `04_exploracion_csv_colab.md` se obtuvieron los siguientes hechos
confirmados que guian las decisiones de este bloque:

| Dato | Valor |
|---|---|
| Separador | `|` (pipe) en los 101 archivos |
| Header | `header=None` (primera fila son datos) |
| Columnas | 110 consistentes en todos los archivos |
| Columnas 100% nulas | 41 (indices: 0, 6, 10, 37, 42, 38, 76, 77, 75, 74, 68, 103, 99, 100, 81, 84, 83, 82, 90, 89, 88, 87, 92, 91, 93, 98, 97, 96, 95, 94, 109, 64, 67, 66, 65, 69, 71, 70, 47, 49, 46) |
| Columnas 0% nulas (core) | 26 (indices: 32, 30, 39, 31, 35, 14, 9, 11, 12, 13, 4, 3, 2, 7, 1, 34, 28, 29, 26, 27, 19, 80, 86, 78, 102, 108) |
| Columnas >50% nulos | 70 |

---

## Paso 1 — Asignar nombres oficiales de Freddie Mac a las columnas

Las 110 columnas actualmente tienen indices numericos (0-109). El User Guide oficial
de Freddie Mac (Single-Family Loan-Level Dataset User Guide) define los nombres
descriptivos de cada campo.

Fuente: Freddie Mac Single-Family Loan-Level Dataset User Guide
https://www.freddiemac.com/fmac-resources/research/pdf/user_guide.pdf

### Esquema de nombres (indices 0-109 → nombres oficiales)

*Este bloque se completara al ejecutar el mapeo en Colab y verificar contra el User Guide.*

---

## Paso 2 — Filtrar columnas

### Criterio de eliminacion definitiva

Eliminar columnas con `pct_nulo_promedio == 100` y `pct_nulo_minimo == 100`:
- 41 columnas vacias en todos los archivos y en todos los periodos.
- No aportan informacion. Reducen el Parquet de forma gratuita.

### Criterio de evaluacion con dominio

Columnas con `pct_nulo_promedio > 50` y alta variabilidad (min=0, max=100):
- Pueden ser campos introducidos en versiones recientes del layout de Freddie Mac.
- Pueden ser campos de liquidation/modification que solo aplican a hipotecas en distress.
- Decision: conservar por ahora y filtrar en la fase de construccion del panel analitico.

### Criterio de conservacion

Columnas con `pct_nulo_maximo == 0`: las 26 columnas core del dataset.
Siempre pobladas. Son el nucleo del analisis.

---

## Paso 3 — Conversion a Parquet con particionamiento

### Esquema de particionamiento

```
year=YYYY/quarter=Q/
```

Ejemplo: `year=2003/quarter=3/` contiene los datos del 2003Q3.

### Parametros de escritura

```python
import pyarrow as pa
import pyarrow.parquet as pq

# Compresion recomendada: Snappy (balance velocidad/tamano)
# Alternativa para mayor compresion: Zstd
compression = 'snappy'

# Tamano de row group: 128 MB (recomendado para Spark)
row_group_size = 128 * 1024 * 1024
```

### Estimacion de tamano

| Formato | Tamano estimado |
|---|---|
| CSV crudo (101 archivos) | ~820 GB |
| Parquet comprimido (Snappy) | ~80-160 GB (reduccion 5x-10x) |
| Parquet comprimido (Zstd) | ~50-100 GB (reduccion 8x-16x) |

---

## Paso 4 — Subida a Azure Blob Storage

### Contenedor destino

```
Container: freddie-mac-parquet
Path: situacion3/parquet/year={year}/quarter={quarter}/
```

### Codigo de referencia

```python
from azure.storage.blob import BlobServiceClient

connection_string = "..."  # Variable de entorno, no en codigo
container_name = "freddie-mac-parquet"

blob_service = BlobServiceClient.from_connection_string(connection_string)
container_client = blob_service.get_container_client(container_name)

# Subir archivo Parquet
with open(local_parquet_path, "rb") as data:
    container_client.upload_blob(
        name=f"situacion3/parquet/year={year}/quarter={quarter}/{filename}",
        data=data,
        overwrite=True
    )
```

### Manejo de sesiones Colab

Las sesiones de Colab tienen ~12 horas de vida. El script debe ser reanudable:

```python
# Verificar cuales archivos ya fueron subidos antes de procesar
already_uploaded = set(b.name for b in container_client.list_blobs(
    name_starts_with="situacion3/parquet/"
))

for csv_file in all_csv_files:
    parquet_name = csv_to_parquet_path(csv_file)
    if parquet_name in already_uploaded:
        print(f"Ya existe: {parquet_name} — saltando")
        continue
    # procesar y subir...
```

---

## Notas tecnicas

- **RAM Colab gratuito:** 12-13 GB. Procesar cada CSV en chunks de 500,000 filas.
- **Comas en SERVICER_NAME:** Usar siempre `sep='|'`. Nunca `sep=None`.
- **Header:** Siempre `header=None`. Asignar `names=column_names` al leer.
- **Columnas 100% nulas:** Eliminarlas al leer, antes de escribir Parquet.
  Esto reduce el tamano y evita warnings de pandas sobre columnas all-NA.

---

## Outputs esperados

Al completar este bloque se actualizaran los archivos:
- `outputs-colab/bloque-12.txt` — log de conversion y subida
- `outputs-colab/bloque-13.txt` — verificacion de integridad en Azure

---

## Referencia arquitectonica

Ver `03_arquitectura_herramientas.md` para la descripcion completa del stack
tecnologico, justificacion de herramientas y estimaciones de costo.
