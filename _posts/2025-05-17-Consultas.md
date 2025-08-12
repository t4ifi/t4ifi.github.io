---
title: "SQL para Principiantes: Consultas básicas con ejemplos prácticos"
date: 2025-05-16 21:00:00 +0000
categories: [Bases de Datos, SQL]
tags: [sql, mysql, mariadb, consultas, base-datos, principiantes]
---

# 🗄️ ¿Qué son las Consultas SQL?

Las consultas SQL (Structured Query Language) son **instrucciones que envías a una base de datos** para realizar operaciones como buscar información, agregar datos, modificar registros o eliminar contenido.

> 💡 **Piénsalo así:** SQL es como el "idioma" que usas para comunicarte con una base de datos. Le dices exactamente qué quieres hacer y ella te responde.

---

## 🔍 Tipos de Consultas SQL

### 📋 DDL - Definición de Datos (Estructura)

**Comandos para crear y modificar la estructura:**

- **`CREATE`**: Crear tablas, bases de datos, índices
- **`ALTER`**: Modificar estructura de tablas existentes  
- **`DROP`**: Eliminar tablas o bases de datos completas
- **`TRUNCATE`**: Vaciar todo el contenido de una tabla (más rápido que DELETE)

### 📊 DML - Manipulación de Datos (Contenido)

**Comandos para trabajar con los datos:**

- **`INSERT`**: Agregar nuevos registros
- **`SELECT`**: Buscar y mostrar información
- **`UPDATE`**: Modificar registros existentes
- **`DELETE`**: Eliminar registros específicos

> ⚠️ **Dato importante:** Los comandos DML son los que normalmente pueden ser vulnerados en ataques de **SQL Injection**.

---

## 🏗️ Creando tu primera base de datos

### Paso 1: Crear la base de datos
```sql
CREATE DATABASE mi_empresa;
```

### Paso 2: Seleccionar la base de datos
```sql
USE mi_empresa;
```

### Paso 3: Crear una tabla con estructura completa
```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    edad INT CHECK (edad >= 18),
    salario DECIMAL(10,2),
    departamento VARCHAR(30),
    fecha_contratacion DATE DEFAULT CURRENT_DATE
);
```

**¿Qué hace cada parte?**
- `id INT PRIMARY KEY AUTO_INCREMENT`: ID único que se incrementa automáticamente
- `VARCHAR(50) NOT NULL`: Texto de máximo 50 caracteres, obligatorio
- `UNIQUE`: No permite valores duplicados  
- `CHECK (edad >= 18)`: Solo permite mayores de edad
- `DECIMAL(10,2)`: Números decimales (10 dígitos total, 2 después del punto)
- `DEFAULT CURRENT_DATE`: Usa la fecha actual por defecto

<div style="text-align: center;">
  <img src="/assets/img/apuntes/Base1.jpeg" alt="crear base de datos" style="width: 100%; max-width: 500px;" />
</div>

---

## ➕ INSERT: Agregando datos

### Insertar un registro completo
```sql
INSERT INTO empleados (nombre, apellido, email, edad, salario, departamento)
VALUES ('Andrés', 'Núñez', 'andres@empresa.com', 25, 3500.00, 'IT');
```

### Insertar múltiples registros a la vez
```sql
INSERT INTO empleados (nombre, apellido, email, edad, salario, departamento) VALUES
('María', 'González', 'maria@empresa.com', 28, 4000.00, 'Marketing'),
('Carlos', 'López', 'carlos@empresa.com', 32, 4500.00, 'Ventas'),
('Ana', 'Martínez', 'ana@empresa.com', 26, 3800.00, 'IT'),
('Pedro', 'Rodríguez', 'pedro@empresa.com', 29, 3200.00, 'Contabilidad');
```

### Insertar solo algunos campos
```sql
INSERT INTO empleados (nombre, apellido, email, edad)
VALUES ('Luis', 'Fernández', 'luis@empresa.com', 24);
-- salario y departamento quedarán como NULL
```

<div style="text-align: center;">
  <img src="/assets/img/apuntes/base2.jpeg" alt="insertar registro" style="width: 100%; max-width: 500px;" />
</div>

---

## 🔍 SELECT: Consultando datos

### Consulta básica - Ver todo
```sql
SELECT * FROM empleados;
```

### Consulta específica - Columnas seleccionadas
```sql
SELECT nombre, apellido, salario FROM empleados;
```

### Consultas con condiciones (WHERE)
```sql
-- Empleados del departamento IT
SELECT * FROM empleados WHERE departamento = 'IT';

-- Empleados con salario mayor a 3500
SELECT nombre, apellido, salario 
FROM empleados 
WHERE salario > 3500;

-- Empleados jóvenes con buen salario
SELECT * FROM empleados 
WHERE edad < 30 AND salario >= 3500;
```

### Ordenar resultados
```sql
-- Ordenar por salario (menor a mayor)
SELECT * FROM empleados ORDER BY salario;

-- Ordenar por salario (mayor a menor)  
SELECT * FROM empleados ORDER BY salario DESC;

-- Ordenar por departamento y luego por salario
SELECT * FROM empleados ORDER BY departamento, salario DESC;
```

### Limitar resultados
```sql
-- Solo los primeros 3 empleados
SELECT * FROM empleados LIMIT 3;

-- Empleados con mejor salario (top 3)
SELECT nombre, apellido, salario 
FROM empleados 
ORDER BY salario DESC 
LIMIT 3;
```

### Búsquedas con patrones
```sql
-- Nombres que empiecen con 'A'
SELECT * FROM empleados WHERE nombre LIKE 'A%';

-- Emails que contengan 'gmail'
SELECT * FROM empleados WHERE email LIKE '%gmail%';

-- Nombres de exactamente 5 letras
SELECT * FROM empleados WHERE nombre LIKE '_____';
```

---

## ✏️ UPDATE: Modificando datos

### Actualizar un campo específico
```sql
UPDATE empleados 
SET salario = 4000.00 
WHERE id = 1;
```

### Actualizar múltiples campos
```sql
UPDATE empleados 
SET salario = 4200.00, departamento = 'Senior IT'
WHERE nombre = 'Andrés' AND apellido = 'Núñez';
```

### Aumentar salarios por departamento
```sql
-- Aumentar 10% a empleados de IT
UPDATE empleados 
SET salario = salario * 1.10 
WHERE departamento = 'IT';
```

### Actualización condicional múltiple
```sql
-- Bono para empleados con más de 2 años
UPDATE empleados 
SET salario = salario + 500 
WHERE fecha_contratacion < '2023-01-01';
```

---

## 🗑️ DELETE: Eliminando datos

### Eliminar registro específico
```sql
DELETE FROM empleados WHERE id = 5;
```

### Eliminar con condiciones múltiples
```sql
-- Eliminar empleados sin departamento asignado
DELETE FROM empleados WHERE departamento IS NULL;

-- Eliminar empleados de contabilidad con salario bajo
DELETE FROM empleados 
WHERE departamento = 'Contabilidad' AND salario < 3000;
```

### ⚠️ Cuidado con DELETE sin WHERE
```sql
-- ¡ESTO ELIMINA TODOS LOS REGISTROS!
DELETE FROM empleados;

-- Mejor usar TRUNCATE para vaciar completamente
TRUNCATE TABLE empleados;
```

---

## 📊 Consultas avanzadas para principiantes

### Funciones de agregación
```sql
-- Contar empleados
SELECT COUNT(*) FROM empleados;

-- Salario promedio
SELECT AVG(salario) as salario_promedio FROM empleados;

-- Salario máximo y mínimo
SELECT MAX(salario) as max_salario, MIN(salario) as min_salario FROM empleados;

-- Suma total de salarios
SELECT SUM(salario) as total_salarios FROM empleados;
```

### Agrupar datos
```sql
-- Empleados por departamento
SELECT departamento, COUNT(*) as cantidad_empleados
FROM empleados 
GROUP BY departamento;

-- Salario promedio por departamento
SELECT departamento, AVG(salario) as salario_promedio
FROM empleados 
GROUP BY departamento
ORDER BY salario_promedio DESC;
```

### Consultas con HAVING (filtrar grupos)
```sql
-- Departamentos con más de 2 empleados
SELECT departamento, COUNT(*) as cantidad
FROM empleados 
GROUP BY departamento
HAVING COUNT(*) > 2;
```

---

## 🛡️ Buenas prácticas de seguridad

### 1. Siempre usar WHERE en UPDATE y DELETE
```sql
-- ✅ BIEN - Con condición específica
UPDATE empleados SET salario = 5000 WHERE id = 1;

-- ❌ MAL - Sin WHERE (afecta TODOS los registros)
UPDATE empleados SET salario = 5000;
```

### 2. Validar datos antes de insertar
```sql
-- ✅ BIEN - Con validaciones
INSERT INTO empleados (nombre, email, edad) 
VALUES ('Juan', 'juan@email.com', 25)
WHERE NOT EXISTS (SELECT 1 FROM empleados WHERE email = 'juan@email.com');
```

### 3. Usar transacciones para operaciones críticas
```sql
START TRANSACTION;
UPDATE empleados SET salario = salario * 1.10 WHERE departamento = 'IT';
-- Si algo sale mal: ROLLBACK;
-- Si todo está bien: COMMIT;
COMMIT;
```

---

## 🎯 Ejercicios prácticos

**Crea la tabla y datos de ejemplo, luego resuelve:**

1. Mostrar empleados que ganen más de 4000
2. Contar cuántos empleados hay en cada departamento  
3. Encontrar el empleado con mayor salario
4. Actualizar el departamento de 'María González' a 'Recursos Humanos'
5. Eliminar empleados con salario menor a 3000

### Soluciones:
```sql
-- 1. Empleados que ganen más de 4000
SELECT * FROM empleados WHERE salario > 4000;

-- 2. Empleados por departamento
SELECT departamento, COUNT(*) FROM empleados GROUP BY departamento;

-- 3. Empleado con mayor salario
SELECT * FROM empleados ORDER BY salario DESC LIMIT 1;

-- 4. Actualizar departamento de María
UPDATE empleados SET departamento = 'Recursos Humanos' 
WHERE nombre = 'María' AND apellido = 'González';

-- 5. Eliminar empleados con salario bajo
DELETE FROM empleados WHERE salario < 3000;
```

---

## 📚 Próximos pasos

1. **Practica** con MySQL o MariaDB instalado en tu sistema
2. **Aprende** sobre JOINS para consultar múltiples tablas
3. **Estudia** índices para optimizar consultas
4. **Explora** funciones específicas de tu motor de base de datos

💡 **Recuerda:** SQL es un lenguaje potente. ¡La práctica constante es clave para dominarlo!

---
**Andrés Nuñez - t4ifi**
