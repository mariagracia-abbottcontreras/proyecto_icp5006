# Procesamiento de resultados presidenciales comunales

- **Proyecto**: Efecto del desempleo comunal sobre el voto opositor en Chile: un análisis electoral 2010–2017
- **Curso**: ICP5006
- **Integrantes**: María Gracía Abbott, Valentina Tesser y Daniel Trujillo

## Objeto del README

Explicar en detalle el flujo de transformación de la base `df_presidenciales` y justificar las decisiones de limpieza y agregación en relación con los objetivos y las hipótesis del estudio.

## Resumen y propósito del script

El script `01_df_presidenciales.Rmd` tiene como objetivo transformar la base original de resultados electorales comunales (archivo Servel: `resultados_elecciones_presidenciales_ce_1989_2017_Chile.xlsx`) en una tabla analítica `df_presidenciales_clean` lista para ser integrada con los datos de desempleo comunal (ENE) y otras bases de control. La tabla final contiene, para cada comuna y elección relevante (2013 y 2017), el candidato electo en el respectivo año (Michelle Bachelet en 2013 y Sebastián Piñera en 2017), sus votos comunales, el total de votos de la comuna en esa elección y el porcentaje de votos del candidato electo respecto del total comunal. Esta estructura responde directamente al objetivo de la investigación: **analizar cómo las variaciones de desempleo a nivel comunal se asocian con el voto por la oposición en las elecciones presidenciales de 2013 y 2017**.

## Fuentes y alcance temporal

La base original proviene del Servicio Electoral (Servel) y contiene resultados comunales desagregados por candidato para múltiples elecciones. Para este proyecto, decidimos focalizar el análisis en las elecciones presidenciales de 2013 y 2017 porque permiten comparar dos ciclos electorales recientes, uno con victoria de la centroizquierda (Bachelet) y otro con victoria de la derecha (Piñera), lo que facilita estudiar el voto opositor en contextos de alternancia política. La decisión de restringir el análisis a estos años busca evitar mezclar contextos políticos demasiado distintos y garantizar que los totales comunales y la competencia electoral sean coherentes por elección.

## Principales decisiones de preprocesamiento y su justificación

1. **Limpieza inicial de nombres de variables (`janitor::clean_names`)**

La primera etapa es estandarizar los nombres de columnas con `clean_names()` para asegurar consistencia al programar y evitar errores tipográficos. Esto facilita la reproducibilidad y la lectura del script por terceros.

```{r}
# Cargar base
df_presidenciales <- read_excel(
  here("00_data", "resultados_elecciones_presidenciales_ce_1989_2017_Chile.xlsx"))

# Limpiar nombres de variables
df_presidenciales <- df_presidenciales %>%
janitor::clean_names()

# Vista general
glimpse(df_presidenciales)
```

2. **Procesar cada elección por separado (`función procesar_eleccion`)**

Se implementó una función `procesar_eleccion(base, ano, candidato)` que encapsula el flujo de transformación para un año y candidato dado: filtrar el año, sumar votos por `region`, `comuna`, `candidato_a`, calcular total comunal (suma de votos de todos los candidatos en esa comuna y elección), filtrar el candidato electo y calcular su porcentaje comunal. La razón para procesar cada elección por separado es doble: (1) garantiza que el `total_de_votos` refleje la competencia específica de esa elección (porque en cada elección participa una cantidad distinta de candidatos), y (2) evita mezclar totales o porcentajes entre años, lo que sería problemático para medir la participación relativa del candidato electo dentro del contexto electoral correcto. Esta decisión es central para medir el efecto del desempleo sobre el voto opositor, pues queremos que el porcentaje comunal del candidato electo represente su rendimiento frente a los competidores reales de esa misma elección.

Calcular el total de votos por comuna y año permite construir la métrica de interés: el porcentaje de votos del candidato electo sobre el total comunal. Esta métrica captura, a nivel territorial, el apoyo relativo del candidato dentro de su propia contienda, lo que es indispensable para comparar la variabilidad territorial del voto oficialista/opositor y vincularla con condiciones económicas locales (p. ej. desempleo).

Aunque tratamos candidatos por separado, cada observación conserva el año de elección (`ano_de_eleccion`) para facilitar empalmes temporales con ENE (que está organizado por año), para permitir controles por periodo, y para poder comparar resultados entre 2013 y 2017 en análisis posteriores.

```{r}
# Función general para procesar una elección
procesar_eleccion <- function(base, ano, candidato) {
  base %>%
    filter(ano_de_eleccion == ano) %>%
    group_by(region, comuna, candidato_a) %>%
    summarise(votos_obtenidos = sum(votos_totales, na.rm = TRUE),
              .groups = "drop") %>%
    group_by(region, comuna) %>%
    mutate(total_de_votos = sum(votos_obtenidos, na.rm = TRUE)) %>%
    ungroup() %>%
    filter(candidato_a == candidato) %>%
    mutate(
      porcentaje_votos = round((votos_obtenidos / total_de_votos) * 100, 1),
      ano_de_eleccion = ano) %>%
    select(ano_de_eleccion, region, comuna, candidato_a,
           votos_obtenidos, total_de_votos, porcentaje_votos)}

# Aplicar la función a ambas elecciones
bachelet_2013 <- procesar_eleccion(df_presidenciales, 2013, "MICHELLE BACHELET JERIA")
pinera_2017 <- procesar_eleccion(df_presidenciales, 2017, "SEBASTIAN PIÑERA ECHENIQUE")

# Unir ambas bases
df_presidenciales_clean <- bind_rows(bachelet_2013, pinera_2017)

# Ordenar observaciones
df_presidenciales_clean <- df_presidenciales_clean %>%
arrange(region, comuna, ano_de_eleccion)

cat("✅ Transformación de variables completada.\n")
```

3. **Normalización de variables de texto**

Se normalizaron todas las columnas de tipo carácter (región, comuna, candidato) transformándolas a minúsculas, eliminando acentos y caracteres especiales y reduciendo espacios redundantes. Esta estandarización es imprescindible para unir `df_presidenciales_clean` con otras bases (`ENE`, `SINIM`) que pueden tener convenciones de nombre diferentes (mayúsculas, tildes, abreviaturas). Una buena normalización minimiza errores en los joins y reduce inspecciones manuales por conflictos de formato.

```{r}
df_presidenciales_clean <- df_presidenciales_clean %>%
  mutate(across(where(is.character), ~ .x %>%
    str_to_lower() %>%                      # todo en minúsculas
    stringi::stri_trans_general("Latin-ASCII") %>%  # elimina tildes y caracteres especiales
    str_trim() %>%                          # elimina espacios al inicio y al final
    str_squish()                            # elimina espacios dobles o múltiples
  ))

# Mensaje de confirmación
cat("✅ Variables de texto normalizadas correctamente.\n")
```

4. **Cálculo de `proporcion_votos`**

Además del porcentaje en 0-100, se creó `proporcion_votos` (valor en 0–1) para facilitar su uso en modelos econométricos y comparaciones estadísticas donde se prefiere escala relativa o transformaciones logísticas. Esta variable sirve como `OppoVoteᵢ,ₜ` (o, en alternativa, se puede construir el % del candidato opositor si el candidato electo fuera considerado "oficialismo").

```{r}
df_presidenciales_clean <- df_presidenciales_clean %>%
mutate(proporcion_votos = votos_obtenidos / total_de_votos)
```
5. **Control de calidad y validaciones**

El script incorpora varias verificaciones: `glimpse()` y `str()` para inspección de tipos, `colSums(is.na(...))` para detectar valores faltantes, y conteos por `region`, `comuna`, `ano_de_eleccion` para detectar duplicados. Además, se recomienda ejecutar una comprobación final que contraste, por ejemplo, que la suma de `total_de_votos` por comuna y año coincida con valores oficiales o con sumas calculadas desde la base original si se desean auditorías más estrictas. Estas verificaciones previenen errores al unir con `ENE` y evitan sesgos por observaciones faltantes o mal agregadas.

```{r}
# Estructura general
str(df_presidenciales_clean)

# Revisión de valores faltantes
colSums(is.na(df_presidenciales_clean))

# Detección de duplicados
df_presidenciales_clean %>%
count(region, comuna, ano_de_eleccion) %>%
filter(n > 1)
```

6. **Exportación reproducible**

La salida se exporta a `02_outputs/df_presidenciales_clean.xlsx` usando here() para construir rutas relativas al proyecto. Se verifica la existencia de la carpeta de outputs antes de escribir el archivo. Esto facilita reproducibilidad y colaboración (otros integrantes del proyecto podrán ejecutar el script sin modificar rutas).

## Contacto y créditos

**Equipo de investigación:**

Proyecto: Efecto del desempleo comunal sobre el voto opositor en Chile (2010–2017)

- María Gracia Abbott

- Valentina Tesser

- Daniel Trujillo

Pontificia Universidad Católica de Chile – Curso ICP5006


