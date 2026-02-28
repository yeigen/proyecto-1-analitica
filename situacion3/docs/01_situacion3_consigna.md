# Situacion 3 -- Analisis de Estructuras Latentes en Portafolios Hipotecarios

**Proyecto 1 de Analitica | Puntaje total: 3.5 puntos**

---

## 1. Contexto y Planteamiento del Problema

El conjunto de datos **Single-Family Fixed-Rate Loan Performance Data** ofrece una vision longitudinal y granular del desempeno de prestamos hipotecarios en Estados Unidos. La alta dimensionalidad y la interdependencia temporal de estas variables (originacion, desempeno mensual, caracteristicas del prestatario) presentan un desafio analitico. La identificacion de patrones subyacentes, mas alla de los indicadores de riesgo tradicionales, es fundamental para una gestion de cartera proactiva, una tarificacion (*pricing*) precisa y el diseno de politicas de originacion robustas.

A pesar de la disponibilidad de indicadores de riesgo convencionales (como el puntaje FICO o la relacion Prestamo-Valor), estos suelen ser estaticos y fallan al capturar la dinamica evolutiva del comportamiento del prestatario y las condiciones macroeconomicas. Existe la necesidad de desarrollar un marco analitico capaz de:

1. Procesar y armonizar datos masivos distribuidos en multiples periodos anuales.
2. Identificar variables latentes (constructos no observables directamente) que expliquen la variabilidad del desempeno crediticio.
3. Implementar modelos de segmentacion que integren tanto el rigor estadistico del Analisis Factorial como la capacidad de representacion no lineal del Deep Learning.

---

## 2. Objetivo General

Disenar e implementar una solucion analitica de alto rendimiento, basada en computacion en la nube y tecnicas de aprendizaje profundo, para identificar, modelar y caracterizar estructuras latentes que permitan una **segmentacion avanzada del riesgo** en un portafolio hipotecario a gran escala.

---

## 3. Objetivos Especificos

| # | Objetivo |
|---|----------|
| 1 | Construir un panel analitico longitudinal utilizando infraestructuras de Cloud Computing y procesamiento distribuido para la informacion. |
| 2 | Ejecutar un Analisis Factorial Exploratorio (AFE) y Confirmatorio (AFC) integrando tecnicas de reduccion de dimensionalidad para datos mixtos y autoencoders. |
| 3 | Desplegar arquitecturas de Deep Learning para capturar la interdependencia temporal de los flujos de pago. |
| 4 | Desarrollar un modelo de agrupamiento (clustering) basado en los espacios latentes obtenidos para definir perfiles de riesgo interpretables. |

---

## 4. Fases del Estudio (Metodologia)

### Fase 1. Construccion del Panel Analitico

Integrar y preparar las diversas tablas del conjunto de datos (originacion, desempeno mensual, atributos del prestatario e inmueble) para construir un panel analitico longitudinal y coherente. Este panel debe ser apto para el estudio conjunto de variables de riesgo, capacidad de pago y desempeno historico.

**Alcance de la fase:**

- Integracion de tablas de originacion, desempeno mensual y atributos.
- Limpieza, transformacion y armonizacion de datos a traves de multiples periodos anuales.
- Construccion de un panel longitudinal coherente y listo para analisis.

---

### Fase 2. Extraccion de Componentes Latentes

Aplicar un metodo de Analisis Latentes adecuado para los datos. El objetivo es extraer componentes multivista que capturen la informacion compartida entre distintos grupos de variables (dominios del prestamo, del prestatario y del comportamiento mensual). Se busca que estas fuentes latentes expliquen la heterogeneidad multivariada del portafolio de manera compacta.

**Alcance de la fase:**

- Aplicacion de metodos de analisis latente apropiados para datos mixtos.
- Extraccion de componentes multivista que cubran los dominios del prestamo, del prestatario y del comportamiento mensual.
- Representacion compacta de la heterogeneidad multivariada del portafolio.

---

### Fase 3. Evaluacion e Interpretacion de Componentes

Evaluar y contrastar los componentes extraidos con los indicadores tradicionales de riesgo crediticio. El analisis debe centrarse en la interpretabilidad, estabilidad y relevancia de los componentes para caracterizar patrones de morosidad, prepago, incumplimiento y amortizacion.

**Criterios de evaluacion de los componentes:**

| Criterio | Descripcion |
|----------|-------------|
| Interpretabilidad | Los componentes deben ser comprensibles en terminos financieros. |
| Estabilidad | Los componentes deben ser robustos a variaciones en los datos. |
| Relevancia | Los componentes deben aportar informacion significativa sobre patrones de morosidad, prepago, incumplimiento y amortizacion. |

---

### Fase 4. Segmentacion Basada en Caracteristicas Latentes

Utilizar las proyecciones (scores) de los componentes independientes obtenidos como base para un proceso de Analisis de Conglomerados. Explorar tecnicas para segmentar los prestamos en grupos homogeneos.

**Tecnicas de clustering a explorar:**

- k-means
- Gaussian Mixture Models (GMM)
- Clustering jerarquico

---

### Fase 5. Caracterizacion de Perfiles de Riesgo

Interpretar y validar los conglomerados resultantes. La caracterizacion debe realizarse en terminos de:

- Perfiles de riesgo.
- Rasgos financieros del prestatario.
- Propiedades estructurales del prestamo.

---

## 5. Requerimientos Tecnicos

### Fase I: Arquitectura y Cloud Computing

| Componente | Detalle |
|------------|---------|
| **Almacenamiento** | Formatos de archivo optimizados (Parquet/Avro) en servicios de almacenamiento de objetos. |
| **Computacion** | Despliegue de clusteres para procesamiento en memoria (ej. Spark, Dask). |

### Fase II: Analisis Multivariado y Latente

1. **Analisis de Componentes Latentes:** Identificacion de dimensiones no observadas que correlacionen atributos del prestatario, el inmueble y el mercado.
2. **Validacion de Estructura:** Aplicar AFC para contrastar las hipotesis de las dimensiones extraidas con la teoria financiera del riesgo.

### Fase III: Componente de Deep Learning e IA

| Componente | Detalle |
|------------|---------|
| **Autoencoders (VAE)** | Para la representacion compacta y no lineal de la alta dimensionalidad. |
| **Frameworks** | Uso de librerias avanzadas para el entrenamiento distribuido. |

---

## 6. Resultados Esperados y Evaluacion

### Entregables esperados

| Entregable | Descripcion |
|------------|-------------|
| **Arquitectura de Solucion** | Diagrama detallado del flujo de datos en la nube y los modelos de IA empleados. |
| **Analisis** | Discusion sobre la estabilidad de los componentes latentes frente a choques economicos. |
| **Impacto de Negocio** | Discusion sobre como la segmentacion obtenida optimiza la tarificacion (*pricing*) y el monitoreo de cartera proactivo. |

### Criterios de evaluacion

| Criterio | Descripcion |
|----------|-------------|
| Integracion tecnologica | Correcta articulacion de las herramientas de nube, procesamiento distribuido y modelos. |
| Validacion estadistica | Rigor en la validacion de los modelos latentes (AFE, AFC, autoencoders). |
| Eficiencia computacional | Rendimiento y escalabilidad de la solucion implementada. |
| Profundidad interpretativa | Calidad y profundidad en la interpretacion de los perfiles de riesgo identificados. |

---

## 7. Resumen de Fases

| Fase | Nombre | Enfoque Principal |
|------|--------|-------------------|
| 1 | Construccion del Panel Analitico | Integracion de datos, panel longitudinal |
| 2 | Extraccion de Componentes Latentes | AFE, reduccion de dimensionalidad, autoencoders |
| 3 | Evaluacion e Interpretacion | Contraste con indicadores tradicionales, interpretabilidad |
| 4 | Segmentacion Latente | Clustering (k-means, GMM, jerarquico) |
| 5 | Caracterizacion de Perfiles | Perfiles de riesgo, validacion de conglomerados |
