---
title: "Operadores SQL: Guía Completa de AND, OR, UNION y más"
date: 2025-05-17 21:00:00 +0000
categories: [Bases de Datos, SQL]
tags: [sql, consultas, operadores, and, or, union, joins, subconsultas, optimizacion]
layout: post
---

## 🎯 Introducción a Operadores SQL

Los operadores SQL son elementos fundamentales que te permiten crear consultas complejas y poderosas. Son las herramientas que conectan, comparan y combinan datos de diferentes maneras.

> 💡 **Analogía:** Los operadores son como conectores y filtros en una tubería de datos. Te permiten controlar exactamente qué información fluye y cómo se combina.

---

## 🔗 Operadores Lógicos: AND y OR

### Operador AND
El operador **AND** requiere que **todas las condiciones** sean verdaderas para que un registro sea incluido en el resultado.

```sql
-- Sintaxis básica
SELECT * FROM tabla WHERE condicion1 AND condicion2;
```

#### Ejemplos prácticos con tabla empleados:

```sql
-- Crear tabla de ejemplo
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    apellido VARCHAR(50),
    edad INT,
    salario DECIMAL(10,2),
    departamento VARCHAR(30),
    fecha_contratacion DATE
);

-- Insertar datos de prueba
INSERT INTO empleados (nombre, apellido, edad, salario, departamento, fecha_contratacion) VALUES
('Juan', 'Pérez', 28, 45000.00, 'IT', '2023-01-15'),
('María', 'González', 35, 52000.00, 'Marketing', '2022-03-20'),
('Carlos', 'López', 42, 48000.00, 'IT', '2021-07-10'),
('Ana', 'Martínez', 29, 38000.00, 'Ventas', '2023-05-12'),
('Pedro', 'Rodríguez', 38, 55000.00, 'IT', '2020-11-08');
```

**Ejemplos de AND:**
```sql
-- Empleados de IT con salario mayor a 45000
SELECT nombre, apellido, salario, departamento 
FROM empleados 
WHERE departamento = 'IT' AND salario > 45000;

-- Empleados entre 30 y 40 años con salario específico
SELECT * FROM empleados 
WHERE edad >= 30 AND edad <= 40 AND salario > 50000;

-- Empleados contratados en 2023 del departamento IT
SELECT nombre, apellido, fecha_contratacion 
FROM empleados 
WHERE YEAR(fecha_contratacion) = 2023 AND departamento = 'IT';
```

### Operador OR
El operador **OR** requiere que **al menos una** de las condiciones sea verdadera.

```sql
-- Sintaxis básica
SELECT * FROM tabla WHERE condicion1 OR condicion2;
```

**Ejemplos de OR:**
```sql
-- Empleados de IT o Marketing
SELECT nombre, apellido, departamento 
FROM empleados 
WHERE departamento = 'IT' OR departamento = 'Marketing';

-- Empleados muy jóvenes o muy experimentados
SELECT nombre, apellido, edad 
FROM empleados 
WHERE edad < 25 OR edad > 40;

-- Empleados con salarios altos o del departamento de Ventas
SELECT * FROM empleados 
WHERE salario > 50000 OR departamento = 'Ventas';
```

### Combinando AND y OR
**¡Cuidado con la precedencia!** Usa paréntesis para clarificar la lógica:

```sql
-- INCORRECTO (ambiguo)
SELECT * FROM empleados 
WHERE departamento = 'IT' OR departamento = 'Marketing' AND salario > 45000;

-- CORRECTO (con paréntesis)
SELECT * FROM empleados 
WHERE (departamento = 'IT' OR departamento = 'Marketing') AND salario > 45000;

-- Empleados de IT con buen salario O empleados senior de cualquier departamento
SELECT * FROM empleados 
WHERE (departamento = 'IT' AND salario > 45000) OR edad > 40;
```

---

## 🔄 Operador UNION y sus variantes

### UNION básico
**UNION** combina resultados de múltiples consultas eliminando duplicados.

```sql
-- Sintaxis básica
SELECT columnas FROM tabla1
UNION
SELECT columnas FROM tabla2;
```

**Ejemplo práctico:**
```sql
-- Crear tabla adicional
CREATE TABLE ex_empleados (
    id INT,
    nombre VARCHAR(50),
    apellido VARCHAR(50),
    ultimo_salario DECIMAL(10,2),
    fecha_salida DATE
);

INSERT INTO ex_empleados VALUES
(101, 'Roberto', 'García', 47000.00, '2023-12-15'),
(102, 'Laura', 'Silva', 51000.00, '2023-11-30');

-- Combinar empleados actuales y ex-empleados (solo nombres)
SELECT nombre, apellido FROM empleados
UNION
SELECT nombre, apellido FROM ex_empleados;
```

### UNION ALL
**UNION ALL** incluye todos los registros, incluso duplicados (más rápido).

```sql
-- Incluir duplicados
SELECT departamento FROM empleados
UNION ALL
SELECT 'Ex-empleado' as departamento FROM ex_empleados;
```

### UNION con ORDER BY
```sql
-- Ordenar resultado final
(SELECT nombre, apellido, salario, 'Actual' as estado FROM empleados WHERE salario > 45000)
UNION
(SELECT nombre, apellido, ultimo_salario, 'Ex-empleado' as estado FROM ex_empleados)
ORDER BY salario DESC;
```

---

## 🔍 Operadores de Comparación

### Operadores básicos
```sql
-- Igualdad y desigualdad
SELECT * FROM empleados WHERE edad = 30;
SELECT * FROM empleados WHERE edad != 25;
SELECT * FROM empleados WHERE edad <> 25;  -- Equivalente a !=

-- Comparaciones numéricas
SELECT * FROM empleados WHERE salario > 45000;
SELECT * FROM empleados WHERE salario >= 45000;
SELECT * FROM empleados WHERE edad < 35;
SELECT * FROM empleados WHERE edad <= 35;
```

### Operador BETWEEN
```sql
-- Rango de valores (más legible que >= y <=)
SELECT nombre, apellido, edad 
FROM empleados 
WHERE edad BETWEEN 25 AND 35;

-- Equivalente a:
SELECT nombre, apellido, edad 
FROM empleados 
WHERE edad >= 25 AND edad <= 35;

-- Rango de fechas
SELECT * FROM empleados 
WHERE fecha_contratacion BETWEEN '2022-01-01' AND '2023-12-31';
```

### Operador IN
```sql
-- Lista de valores específicos
SELECT * FROM empleados 
WHERE departamento IN ('IT', 'Marketing', 'Ventas');

-- Equivalente a:
SELECT * FROM empleados 
WHERE departamento = 'IT' OR departamento = 'Marketing' OR departamento = 'Ventas';

-- Con subconsulta
SELECT * FROM empleados 
WHERE id IN (SELECT empleado_id FROM proyectos WHERE estado = 'Activo');
```

### Operador LIKE
```sql
-- Búsqueda de patrones
SELECT * FROM empleados WHERE nombre LIKE 'Mar%';      -- Empieza con "Mar"
SELECT * FROM empleados WHERE apellido LIKE '%ez';     -- Termina con "ez"
SELECT * FROM empleados WHERE nombre LIKE '_ana';      -- "Ana", "Lana", etc.
SELECT * FROM empleados WHERE email LIKE '%@gmail.%'; -- Emails de Gmail
```

### Operador IS NULL / IS NOT NULL
```sql
-- Valores nulos
SELECT * FROM empleados WHERE salario IS NULL;
SELECT * FROM empleados WHERE salario IS NOT NULL;

-- NUNCA uses = o != con NULL
-- INCORRECTO: WHERE salario = NULL
-- CORRECTO:   WHERE salario IS NULL
```

---

## 🏗️ Operadores de Conjunto Avanzados

### INTERSECT (MySQL 8.0+)
```sql
-- Empleados que están en ambas consultas
SELECT nombre FROM empleados WHERE departamento = 'IT'
INTERSECT
SELECT nombre FROM empleados WHERE salario > 45000;
```

### EXCEPT / MINUS
```sql
-- En PostgreSQL (EXCEPT)
SELECT nombre FROM empleados WHERE departamento = 'IT'
EXCEPT
SELECT nombre FROM empleados WHERE edad < 30;

-- En Oracle (MINUS)
SELECT nombre FROM empleados WHERE departamento = 'IT'
MINUS
SELECT nombre FROM empleados WHERE edad < 30;
```

---

## 📊 Casos de Uso Profesionales

### 1. Dashboard de Recursos Humanos
```sql
-- Reporte completo de empleados críticos
SELECT 
    nombre,
    apellido,
    departamento,
    salario,
    CASE 
        WHEN salario > 50000 THEN 'Alto'
        WHEN salario BETWEEN 35000 AND 50000 THEN 'Medio'
        ELSE 'Bajo'
    END as nivel_salario,
    CASE 
        WHEN edad > 40 THEN 'Senior'
        WHEN edad BETWEEN 30 AND 40 THEN 'Medio'
        ELSE 'Junior'
    END as nivel_experiencia
FROM empleados
WHERE (departamento = 'IT' AND salario < 45000)
   OR (edad > 35 AND departamento IN ('Marketing', 'Ventas'))
ORDER BY salario DESC;
```

### 2. Análisis de Retención
```sql
-- Empleados en riesgo de fuga
SELECT e1.nombre, e1.apellido, e1.salario, e1.departamento
FROM empleados e1
WHERE e1.salario < (
    SELECT AVG(e2.salario) * 0.9  -- 10% por debajo del promedio
    FROM empleados e2 
    WHERE e2.departamento = e1.departamento
)
AND e1.fecha_contratacion < DATE_SUB(NOW(), INTERVAL 2 YEAR);
```

### 3. Reporte de Diversidad Departamental
```sql
-- Distribución por departamento y rangos de edad
SELECT 
    departamento,
    COUNT(*) as total_empleados,
    SUM(CASE WHEN edad < 30 THEN 1 ELSE 0 END) as jovenes,
    SUM(CASE WHEN edad BETWEEN 30 AND 40 THEN 1 ELSE 0 END) as medios,
    SUM(CASE WHEN edad > 40 THEN 1 ELSE 0 END) as seniors,
    AVG(salario) as salario_promedio
FROM empleados
GROUP BY departamento
HAVING COUNT(*) > 1  -- Solo departamentos con más de 1 empleado
ORDER BY total_empleados DESC;
```

---

## ⚡ Optimización de Consultas con Operadores

### Tips de rendimiento:

1. **Orden de condiciones en AND:**
```sql
-- MÁS EFICIENTE: condición más selectiva primero
SELECT * FROM empleados 
WHERE id = 123 AND departamento = 'IT';

-- MENOS EFICIENTE
SELECT * FROM empleados 
WHERE departamento = 'IT' AND id = 123;
```

2. **Uso de índices:**
```sql
-- Crear índices para mejorar performance
CREATE INDEX idx_departamento ON empleados(departamento);
CREATE INDEX idx_salario ON empleados(salario);
CREATE INDEX idx_dept_salario ON empleados(departamento, salario);
```

3. **UNION vs UNION ALL:**
```sql
-- Si sabes que no hay duplicados, usa UNION ALL (más rápido)
SELECT nombre FROM empleados_madrid
UNION ALL
SELECT nombre FROM empleados_barcelona;

-- Solo usa UNION si necesitas eliminar duplicados
SELECT departamento FROM empleados
UNION
SELECT departamento FROM ex_empleados;
```

4. **EXISTS vs IN:**
```sql
-- Para subconsultas grandes, EXISTS suele ser más eficiente
SELECT * FROM empleados e
WHERE EXISTS (
    SELECT 1 FROM proyectos p 
    WHERE p.empleado_id = e.id AND p.estado = 'Activo'
);

-- En lugar de:
SELECT * FROM empleados 
WHERE id IN (SELECT empleado_id FROM proyectos WHERE estado = 'Activo');
```

---

## 🧪 Ejercicios Prácticos

### Ejercicio 1: Filtros Complejos
```sql
-- Encontrar empleados que cumplan ALGUNA de estas condiciones:
-- 1. Son de IT con más de 3 años en la empresa
-- 2. Son de Marketing con salario > 50000
-- 3. Tienen más de 40 años independientemente del departamento

-- Tu solución aquí:
SELECT * FROM empleados 
WHERE (departamento = 'IT' AND fecha_contratacion <= DATE_SUB(NOW(), INTERVAL 3 YEAR))
   OR (departamento = 'Marketing' AND salario > 50000)
   OR edad > 40;
```

### Ejercicio 2: Union de Datos
```sql
-- Crear un reporte unificado que muestre:
-- - Empleados actuales con salario > 45000 (marcar como "Empleado Activo")
-- - Ex-empleados con último salario > 45000 (marcar como "Ex-empleado")
-- Ordenar por salario descendente

-- Tu solución:
(SELECT nombre, apellido, salario, 'Empleado Activo' as estado
 FROM empleados WHERE salario > 45000)
UNION
(SELECT nombre, apellido, ultimo_salario, 'Ex-empleado' as estado
 FROM ex_empleados WHERE ultimo_salario > 45000)
ORDER BY salario DESC;
```

---

## 🎯 Próximos Pasos

1. **Practica con datos reales**: Crea tablas con más registros
2. **Aprende JOINs**: Combina datos de múltiples tablas
3. **Estudia subconsultas**: Consultas dentro de consultas
4. **Explora funciones de ventana**: ROW_NUMBER(), RANK(), etc.
5. **Optimización avanzada**: EXPLAIN PLAN y análisis de rendimiento

💡 **Recuerda:** Los operadores SQL son herramientas poderosas. La clave está en combinarlos de manera lógica y eficiente para resolver problemas reales de negocio.

**Andrés Nuñez**
