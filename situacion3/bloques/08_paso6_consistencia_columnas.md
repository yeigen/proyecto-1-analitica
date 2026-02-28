# Paso 6 — Verificar consistencia de columnas entre todos los archivos

```python
reference_file = all_files[0]
reference_cols = columns_per_file[reference_file]

all_match = True
mismatches = []

for filename, cols in columns_per_file.items():
    if len(cols) != len(reference_cols):
        all_match = False
        mismatches.append({
            'archivo': filename,
            'num_columnas': len(cols),
            'diferencia': len(cols) - len(reference_cols),
        })

if all_match:
    print(f"RESULTADO: Todos los archivos tienen {len(reference_cols)} columnas.")
    print(f"Archivo de referencia: {reference_file}")
else:
    print(f"RESULTADO: HAY DIFERENCIAS en el numero de columnas.")
    print(f"Archivo de referencia: {reference_file} ({len(reference_cols)} columnas)")
    print(f"Archivos con diferencias: {len(mismatches)}")
    print()
    mismatch_df = pd.DataFrame(mismatches)
    print(mismatch_df.to_string(index=False))
```

Output esperado: ver `outputs-colab/bloque-6.txt`
