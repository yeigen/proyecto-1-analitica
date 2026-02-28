# Contexto y hallazgos confirmados de la exploracion

Este documento forma parte de la serie de bloques de exploracion ligera de los
101 archivos CSV de Freddie Mac (Single-Family Fixed-Rate Loan Performance Data).

El notebook completo se ejecuta en Google Colab sobre datos almacenados en
Google Drive (~820 GB, ~9 billones de filas totales).

**Objetivo:** responder tres preguntas concretas sin cargar ningun archivo completo:
1. Todos los CSVs tienen las mismas columnas?
2. Cuantas columnas tienen valores nulos o un porcentaje alto de nulos?
3. Cual es el separador real de cada archivo?

Nada mas. Sin analisis estadistico, sin graficas, sin transformaciones.

---

## Hallazgos confirmados (resultados de ejecuciones anteriores)

### Bloque 4 — Separador

Todos los 101 archivos retornan exactamente **110 columnas** con `sep='|'` y estado `OK`.

El separador real es `|` (pipe) en los 101 archivos sin excepcion. La deteccion automatica
con `csv.Sniffer` es **no confiable** para estos archivos por un defecto estructural:

- La lista interna `preferred = [',', '\t', ';', ' ', ':']` no incluye `|`
- Si la coma o el espacio tienen frecuencia consistente entre las primeras filas
  (por comas literales en `SERVICER_NAME`, ej: `JPMorgan Chase Bank, N.A.`),
  el Sniffer los prefiere sobre `|` aunque `|` sea el separador real
- Resultado con Sniffer: 52 archivos con `|`, 38 con `,`, 11 con ` ` — 49 errores sobre 101
- Resultado con `sep='|'` fijo: 101/101 correctos

Las comas dentro de `SERVICER_NAME` no corrumpen la lectura con `sep='|'`. El motor C
de pandas hace split estricto por `|` y la coma queda como texto literal dentro del campo.

### Bloque 6 — Columnas

Todos los archivos tienen exactamente **110 columnas** (indices 0 a 109) con `header=None`
y `sep='|'`. No hay diferencias entre archivos. El bloque 6 no genero output visible porque
todos los archivos pasaron la verificacion (rama `all_match = True`).

### Bloque 10 — Nulos por columna (muestra de 1000 filas, NO aleatoria)

De las 110 columnas:

| Categoria | Columnas | Indices |
|---|---|---|
| 100% nulas en todos los archivos | 41 | 0, 6, 10, 37, 42, 38, 76, 77, 75, 74, 68, 103, 99, 100, 81, 84, 83, 82, 90, 89, 88, 87, 92, 91, 93, 98, 97, 96, 95, 94, 109, 64, 67, 66, 65, 69, 71, 70, 47, 49, 46 |
| ~99.98% nulas (casi vacias) | 12 | 51, 50, 53, 52, 60, 58, 55, 56, 57, 61, 59, 54 |
| >50% nulas en promedio | 70 | (incluye los 53 anteriores mas: 107, 106, 63, 79, 104, 45, 44, 43, 62, 33, 72, 85, 48, 40, 105, 101, 24) |
| 0% nulas en todos los archivos | 26 | 32, 30, 39, 31, 35, 14, 9, 11, 12, 13, 4, 3, 2, 7, 1, 34, 28, 29, 26, 27, 19, 80, 86, 78, 102, 108 |
| Con algun nulo | 84 | — |

Columnas con nulos variables (alta dispersion min/max — requieren muestreo aleatorio correcto):

| Columna | Promedio | Min | Max |
|---|---|---|---|
| 33 | 75.76% | 0.0% | 100.0% |
| 72 | 75.76% | 0.0% | 100.0% |
| 85 | 67.33% | 0.0% | 100.0% |
| 48 | 67.32% | 2.0% | 100.0% |
| 40 | 66.01% | 0.1% | 100.0% |
| 24 | 51.94% | 18.9% | 84.3% |
| 5  |  4.14% | 0.1% | 67.8% |
| 20 |  1.24% | 0.0% | 100.0% |

### Bloque 11 — Resumen final

```
Archivos CSV encontrados : 101
Separador real           : | (pipe) en todos los archivos
Header                   : ninguno (header=None)
Columnas                 : 110 consistentes en todos los archivos
Columnas con algun nulo  : 84
Columnas con >50% nulos  : 70
Columnas 100% nulas      : 41
```

---

## Problema critico: el muestreo anterior NO era aleatorio

### Que hace `nrows=1000`

```python
pd.read_csv(filepath, sep='|', header=None, nrows=1000)
```

Lee las **primeras 1000 filas secuenciales** desde el inicio del archivo. Esto es
muestreo de conveniencia, no muestreo aleatorio. Para datos de hipotecas de Freddie Mac,
introduce los siguientes sesgos:

| Sesgo | Impacto |
|---|---|
| Temporal | Las primeras filas son los registros mas antiguos del trimestre. Hipotecas que entraron en default al final del periodo no estan representadas |
| Originador | Los primeros loans tienden a ser del mismo banco (procesados en batch) |
| Performance | Los campos de loss mitigation, modification y liquidation (columnas 43-109) aparecen poblados en registros tardios, no en los primeros meses de vida de la hipoteca |
| Geografico | Si los registros estan ordenados por estado o ZIP, las primeras filas se concentran en codigos bajos |

**Impacto concreto:** las 41 columnas 100% nulas y las 26 columnas 0% nulas son robustas
a cualquier metodo — su resultado es identico con cualquier muestra. Pero las ~17 columnas
con nulos variables (columnas 33, 72, 85, 48, 40, 20, 5...) tienen porcentajes calculados
con sesgo. La columna 33, por ejemplo, va de 0% a 100% segun el archivo — su promedio de
75.76% esta calculado sobre las primeras filas de cada trimestre, no sobre filas representativas.
