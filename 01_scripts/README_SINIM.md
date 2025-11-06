# Procesamiento de SINIM

- **Proyecto**: Efecto del desempleo comunal sobre el voto opositor en Chile: un análisis electoral 2010–2017
- **Curso**: ICP5006
- **Integrantes**: María Gracía Abbott, Valentina Tesser y Daniel Trujillo

## Objeto del README

Explicar en detalle el flujo de transformación de la base `base_control_SINIM` y justificar las decisiones de limpieza y agregación en relación con los objetivos y las hipótesis del estudio.

## Resumen y propósito del script

El script `01_df_SINIM.Rmd` tiene como objetivo transformar la base original de variables demográficas comunales (archivo original: `datos_comunales_SINIM.xlsx`) en la tabla analítica `base_control_SINIM` lista para ser integrada con los datos de desempleo comunal `(ENE)` y los resultados electorales comunales `(df_presidenciales_clean)`. La tabla final contiene, por comuna y año (2010–2017), el código comunal y las variables de control seleccionadas:

- `densidad_pob` — Densidad de población (hab/km²)

- `poblacion_tot` — Población comunal total

- `porc_femenina` — % de población femenina

- `porc_rural` — % de población rural

Estas variables permiten controlar características demográficas locales al analizar la relación entre desempleo comunal (ENE) y el voto opositor en las presidenciales 2013 y 2017.

## Fuentes y alcance temporal

La base original proviene del Sistema Nacional de Información Municipal (SINIM), administrado por la Subsecretaría de Desarrollo Regional y Administrativo (SUBDERE) y contiene indicadores comunales como población, densidad, porcentaje femenino y ruralidad. Para este proyecto, decidimos focalizar el análisis dentro del rango 2010-2017, pues permite vincular estos indicadores y los de la ENE con las elecciones presidenciales realizadas dentro de esos años (2013 y 2017). Esto permite robustecer el estudio del voto opositor en contextos de alternancia política. Si bien, se intentó que todas los indicadores tuvieran datos de todos los años, la variable `densidad_pob` solamente contiene datos entre 2010 y 2013. Se buscará resolver este problema en siguientes versiones.

## Estructura del procesamiento y decisiones principales

0. **Configuración y paquetes**

La primera etapa es cargar los paquetes necesarios y limpiar el entorno para reproducibilidad.

```{r}
rm(list = ls())
library(here)   
library(readxl)
library(tidyverse)   
library(stringr)
library(stringi)
library(janitor)
```

1. **Lectura del archivo**

Se importa la base original descargada desde el Sistema Nacional de Información Municipal (SINIM).
Para evitar que los encabezados o comentarios afecten la estructura de la tabla, se omiten las tres primeras filas (`skip = 3`) y se indica que las columnas no tienen nombres (`col_names = FALSE`), lo que facilita asignarlos manualmente más adelante.

```{r}
df_0 <- read_excel(here("00_data","datos_comunales_SINIM.xlsx"), skip = 3, col_names = FALSE)
```

2. **Asignación de nombres de columnas**

Se determinaron primero los años a utilizar. Luego se crearon con esto nombres consistentes para cada variable por año. De esta manera el formato de `variable_año` permite identificar rápidamente las series temporales y facilita la transformación posterior al formato largo, esencial para el análisis longitudinal.

```{r}
#Definimos los años en orden descendente (como en el archivo)
años <- 2017:2010

#Con eso, asignamos nombres a las columnas
colnames(df_0) <- c(
  "codigo", "comuna",
  paste("densidad_pob", años, sep = "_"),
  paste("poblacion_tot", años, sep = "_"), 
  paste("porc_femenina", años, sep = "_"),
  paste("porc_rural", años, sep = "_")
)

#Revisamos los nombres de las columnas
colnames(df_0)
```

3. **Limpieza y conversión de tipos**

Se convierten las variables a formato numérico y se reemplazan los valores no válidos (como `"No Recepcionado"`) por `NA`. Además, se eliminan las observaciones sin código comunal. Este paso asegura coherencia tipológica y elimina ruido, lo que garantiza la validez del análisis estadístico posterior.

```{r}
df_1 <- df_0 %>%
  mutate(
    codigo = as.numeric(codigo),
    across(starts_with("densidad_"), as.numeric),
    across(starts_with("poblacion_"), as.numeric),
    across(starts_with("porc_femenina_"), as.numeric),
    across(starts_with("porc_rural_"), ~ ifelse(.x == "No Recepcionado", NA, as.numeric(.x)))
  ) %>%
  filter(!is.na(codigo))

summary(df_1)
```

4. **Reestructuración a formato largo**

Se transforma la base de formato ancho (con una columna por año) a formato largo, donde cada fila representa una comuna en un año determinado. Esta estructura es la más adecuada para análisis comparativos en el tiempo y facilita la integración con otras fuentes longitudinales.

```{r}
df_2 <- df_1 %>%
  pivot_longer(
    cols = -c(codigo, comuna),  # Todas excepto ID's
    names_to = c(".value", "año"),
    names_pattern = "(.*)_(\\d+)",  # Patrón: variable_año
    values_drop_na = FALSE  # Mantiene NA's si quieres analizarlos
  ) %>%
  mutate(año = as.integer(año))
```

5. **Ordenar el resultado**

Se ordenan las observaciones por comuna y año, y se reordenan las columnas en un orden lógico y legible. Esto estandariza la estructura final y permite una rápida exploración y verificación de los datos.

```{r}
df_completa <- df_2 %>%
  arrange(comuna, año) %>%
  select(codigo, comuna, año, densidad_pob, poblacion_tot, porc_femenina, porc_rural)  # Orden de columnas deseado

head(df_completa)
```

6. **Limpieza de los nombres de comunas**

Se limpian los nombres de comunas convirtiéndolos a minúsculas, eliminando acentos y espacios extra.Este proceso homogeneiza la escritura de los nombres, evitando errores al unir bases que pueden tener diferencias menores de formato (como “Ñuñoa” vs “nunoa”).

```{r}
df_final <- df_completa |> 
  mutate(
    comuna = comuna |> 
      str_to_lower() |> 
      stri_trans_general("Latin-ASCII") |> 
      str_trim() |> 
      str_squish()
         )

head(df_final)
```

7. **Normalizar con base de resultados presidenciales**

Se comparan los nombres de comunas con los de la base `df_presidenciales_clean` para detectar diferencias ortográficas. Posteriormente se corrigen manualmente las discrepancias conocidas, lo que asegura la compatibilidad entre ambas fuentes y posibilita futuras integraciones sin pérdida de observaciones.

```{r}
df_final |> 
  anti_join(df_presidenciales_clean, by = "comuna") |> 
  select(comuna) |> 
  distinct()
  
df_final <- df_final |>
  mutate(comuna = case_when(
    comuna == "aysen" ~ "aisen",
    comuna == "trehuaco" ~ "treguaco",
    comuna == "llaillay" ~ "llay-llay",
    comuna == "marchihue" ~ "marchigue",
    comuna == "paiguano" ~ "paihuano",
    comuna == "o´higgins" ~ "o'higgins",
    TRUE ~ comuna
  ))

```

8. **Normalizar con base ENE**

Se intenta alinear la base del SINIM con la de la Encuesta Nacional de Empleo (ENE) usando los códigos comunales. Durante el proceso se identifican comunas faltantes en la ENE, lo cual se documenta para resolver en una etapa posterior. Esto permite detectar vacíos en la cobertura de datos y preparar una integración más precisa.

```{r}
ENE_2020_2017_full_vadjust1 <- ENE_2010_2017_full_vadjust |> 
  rename(comuna = com_caracter, codigo = r_p_c)


codigos_faltantes <- df_final |> 
  anti_join(ENE_2020_2017_full_vadjust1, by = "codigo") |> 
  select(codigo) |> 
  distinct() |> 
  pull()


df_final |>
  filter(codigo %in% codigos_faltantes) |> 
  distinct(codigo, .keep_all = TRUE)

```

9. **Exportación reproducible**

Se exporta la base final depurada en formato .xlsx dentro de la carpeta 02_outputs. Gracias al uso de here(), la ruta es relativa y funcionará sin cambios en cualquier entorno del proyecto. El archivo resultante base_control_SINIM.xlsx constituye la versión final y lista para análisis o vinculación con otras bases.

```{r}
writexl::write_xlsx(df_final, here("02_outputs","base_control_SINIM.xlsx"))

```

## Contacto y créditos

**Equipo de investigación:**

Proyecto: Efecto del desempleo comunal sobre el voto opositor en Chile (2010–2017)

- María Gracia Abbott

- Valentina Tesser

- Daniel Trujillo

Pontificia Universidad Católica de Chile – Curso ICP5006


