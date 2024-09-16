# Algunos de los ejercicios del curso de PostgreSQL Summary Stats and Window Functions

Puedes acceder al curso  [aquí](https://app.datacamp.com/learn/courses/postgresql-summary-stats-and-window-functions), tiene una duración de 4 horas, 12 videos y 44 ejercicios. 👩‍💻

Certificado  [aquí](https://www.datacamp.com/completed/statement-of-accomplishment/course/d29991f7096b83f70bd19aa9df980f6f9595d27e)

## Descripción del curso:

Aprenderás a crear consultas para análisis e ingeniería de datos con funciones de ventana, ¡el arma secreta de SQL! Utilizando datos de vuelos, descubrirás lo sencillo que es utilizar las funciones de ventana, y lo flexibles y eficaces que son.

## 1️⃣ Introduccion a las funciones de ventana:

Aprenderás qué son las funciones ventana y las dos subcláusulas básicas de las funciones ventana, ORDER BY y PARTITION BY.

### Numeracion de filas: 

La aplicacion mas sencilla de las funciones de ventana es la numeracion de filas. Numerar las filas te permite buscar facilmente la fila n.

-- Asignar números a cada fila
```
SELECT *,
ROW_NUMBER() OVER() AS Row_N
FROM Summer_Medals
ORDER BY Row_N ASC;
```

-- Asigna un número a cada año en que se celebraron los Juegos Olímpicos de Verano.
```
SELECT
  Year,

  -- Assign numbers to each year
  ROW_NUMBER() OVER() AS Row_N
FROM (
  SELECT DISTINCT Year
  FROM Summer_Medals
  ORDER BY Year ASC
) AS Years
ORDER BY Year ASC;
```

### ORDER BY

-- Asigna un número a cada año en que se celebraron los Juegos Olímpicos de Verano, de modo que las filas con los años más recientes tengan números de fila más bajos.

```
SELECT
  Year,
  -- Assign the lowest numbers to the most recent years
  ROW_NUMBER() OVER (ORDER BY Year DESC) AS Row_N
FROM (
  SELECT DISTINCT Year
  FROM Summer_Medals
) AS Years
ORDER BY Year;
```

-- Para cada atleta, cuenta el número de medallas que ha ganado.

```
SELECT
  -- Count the number of medals each athlete has earned
  athlete, COUNT(*) AS Medals
FROM Summer_Medals
GROUP BY Athlete
ORDER BY Medals DESC;
```

-- Devuelve los medallistas de oro de cada año en la competición masculina de halterofilia de 69 kg.

```
SELECT
  -- Return each year's champions' countries
  Year,
  Country AS champion
FROM Summer_Medals
WHERE
  Discipline = 'Weightlifting' AND
  Event = '69KG' AND
  Gender = 'Men' AND
  Medal = 'Gold';
```

### PARTITION BY 

-- Devuelve los campeones anteriores del evento de cada año por sexo.

```
WITH Tennis_Gold AS (
  SELECT DISTINCT
    Gender, Year, Country
  FROM Summer_Medals
  WHERE
    Year >= 2000 AND
    Event = 'Javelin Throw' AND
    Medal = 'Gold')

SELECT
  Gender, Year,
  Country AS Champion,
  -- Fetch the previous year's champion by gender
  LAG(Country) OVER (PARTITION BY Gender
            ORDER BY Year ASC) AS Last_Champion
FROM Tennis_Gold
ORDER BY Gender ASC, Year ASC;
```

-- Devuelve los campeones anteriores de las pruebas de cada año por sexo y prueba.

```
WITH Athletics_Gold AS (
  SELECT DISTINCT
    Gender, Year, Event, Country
  FROM Summer_Medals
  WHERE
    Year >= 2000 AND
    Discipline = 'Athletics' AND
    Event IN ('100M', '10000M') AND
    Medal = 'Gold')

SELECT
  Gender, Year, Event,
  Country AS Champion,
  -- Fetch the previous year's champion by gender and event
   LAG(Country) OVER (PARTITION BY Event, Gender
            ORDER BY Year ASC) AS Last_Champion
FROM Athletics_Gold
ORDER BY Event ASC, Gender ASC, Year ASC;
```


## 2️⃣ Búsqueda, clasificación y paginación: 

Aprenderás tres aplicaciones prácticas de las funciones ventana: obtener valores de distintas partes de la tabla, clasificar filas según sus valores y agrupar filas en distintas tablas.


### 1- Búsqueda

#### LEAD: 

Si tienes datos ordenados en el tiempo, puedes hacerlo con la funcion de busqueda LEAD. Esto es especialmente útil si quieres comparar un valor actual con un valor futura.

-- Para cada afo, busca el medallista de aro actual y el medalliata de oro de 3 competicianes anteriores a la fila actual.

```
WITH Discus_Medalists AS (
SELECT DISTINCT

Athlete
FROM Suner_Hedals
WHERE Medal = 'Gold'
AND Event = 'Discus Throw"
AND Gender = 'Women'
AND Year >= 2000)

SELECT
-- For each year, fetch the current and future nedalists
athlete,
year,
LEAD(athlete, 3)OVER (ORDER BY year ASC) AS Future_Champion
FROM Discus_Medalists
ORDER BY Year ASC;
```

#### FIRST_VALUE: 

A menudo es útil obtener el primer o el último valor de un conjunto de datos para comparar con el todos los demas valores. Con funciones de obtención absoluta como FIRST_VALUE, puedes obtener un valor en una partición absoluta de la tabla, como su principio o su final.

-- Devuelve todos los atletas y el primer atleta ardenados par orden alfabetico.

```
WITH ALL_Male_Medalists AS (
SELECT DISTINCT
Athlete
FROM Summer_Medals
WHERE Medal = 'Gold'
AND Gender = 'Men')

SELECT
-- Fetch all athletes and the first athlete alphabetically
athlete
FIRST_VALUE(athlete) OVER (
ORDER BY athlete ASC
) AS First_Athlete
FROM ALL_Male_Medalists;
```

-- Devuelve el año y la ciudad en que se celebraron cada uno de los Juegos Olimpicos y busca la ultima ciudad en la que se celebraron los Juegos Olimpicos.

```
WITH Hosts AS (
SELECT DISTINCT Year,
FROM Sumner_Medals)

SELECT
Vear
City,
-- Get the last city in which the Olympic ganes were held
LAST_VALUE(city) OVER (
ORDER BY Year ASC
RANGE BETWEEN
UNBOUNDED PRECEDING AND
UNBOUNDED FOLLOWING
) AS Last_City
FROM Hosts
ORDER BY Year ASC;
```

### 2- Clasificación

#### RANK() omite numeros en caso de valores identicos.

-- Clasifica a cada atleta por el numero de medallas que ha ganado -cuanto mayor sea el recuento, mayor será la clasificación- con números idénticos en caso de valores idénticos.

```
WITH Athlete_Medals AS (
SELECT
Athlete,
COUNT (*) AS Medals
FROM Summer_Medals
GROUP BY Athlete)

SELECT
Athlete,
Medals,
-- Rank athletes by the medals they've won
RANK() OVER (ORDER BY medals DESC) AS Rank_N
FROM Athlete_Medals
ORDER BY Medals DESC;
```

#### DENSE_RANK: no omite numeros en caso de valores identicos.

-- Clasifica a los atletas de cada pais por el numero de medallas que han ganado -cuanto mayor sea el numero, mayor sera la clasificación- sin omitir números en caso de valores idénticos. 

```
WITH Athlete_Medals AS (
SELECT
Country, Athlete, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country IN ('JPN', 'KOR')
AND Year >= 2000
GROUP BY Country, Athlete
HAVING COUNT(*) > 1)

SELECT
Country,
-- Rank athletes in each country by the medals they've won
Athlete,
DENSE_RANK() OVER (PARTITION BY country
ORDER BY Medals DESC) AS Rank_N
FROM Athlete_Medals
ORDER BY Country ASC, RANK_N ASC;
```


### 3- Buscapersonas 

#### NTILE: 

-- Divide los distintos eventos en exactamente 111 grupos, ordenados por evento en orden alfabético.

```
WITH Events AS (
SELECT DISTINCT Event
FROM Summer_Medals)

SELECT
--- Split up the distinct events into 111 unique groups
event,
NTILE(111) OVER (ORDER BY event ASC) AS Page
FROM Events
ORDER BY Event ASC;
```

#### Tercios superior, medio e inferior Dividir tus datos en tercios o cuartiles suele ser util para comprender como se distribuyen los valores de tu conjunto de datos. Obtener estadisticas resumidas (medias, sumas, desviaciones tipicas, etc.) de los tercios superior, medio e inferior puede ayudarte a determinar que distribucion siguen tus valores.

-- Divide a los atletas en tercios superior, medio e inferior en funcion de su numero de medallas.

```
WITH Athlete_Medals AS (
SELECT Athlete, COUNT(*) AS Medals
FROM Summer_Medals
GROUP BY Athlete
HAVING COUNT(*) > 1)

SELECT
Athlete,
Medals,
-- Split athletes into thirds by their earned medals
NTILE(3) OVER(ORDER BY Medals DESC) AS Third
FROM Athlete_Medals
ORDER BY Medals DESC, Athlete ASC;
```

## 3️⃣ Funciones y marcos de ventana agregados:

Aprenderás a utilizar funciones agregadas con las que estás familiarizado, como `AVG()` y `SUM()`, como funciones de ventana, así como a definir marcos para cambiar la salida de una función de ventana.

### Funciones de Ventana Agregadas: 

#### El total acumulado (o suma acumulada) de una columna te ayuda a determinar cuál es la contribución de cada fila a la suma total.

-- Devuelve los atletas, el numero de medallas que han conseguido y el total de medallas en carrera, ordenados por los nombres de los atletas en orden alfabético.

```
WITH Athlete_Medals AS (
SELECT
Athlete, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country = 'USA' AND Medal = 'Gold'
AND Year >= 2000
GROUP BY Athlete)

SELECT
-- Caloulate the punning total of athlete medals
Athlete,
Medals,
SUM(Medals) OVER (ORDER BY Athlete ASC) AS Max_Medals
FROM Athlete_Medals
ORDER BY Athlete ASC;
```

#### Obtener el máximo

-- Devuelve el año, el pais, las medallas y las medallas maximas conseguidas hasta el momento por cada pais, ordenadas por ano en orden ascendente.

```
WITH Country_Medals AS (
SELECT
Year, Country, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country IN ('CHN', 'KOR', 'JPN')
AND Medal = 'Gold' AND Year >= 2000
GROUP BY Year, Country)

SELECT
-- Return the max medals earned so far per country
Year,
Country,
Medals,
MAX(medals) OVER (PARTITION BY country
ORDER BY Year ASC) AS Max_Medals
FROM Country_Medals
ORDER BY Country ASC, Year ASC;
```

#### MAX y SUM, funciones agregadas que normalmente se utilizan con GROUP BY, se utilizan como funciones de ventana. También puedes utilizar las otras funciones agregadas, como MIN, como funciones de ventana.

-- Devuelve el año, las medallas conseguidas y el mínimo de medallas conseguidas hasta el momento.

```
WITH France_Medals AS (
SELECT
Year, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country = 'FRA'
AND Medal = 'Gold' AND Year >= 2000
GROUP BY Year)

SELECT
Year,
Medals,
MIN(Medals) OVER (ORDER BY Year ASC) AS Min_Medals
FROM France_Medals
ORDER BY Year ASC;
```

### Marcos:

Los marcos te permiten restringir las filas pasadas como entrada a tu funcion de ventana a una ventana deslizante para que definas el inicio y el final.

Afadir un marco a tu funcion de ventana te permite calcular metricas "moviles", cuyas entradas se deslizan de fila en fila.

-- Devuelve el año, las medallas ganadas y el maximo de medallas ganadas, comparando sólo el afo actual y el siguiente.

```
WITH Scandinavian_Medals AS (
SELECT
Year, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country IN ('DEN', 'NOR', 'FIN', 'SWE', 'ISL ')
AND Medal = 'Gold'
GROUP BY Year)

SELECT
-- Select each year's medals
year
medals,
-- Get the max of the current and next years'
MAX(medals) OVER (ORDER BY year ASC
ROWS BETWEEN CURRENT ROW
AND 1 FOLLOWING) AS Max_Medals
FROM Scandinavian_Medals
ORDERY BY Year ASC;
```

#### LAG, LEAD

Los marcos te permiten "echar un vistazo" hacia delante o hacia atras sin utilizar primero las funciones de obtencion relativa, LAG y LEAD , para obtener los valores de filas anteriores en la fila actual.

-- Devuelve los atletas, las medallas conseguidas y el maximo de medallas conseguidas, comparando solo los dos ultimos y los atletas actuales, ordenados por los nombres de los atletas en orden alfabético.

```
WITH Chinese_Medals AS (
SELECT
Athlete, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country = 'CHN' AND Medal = 'Gold'
AND Year >= 2000
GROUP BY Athlete)

SELECT
-- Select the athletes and the medals they've earned
Athlete,
medals,
-- Get the max of the last two and current rows' medals
MAX(Medals) OVER (ORDER BY Athlete ASC
ROWS BETWEEN 2 PRECEDING
AND CURRENT ROW) AS Max_Medals
FROM Chinese_Medals
ORDER BY Athlete ASC;
```

#### Medios móviles y totales

Utilizar marcos con funciones de ventana agregada te permite calcular muchas métricas habituales, como medias móviles y totales. Estas métricas siguen la evolución del rendimiento a lo largo del tiempo.

-- Calcula la media móvil de 3 años de medallas ganadas.

```
WITH Russian_Medals AS (
SELECT
Year, COUNT(*) AS Medals
FROM Summer_Medals
WHERE
Country = 'RUS'
AND Medal = 'Gold'
AND Year >= 1980
GROUP BY Year)

SELECT
Year, Medals,
--- Calculate the 3-year moving average of medals earned
AVG(medals) OVER
CORDER BY Year ASC
ROWS BETWEEN
2 PRECEDING AND CURRENT ROW) AS Medals_MA
FROM Russian_Medals
ORDER BY Year ASC;
```

-- Calcula la suma móvil de 3 años de medallas ganadas por país.

```
WITH Country_Medals AS (
SELECT
Year, Country, COUNT(*) AS Medals
FROM Summer_Medals
GROUP BY Year, Country)

SELECT
Year, Country, Medals,
-- Calculate each country's 3-game moving total
SUM(Medals) OVER
(PARTITION BY country
ORDER BY Year ASC
ROWS BETWEEN
2 PRECEDING AND CURRENT ROW) AS Medals_MA
FROM Country_Medals
ORDER BY Country ASC, Year ASC;
```

## 4️⃣ Más allá de las funciones de ventana:

Aprenderás algunas técnicas y funciones que son útiles cuando se utilizan junto con las funciones de ventana.

### Pivotante

#### Un pivote básico, Pivótala en Year para obtener la siguiente tabla remodelada y más limpia.

-- Crea la extensión correcta. Rellena los nombres de las columnas de la tabla pivotante.

```
F- Create the correct extension to enable CROSSTAB
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM CROSSTABC$$
SELECT
Gender, Year, Country
FROM Summer_Medals
WHERE
Year IN (2008, 2012)
AND Medal = 'Gold'
AND Event = 'Pole Vault'
ORDER By Gender ASC, Year ASC;
-- Fill in the correct column names for the pivoted table
$$) AS ct (Gender VARCHAR,
"2008" VARCHAR,
"2012" VARCHAR)

ORDER BY Gender ASC;
```

#### Pivotar con clasificacion

-- Cuenta las medallas de oro que han ganado Francia (FRA), el Reino Unido (GBR) y Alemania (GER) por país y año.

```
-- Count the gold medals per country and year
SELECT
Country,
Year.
Count(*) AS Awards
FROM Summer_Medals
WHERE
Country IN ('FRA', 'GBR', 'GER')
AND Year IN (2004, 2008, 2012)
AND Medal = 'Gold'
GROUP BY country, year
ORDER BY Country ASC, Year ASC
```

### ROLLUP y CUBE

-- Cuenta las medallas de oro concedidas por pais y sexo. Genera recuentos de premios de oro de nivel Country.

```
-- Count the gold medals per country and gender
SELECT
country,
gender
COUNT(*) AS Gold_Awards
FROM Summer_Medals
WHERE
Year = 2004
AND Medal = 'Gold'
AND Country IN ('DEN', 'NOR', 'SWE')
-- Generate Country-Level subtotals
GROUP BY Country, ROLLUP(Gender)
ORDER BY Country ASC, Gender ASC;
```

-- Cuenta las medallas concedidas por sexo y tipo de medalla. Genera todos los recuentos posibles a nivel de grupo (subtotales por sexo y tipo de medalla y el total general).

```
-- Count the medals per gender and medal type
SELECT
gender,
medal,
Count(*) AS Awards
FROM Summer_Medals
WHERE
Year = 2012
AND Country = 'RUS'
-- Get all possible group-level subtotals
GROUP BY CUBE(Gender, Medal)
ORDER BY Gender ASC, Medal ASC;
```

#### Limpiar los resultados
Volviendo al desglose de los premios escandinavos que hiciste anteriormente, quieres limpiar los resultados sustituyendo las nuLl s por texto con sentido.

-- Convierte los nuLl s de la columna Country en ALL countries , y los nuLl s de la columna Gender en ALL genders.

```
SELECT
-- Replace the nulls in the columns with meaningful text
COALESCE(Country, 'All countries') AS Country,
COALESCE(Gender, 'All genders') AS Gender,
COUNT(*) AS Awards
FROM Summer_Medals
WHERE
Year = 2004
AND Medal = 'Gold'
AND Country IN ('DEN', 'NOR', 'SWE')
GROUP BY ROLLUP(Country, Gender)
ORDER BY Country ASC, Gender ASC;
```

#### Resumir los resultados

-- Clasifica los paises segun las medallas que se les han concedido.

```
WITH Country_Medals AS (
SELECT
Country,
COUNT(*) AS Medals
FROM Summer_Medals
WHERE Year = 2000
AND Medal = 'Gold'
GROUP BY Country)

SELECT
Country,
-- Rank countries by the medals awarded
RANK() OVER (ORDER BY Medals DESC) AS Rank
FROM Country_Medals
ORDER BY Rank ASC;
```
