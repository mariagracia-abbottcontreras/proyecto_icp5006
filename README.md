# Descripción de Proyecto - ICP5006

## 1. Tema del proyecto
La influencia del desempleo comunal en el comportamiento electoral presidencial en Chile (2010–2017): un análisis del voto de castigo y apoyo en contextos de alternancia política y transformaciones laborales.

## 2. Objetivo
Analizar cómo las variaciones en la tasa de desempleo comunal influyen en la probabilidad de que los electores voten por candidatos presidenciales de oposición en Chile entre 2010 y 2017, examinando si los cambios en las condiciones laborales previas a las elecciones de 2013 y 2017 se asocian con un voto de castigo o de apoyo hacia el gobierno en ejercicio, en un contexto de alternancia política y transformaciones estructurales del mercado laboral. 

### 2.1 Objetivos específicos 
- Examinar la evolución de la tasa de desempleo comunal en Chile entre 2010 y 2017.
- Describir las variaciones territoriales en el apoyo electoral a los candidatos de gobierno y de oposición en las elecciones presidenciales de 2013 y 2017.
- Estimar el efecto de la tasa de desempleo comunal sobre el voto opositor, controlando por variables sociodemográficas y territoriales.
- Analizar si la relación entre desempleo y voto opositor varía según tipo de comuna, región o ciclo político. 

## 3. Pregunta de investigación
¿En qué medida la tasa de desempleo comunal influye en la probabilidad de que los electores voten por candidatos presidenciales de oposición en Chile entre 2010 y 2017?

## 4. Hipótesis
A medida que aumenta la tasa de desocupación en una comuna, dicha comuna tenderá, en promedio, a votar en mayor proporción por la oposición. 

### 4.1 Hipótesis secundarias
- H1: El efecto del desempleo sobre el voto opositor es más fuerte en comunas urbanas que en rurales.
- H2: El impacto del desempleo en el voto opositor se intensifica en contextos de desaceleración económica nacional.
- H3: En comunas con mayores niveles de pobreza o informalidad laboral, el desempleo influye con mayor intensidad en las preferencias electorales. 

### 4.2 Justificación
El período seleccionado (2010–2017) permite observar cómo fluctúan las tasas de desempleo en los años previos y posteriores a las elecciones presidenciales de 2013 y 2017. De esta forma, se podrá analizar si los cambios en las condiciones laborales comunales previas a cada elección se asocian con un voto de castigo o de apoyo hacia el gobierno en ejercicio. Este enfoque busca capturar el impacto de los ciclos económicos y políticos recientes, caracterizados por alternancia de coaliciones y transformaciones estructurales en el mercado laboral.

## 5. Diagnóstico del problema
La literatura sobre comportamiento electoral ha demostrado consistentemente la existencia de una relación entre desempeño económico y resultados electorales, fenómeno conocido como voto económico. En términos generales, los gobiernos tienden a ser castigados cuando la economía se deteriora y recompensados cuando esta mejora (Lewis-Beck y Stegmaier, 2019). Este patrón ha sido ampliamente documentado en democracias consolidadas, donde los votantes actúan como evaluadores retrospectivos del desempeño gubernamental. Sin embargo, en contextos latinoamericanos, y particularmente en Chile, dicha relación parece más ambigua y dependiente de factores institucionales, políticos y sociales. 

En el caso chileno, los estudios de Cerda y Vergara (2009) ofrecen evidencia de un voto económico retrospectivo durante los primeros años de la posdictadura, cuando los niveles de crecimiento y empleo favorecían sistemáticamente a las candidaturas oficialistas. No obstante, investigaciones más recientes, como las de Navia y Soto (2015) o Sáez-Lozano, Jaime-Castillo y Letelier-Saavedra (2014), advierten que la relación entre economía y voto se ha vuelto menos lineal. Factores como la aprobación presidencial, la identificación partidaria y la valoración del liderazgo político tienden a moderar o incluso desplazar el peso del desempeño económico en la decisión electoral. Así, aunque los votantes continúan evaluando la gestión económica, su comportamiento no responde mecánicamente a las variaciones macroeconómicas. 

Pese a este avance, la mayoría de los estudios sobre voto económico en Chile se han concentrado en indicadores nacionales —como el crecimiento del PIB o la inflación— y en la percepción individual del desempeño económico. En cambio, la dimensión territorial del fenómeno ha recibido menor atención. Las diferencias locales en materia de empleo, pobreza o estructura productiva pueden generar dinámicas electorales específicas, especialmente en un país caracterizado por una alta heterogeneidad socioeconómica entre comunas y regiones. En este sentido, el desempleo comunal constituye una variable clave para comprender cómo las condiciones económicas locales moldean las preferencias políticas. 

El período comprendido entre 2010 y 2017 ofrece una oportunidad analítica relevante para examinar este vínculo. Durante estos años, Chile experimentó alternancia política entre coaliciones, fluctuaciones en las tasas de desempleo y episodios de desaceleración económica. Observar cómo evolucionan las tasas de desocupación comunal antes y después de las elecciones presidenciales de 2013 y 2017 permite evaluar si el deterioro del mercado laboral se asocia con un voto de castigo hacia el gobierno o, por el contrario, con la mantención del apoyo oficialista. 

En síntesis, el problema que motiva esta investigación radica en la escasa evidencia empírica sobre el impacto territorial del desempleo en el comportamiento electoral chileno. A diferencia de los enfoques centrados en percepciones nacionales o individuales, este estudio busca examinar cómo las condiciones económicas locales inciden en la probabilidad de votar por la oposición, contribuyendo a una comprensión más desagregada y contextual del voto económico en Chile contemporáneo. 
## 6. Marco teórico


## 7. Metodología

### 7.1 Unidad de análisis y variables

### 7.2 Estrategia analítica
Se aplicarán modelos de regresión lineal múltiple (OLS) para estimar el efecto del desempleo comunal sobre el voto por la oposición. 
Para fortalecer la validez del análisis, se incluirán efectos fijos por región y año, con el fin de controlar diferencias estructurales y coyunturales entre territorios y ciclos electorales. 
El análisis se desarrollará en dos etapas:
      1. **Análisis descriptivo:** Revisión de la evolución temporal de las tasas de desempleo y los resultados electorales por comuna, identificando tendencias y correlaciones preliminares

### 7.3 Procedimiento y herramientas
El procesamiento y análisis de los datos se realizará mediante software estadístico R, utilizando técnicas de limpieza, normalización y depuración de bases de datos. 
Los resultados se presentarán mediante tablas de coeficientes, intervalos de confianza y gráficos comparativos que ilustren la relación entre desempleo y comportamiento electoral. 


## 8. Bases de datos a utilizar
### 8.1 Base de datos de ocupación y desocupación (INE)
- **Fuente:** Instituto Nacional de Estadísticas (INE)
- **Cobertura:** Información trimestral y anual sobre empleo y desempleo a nivel comunal y regional.
- **Variables principales:**
      - Tasa de desocupación (%)
      - Tasa de ocupación (%)
      - Población económicamente activa (PEA)
      - Población ocupada y desocupada por sexo
      - Participación laboral por tramo etario
  - **Uso en el estudio**: Se utilizarán las tasas promedio anuales de desempleo por comuna para los años 2010 a 2017, permitiendo comparar su evolución con los resultados electorales presidenciales. 

### 8.2 Base de datos de resultados presidenciales (Servel/CEP)
- **Fuente:** Base oficial del Centro de Estudios Públicos (CEP) elaborada a partir de datos del **Servicio Electoral de Chile (Servel)**
       — archivo: “resultados_elecciones_presidenciales_ce_1989_2017_Chile.xlsx”.
- **Cobertura:** Resultados comunales de todas las elecciones presidenciales entre 1989 y 2017.
- **Variables principales:**
      - Año de elección
      - Nombre del candidato/a
      - Pacto o coalición política
      - Votos por candidato
      - Porcentaje de votación por candidato

## 9. Referencias

Cerda, R., & Vergara, R. (2009). Business cycle and political economy: The Chilean case. Economía Chilena (The Chilean Economy), 12(2), 5–28.

Lewis-Beck, M. S., & Stegmaier, M. (2019). Economic voting. In R. S. Katz & W. Crotty (Eds.), The Oxford Handbook of Political Science (pp. 1–20). Oxford University Press. https://doi.org/10.1093/oxfordhb/9780199234479.003.0009

Navia, P., & Soto, A. (2015). Economic voting and incumbent support in Chile: Evidence from the 2009 presidential election. Latin American Politics and Society, 57(2), 89–111. https://doi.org/10.1111/j.1548-2456.2015.00269.x

Sáez-Lozano, J. L., Jaime-Castillo, A. M., & Letelier-Saavedra, F. (2014). Economic performance and support for the incumbent government: The case of Chile, 1990–2009. Revista Española de Investigaciones Sociológicas, 147, 3–26. https://doi.org/10.5477/cis/reis.147.3

