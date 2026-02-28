# Paso 10 — Detalle de nulos por archivo y columna

```python
if valid_null_data:
    display_df = null_df.copy()
    display_df = display_df.reset_index()
    display_df = display_df.rename(columns={'index': 'archivo'})

    cols_with_any_null = [
        c for c in display_df.columns
        if c != 'archivo' and display_df[c].max() > 0
    ]

    if cols_with_any_null:
        print(f"Columnas que tienen al menos algun nulo en algun archivo: {len(cols_with_any_null)}")
        print()
        print(display_df[['archivo'] + cols_with_any_null].to_string(index=False))
    else:
        print("Ninguna columna tiene nulos en ninguno de los archivos.")
else:
    print("No hay datos disponibles.")
```

Output esperado: ver `outputs-colab/bloque-10.txt`
