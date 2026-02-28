# Paso 7 — Tabla resumen de numero de columnas por archivo

```python
col_count_summary = pd.DataFrame({
    'archivo': list(columns_per_file.keys()),
    'num_columnas': [len(cols) for cols in columns_per_file.values()]
})

unique_counts = col_count_summary['num_columnas'].value_counts()
print("Distribucion de cantidad de columnas detectadas:")
print(unique_counts.to_string())
print()

if unique_counts.shape[0] > 1:
    print("ATENCION: No todos los archivos tienen la misma cantidad de columnas.")
    print()
    print(col_count_summary.to_string(index=False))
else:
    print(f"Todos los archivos tienen {unique_counts.index[0]} columnas.")
```

Output esperado: ver `outputs-colab/bloque-7.txt`
