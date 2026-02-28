# Estrategia de muestreo: chunksize + sample + early break

## Por que `skiprows` con callable NO es viable

```python
pd.read_csv(file, skiprows=lambda x: x not in selected_rows)
```

La lambda se evalua en cada fila del archivo. Para un archivo de 37 GB esto lee los
37 GB completos con overhead adicional — peor que `nrows`.

---

## La estrategia correcta

Se lee el archivo en chunks de 50,000 filas. De cada chunk se toman 20 filas aleatorias.
Se corta despues de 5 chunks. En total se leen 250,000 filas pero se retienen solo 100.
El `break` es critico para no leer el archivo completo desde Drive.

- Filas leidas por archivo: ~250,000 (solo los primeros 5 chunks)
- Filas retenidas: ~100 (20 aleatorias x 5 chunks)
- Filas totales retenidas (101 archivos): ~10,100 > 9,604 requeridas
- Las filas vienen de 5 posiciones diferentes del archivo, no solo del inicio
