# Descripción de Proyecto - ICP5006

## 1. Tema del proyecto:
La influencia del desempleo comunal en el comportamiento electoral presidencial en Chile (2010–2017): un análisis del voto de castigo y apoyo en contextos de alternancia política y transformaciones laborales.

## 2. Objetivo: 
Analizar cómo las variaciones en la tasa de desempleo comunal influyen en la probabilidad de que los electores voten por candidatos presidenciales de oposición en Chile entre 2010 y 2017, examinando si los cambios en las condiciones laborales previas a las elecciones de 2013 y 2017 se asocian con un voto de castigo o de apoyo hacia el gobierno en ejercicio, en un contexto de alternancia política y transformaciones estructurales del mercado laboral. 

### 2.1 Objetivos específicos: 
- Examinar la evolución de la tasa de desempleo comunal en Chile entre 2010 y 2017.
- Describir las variaciones territoriales en el apoyo electoral a los candidatos de gobierno y de oposición en las elecciones presidenciales de 2013 y 2017.
- Estimar el efecto de la tasa de desempleo comunal sobre el voto opositor, controlando por variables sociodemográficas y territoriales.
- Analizar si la relación entre desempleo y voto opositor varía según tipo de comuna, región o ciclo político. 

## 3. Pregunta de investigación: 
¿En qué medida la tasa de desempleo comunal influye en la probabilidad de que los electores voten por candidatos presidenciales de oposición en Chile entre 2010 y 2017?

## 4. Hipótesis: 
A medida que aumenta la tasa de desocupación en una comuna, dicha comuna tenderá, en promedio, a votar en mayor proporción por la oposición. 

### 4.1 Hipótesis secundarias: 
- H1: El efecto del desempleo sobre el voto opositor es más fuerte en comunas urbanas que en rurales.
- H2: El impacto del desempleo en el voto opositor se intensifica en contextos de desaceleración económica nacional.
- H3: En comunas con mayores niveles de pobreza o informalidad laboral, el desempleo influye con mayor intensidad en las preferencias electorales. 

### 4.2 Justificación
El período seleccionado (2010–2017) permite observar cómo fluctúan las tasas de desempleo en los años previos y posteriores a las elecciones presidenciales de 2013 y 2017. De esta forma, se podrá analizar si los cambios en las condiciones laborales comunales previas a cada elección se asocian con un voto de castigo o de apoyo hacia el gobierno en ejercicio. Este enfoque busca capturar el impacto de los ciclos económicos y políticos recientes, caracterizados por alternancia de coaliciones y transformaciones estructurales en el mercado laboral.

### 4.3 Diagnóstico del problema

## 5. Marco teórico


## 6. Metodología

### 6.1 Unidad de análisis y variables

### 6.2 Estrategia analítica
Se aplicarán modelos de regresión lineal múltiple (OLS) para estimar el efecto del desempleo comunal sobre el voto por la oposición. 
Para fortalecer la validez del análisis, se incluirán efectos fijos por región y año, con el fin de controlar diferencias estructurales y coyunturales entre territorios y ciclos electorales. 
El análisis se desarrollará en dos etapas:
      1. **Análisis descriptivo:** Revisión de la evolución temporal de las tasas de desempleo y los resultados electorales por comuna, identificando tendencias y correlaciones preliminares

### 6.3 Procedimiento y herramientas
El procesamiento y análisis de los datos se realizará mediante software estadístico R, utilizando técnicas de limpieza, normalización y depuración de bases de datos. 
Los resultados se presentarán mediante tablas de coeficientes, intervalos de confianza y gráficos comparativos que ilustren la relación entre desempleo y comportamiento electoral. 


## 7. Bases de datos a utilizar
### 7.1 Base de datos de ocupación y desocupación (INE)
- **Fuente:** Instituto Nacional de Estadísticas (INE)
- **Cobertura:** Información trimestral y anual sobre empleo y desempleo a nivel comunal y regional.
- **Variables principales:**
      - Tasa de desocupación (%)
      - Tasa de ocupación (%)
      - Población económicamente activa (PEA)
      - Población ocupada y desocupada por sexo
      - Participación laboral por tramo etario
  - **Uso en el estudio**: Se utilizarán las tasas promedio anuales de desempleo por comuna para los años 2010 a 2017, permitiendo comparar su evolución con los resultados electorales presidenciales. 

### 7.2 Base de datos de resultados presidenciales (Servel/CEP)
- **Fuente:** Base oficial del Centro de Estudios Públicos (CEP) elaborada a partir de datos del **Servicio Electoral de Chile (Servel)**
       — archivo: “resultados_elecciones_presidenciales_ce_1989_2017_Chile.xlsx”.
- **Cobertura:** Resultados comunales de todas las elecciones presidenciales entre 1989 y 2017.
- **Variables principales:**
      - Año de elección
      - Nombre del candidato/a
      - Pacto o coalición política
      - Votos por candidato
      - Porcentaje de votación por candidato
