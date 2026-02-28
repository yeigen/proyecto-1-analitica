# Arquitectura Tecnologica y Herramientas - Situacion 3

## 1. Vision General

Este documento describe la arquitectura tecnologica disenada para procesar, transformar y analizar los datos de la Situacion 3 del proyecto. El volumen de datos (~820 GB en 101 archivos CSV) exige una estrategia que combine almacenamiento eficiente, formatos columnares optimizados y procesamiento distribuido.

---

## 2. Diagrama de Arquitectura

```
+---------------------+        +---------------------+        +-------------------------+
|                     |        |                     |        |                         |
|   Google Drive      |------->|   Google Colab      |------->|   Azure Blob Storage    |
|   (CSV crudos)      |  (1)   |   (Exploracion +    |  (3)   |   (Parquet particionado)|
|                     |        |    Conversion)       |        |                         |
+---------------------+        +---------------------+        +-------------------------+
                                                                          |
                                                                          | (4)
                                                                          v
                                                               +-------------------------+
                                                               |                         |
                                                               |   Databricks / Spark    |
                                                               |   (Procesamiento        |
                                                               |    distribuido)          |
                                                               |                         |
                                                               +-------------------------+
                                                                          |
                                                                          | (5)
                                                                          v
                                                               +-------------------------+
                                                               |                         |
                                                               |   Resultados / Analisis |
                                                               |   (Tablas, agregados,   |
                                                               |    visualizaciones)      |
                                                               |                         |
                                                               +-------------------------+
```

### Descripcion del flujo

| Paso | Origen                | Destino               | Accion                                                      |
|------|-----------------------|-----------------------|-------------------------------------------------------------|
| (1)  | Google Drive          | Google Colab          | Lectura de CSVs crudos montando Drive en Colab              |
| (2)  | Google Colab          | Google Colab          | Exploracion inicial: columnas, tipos, nulos, separador      |
| (3)  | Google Colab          | Azure Blob Storage    | Conversion CSV a Parquet particionado y subida a Azure      |
| (4)  | Azure Blob Storage    | Databricks / Spark    | Lectura de Parquets para procesamiento distribuido          |
| (5)  | Databricks / Spark    | Resultados            | Generacion de tablas agregadas, metricas y visualizaciones  |

---

## 3. Herramientas Seleccionadas

### 3.1 Google Colab - Entorno de exploracion y conversion

**Estado:** Ya disponible.

**Rol en la arquitectura:** Punto de entrada para la exploracion inicial de los datos y la conversion de formato CSV a Parquet.

**Justificacion:**
- Acceso directo a Google Drive mediante montaje nativo (`drive.mount`), lo que elimina la necesidad de transferencias intermedias para los CSVs crudos.
- Entorno preconfigurado con Python, pandas, PyArrow y bibliotecas de conectividad a Azure.
- Suficiente para procesar archivos de forma individual o en lotes pequenos durante la fase de conversion.
- Sin costo adicional en su version gratuita para tareas de exploracion y conversion secuencial.

**Limitaciones:**
- La RAM disponible (12-13 GB en la version gratuita) impide cargar archivos CSV muy grandes de una sola vez. Se debe procesar por fragmentos (`chunks`).
- Las sesiones tienen un tiempo de vida limitado (~12 horas), lo que requiere diseno de scripts reanudables para la conversion de los 101 archivos.

### 3.2 Google Drive - Almacenamiento de datos crudos

**Estado:** Ya disponible.  
**Ruta:** `/content/drive/MyDrive/estadistica_proyecto_1/situacion3/csv_raw/`

**Rol en la arquitectura:** Repositorio de los 101 archivos CSV originales (819.95 GB en total).

**Justificacion:**
- Los datos ya se encuentran almacenados en esta ubicacion, lo que evita costos y tiempos de transferencia adicionales.
- La integracion nativa con Google Colab permite acceso directo sin configuracion de credenciales.
- Sirve como capa de respaldo de los datos originales sin transformar.

**Limitaciones:**
- No es un sistema de almacenamiento disenado para consultas analiticas; solo cumple el rol de repositorio de datos crudos.
- La velocidad de lectura depende del ancho de banda de la conexion entre Colab y Drive.

### 3.3 Azure Blob Storage - Almacenamiento de datos procesados

**Estado:** Cuenta de almacenamiento ya disponible.

**Rol en la arquitectura:** Repositorio centralizado de los archivos Parquet particionados, optimizados para lectura por motores de procesamiento distribuido.

**Justificacion:**
- Escalabilidad practicamente ilimitada para almacenar los Parquets resultantes (estimados en ~80-160 GB tras la compresion columnar).
- Compatibilidad nativa con Databricks, Azure Synapse y cualquier motor Spark, que pueden leer directamente desde Blob Storage o Azure Data Lake Storage Gen2.
- Modelo de costos por uso (pay-as-you-go) con tarifas competitivas para almacenamiento en la capa "Hot" o "Cool" segun la frecuencia de acceso.
- Soporte para organizacion jerarquica que permite el esquema de particionamiento `year=YYYY/quarter=Q`.

**Esquema de particionamiento propuesto:**

```
container/
  situacion3/
    parquet/
      year=2010/
        quarter=1/
          parte-00000.parquet
          parte-00001.parquet
        quarter=2/
          ...
      year=2011/
        quarter=1/
          ...
```

### 3.4 Databricks (o alternativa equivalente) - Procesamiento distribuido

**Estado:** Por configurar.

**Rol en la arquitectura:** Motor de procesamiento distribuido para ejecutar consultas, agregaciones y transformaciones sobre el volumen completo de datos en formato Parquet.

**Justificacion:**
- Apache Spark (el motor subyacente de Databricks) esta disenado para procesamiento de datos a escala, con capacidad de leer Parquets particionados y aplicar predicate pushdown y partition pruning de forma automatica.
- Databricks ofrece integracion nativa con Azure Blob Storage, lo que simplifica la configuracion de acceso.
- El entorno de notebooks de Databricks permite combinar codigo, documentacion y visualizaciones en un flujo de trabajo reproducible.
- La escalabilidad horizontal (agregar nodos al cluster) permite ajustar la capacidad de computo al volumen de datos.

---

## 4. Ventajas de Parquet sobre CSV para ~820 GB

La decision de convertir los datos de CSV a Parquet es una de las mas relevantes de esta arquitectura. A continuacion se detallan las ventajas concretas para este caso de uso:

| Aspecto                     | CSV                                          | Parquet                                        |
|-----------------------------|----------------------------------------------|------------------------------------------------|
| **Tamano en disco**         | ~820 GB (sin comprimir)                      | ~80-160 GB estimado (compresion columnar Snappy/Zstd, reduccion 5x-10x) |
| **Lectura selectiva**       | Requiere leer filas completas                | Lectura solo de columnas necesarias (projeccion columnar) |
| **Tipos de datos**          | Todo es texto; requiere parsing en cada lectura | Tipos nativos embebidos (int, float, string, date); sin parsing repetido |
| **Particionamiento**        | No soportado nativamente                     | Soporte nativo por carpetas (year/quarter); permite partition pruning |
| **Predicate pushdown**      | No disponible                                | Los motores Spark/Databricks filtran a nivel de archivo sin cargar datos innecesarios |
| **Esquema**                 | Implicito; puede variar entre archivos       | Explicito y embebido en los metadatos del archivo |
| **Compatibilidad analitica**| Universal pero ineficiente                   | Estandar de facto en ecosistemas de datos modernos (Spark, Databricks, BigQuery, Athena) |
| **Costo de almacenamiento** | Alto por volumen sin comprimir               | Significativamente menor por la compresion inherente |
| **Costo de procesamiento**  | Mayor tiempo de computo = mayor costo        | Menor I/O y parsing = menor tiempo y costo de computo |

### Impacto estimado para este proyecto

- **Almacenamiento:** De ~820 GB (CSV) a ~80-160 GB (Parquet), lo que reduce los costos de Azure Blob Storage proporcionalmente.
- **Tiempo de consulta:** Consultas que acceden a un subconjunto de columnas pueden ser entre 10x y 100x mas rapidas gracias a la lectura columnar y el partition pruning.
- **Costos de computo:** Menor volumen de datos leidos implica menor tiempo de ejecucion en clusters de Databricks, que se facturan por hora de DBU (Databricks Units).

---

## 5. Tabla Comparativa de Opciones de Procesamiento Distribuido

| Criterio                        | Databricks (en Azure)               | Azure Synapse Analytics              | Dask (en Colab o VM)                 |
|---------------------------------|--------------------------------------|--------------------------------------|--------------------------------------|
| **Motor subyacente**            | Apache Spark optimizado (Photon)     | Spark pools + SQL serverless         | Dask (Python nativo, paralelismo)    |
| **Integracion con Azure Blob**  | Nativa, con montaje DBFS             | Nativa, con linked services          | Requiere configuracion manual (fsspec + adlfs) |
| **Escalabilidad**               | Horizontal (clusters auto-escalables)| Horizontal (Spark pools) + SQL on-demand | Limitada a los recursos de la maquina o VM |
| **Facilidad de configuracion**  | Media (requiere workspace y cluster) | Media-Alta (interfaz integrada en Azure) | Alta (pip install, sin infraestructura cloud adicional) |
| **Soporte para Parquet**        | Excelente (Delta Lake, partition pruning) | Excelente (SQL serverless sobre Parquet) | Bueno (via PyArrow/fastparquet)      |
| **Modelo de costos**            | DBU por hora + VM del cluster        | Pay-per-query (serverless) o Spark pools por hora | Solo el costo de la VM o Colab Pro   |
| **Capacidad de procesamiento para ~160 GB Parquet** | Excelente (disenado para esto) | Excelente (disenado para esto)       | Viable pero limitada por memoria de una sola maquina |
| **Ecosistema de notebooks**     | Notebooks integrados, colaborativos  | Notebooks de Synapse Studio          | Jupyter/Colab (requiere adaptacion)  |
| **Curva de aprendizaje**        | Media (API de Spark/PySpark)         | Media (T-SQL + Spark)                | Baja (API similar a pandas)          |
| **Idoneidad para el proyecto**  | **Alta**                             | Alta                                 | Media (suficiente para prototipos, limitada para escala completa) |

### Recomendacion

**Databricks** es la opcion principal recomendada por las siguientes razones:
1. Esta disenado para cargas de trabajo de este volumen (~160 GB en Parquet, escalable a TB).
2. La integracion con Azure Blob Storage es directa y bien documentada.
3. Los notebooks de Databricks permiten documentar y reproducir el analisis.
4. El auto-escalado de clusters permite controlar costos apagando recursos cuando no se usan.

**Dask** se considera como alternativa viable para prototipos rapidos o si las restricciones presupuestarias impiden usar Databricks. Sin embargo, no es la opcion ideal para el volumen completo de datos.

---

## 6. Consideraciones de Costo y Escalabilidad

### 6.1 Costos estimados

| Componente                  | Estimacion de costo                          | Notas                                         |
|-----------------------------|----------------------------------------------|-----------------------------------------------|
| Google Drive                | Sin costo adicional (almacenamiento existente) | Los CSVs ya estan almacenados                |
| Google Colab                | Gratuito (o ~10 USD/mes con Colab Pro)       | Suficiente para exploracion y conversion      |
| Azure Blob Storage (Hot)    | ~3.2 USD/mes por 160 GB                     | Tarifa aproximada de 0.02 USD/GB/mes (Hot tier) |
| Azure Blob Storage (Cool)   | ~1.6 USD/mes por 160 GB                     | Alternativa si el acceso es infrecuente       |
| Databricks (cluster basico) | ~0.40-1.00 USD/DBU-hora                     | Depende del tipo de VM; un cluster con 2-4 nodos Standard_DS3_v2 puede costar ~2-5 USD/hora |
| Transferencia de datos      | Ingress a Azure es gratuito                  | La subida de Parquets desde Colab no tiene costo de transferencia |

### 6.2 Estrategias de optimizacion de costos

1. **Clusters efimeros en Databricks:** Crear clusters solo cuando se necesiten y configurar apagado automatico tras inactividad (por ejemplo, 30 minutos).
2. **Partition pruning:** El particionamiento por year/quarter permite que las consultas lean solo las particiones relevantes, reduciendo el volumen de datos procesados y el tiempo de computo.
3. **Capa de almacenamiento adecuada:** Usar Azure Blob Storage en tier "Cool" si los datos se consultan de forma esporadica; migrar a "Hot" durante periodos de analisis intensivo.
4. **Tamano de cluster ajustado:** Comenzar con un cluster pequeno (2 workers) y escalar solo si el tiempo de ejecucion lo justifica.

### 6.3 Escalabilidad

- **Almacenamiento:** Azure Blob Storage escala automaticamente; no hay limite practico para el volumen de Parquets almacenados.
- **Procesamiento:** Databricks permite agregar nodos al cluster de forma horizontal. Si los datos crecen (por ejemplo, nuevos periodos o fuentes adicionales), basta con aumentar el tamano del cluster temporalmente.
- **Formato Parquet:** Al ser un formato columnar con metadatos embebidos, soporta eficientemente la adicion de nuevos archivos sin necesidad de reorganizar los existentes.

---

## 7. Prerequisitos y Accesos Necesarios

### 7.1 Accesos ya disponibles

| Recurso            | Estado       | Detalles                                                              |
|---------------------|-------------|-----------------------------------------------------------------------|
| Google Colab        | Disponible  | Cuenta de Google con acceso al entorno de notebooks                   |
| Google Drive        | Disponible  | Ruta: `/content/drive/MyDrive/estadistica_proyecto_1/situacion3/csv_raw/` |
| Azure Blob Storage  | Disponible  | Cuenta de almacenamiento ya creada                                    |

### 7.2 Accesos y configuraciones pendientes

| Recurso / Configuracion         | Accion requerida                                                        | Responsable |
|----------------------------------|-------------------------------------------------------------------------|-------------|
| Contenedor en Azure Blob Storage | Crear un contenedor dedicado (ej. `situacion3-parquet`)                 | Por definir |
| Credenciales de Azure en Colab   | Configurar connection string o SAS token para subir Parquets desde Colab | Por definir |
| Workspace de Databricks          | Crear workspace en Azure Databricks (o solicitar acceso a uno existente) | Por definir |
| Cluster de Databricks            | Configurar cluster con Spark runtime, tamano de nodos y auto-apagado    | Por definir |
| Montaje de Blob Storage en Databricks | Configurar acceso del cluster al contenedor de Parquets via DBFS mount o credenciales | Por definir |

### 7.3 Bibliotecas y dependencias (Google Colab)

```python
# Exploracion y conversion
pandas
pyarrow          # Lectura/escritura de Parquet
fastparquet      # Alternativa a PyArrow (opcional)

# Conectividad con Azure
azure-storage-blob   # SDK oficial para subir archivos a Azure Blob Storage

# Utilidades
tqdm             # Barras de progreso para la conversion de archivos
```

### 7.4 Configuracion minima de Databricks

```
Runtime:          Databricks Runtime 13.x LTS o superior (incluye Spark 3.4+)
Tipo de nodo:     Standard_DS3_v2 (4 cores, 14 GB RAM) o equivalente
Workers:          2-4 nodos (ajustable segun carga)
Auto-apagado:     30 minutos de inactividad
Modo del cluster: Standard (no High Concurrency, para optimizar costos)
```

---

## 8. Resumen de Decisiones Arquitectonicas

| Decision                                | Justificacion                                                                 |
|-----------------------------------------|-------------------------------------------------------------------------------|
| Mantener CSVs en Google Drive           | Ya estan ahi; evitar transferencias innecesarias del dato crudo               |
| Usar Colab para exploracion y conversion| Acceso directo a Drive; entorno listo sin configuracion adicional             |
| Convertir a Parquet con particionamiento| Reduccion de tamano 5x-10x; lectura columnar; partition pruning              |
| Almacenar Parquets en Azure Blob Storage| Escalabilidad, costo competitivo, integracion nativa con Databricks          |
| Procesar con Databricks/Spark           | Motor distribuido disenado para este volumen; auto-escalado; notebooks integrados |
| Particionar por year/quarter            | Alineado con la naturaleza temporal de los datos; permite filtrado eficiente |
