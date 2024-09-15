# Algunos de los ejercicios del curso de SQL Intermedio de DataCamp 

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

## 3️⃣ Funciones y marcos de ventana agregados:

Aprenderás a utilizar funciones agregadas con las que estás familiarizado, como `AVG()` y `SUM()`, como funciones de ventana, así como a definir marcos para cambiar la salida de una función de ventana.

## 4️⃣ Más allá de las funciones de ventana:

Aprenderás algunas técnicas y funciones que son útiles cuando se utilizan junto con las funciones de ventana.

