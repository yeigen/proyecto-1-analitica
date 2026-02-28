# Paso 4 — Verificar separador con lectura directa pipe

Se verifica el separador leyendo 5 filas con `sep='|'` y `header=None` y contando
las columnas resultantes. Si el resultado es 110, el separador es pipe.

No se usa `csv.Sniffer` — falla en 49/101 archivos por las comas en `SERVICER_NAME`.

```python
sep_verification = {}

for filename in all_files:
    filepath = os.path.join(csv_dir, filename)
    try:
        df_probe = pd.read_csv(
            filepath,
            sep='|',
            header=None,
            nrows=5,
            encoding='utf-8',
            on_bad_lines='warn',
        )
        sep_verification[filename] = {
            'num_columnas': df_probe.shape[1],
            'estado': 'OK' if df_probe.shape[1] >= 100 else 'REVISAR',
        }
    except Exception as e:
        sep_verification[filename] = {
            'num_columnas': -1,
            'estado': f'ERROR: {e}',
        }

sep_ver_df = pd.DataFrame(sep_verification).T
sep_ver_df.index.name = 'archivo'
sep_ver_df = sep_ver_df.reset_index()

print("Verificacion de separador pipe en todos los archivos:")
print()
print(sep_ver_df.to_string(index=False))
print()
print("Resumen:")
print(sep_ver_df['estado'].value_counts().to_string())
```

Output esperado: ver `outputs-colab/bloque-4.txt`
