# Paso 8 — Muestreo representativo aleatorio (chunksize + sample + early break)

## Justificacion del tamano de muestra

Se usa la formula de Cochran para estimar proporciones en poblaciones grandes
(ver `01_formula_muestra.md` para detalle completo):

```
n0 = z^2 * p * (1 - p) / e^2
n0 = (1.96^2 * 0.5 * 0.5) / 0.01^2 = 9,604 filas
```

Distribuidas en 101 archivos: 9,604 / 101 = ~95 filas por archivo.
Se usan **100 filas por archivo** (10,100 total) para tener margen.

## Por que el metodo es aleatorio

`chunk.sample(n=20, random_state=42)` selecciona 20 filas al azar sin reemplazo
del chunk usando muestreo aleatorio simple (Fisher-Yates shuffle internamente).
El resultado es aleatorio dentro de cada chunk. El `random_state=42` solo garantiza
reproducibilidad entre ejecuciones — no elimina la aleatoriedad, la fija.

Al leer 5 chunks de posiciones distintas del archivo (filas 0-50K, 50K-100K,
100K-150K, 150K-200K, 200K-250K), la muestra final de 100 filas cubre distintos
momentos del archivo y distintos prestatarios, rompiendo el sesgo de `nrows=N`
que solo tomaba las primeras filas.

Se lee cada archivo en chunks de 50,000 filas. De cada chunk se toman 20 filas
aleatorias. Se corta despues de 5 chunks (250,000 filas leidas, 100 retenidas por archivo).

```python
null_info_per_file = {}

for filename in all_files:
    filepath = os.path.join(csv_dir, filename)
    try:
        samples = []
        chunks_read = 0

        reader = pd.read_csv(
            filepath,
            sep='|',
            header=None,
            chunksize=CHUNK_SIZE,
            encoding='utf-8',
            on_bad_lines='warn',
            low_memory=False,
        )

        for chunk in reader:
            n = min(SAMPLE_PER_CHUNK, len(chunk))
            samples.append(chunk.sample(n=n, random_state=42))
            chunks_read += 1
            if chunks_read >= CHUNKS_TO_READ:
                break

        df_sample = pd.concat(samples, ignore_index=True)
        total_rows = len(df_sample)
        null_pct = (df_sample.isnull().sum() / total_rows * 100).round(2)
        null_info_per_file[filename] = null_pct

    except Exception as e:
        null_info_per_file[filename] = f'ERROR: {e}'

print(f"Archivos analizados: {len(null_info_per_file)}")
print(f"Filas leidas por archivo: ~{CHUNK_SIZE * CHUNKS_TO_READ:,}")
print(f"Filas retenidas por archivo: ~{SAMPLE_PER_CHUNK * CHUNKS_TO_READ}")

errors_8 = {k: v for k, v in null_info_per_file.items() if isinstance(v, str)}
if errors_8:
    print()
    print(f"Archivos con error: {len(errors_8)}")
    for k, v in errors_8.items():
        print(f"  {k}: {v}")
```

Output esperado: ver `outputs-colab/bloque-8.txt`
