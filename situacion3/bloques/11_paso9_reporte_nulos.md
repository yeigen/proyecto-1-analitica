# Paso 9 — Reporte consolidado de nulos por columna

```python
valid_null_data = {
    k: v for k, v in null_info_per_file.items()
    if isinstance(v, pd.Series)
}

if valid_null_data:
    null_df = pd.DataFrame(valid_null_data).T
    null_df.index.name = 'archivo'

    avg_null_pct = null_df.mean(axis=0).round(2).sort_values(ascending=False)
    min_null_pct = null_df.min(axis=0).round(2)
    max_null_pct = null_df.max(axis=0).round(2)

    null_summary = pd.DataFrame({
        'columna': avg_null_pct.index,
        'pct_nulo_promedio': avg_null_pct.values,
        'pct_nulo_minimo': min_null_pct.reindex(avg_null_pct.index).values,
        'pct_nulo_maximo': max_null_pct.reindex(avg_null_pct.index).values,
    })

    print("Porcentaje de nulos por columna (muestra representativa aleatoria):")
    print()
    print(null_summary.to_string(index=False))

    high_null_cols = null_summary[null_summary['pct_nulo_promedio'] > 50]
    print()
    if not high_null_cols.empty:
        print(f"Columnas con MAS del 50% de nulos en promedio: {len(high_null_cols)}")
        print()
        print(high_null_cols.to_string(index=False))
    else:
        print("Ninguna columna supera el 50% de nulos en promedio.")

    zero_null_cols = null_summary[null_summary['pct_nulo_maximo'] == 0]
    print()
    print(f"Columnas SIN ningun nulo en ninguno de los archivos: {len(zero_null_cols)}")
    if not zero_null_cols.empty:
        print(zero_null_cols['columna'].tolist())
else:
    print("No se pudo obtener informacion de nulos de ningun archivo.")
```

Output esperado: ver `outputs-colab/bloque-9.txt`
