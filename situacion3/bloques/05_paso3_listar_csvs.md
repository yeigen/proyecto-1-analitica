# Paso 3 — Listar todos los archivos CSV

```python
all_files = sorted([
    f for f in os.listdir(csv_dir)
    if f.lower().endswith('.csv')
])

print(f"Total de archivos CSV encontrados: {len(all_files)}")
for i, f in enumerate(all_files, 1):
    print(f"  {i:>3d}. {f}")
```

Output esperado: ver `outputs-colab/bloque-3.txt`
