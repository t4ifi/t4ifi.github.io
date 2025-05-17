---
title: "Operadores SQL: AND, OR y UNION"
date: 2025-05-16 21:00:00 +0000
categories: [Apuntes, SQL]
tags: [sql, consultas, operadores, and, or, union]
layout: post
---

## AND y OR

El operador lógico AND y el operador OR son fundamentales en la lógica y la programación, incluyendo SQL.

## AND

- En la lógica booleana, **AND** es un operador que devuelve verdadero (true) si y solo si ambas expresiones que conecta son verdaderas.
- En SQL, el **AND** se usa para combinar múltiples condiciones en una consulta. Por ejemplo, en una consulta **SELECT**, puedes usar **AND** para especificar que quieres filas que cumplan con dos o más condiciones al mismo tiempo.

### Ejemplo:

Supongamos que tenemos una tabla llamada empleados con las siguientes columnas: id, nombre, apellido, edad, y queremos obtener los empleados cuyo nombre sea "Juan" y cuya edad sea mayor de 30 años. La consulta SQL sería:
```sql
SELECT * FROM empleados WHERE nombre = 'Juan' AND edad > 30;
```
Esta consulta selecciona solo los empleados cuyo nombre sea "Juan" y cuya edad sea mayor de 30 años. Ambas condiciones deben cumplirse al mismo tiempo.

---

## OR

Supongamos que queremos seleccionar los empleados cuyo nombre sea "Juan" o cuya edad sea mayor de 30 años. La consulta SQL sería:
```sql
SELECT * FROM empleados WHERE nombre = 'Juan' OR edad > 30;
```
Esta consulta selecciona todos los empleados que cumplan al menos una de las condiciones: que su nombre sea "Juan" o que su edad sea mayor de 30 años.

---

## UNION

El operador UNION se usa para combinar los resultados de dos o más consultas SQL en una sola tabla de resultados.
```sql
SELECT id, nombre, apellido, edad FROM empleados WHERE nombre = 'Juan'
UNION
SELECT id, nombre, apellido, edad FROM empleados WHERE edad > 30;
```
- La primera consulta selecciona los empleados cuyo nombre sea "Juan".
- La segunda consulta selecciona los empleados cuya edad sea mayor de 30 años.
- El operador UNION combina ambos resultados en una sola tabla sin duplicados.
