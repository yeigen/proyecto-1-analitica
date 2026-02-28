# Justificacion estadistica: muestra de 9,604 filas vs. poblacion completa

## Contexto del proyecto

Este documento justifica por que una muestra aleatoria de **n = 9,604 filas** es
estadisticamente suficiente para representar una poblacion de **N ~ 9 billones de filas**
(101 archivos CSV de Freddie Mac, 110 columnas originales, ~69 columnas utiles tras
eliminar las 41 columnas 100% nulas).

El muestreo se realiza tomando ~100 filas aleatorias por archivo (101 archivos),
lo que da ~10,100 filas brutas, de las cuales 9,604 es el minimo requerido por la
formula de Cochran.

---

## 1. Respuesta directa: si, la muestra de 9,604 filas es suficiente

**Si.** Despues de eliminar las columnas 100% nulas, trabajar con 9,604 filas aleatorias
en lugar de los ~9 billones de filas completas es estadisticamente valido. La justificacion
no es una opinion — es el resultado de tres pilares matematicos:

| Pilar | Que garantiza |
|-------|---------------|
| **Formula de Cochran** | Con n = 9,604, cualquier proporcion estimada tiene un margen de error <= 1% con 95% de confianza |
| **Teorema Central del Limite (TCL)** | Con n >= 30 (y aqui n = 9,604), la distribucion de las medias muestrales es aproximadamente normal, sin importar la distribucion original de los datos |
| **Ley de los Grandes Numeros (LGN)** | A medida que n crece, el promedio muestral converge al promedio poblacional. Con n = 9,604, la convergencia es altamente precisa |

La condicion critica es que el muestreo sea **aleatorio simple** (cada fila de la
poblacion tiene la misma probabilidad de ser seleccionada). El muestreo anterior con
`nrows=1000` NO cumplia esto — leia las primeras 1,000 filas secuenciales, introduciendo
sesgo temporal, de originador y de performance. El muestreo aleatorio corrige ese defecto.

---

## 2. Que es el estadistico t y para que sirve realmente

### Definicion

El estadistico t (Student's t) es una medida de cuantas desviaciones estandar esta
una media muestral de un valor de referencia, ajustando por el tamano de muestra.

**Formula del t de una muestra:**

```
t = (x̄ - μ₀) / (s / √n)
```

Donde:

| Simbolo | Significado |
|---------|-------------|
| x̄ | Media de la muestra |
| μ₀ | Valor de referencia (la media poblacional hipotetica) |
| s | Desviacion estandar de la muestra |
| n | Tamano de la muestra |

El estadistico t sigue una distribucion t de Student con **(n - 1)** grados de libertad.
Cuando n es grande (como n = 9,604), la distribucion t es practicamente identica a la
normal estandar.

### Para que sirve

El t-test responde una pregunta especifica:

> "¿La media de mi muestra es estadisticamente diferente de un valor de referencia μ₀?"

Se usa para **contrastar hipotesis** sobre medias. La hipotesis nula es H₀: x̄ = μ₀,
y el t-test calcula la probabilidad (p-valor) de observar una diferencia tan grande
o mayor si H₀ fuera cierta.

**Intervalo de confianza asociado al t-test de una muestra:**

```
IC = x̄ ± t_(α/2, n-1) × (s / √n)
```

Para n = 9,604 y confianza del 95%, t_(0.025, 9603) ≈ 1.960 (esencialmente igual a z).
Esto significa que el intervalo de confianza para la media poblacional es:

```
IC = x̄ ± 1.960 × (s / √9604)
   = x̄ ± 1.960 × (s / 98.0)
   = x̄ ± 0.02 × s
```

Es decir, el intervalo de confianza para cualquier media tiene un ancho de apenas
±2% de la desviacion estandar de esa variable. Esto es muy preciso.

---

## 3. Por que el t-test NO es la herramienta principal aqui

### El problema logico

Para ejecutar un t-test de una muestra necesitas conocer **μ₀** (la media poblacional real).
Pero si ya conocieras la media de los 9 billones de filas, no necesitarias la muestra.

Existe una **circularidad**: el t-test compara muestra contra poblacion, pero no tenemos
acceso computable a la poblacion completa (820 GB, 9 billones de filas). No podemos
calcular μ₀ para cada una de las ~69 columnas utiles.

### Que NO hace el t-test

- **No demuestra que la muestra sea representativa.** Demuestra (o no) que una media
  muestral difiere de un valor hipotetico.
- **No calcula el tamano de muestra necesario.** Eso lo hace Cochran.
- **No garantiza convergencia.** Eso lo hacen el TCL y la LGN.

### El t-test tiene un rol, pero es de verificacion, no de justificacion

El t-test **si es util** en este proyecto como herramienta de **verificacion empirica**:

1. Tomar la muestra de 9,604 filas
2. Dividirla en dos subconjuntos aleatorios (ej. 4,802 + 4,802)
3. Para cada columna numerica, hacer un t-test de dos muestras entre ambos subconjuntos
4. Si los p-valores son altos (> 0.05) para todas las columnas, se confirma que la
   muestra es internamente consistente y no tiene sesgo evidente

Tambien se puede usar para comparar la muestra contra una segunda muestra independiente
de los mismos archivos. Si las medias no difieren significativamente, eso refuerza
empiricamente la representatividad.

**Pero la justificacion teorica viene de Cochran + TCL + LGN, no del t-test.**

---

## 4. Que SI demuestra la confiabilidad de la muestra

### 4.1 Formula de Cochran (tamano de muestra para proporciones)

La formula de Cochran calcula el tamano minimo de muestra para estimar una proporcion
con un margen de error y confianza dados:

```
n₀ = z² × p × (1 - p) / e²
```

Con los parametros de este proyecto:

```
n₀ = 1.96² × 0.5 × 0.5 / 0.01²
   = 3.8416 × 0.25 / 0.0001
   = 9,604
```

| Parametro | Valor | Justificacion |
|-----------|-------|---------------|
| z = 1.96 | Nivel de confianza 95% | Estandar en investigacion |
| p = 0.5 | Proporcion estimada | Maxima varianza (caso mas conservador posible) |
| e = 0.01 | Margen de error 1% | Precision alta |
| N ~ 9 × 10⁹ | Poblacion | 101 archivos Freddie Mac |

**Nota sobre p = 0.5:** Usar p = 0.5 maximiza el producto p(1-p) = 0.25. Cualquier
proporcion real (ej. 30% de defaults) daria un n₀ menor. Es decir, 9,604 es el
tamano mas conservador posible — funciona para el peor caso.

**Correccion de poblacion finita:**

```
n = n₀ / (1 + (n₀ - 1) / N)
```

Para N = 9,000,000,000:

```
n = 9,604 / (1 + 9,603 / 9,000,000,000)
  = 9,604 / (1 + 0.00000107)
  = 9,604 / 1.00000107
  = 9,603.99
  ≈ 9,604
```

La correccion no cambia nada. El ratio n/N = 9,604 / 9,000,000,000 = 0.000000107
(0.0000107%), que es **mas de 4 millones de veces menor** al umbral del 5% donde la
correccion de poblacion finita empieza a ser relevante.

**Conclusion de Cochran:** n = 9,604 es matematicamente suficiente para estimar
cualquier proporcion en la poblacion con ±1% de error y 95% de confianza.

### 4.2 Teorema Central del Limite (TCL)

**Enunciado:** Si X₁, X₂, ..., Xₙ son variables aleatorias independientes e
identicamente distribuidas (i.i.d.) con media μ y varianza finita σ², entonces
la distribucion de la media muestral:

```
X̄ₙ = (X₁ + X₂ + ... + Xₙ) / n
```

converge en distribucion a una normal:

```
√n × (X̄ₙ - μ) / σ  →  N(0, 1)    cuando n → ∞
```

**Condiciones:**
1. Las observaciones deben ser **independientes** (el muestreo aleatorio simple lo garantiza)
2. Las observaciones deben ser **identicamente distribuidas** (todas provienen de la misma poblacion)
3. La varianza debe ser **finita** (se cumple para cualquier variable financiera acotada)

**Aplicacion al proyecto:**

Con n = 9,604, el TCL garantiza que la media muestral de cualquier columna tiene
distribucion aproximadamente normal, sin importar que la distribucion original sea
sesgada, bimodal o con colas pesadas. Esto es lo que permite construir intervalos
de confianza y hacer inferencia.

El error estandar de la media es:

```
SE = σ / √n = σ / √9604 = σ / 98.0
```

Es decir, la media muestral tiene una dispersion que es apenas **1/98 de la desviacion
estandar original** de la variable. Para una columna con σ = 100, el error estandar
de la media seria apenas 1.02.

### 4.3 Ley de los Grandes Numeros (LGN)

**Enunciado (LGN debil):** Si X₁, X₂, ..., Xₙ son variables aleatorias i.i.d. con
media μ y varianza finita, entonces para todo ε > 0:

```
P(|X̄ₙ - μ| > ε) → 0    cuando n → ∞
```

En palabras: la probabilidad de que la media muestral difiera de la media poblacional
en mas de cualquier cantidad fija ε se hace arbitrariamente pequena conforme n crece.

**Aplicacion al proyecto:**

La LGN garantiza que con n = 9,604 observaciones aleatorias, los estadisticos muestrales
(medias, proporciones, varianzas) estan muy cerca de sus valores poblacionales. No da
una cifra exacta de error (eso lo da Cochran), pero proporciona la **base teorica**
de por que el muestreo funciona: las fluctuaciones aleatorias se cancelan entre si.

---

## 5. Demostracion numerica con los datos reales del proyecto

### 5.1 Precision en proporciones (formula de Cochran)

Supongamos que una columna binaria tiene una proporcion real de p = 0.30 (30% de
algun evento — ej. default, prepago, etc.) en la poblacion. Con n = 9,604:

```
Margen de error = z × √(p(1-p)/n)
                = 1.96 × √(0.30 × 0.70 / 9604)
                = 1.96 × √(0.0000218)
                = 1.96 × 0.00467
                = 0.00916
                ≈ 0.92%
```

Es decir, si la muestra dice "30.5% de defaults", la proporcion real esta entre
29.6% y 31.4% con 95% de confianza. Para el caso conservador de p = 0.5:

```
Margen de error = 1.96 × √(0.5 × 0.5 / 9604)
                = 1.96 × √(0.0000261)
                = 1.96 × 0.00510
                = 1.00%
```

Exactamente 1%, que es el margen que elegimos.

### 5.2 Precision en medias (aplicacion del TCL)

Para una columna continua (ej. tasa de interes) con desviacion estandar σ = 1.5
puntos porcentuales:

```
Error estandar = σ / √n = 1.5 / √9604 = 1.5 / 98.0 = 0.0153

Intervalo de confianza al 95%:
IC = x̄ ± 1.96 × 0.0153
   = x̄ ± 0.030
```

La media estimada de la tasa de interes tendria un margen de ±0.03 puntos
porcentuales. Si la muestra dice "media = 4.25%", la media real esta entre
4.22% y 4.28% con 95% de confianza.

### 5.3 Precision por columnas del proyecto

| Variable ejemplo | σ estimada | SE = σ/√9604 | IC 95% (±) | Interpretacion |
|------------------|-----------|---------------|------------|----------------|
| Tasa de interes | ~1.5 pp | 0.0153 | ±0.030 pp | Precision de 3 centesimas de punto |
| FICO score | ~50 puntos | 0.510 | ±1.0 punto | Precision de 1 punto FICO |
| LTV ratio | ~15% | 0.153 | ±0.30% | Precision de 3 decimas de punto |
| Proporcion (p=0.5) | — | 0.0051 | ±1.0% | Caso mas conservador |
| Proporcion (p=0.1) | — | 0.0031 | ±0.60% | Evento poco frecuente |

Todos estos margenes son suficientes para el analisis factorial, clustering y
segmentacion que requiere el proyecto.

---

## 6. Tabla resumen de garantias estadisticas

| Garantia | Herramienta | Resultado | Condicion requerida |
|----------|-------------|-----------|---------------------|
| "9,604 filas son suficientes para representar 9 billones con ±1% de error" | Formula de Cochran | **Demostrado** (calculo exacto) | Muestreo aleatorio simple |
| "La media muestral tiene distribucion normal" | TCL | **Garantizado** para n = 9,604 >> 30 | Independencia, varianza finita |
| "Los estadisticos muestrales convergen a los poblacionales" | LGN | **Garantizado** asintoticamente | i.i.d., varianza finita |
| "La muestra no tiene sesgo sistematico" | t-test (verificacion) | **Verificable empiricamente** | Se requiere un segundo subconjunto o particion |
| "La correccion de poblacion finita es irrelevante" | FPC | **Demostrado**: n/N = 0.0000107% | n/N << 5% |

### Supuestos que deben cumplirse (y como se cumplen)

| Supuesto | Estado en este proyecto |
|----------|----------------------|
| Muestreo aleatorio simple | **Se cumple** si las filas se seleccionan con `sample()` aleatorio de cada archivo, no con `nrows` |
| Independencia entre observaciones | **Razonable** si se muestrea de los 101 archivos (diferentes trimestres). Nota: un mismo loan puede aparecer en multiples meses — si se requiere independencia estricta a nivel de loan, se deberia deduplicar por loan ID |
| Varianza finita | **Se cumple** para todas las variables financieras del dataset (tasas, scores, ratios, montos acotados) |
| Poblacion mucho mayor que muestra | **Se cumple**: N/n > 900,000 |

### Supuesto a vigilar: independencia a nivel de loan

Los datos de Freddie Mac son **longitudinales** — un mismo prestamo aparece en
multiples filas (una por mes de performance). Si la muestra aleatoria selecciona
varias filas del mismo loan, las observaciones no son estrictamente independientes.

Esto **no invalida** la muestra para la exploracion inicial (deteccion de nulos,
distribuciones marginales, tipos de datos), pero debe tenerse en cuenta para el
analisis factorial y clustering de las fases posteriores, donde puede ser necesario
agregar a nivel de loan o usar metodos que modelen la correlacion intra-loan.

---

## 7. Conclusion: que decirle al profesor

> **"La muestra de 9,604 filas aleatorias es estadisticamente representativa de la
> poblacion de ~9 billones de filas. Esto no es una estimacion — es el resultado
> directo de la formula de Cochran (Cochran, 1977, Cap. 4), que garantiza un margen
> de error del 1% con 95% de confianza bajo muestreo aleatorio simple."**

Los tres argumentos formales:

1. **Cochran dice que 9,604 es suficiente.** La formula n₀ = z²p(1-p)/e² con z=1.96,
   p=0.5, e=0.01 da exactamente 9,604. La correccion de poblacion finita no cambia
   el resultado porque n/N < 0.00002%.

2. **El TCL garantiza que las medias muestrales son normales.** Con n = 9,604, el error
   estandar de cualquier media es σ/98 — es decir, una centesima de la variabilidad
   original. Esto permite construir intervalos de confianza estrechos para cualquier
   parametro.

3. **La LGN garantiza convergencia.** Los estadisticos muestrales (medias, proporciones,
   varianzas) convergen a sus valores poblacionales. Con n = 9,604, la convergencia
   es precisa.

**Sobre el estadistico t:** no es la herramienta para *justificar* el tamano de muestra
(eso es Cochran), pero si sirve para *verificar empiricamente* que la muestra no tiene
sesgo. Se puede dividir la muestra en dos mitades y correr t-tests por columna: si
los p-valores son altos, la muestra es internamente consistente.

**Honestidad sobre los limites:**
- La garantia depende de que el muestreo sea realmente aleatorio (no secuencial).
- Los datos longitudinales tienen correlacion intra-loan que debe manejarse en fases
  posteriores del analisis.
- La muestra es suficiente para estimacion de parametros univariados con ±1% de error.
  Para estimaciones multivariadas complejas (interacciones de orden alto), el error
  puede ser mayor, pero sigue siendo manejable para 69 columnas.

---

## Referencias

- Cochran, W. G. (1977). *Sampling Techniques* (3rd ed.). Wiley. ISBN: 978-0-471-16240-7.
  Capitulo 4, Seccion 4.2 (formula para proporciones). Seccion 2.10 (correccion de
  poblacion finita).
- Rice, J. A. (2007). *Mathematical Statistics and Data Analysis* (3rd ed.). Cengage.
  Capitulo 6 (TCL), Capitulo 5 (LGN).
- Casella, G. & Berger, R. L. (2002). *Statistical Inference* (2nd ed.). Cengage.
  Capitulo 5 (propiedades de estimadores), Capitulo 10 (intervalos de confianza).
