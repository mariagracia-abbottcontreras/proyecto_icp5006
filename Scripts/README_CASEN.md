# Procesamiento y Armonización CASEN 2003–2022

- Curso: ICP5006
- Integrantes: María Gracía Abbott, Valentina Tesser y Daniel Trujillo

Este documento (`casen.Rmd`) contiene el código en **R** utilizado para **procesar, limpiar, filtrar y armonizar** las bases de datos de la **Encuesta CASEN (2003–2022)**, con el objetivo de construir un **panel comunal** comparable entre años, centrado en los periodos de interés del estudio.

## Objetivo del script

El propósito principal de este código es generar una **base de datos comunal comparable en el tiempo** que sirva como insumo para el análisis cuantitativo del proyecto:

> **“Efecto del desempleo comunal sobre el voto opositor en Chile (2010–2017)”**

En este contexto, el procesamiento de las bases CASEN tiene tres objetivos específicos:

1. **Armonizar la información comunal** proveniente de diferentes rondas de la encuesta (2003–2022), garantizando coherencia entre años.  
2. **Seleccionar los años 2003, 2009, 2013 y 2017**, que permiten construir un panel de mediano plazo que capture la dinámica del desempleo antes y durante los periodos electorales de interés (2013 y 2017).  
3. **Construir indicadores comunales** —especialmente la **tasa de desempleo**— que puedan vincularse posteriormente con los resultados electorales del Servel.

El enfoque comunal y temporal responde al **diseño longitudinal del estudio**, orientado a observar cómo la evolución del desempleo en los territorios se asocia a cambios en el voto opositor en distintos contextos económicos y políticos.

## Flujo general del procesamiento

1. **Importación de datos** CASEN en formato `.dta` (Stata).  
2. **Selección de variables relevantes** (empleo, ingreso, comuna, ponderadores, región, etc.).  
3. **Estandarización de nombres y códigos de comunas** para permitir la comparación entre años, considerando las modificaciones territoriales realizadas por el INE.  
4. **Selección de años del estudio (2003, 2009, 2013 y 2017)** mediante la instrucción:

  ```r
   bases <- ls(pattern = "^casen_(2003|2009|2013|2017)$")
  ```
  
Este filtro permite acotar el panel a los años pre-electorales que reflejan con mayor precisión las condiciones socioeconómicas comunales previas a los comicios presidenciales de 2013 y 2017, incorporando además observaciones previas (2003 y 2009) para fortalecer la perspectiva longitudinal.

5. **Cálculo de indicadores comunales** (tasa de desempleo, ingreso promedio, tasa de participación laboral, etc.) utilizando ponderadores adecuados para cada año.
6. **Consolidación y exportación** del panel en formato .xlsx y .csv.

## Estructura del código (`casen.Rmd`)

El archivo `casen.Rmd` está organizado en los siguientes bloques:

| Sección | Descripción | Justificación |
|----------|-------------|-----------------|
| **1. Carga de librerías y entorno de trabajo** | Configura los paquetes necesarios (`tidyverse`, `haven`, `dplyr`, `readxl`, `writexl`, `janitor`, `stringr`). |Permite manipular bases grandes y asegurar limpieza reproducible |
| **2. Importación de bases CASEN** | Se cargan todas las bases disponibles (2003–2022) desde /data/, verificando consistencia de nombres de variables | Se incluyen todos los años originales para garantizar que el filtrado posterior sea reproducible |
| **3. Armonización de variables** | Se renombran y recodifican variables clave (comuna, region, ocupado, ingreso, factor_expansion) | Cada ronda de CASEN tiene nombres y codificaciones distintas; esta etapa estandariza los campos para permitir comparabilidad |
| **4. Selección de años del estudio** | Se filtran las bases para conservar solo los años 2003, 2009, 2013 y 2017. | Estos años permiten construir un panel que capture la dinámica del desempleo antes y durante los periodos electorales de interés. |
| **5. Cálculo de indicadores comunales** | Se agrupan observaciones por comuna (ponderadas por factor de expansión) y se calculan tasas e indicadores relevantes | Permite obtener medidas agregadas comparables con los resultados electorales comunales |
| **6. Consolidación y exportación del panel** | Se combinan los resultados de todos los años en un único dataframe y se guarda como `casen_comunal.xlsx` y `casen_comunal.csv`. | Facilita la vinculación con otras fuentes para el análisis final |

## Requisitos y librerías

Antes de ejecutar el código, instala los siguientes paquetes en R:

```r
install.packages(c("tidyverse", "haven", "janitor", "writexl", "readxl", "stringr", "dplyr"))
```

Luego, coloca las bases CASEN en formato .dta dentro de la carpeta /data/ del proyecto.

## Ejecución del código

1. Abre el archivo `casen.Rmd` en RStudio.
2. Verifica que la ruta de trabajo esté configurada en el directorio raíz del proyecto.
3. Ejecuta los bloques en orden o selecciona **"Knit"** para generar el documento final con los resultados del procesamiento.

El archivo final se guardará automáticamente en /outputs/ bajo los nombres:

- `casen_comunal.xlsx`
- `casen_comunal.csv`

## Notas metodológicas

1. **Justificación temporal:**

Se seleccionan los años 2003, 2009, 2013 y 2017 porque corresponden a momentos previos a elecciones presidenciales, donde la percepción económica (capturada por CASEN) puede influir en el comportamiento electoral. Además, la inclusión de 2003 y 2009 amplía la base histórica y fortalece la robustez del análisis longitudinal.

2. **Justificación territorial:**

El nivel de agregación comunal es el más adecuado para este estudio, ya que permite vincular información socioeconómica (CASEN) con resultados electorales (Servel) y datos institucionales (SINIM).

3. **Reproducibilidad:**

Todo el proceso está automatizado y documentado, de modo que otros investigadores puedan replicar o actualizar el panel en futuras versiones de CASEN.

## Contacto y créditos

**Equipo de investigación:**

Proyecto: Efecto del desempleo comunal sobre el voto opositor en Chile (2010–2017)

- María Gracia Abbott

- Valentina Tesser

- Daniel Trujillo

Pontificia Universidad Católica de Chile – Curso ICP5006




