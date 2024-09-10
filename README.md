# SQL_Funciones_de_Ventana_Ejercicios_Curso_DataCamp







## 1️⃣ Introduccion a las funciones de ventana
En este capítulo aprenderás qué son las funciones ventana y las dos subcláusulas básicas de las funciones ventana, ORDER BY y PARTITION BY.

Numeracion de filas: La aplicacion mas sencilla de las funciones de ventana es la numeracion de filas. Numerar las filas te permite buscar facilmente la fila n.

-- Asignar números a cada fila

```
SELECT *,
ROW_NUMBER() OVER() AS Row_N
FROM Summer_Medals
ORDER BY Row_N ASC;
```
