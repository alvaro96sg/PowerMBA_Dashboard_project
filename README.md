# Analítica de Rendimiento y Táctica en la NBA: Dashboard de Eficiencia con Excel y Power Query

Proyecto del módulo '4. Dashboard & Análisis de Datos' para el **Máster en Data & Analytics** en **PowerMBA**.

---

# 1. Descripción del Proyecto

Este proyecto realiza un análisis estadístico y táctico a partir de los datos históricos de partidos de la NBA. A través del procesamiento y transformación de datos en **Power Query** y el modelado dinámico en **Microsoft Excel 2019**, se ha construido un **Dashboard interactivo** capaz de evaluar la eficiencia en distintas métricas de rendimiento, las tendencias de tiro por zonas, los factores clave en la victoria y la evolución temporal de los equipos.

El dataset original (`TeamStatistics.csv`) procede de Kaggle ([NBA Dataset: Box Scores and Stats](https://www.kaggle.com/datasets/eoinamoore/historical-nba-data-and-player-box-scores/)) y se ha filtrado para analizar exclusivamente el comportamiento de la temporada **2025-2026** (a partir del 21 de octubre de 2025) y omitiendo partidos de exhibición.

---

# 2. Estructura del Repositorio

Este repositorio está organizado siguiendo los requisitos del proyecto.

PowerMBA_Dashboard_project/
│
├── dashboard/
│   └── NBAteams_Analysis_and_Dashboard.xlsx
│
├── data/
│   └── TeamStatistics.csv
│
└── README.md

### 2.1. Descripción de las Carpetas y Archivos

* **dashboard/** → Archivo ejecutable de Excel que alberga el modelo de datos, las tablas dinámicas auxiliares y el Panel de Control interactivo.
* **data/raw/** → Dataset original con estadísticas acumuladas por partido y equipo.
* **docs/** → Capturas y documentación visual del proyecto.

---

# 3. Flujo de Trabajo y Tratamiento de Datos

El proyecto se desarrolló siguiendo los siguientes pasos principales:

### 3.1. Extracción y Limpieza en Power Query

* **Importación y Tipado**: Carga del archivo `TeamStatistics.csv` en **Power Query** y anulación de la detección automática de tipos para evitar errores de conversión.
* **Ingeniería de Claves Temporales**: Duplicación de la columna gameDate para aislar la fecha pura (`gameOnlyDate`) y extracción de campos derivados (Mes y Año).
* **Filtrado y Eliminación**:
  * Partidos de exhibición (All-Star) (`gameType`) se eliminaron.
  * La temporada 2025-2026 (partidos disputados desde el 21/10/2025) es la elegida para el análisis.
* **Depuración de Variables**: Selección de identificadores primarios (`gameId`, `teamId`), fechas y estadísticas de juego regulares.
* **Tratamiento Numérico y Formato**: Sustitución de puntos por comas en valores decimales, ajuste de redondeos y tipado a número entero/decimal según la métrica.

### 3.2. Auditoría y Control de Calidad

* **Generación de identificador único**: Creación del campo concatenado teamgameId (gameId & teamId).
* **Auditoría de Duplicados**: Validación y depuración previa para garantizar la integridad de los datos (0 registros duplicados encontrados).
* **Tratamiento de Nulos**: La eliminación previa de registros del All-Star y columnas no relevantes resultó en la eliminación total de celdas vacías en el conjunto final.

### 3.3. Creación de métricas

* **TrueShooting%**: mide la eficiencia de anotación en baloncesto. Se define como:

$$
TrueShooting\% =\frac{Points}{2 \cdot ( FieldGoalsAttempted + 0.44 \cdot FreeThrowsAttempted)}
$$

* **EstimatedPossessions y EstimatedPossessionsOpponent**: número aproximado de veces que un equipo tiene el balón para atacar en un partido. Se define como:

$$
EP = FieldGoalsAttempted + 0.44 \cdot FreeThrowsAttempted + TurnOvers - OffensiveRebounds
$$

* **OffensiveRating**: cantidad de puntos que anota un equipo por cada 100 posesiones. Se define como:

$$
OR = \frac{Points}{EP}\cdot 100
$$

* **DefensiveRating**: cantidad de puntos que permite un equipo por cada 100 posesiones. Se define como:

$$
DR = \frac{OpponentPoints}{EPOpponent}\cdot 100
$$

* **NetRating**: diferencia de puntos anotados y permitidos por un equipo o jugador por cada 100 posesiones. Se define como:

$$
NetRating = OR - DR
$$

* **Assists/TurnOvers Ratio**: mide la eficacia de un equipo al manejar el balón. Se calcula dividiendo el número total de asistencias entre las pérdidas de balón (TurnOvers). Se define como:

$$
Assists/TurnOvers Ratio = \frac{Assists}{TurnOvers}
$$

---

# 4. Arquitectura del Libro de Excel

El proyecto se estructuró mediante distintas hojas de cálculo para modularizar el procesamiento de datos y la presentación.

* **Datos**: Contiene la tabla base procesada. Se añaden métricas calculadas adicionales (medidores de efectividad, indicadores de dominio, métricas de rendimiento, etc.).
* **KPI's**: Almacena los indicadores clave consolidados (Big Numbers) proyectados en la cabecera del Dashboard.
* **Análisis_Influencia_estadística**: Tabla dinámica que promedia el rendimiento de cada variable estadística segregada por resultado ($W / L$). Base para el gráfico 'Influencia de Variables en la Victoria' para el Dashboard.
* **Análisis_lanzamientos**: Segmentación de puntos anotados según la zona ($PTS_{2P}$ dentro del área vs $PTS_{3P}$ fuera del área). Genera las coordenadas para el Diagrama de Dispersión.
* **Análisis_temporal**: Hoja que modela la línea cronológica de métricas por fecha y resultado ($W / L$).
* **Análisis_TOP** y **Análisis_TOP_winrate**: Tablas dinámicas con los rankings de los 5 mejores y 5 peores equipos según la métrica seleccionada, asociando su impacto directo en el Winrate ($\% \text{Victorias}$).
* **Dashboard**: Panel de control interactivo con segmentadores de datos (Slicers) conectados a todas las tablas dinámicas auxiliares.

---

# 5. Componentes del Dashboard Interactivo

1. **Indicadores clave de Rendimiento (KPIs)**: Tarjetas dinámicas que muestran el promedio de Puntos Anotados, Asistencias y Porcentajes de Acierto en Tiros de Campo, Triples y Tiros Libres según la selección.
2. **Panel de Control**:
   * **Segmentador de Datos**: Permite el filtrado dinámico por equipos, actualizando a tiempo real la totalidad de los gráficos e indicadores a excepción del *Ranking TOP 5*.
   * **Validación de Datos**: Lista desplegable que alterna la Métrica activa, modificando las visualizaciones de *Evolución Temporal* y *Ranking TOP 5*.
3. **Mapa de Dispersión (Zonas de Anotación)**: Matriz que clasifica el perfil de los equipos en 4 cuadrantes según el balance entre anotación perimetral ($PTS_{3P}$) e interior/media distancia ($PTS_{2P}$) (*Juego Perimetral*, *Ataque Total*, *Ataque Ineficiente* y *Juego Interior*).
4. **Factores de Influencia en la Victoria**: Gráfico de barras apiladas que enfrenta el peso porcentual de cada métrica (Rebotes, Robos, Pérdidas, etc.) cuando el equipo gana frente a cuando pierde ($W / L$).
5. **Evolución Temporal de Métricas**: Gráfico de líneas que compara la tendencia temporal de cualquier métrica seleccionada entre victorias y derrotas a lo largo de la temporada.
6. **Ranking TOP 5 (Mejores y Peores Equipos)**: Tabla comparativa que evalúa el rendimiento extremo de los equipos en la métrica activa y su correlación directa con su porcentaje global de victorias (Winrate).

---

# 6. Principales Hallazgos e Informe del Análisis

Tras el procesamiento de los datos y la exploración a través del Dashboard, se han extraído las siguientes conclusiones estratégicas sobre la dinámica de juego en la temporada 2025-2026:

### Perfiles según Zonas de Anotación

* **Dominación por Adaptabilidad:** Los equipos dominantes presentan un diagrama de dispersión considerablemente más abierto, lo que podría indicar una mayor capacidad para adaptar su juego a las características del rival y a distintas situaciones de partido. En cambio, los equipos con un menor *Winrate* muestran una distribución de los lanzamientos más compacta y concentrada en la zona central del mapa.
  Esta mayor concentración sugiere un estilo de juego más encasillado, con una menor variedad en la selección de lanzamientos. Esta falta de adaptabilidad podría suponer una desventaja frente a equipos capaces de plantear diferentes estrategias defensivas, como una mayor presión en las zonas de lanzamiento más frecuentes o la utilización de jugadores exteriores con velocidad suficiente para dificultar los tiros desde mayores distancias.
* **Ataque Total:** Los equipos con mayor *Winrate* de la liga no dependen de una sola zona; se ubican predominantemente en el cuadrante de Ataque Total (alto volumen de $PTS_{2P}$ y $PTS_{3P}$).

### Determinantes Críticos de la Victoria

* **El dominio del rebote defensivo, los robos y los tapones:** Los equipos más dominantes presentan una superioridad constante en las variables defensivas, situándose estas de forma recurrente entre las cinco características más influyentes en la consecución de la victoria.
* **Impacto del factor cancha:** Los equipos más ganadores presentan una proporción de victorias en casa considerablemente superior a la de los equipos de la parte baja de la clasificación. Esto pone de manifiesto la relevancia del factor cancha como uno de los condicionantes asociados al éxito de los equipos en la lucha por el título de liga.

### Evolución Temporal de Métricas

* **Predominio de la eficiencia ofensiva:** Las victorias presentan valores superiores en Puntos Anotados, % de Tiro Real, Calificación Ofensiva, % de Tiros de Campo y % de Triples, destacando la importancia de la eficiencia ofensiva para lograr la victoria.
* **Mayor Calificación Defensiva en las derrotas:** La Calificación Defensiva presenta valores superiores en las derrotas. Esto refleja un mayor rendimiento defensivo y una dominancia en facetas ofensivas y de posesión del equipo rival. Esto puede provocar una mayor realización de acciones defensivas exitosas, aunque no suficientes para la victoria.
* **Influencia de las jugadas puntuales:** Las Asistencias/Pérdidas, Pérdidas, Puntos tras Pérdida y Puntos al Contrataque muestran un comportamiento más variable, reflejando la influencia de situaciones puntuales durante el partido.
* **Mayor estabilidad en las derrotas:** Los Rebotes Ofensivos y Rebotes Defensivos presentan un comportamiento más constante en las derrotas y mayor variabilidad en las victorias, por lo que no parecen ser factores determinantes por sí solos.
* **Diferencias en el comportamiento según la posición en la clasificación:** Las Asistencias presentan un comportamiento particular. Los equipos más dominantes no muestran una diferencia significativa en el número de Asistencias entre los partidos ganados y perdidos. En cambio, en los equipos de media y baja tabla se observa una diferencia más clara, registrando un mayor número de Asistencias en las victorias.

### Correlación entre Rankings y Winrate

* **Equipos fuertes defensivamente:** Se corrobora la hipótesis planteada anteriormente. Los equipos con una Calificación Defensiva alta presentan los *Winrate* más bajos de la competición, mientras que aquellos con una peor calificación en esta métrica alcanzan *Winrate* excepcionalmente altos.
* **Los detalles marcan la diferencia:** Aunque anteriormente las pérdidas de balón se presentaban como acciones puntuales con una influencia situacional, su acumulación se muestra como un claro indicador del rendimiento a largo plazo. Los equipos que registran un menor número de pérdidas presentan los mejores *Winrate* de la competición.
* **Acierto en los tiros:** El porcentaje de acierto en  Tiros de Campo , Triples y la métrica de Tiro Real se posicionan como algunos de los mejores indicadores del éxito colectivo.

# 7. Herramientas Utilizadas

Microsoft Excel 2019:

* *Power Query*: Extracción, transformación y limpieza de datos (ETL).
* Funciones Avanzadas de Microsoft Excel 2019.
* Tablas y Gráficos Dinámicos.
* Formato Condicional, Segmentador de datos y Validación de datos.
