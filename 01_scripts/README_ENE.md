# Procesamiento de bases ENE 2010–2017: 
### Generación de tasas comunales de desocupación — versiones simple y ponderada  

- Curso: ICP5006
- Integrantes: María Gracía Abbott, Valentina Tesser y Daniel Trujillo

## Resumen general

Este script (`01_df_ENE.Rmd`) tiene como objetivo construir una **base comunal longitudinal (panel)** con indicadores de **tasa de desocupación** para el período **2010–2017**, a partir de las microbases de la **Encuesta Nacional de Empleo (ENE)** del Instituto Nacional de Estadísticas (INE).

Se elaboran **dos versiones**:

1. **Versión “Raw” (simple, sin ponderadores):**  
   - Calcula tasas de desocupación por comuna usando proporciones simples, sin aplicar factores de expansión.  
   - Es útil para análisis exploratorios o para validar rutinas de procesamiento.

2. **Versión “Adjusted” (ponderada, representativa):**  
   - Utiliza los **ponderadores muestrales (`fact`)** de la ENE, lo que permite estimaciones **representativas de la población total**.  
   - Esta versión refleja con precisión la situación laboral real de cada comuna.

Ambas versiones permiten calcular **diferencias interanuales** y **promedios por período  de análisis (2010–2013 y 2014–2017)**,  lo que facilita el análisis de las tendencias del desempleo en Chile antes de una elección presidencial; en este caso, las elecciones presidenciales de 2013 y 2017.

## 0. Configuración inicial

Antes de procesar los datos, se limpian los objetos del entorno y se cargan las librerías necesarias:

```{r}
rm(list = ls())

library(haven)      # Leer archivos de Stata (.dta)
library(readxl)     # Leer archivos de Excel (.xlsx)
library(tidyverse)  # Conjunto de herramientas para manipular datos
library(stringr)    # Manejo y limpieza de cadenas de texto
library(purrr)      # Programación funcional, útil para iterar sobre listas
library(dplyr)      # Manipulación y transformación de data frames
library(tidyr)      # Organización y reestructuración de bases
library(stringi)    # Manejo avanzado de texto (p.ej. quitar acentos)
library(zoo)        # Funciones para series temporales (p.ej. arrastrar último valor)
```
Cada librería cumple un rol esencial en la limpieza, transformación y organización del panel comunal.

## 1. Importación de bases ENE

El primer paso consiste en **identificar y cargar automáticamente todas las bases de datos ENE** disponibles en el directorio de trabajo, las cuales se asumen en formato `.dta`.

```{r}
files <- list.files(pattern = "\.dta$", ignore.case = TRUE)
obj_names <- paste0("ENE", str_extract(files, "(?<!\d)(20\d{2})(?!\d)"))
```

- Se buscan todos los archivos `.dta` (uno por año).
- Se extrae el año desde el nombre del archivo para crear objetos como `ENE2010`, `ENE2011`, etc.

Luego se depura esta lista por si algún archivo no calza con el patrón o tiene años repetidos:

```{r}
valid <- !is.na(obj_names)
files <- files[valid]
obj_names <- make.unique(obj_names[valid], sep = "_")
```

Finalmente, se leen todas las bases y se asignan directamente al entorno global:

```{r}
dta_list <- set_names(files, obj_names) |> map(read_dta)
list2env(dta_list, envir = .GlobalEnv)
```

De esta forma, cada base ENE queda disponible como objeto independiente en el entorno.

## 2. Versión “Raw” (No ponderada)

En esta sección se construye un **panel comunal no ponderado** a partir de los microdatos de la Encuesta Nacional de Empleo (ENE) para el período 2010–2017. El objetivo es generar una primera aproximación descriptiva al comportamiento del desempleo en las comunas chilenas, sin aplicar aún los **factores de expansión** o ponderadores muestrales. En otras palabras, cada registro individual dentro de la base de datos tiene el mismo peso en el cálculo de las tasas, independientemente del tamaño o la estructura demográfica de la población que representa.

Trabajar con una versión “raw” o sin ponderar cumple una función exploratoria y metodológica importante. Por un lado, permite verificar la **consistencia interna** de las bases antes de aplicar ajustes estadísticos más complejos. Esto incluye la revisión de nombres de variables, codificación de comunas, tratamiento de valores perdidos y detección de posibles errores en la integración anual. Por otro lado, posibilita observar **patrones relativos** entre comunas, ya que aunque las tasas obtenidas no sean representativas a nivel nacional, sí reflejan las diferencias sistemáticas en el número de personas ocupadas y desocupadas en cada territorio bajo los mismos criterios de cálculo.

La generación de esta versión inicial también es clave para construir la **estructura del panel comunal**, que servirá como base para las siguientes etapas del procesamiento. Una vez definidas las tasas de desempleo comunales no ponderadas por año, se pueden calcular variaciones interanuales y promedios para cada período analizado previo a la elección de análisis, lo que facilita identificar tendencias locales y preparar la información para su posterior comparación con la versión ajustada (ponderada).

De esta manera, esta sección busca garantizar la trazabilidad y coherencia del proceso de análisis. La versión no ponderada funciona como un **punto de partida limpio y estandarizado**, que permite entender la distribución bruta del desempleo y establecer un marco de referencia antes de aplicar los ponderadores que corrigen las diferencias de tamaño y composición poblacional entre comunas.

### 2.1 Limpieza de variables y loop anual

Primero se define una función auxiliar para estandarizar los nombres comunales:

```{r}
normalize_commune <- function(x) {
x |>
as.character() |>
stringi::stri_trans_general("Latin-ASCII") |>
str_to_lower() |>
str_replace_all("[^a-z0-9]+", "")
}
```
Luego, un **bucle** `for` recorre cada base anual (`ENE2010`, `ENE2011`, …) para:

- Seleccionar variables relevantes (`ano_encuesta`, `r_p_c`, `activ`).
- Depurar la variable `activ`, conservando solo códigos válidos (1: ocupado, 2: desocupado, 3: inactivo).
- Crear una variable binaria `desocupado` (1 si activ==2, 0 en caso contrario).
- Agrupar por año y comuna y calcular la proporción simple:

    ```{r}
    prop_desocup_anual = mean(desocupado, na.rm = TRUE)
    ```
Cada resumen anual se guarda dinámicamente (`summary_ENE2010`, `summary_ENE2011`, …).

### 2.3 Unión de resultados en panel

Todos los resúmenes anuales se combinan:

```{r}
ENE_2010_2017 <- bind_rows(mget(summary_names)) %>%
  mutate(
    ano_encuesta = as.integer(ano_encuesta),
    com_caracter = str_trim(as.character(com_caracter))
  ) %>%
  arrange(ano_encuesta, com_caracter)
```
El resultado es un **panel ordenado año–comuna**, listo para análisis longitudinales.

### 2.4 Diferencias interanuales y promedios por periodo

En esta etapa del procesamiento se busca caracterizar la evolución temporal del desempleo comunal sin aplicar ponderadores, es decir, trabajando sobre tasas comunales previamente calculadas a partir de los microdatos de la encuesta. A partir de este panel limpio y consolidado, se construyen indicadores de cambio y tendencia que permiten interpretar la dinámica del desempleo desde una perspectiva comparativa e histórica.

El primer indicador calculado corresponde a las **diferencias interanuales** (`laggdiff`), que miden la variación del desempleo entre un año y el anterior dentro de cada comuna. Este cálculo se realiza restando la tasa comunal de desempleo del año previo a la del año en curso. De esta manera, se obtiene una medida del cambio neto que experimenta cada territorio en su nivel de desempleo año a año. Este indicador es especialmente útil para detectar momentos de crecimiento o reducción del desempleo local, así como para identificar comunas que presentan comportamientos atípicos frente a las tendencias nacionales o regionales.

```{r}
# Cálculo la diferencia interanual (`laggdiff`)

ENE_2010_2017_full <- ENE_2010_2017_full %>%
  group_by(com_caracter) %>%
  arrange(ano_encuesta, .by_group = TRUE) %>%
  mutate(
    prev_prop = dplyr::lag(zoo::na.locf(prop_desocup_anual, na.rm = FALSE)),
    laggdiff  = ifelse(!is.na(prop_desocup_anual) & !is.na(prev_prop),
                       prop_desocup_anual - prev_prop, NA_real_)
  ) %>%
  ungroup()
```

En segundo lugar, se calculan los **promedios de variación por período de análisis**, agrupando los años de observación en dos bloques: 2010–2013 y 2014–2017. Esta división responde a la lógica electoral y política del estudio, ya que permite vincular las fluctuaciones del desempleo comunal con los contextos previos a las elecciones presidenciales de 2013 y 2017. De este modo, el promedio de variación se obtiene como la media aritmética simple de las diferencias interanuales dentro de cada bloque de años, generando un indicador sintético de la evolución del desempleo comunal en los periodos previos a cada elección presidencial. Este indicador será posteriormente contrastado con los resultados electorales, con el fin de examinar si los cambios en las condiciones laborales locales se asocian con un voto de apoyo o castigo hacia el gobierno en ejercicio.

```{r}
# Definición periodos políticos

ENE_2010_2017_full <- ENE_2010_2017_full %>%
  mutate(
    period = dplyr::case_when(
      ano_encuesta >= 2010 & ano_encuesta <= 2013 ~ "2010-2013",
      ano_encuesta >= 2014 & ano_encuesta <= 2017 ~ "2014-2017",
      TRUE ~ NA_character_
    )
  )

# Calcular promedios por comuna y periodo
  
ENE_2010_2017_full <- ENE_2010_2017_full %>%
  group_by(com_caracter, period) %>%
  mutate(
    mean_laggdiff_period    = if (all(is.na(laggdiff))) NA_real_ else mean(laggdiff, na.rm = TRUE),
    n_years_laggdiff_period = sum(!is.na(laggdiff))
  ) %>%
  ungroup()

# Creación de una versión compacta

ENE_2010_2017_compact <- ENE_2010_2017_full %>%
  filter(!is.na(period)) %>%
  group_by(com_caracter, period) %>%
  summarise(
    mean_laggdiff = if (all(is.na(laggdiff))) NA_real_ else mean(laggdiff, na.rm = TRUE),
    n_years_used  = sum(!is.na(laggdiff)),
    .groups = "drop"
  ) %>%
  left_join(rpc_mode, by = "com_caracter") %>%
  rename(r_p_c = r_p_c_mode) %>%
  arrange(com_caracter, period)
```

El propósito de esta etapa es doble. Por un lado, facilita una lectura comparada de la trayectoria del desempleo comunal en el tiempo, más allá de las fluctuaciones puntuales que pueden observarse entre años consecutivos. Por otro, permite identificar patrones estructurales y tendencias generales del mercado laboral local, proporcionando evidencia empírica para vincular las variaciones del desempleo con comportamientos políticos y electorales a nivel territorial. En suma, estas métricas transforman el panel comunal en una herramienta analítica que no solo refleja niveles estáticos de desempleo, sino también su ritmo, dirección y coherencia temporal en los contextos presidenciales previos a las elecciones de 2013 y 2017.

## 3. Versión “Adjusted” — Con ponderadores

### 3.1 ¿Qué son los ponderadores (fact)?

En encuestas como la **Encuesta Nacional de Empleo (ENE)**, los datos provienen de una muestra de la población, es decir, no se entrevista a todas las personas del país, sino a un grupo seleccionado que debe representar adecuadamente a la población total. Sin embargo, debido a factores como la distribución geográfica, el tamaño comunal o la no respuesta de algunos hogares, no todas las observaciones tienen la misma probabilidad de ser seleccionadas.

Para corregir esas diferencias y garantizar que los resultados sean representativos del total de la población, se utilizan ponderadores o factores de expansión, identificados en la base de datos como `fact`.

El ponderador indica cuántas personas “reales” de la población representa cada individuo encuestado.

> **Ejemplo**: Si una persona tiene fact = 400, significa que representa a 400 personas en la población total.

En otras palabras, los ponderadores permiten “expandir” la muestra hasta aproximarla al tamaño y estructura de la población total del país.

Aplicar ponderadores es esencial porque:

- **Corrige desequilibrios muestrales**: algunas comunas o grupos poblacionales pueden estar sobre o subrepresentados en la muestra.
- **Asegura la representatividad**: los resultados obtenidos reflejan la realidad de toda la población y no solo la de las personas encuestadas.
- **Permite comparabilidad**: las estimaciones se alinean con los estándares oficiales del Instituto Nacional de Estadísticas (INE).

Sin ponderadores, las tasas reflejan solo la muestra; con ponderadores, reflejan la población real.

### 3.2 Loop: tasas trimestrales ponderadas

Esta sección constituye el **núcleo operativo del procesamiento de la ENE**, ya que automatiza la limpieza, depuración y cálculo de la **tasa de desocupación ponderada** para cada año del periodo de estudio (2010–2017). El objetivo es transformar cada base anual original —con sus millones de observaciones individuales— en un **resumen comunal-trimestral representativo**, que capture la proporción real de personas desocupadas en la población, **respetando el diseño muestral** del instrumento.

El proceso comienza con un **loop** que recorre todas las bases de la ENE identificadas en el entorno (`enes`). En cada iteración, el comando `get(nm)` carga la base correspondiente al año en curso. A continuación, se realiza una **selección precisa de variables clave**, conservando sólo aquellas necesarias para el análisis: el año de la encuesta (`ano_encuesta`), el código de comuna (`r_p_c`), la condición de actividad laboral (`activ`), el mes central del trimestre móvil (`mes_central`) y el **factor de expansión (`fact`)**. Este último —el **ponderador muestral**— es un componente esencial: en la ENE, **cada persona encuestada representa a un número determinado de individuos en la población total**, según criterios de diseño, estratificación y probabilidad de selección. Aplicar estos ponderadores permite que los resultados sean **representativos a nivel comunal y nacional**, evitando sesgos derivados de la estructura de la muestra.

```{r}
for (nm in enes) {

  df_raw <- get(nm)

  # --- Selección y recodificación base ---
  #     IMPORTANTE: 'fact' es el factor de expansión (ponderador) que provee la ENE
  df <- df_raw %>%
    select(
      ano_encuesta,   # año de la encuesta (puede venir labelled)
      r_p_c,          # código de comuna (con etiquetas)
      activ,          # condición de actividad (1=ocupado, 2=desocupado, 3=fuera FT)
      mes_central,    # mes central del trimestre móvil (1..12)
      fact            # factor de expansión (ponderador)
    ) %>%
    mutate(
      # Recodificar activ: conservar 1/2/3 y pasar todo lo demás a NA
      activ = ifelse(activ %in% c(1L, 2L, 3L), activ, NA_integer_),

      # Etiqueta legible de comuna + versión normalizada para "match" robusto
      com_caracter = normalize_commune(as_factor(r_p_c, levels = "labels", ordered = FALSE)),

      # Indicador 0/1 de desocupación (1 si activ==2; 0 si activ==1 o 3; NA si activ==NA)
      desocupado = as.numeric(activ == 2L)
    )

  # --- Resumen por (año, comuna, mes_central): proporción trimestral ponderada ---
  #     - prop_desocup_trim: tasa ponderada usando 'fact' DENTRO del trimestre-comuna
  #     - sum_w_trim/num_w_trim: acumuladores para auditoría (no se usan en el promedio anual simple)
  #     - n_obs: tamaño muestral bruto (no ponderado) meramente informativo
  resumen_trim <- df %>%
    group_by(ano_encuesta, r_p_c, com_caracter, mes_central) %>%
    summarise(
      sum_w_trim        = sum(ifelse(!is.na(desocupado) & !is.na(fact), fact, 0), na.rm = TRUE),
      num_w_trim        = sum(ifelse(!is.na(desocupado) & !is.na(fact), fact * desocupado, 0), na.rm = TRUE),
      prop_desocup_trim = ifelse(sum_w_trim > 0, num_w_trim / sum_w_trim, NA_real_),
      n_obs             = sum(!is.na(desocupado)),
      .groups = "drop"
    ) %>%
    arrange(ano_encuesta, com_caracter, mes_central)

  # Crear objeto summaryTrim_ENE#### en el entorno (p. ej., summaryTrim_ENE2010)
  summary_nm <- paste0("summaryTrim_", nm)
  assign(summary_nm, resumen_trim, envir = .GlobalEnv)
  summary_trim_names <- c(summary_trim_names, summary_nm)
}
```

Una vez seleccionadas las variables, se procede a la **recodificación y generación de indicadores**. La variable `activ` se limpia para mantener únicamente los valores válidos (1 = ocupado, 2 = desocupado, 3 = fuera de la fuerza de trabajo), eliminando cualquier código especial o error de captura. Luego, se genera una versión textual y estandarizada del nombre de la comuna (`com_caracter`) utilizando la función `normalize_commune`, la cual transforma las etiquetas originales en cadenas **sin acentos ni espacios irregulares**. Esta normalización es fundamental para garantizar **consistencia en las uniones y comparaciones posteriores**, especialmente cuando se integran bases de distintos años o fuentes. Finalmente, se crea una **variable binaria `desocupado`**, que adopta el valor 1 si el individuo está desocupado (`activ == 2`) y 0 en caso contrario.

Con la base ya limpia y estandarizada, el código procede a **agrupar las observaciones por comuna, año y mes central del trimestre móvil**. Este nivel de agregación es crucial, ya que la ENE utiliza un **diseño trimestral móvil** (por ejemplo, enero–marzo, febrero–abril, etc.), y el mes central permite identificar el periodo de referencia de cada medición. Dentro de cada grupo, se calculan tres elementos clave:

- **`sum_w_trim`**: la suma total de los ponderadores (`fact`), que representa la población estimada en ese trimestre-comuna;  
- **`num_w_trim`**: la suma ponderada de los casos desocupados, es decir, el número estimado de personas sin empleo en la población;  
- **`prop_desocup_trim`**: la proporción ponderada de desocupados sobre el total ponderado (`num_w_trim / sum_w_trim`), que constituye la **tasa de desocupación trimestral ponderada**.

Adicionalmente, se incluye el conteo no ponderado de observaciones (`n_obs`), útil como indicador de **tamaño muestral efectivo**, permitiendo evaluar la confiabilidad de las estimaciones en comunas con pocos casos.

Finalmente, los resultados se **almacenan dinámicamente**: cada tabla de resumen trimestral se guarda en el entorno global con un nombre único (`summaryTrim_ENE2010`, `summaryTrim_ENE2011`, etc.), y su referencia se agrega a una lista (`summary_trim_names`) que facilitará su combinación posterior.

Por lo tanto, este loop cumple una **doble función**: automatiza el procesamiento homogéneo de múltiples bases anuales y genera un conjunto estandarizado de **tasas comunales ponderadas**, comparables entre años y coherentes con la estructura probabilística de la ENE. Este procedimiento permite construir una **base sólida para el panel longitudinal**, asegurando **representatividad estadística y consistencia metodológica** en el análisis de la evolución del desempleo comunal. De este modo, se sientan las bases empíricas para examinar la relación entre las condiciones laborales locales y el comportamiento electoral en las elecciones presidenciales de 2013 y 2017.

Cada resumen trimestral contiene:

| Variable | Descripción |
|----------|-------------|
| `ano_encuesta` | Año de la encuesta |
| `r_p_c` | Código de comuna |
| `com_caracter` | Nombre normalizado de comuna |
| `mes_central` | Mes central del trimestre |
| `prop_desocup_trim` | Tasa de desocupación ponderada |
| `sum_w_trim` | Suma total de ponderadores |
| `n_obs` | Número de observaciones válidas |

### 3.3 Unión de bases trimestrales

Los resúmenes anuales se combinan en una sola tabla ordenada:

```{r}
ENE_2010_2017_trim <- bind_rows(mget(summary_trim_names)) %>%
  mutate(
    ano_encuesta = as.integer(ano_encuesta),
    com_caracter = str_trim(as.character(com_caracter)),
    mes_central  = as.integer(mes_central)
  ) %>%
  arrange(ano_encuesta, com_caracter, mes_central)
```
Esto forma una base panel año × comuna × trimestre, ideal para análisis de tendencias temporales.

### 3.4 Diferencias interanuales y promedios por periodo (ponderado)

En esta etapa se trabaja sobre el panel comunal **ponderado**, es decir, aquel en el que las tasas de desempleo han sido ajustadas mediante los **factores de expansión** (`fact`) de la ENE. Estos factores permiten extrapolar los resultados muestrales a la población total, corrigiendo los sesgos derivados del diseño de la encuesta y otorgando representatividad estadística a las estimaciones. A partir de esta base, se analiza la **evolución temporal del desempleo comunal** entre 2010 y 2017, considerando no solo sus niveles absolutos, sino también sus variaciones interanuales y promedios por períodos presidenciales. Este enfoque permite observar las tendencias estructurales del mercado laboral local y preparar la información para su posterior contraste con los resultados electorales, en línea con el objetivo de examinar cómo las condiciones laborales previas a las elecciones de 2013 y 2017 se asocian con el voto opositor.

El primer paso consiste en calcular, para cada comuna, la diferencia interanual ponderada del desempleo. Esto se logra agrupando los datos por comuna, ordenando los años de observación y aplicando la función lag() junto a na.locf() (del paquete zoo) para obtener el valor previo más reciente disponible. De este modo, se define una nueva variable, laggdiff, que representa el cambio en la tasa de desocupación entre un año y el anterior. Este indicador resulta útil para detectar tendencias locales —por ejemplo, comunas donde el desempleo ha aumentado o disminuido sistemáticamente—, y permite analizar el comportamiento del mercado laboral más allá de las variaciones puntuales de un año determinado.

```{r}
# Cálculo de diferencias interanuales `(laggdiff)`

ENE_2010_2017_full <- ENE_2010_2017_full %>%
  group_by(com_caracter) %>%
  arrange(ano_encuesta, .by_group = TRUE) %>%
  mutate(
    prev_prop = dplyr::lag(zoo::na.locf(prop_desocup_anual, na.rm = FALSE)),
    laggdiff  = ifelse(!is.na(prop_desocup_anual) & !is.na(prev_prop),
                       prop_desocup_anual - prev_prop, NA_real_)
  ) %>%
  ungroup()
```

Posteriormente, se introducen los periodos de análisis como categorías analíticas que agrupan los años 2010–2013 y 2014–2017. Esta clasificación facilita la interpretación política y estructural de los cambios observados, permitiendo comparar la evolución del desempleo bajo distintos contextos institucionales y económicos. Para cada comuna y periodo se calcula el promedio de las diferencias interanuales (mean_laggdiff_period), junto con el número de años efectivamente observados con valores válidos (n_years_laggdiff_period).

```{r}
# Definición de periodos políticos

ENE_2010_2017_full <- ENE_2010_2017_full %>%
  mutate(
    period = dplyr::case_when(
      ano_encuesta >= 2010 & ano_encuesta <= 2013 ~ "2010-2013",
      ano_encuesta >= 2014 & ano_encuesta <= 2017 ~ "2014-2017",
      TRUE ~ NA_character_
    )
  )
  
# Calcular promedios por comuna y periodo

ENE_2010_2017_full <- ENE_2010_2017_full %>%
  group_by(com_caracter, period) %>%
  mutate(
    mean_laggdiff_period    = if (all(is.na(laggdiff))) NA_real_ else mean(laggdiff, na.rm = TRUE),
    n_years_laggdiff_period = sum(!is.na(laggdiff))  # cuántos años con diferencia válida aportan
  ) %>%
  ungroup()
```

El resultado final es un panel robusto que combina representatividad poblacional (por los ponderadores) con consistencia temporal (por las diferencias interanuales). Esto ofrece una visión más precisa y comparativa del desempleo comunal, tanto a nivel de corto plazo (año a año) como en horizontes políticos más amplios (previos a una elección). La información resultante permite detectar patrones estructurales de desempleo, evaluar la estabilidad del mercado laboral y sentar las bases para análisis posteriores vinculados a dinámicas económicas, territoriales o electorales.

```{r}
# Creación de una versión compacta

ENE_2010_2017_compact <- ENE_2010_2017_full %>%
  filter(!is.na(period)) %>%
  group_by(com_caracter, period) %>%
  summarise(
    mean_laggdiff = if (all(is.na(laggdiff))) NA_real_ else mean(laggdiff, na.rm = TRUE),
    n_years_used  = sum(!is.na(laggdiff)),
    .groups = "drop"
  ) %>%
  left_join(rpc_mode, by = "com_caracter") %>%
  rename(r_p_c = r_p_c_mode) %>%
  arrange(com_caracter, period)
```

## 4. Exportación de resultados

Los objetos finales se exportan en formato Excel:

```{r}
writexl::write_xlsx(ENE_2010_2017_full, "ENE_2010_2017_full_vraw.xlsx")
writexl::write_xlsx(ENE_2010_2017_compact, "ENE_2010_2017_compact_vraw.xlsx")
writexl::write_xlsx(ENE_2010_2017_full, "ENE_2010_2017_full_vadjust.xlsx")
writexl::write_xlsx(ENE_2010_2017_compact, "ENE_2010_2017_compact_vadjust.xlsx")
```

## Contacto y créditos

**Equipo de investigación:**

Proyecto: Efecto del desempleo comunal sobre el voto opositor en Chile (2010–2017)

- María Gracia Abbott

- Valentina Tesser

- Daniel Trujillo

Pontificia Universidad Católica de Chile – Curso ICP5006




