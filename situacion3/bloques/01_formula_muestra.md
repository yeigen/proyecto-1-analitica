# Formula de muestra representativa

## Referencia bibliografica

Se usa la formula de Cochran para estimar el tamano de muestra en proporciones con
poblaciones grandes. La fuente original es:

> Cochran, W. G. (1977). **Sampling Techniques** (3rd ed.). Wiley.
> ISBN: 978-0-471-16240-7
> https://www.wiley.com/en-us/Sampling+Techniques%2C+3rd+Edition-p-9780471162407

La formula se encuentra en el **Capitulo 4** ("The Estimation of Proportions"),
especificamente en la seccion 4.2 (ecuacion 4.2) para muestras de proporciones,
y en la seccion 2.10 para la correccion de poblacion finita (finite population correction, FPC).

**Estado formal de la formula:** No es un teorema en el sentido estricto (no tiene
demostracion por deduccion logica pura). Es un **resultado derivado** de la teoria
del muestreo aleatorio simple (Simple Random Sampling, SRS) bajo el Teorema Central
del Limite. Cochran demuestra su derivacion en los capitulos 2 y 4 del libro. En la
literatura estadistica se lo trata como resultado comprobado y es el estandar de facto
para justificar tamanos muestrales en estudios de poblaciones grandes.

---

## Parametros

- Nivel de confianza: 95% — z = 1.96
- Margen de error: 1% — e = 0.01
- Proporcion estimada: p = 0.5 (maxima varianza, caso conservador)
- Poblacion: N ~ 3,000,000,000 a 9,000,000,000 filas

---

## Calculo

Formula de poblacion infinita:

```
n0 = z^2 x p x (1-p) / e^2
n0 = 1.96^2 x 0.5 x 0.5 / 0.01^2
n0 = 3.8416 x 0.25 / 0.0001
n0 = 9,604 filas
```

Correccion de poblacion finita:

```
n = n0 / (1 + (n0 - 1) / N)

Para N = 3,000,000,000:
  n = 9604 / (1 + 9603 / 3,000,000,000) = 9,603.97 ~ 9,604

Para N = 9,000,000,000:
  n = 9604 / (1 + 9603 / 9,000,000,000) = 9,603.99 ~ 9,604
```

---

## Por que la formula de poblacion finita no cambia nada

| N | n finita | n infinita | Diferencia |
|---|---|---|---|
| 3 billones | 9,603.97 | 9,604 | 0.03 filas (0.0003%) |
| 9 billones | 9,603.99 | 9,604 | 0.01 filas (0.0001%) |

El ratio n/N = 9,604 / 3,000,000,000 = 0.000032%, que es 150,000 veces menor al umbral
del 5% donde la correccion empieza a ser relevante. Para cualquier N >= 1,000,000, usar
**n = 9,604** es estadisticamente correcto sin correccion.

---

## Distribucion por archivo

```
9,604 / 101 archivos = ~95 filas por archivo
```

Se usa **100 filas por archivo** (10,100 total) para tener margen. Esto es 10 veces menos
que el `nrows=1000` anterior pero estadisticamente superior porque las filas son aleatorias.
