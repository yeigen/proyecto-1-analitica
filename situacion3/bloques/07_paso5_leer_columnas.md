# Paso 5 — Leer columnas de cada archivo

Se usa `nrows=0` para obtener unicamente los indices de columna sin cargar datos.

```python
columns_per_file = {}

for filename in all_files:
    filepath = os.path.join(csv_dir, filename)
    try:
        df_header = pd.read_csv(
            filepath,
            sep='|',
            header=None,
            nrows=0,
            encoding='utf-8',
        )
        columns_per_file[filename] = list(df_header.columns)
    except Exception as e:
        columns_per_file[filename] = [f'ERROR: {e}']

print(f"Archivos procesados: {len(columns_per_file)}")
```

Output esperado: ver `outputs-colab/bloque-5.txt`
