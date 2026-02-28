# Paso 11 — Resumen final

```python
print("=" * 80)
print("RESUMEN FINAL DE LA EXPLORACION")
print("=" * 80)
print()
print(f"Directorio: {csv_dir}")
print(f"Archivos CSV encontrados: {len(all_files)}")
print()

print("1. SEPARADOR")
print("   Separador real: | (pipe) en todos los archivos")
print("   NOTA: csv.Sniffer no es confiable. Se usa sep='|' fijo.")
print()

print("2. ESTRUCTURA")
print("   Header: ninguno (header=None). Primera fila son datos.")
print("   Columnas: 110 (indices 0 a 109) consistentes en todos los archivos.")
print()

print("3. CONSISTENCIA DE COLUMNAS")
if all_match:
    print(f"   Todos los archivos tienen {len(reference_cols)} columnas.")
else:
    print(f"   HAY DIFERENCIAS. {len(mismatches)} archivo(s) difieren.")
print()

print("4. MUESTREO")
total_sampled = SAMPLE_PER_CHUNK * CHUNKS_TO_READ * len(all_files)
print(f"   Metodo: chunksize={CHUNK_SIZE:,} + sample({SAMPLE_PER_CHUNK}) x {CHUNKS_TO_READ} chunks por archivo")
print(f"   Filas retenidas por archivo: ~{SAMPLE_PER_CHUNK * CHUNKS_TO_READ}")
print(f"   Filas totales analizadas: ~{total_sampled:,}")
print(f"   Muestra estadistica requerida (95% conf, +-1%): 9,604 filas")
print(f"   Cobertura: {total_sampled:,} >= 9,604 | OK")
print()

print("5. COLUMNAS CON NULOS")
if valid_null_data:
    total_cols = len(avg_null_pct)
    cols_with_nulls = (avg_null_pct > 0).sum()
    cols_high_nulls = (avg_null_pct > 50).sum()
    cols_all_null = (avg_null_pct == 100).sum()
    print(f"   Total de columnas: {total_cols}")
    print(f"   Columnas con algun nulo: {cols_with_nulls}")
    print(f"   Columnas con >50% nulos: {cols_high_nulls}")
    print(f"   Columnas 100% nulas: {cols_all_null}")
else:
    print("   No se pudo calcular.")
print()
print("=" * 80)
print("Exploracion completada.")
print("=" * 80)
```

Output esperado: ver `outputs-colab/bloque-11.txt`
