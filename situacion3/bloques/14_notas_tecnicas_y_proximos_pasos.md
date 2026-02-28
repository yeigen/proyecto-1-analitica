# Notas tecnicas y proximos pasos

## Notas tecnicas

- **Separador:** siempre `sep='|'`. No usar `csv.Sniffer`, `sep=None`, ni `sep=','`.
- **Header:** siempre `header=None`. La primera fila ya son datos.
- **Muestreo:** `chunksize + sample + break` en lugar de `nrows`. Lee ~250K filas por
  archivo pero retiene solo ~100. Elimina el sesgo de inicio de archivo.
- **Formula de muestra:** n = z^2 * p * (1-p) / e^2 = 9,604 filas totales para 95% conf +-1%.
  La correccion de poblacion finita es despreciable para N >= 1,000,000.
- **Comas en valores:** `SERVICER_NAME` puede contener comas literales.
  Con `sep='|'` no genera ninguna corrupcion.
- **Campos vacios:** representados como `||`, pandas los convierte a `NaN`.
- **low_memory=False:** evita advertencias de tipos mixtos.
- **on_bad_lines='warn':** reporta filas problematicas sin detener la lectura.

---

## Proximos pasos: decision de columnas a descartar

Con los resultados de esta exploracion se puede construir la lista de columnas a eliminar
antes de la conversion a Parquet:

- **Eliminar definitivamente:** columnas con `pct_nulo_promedio == 100` y `pct_nulo_minimo == 100`
  (41 columnas — vacias en todos los archivos y en todos los periodos)
- **Evaluar con criterio de dominio:** columnas con `pct_nulo_promedio > 50` y alta variabilidad
  (min=0, max=100) — pueden ser campos introducidos en versiones recientes del layout de Freddie Mac
  o campos de liquidation que solo aplican a hipotecas en distress
- **Conservar:** columnas con `pct_nulo_maximo == 0` (26 columnas — core del dataset,
  siempre pobladas)

El siguiente paso aplicara el esquema de nombres oficiales de Freddie Mac
(Single-Family Loan-Level Dataset User Guide) para reemplazar los indices 0-109 por
nombres descriptivos antes de filtrar y convertir a Parquet.

Ver `docs/05_conversion_parquet.md` para el detalle del bloque siguiente.
